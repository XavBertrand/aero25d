# Prise en main et vérification prévues

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

Ce guide est une cible de validation pour la future réalisation. À la date du plan, les exécutables, presets, fixtures et tests ci-dessous ne sont pas encore présents : les commandes ne sont pas présentées comme exécutées ou réussies. Le plan ne crée ni solveur ni campagne aérodynamique.

## Prérequis et construction CPU

Linux/WSL2, compilateur C++20, CMake ≥3.25, Ninja, OpenSSL libcrypto et Python ≥3.11. Les dépendances C++ de source sont figées selon [research.md](research.md), présentes localement pour une construction sans réseau. CUDA et les pilotes NVIDIA ne sont pas nécessaires. Les dépendances de qualification (dont lecteur VTK indépendant) restent séparées de celles d'exécution.

Depuis la racine du dépôt :

```bash
cmake --preset cpu-debug
cmake --build --preset cpu-debug
ctest --preset cpu-debug --output-on-failure
cmake --install build/cpu-debug --prefix build/install
build/install/bin/aero25d-mesh --help
```

Le preset choisit un projet `LANGUAGES CXX`, sans détection CUDA, avec une compilation sans `fast-math`. CLion ouvre le `CMakePresets.json` du dépôt et utilise le même preset/compilateur WSL. `cpu-release` sert à caractériser ultérieurement temps et mémoire, sans modifier les tolérances ; `cpu-sanitize` sert aux contrôles mémoire/comportement indéfini. Les paramètres locaux sont dans `CMakeUserPresets.json`, non dans les presets partagés.

## Premier jalon : preuve géométrique des topologies

Avant l'ensemble des parcours utilisateur, exécuter uniquement la démonstration future suivante :

```bash
ctest --preset cpu-debug -L topology-demo --output-on-failure
```

Le label doit sélectionner V-01, V-02 et V-03 aux flèches 0°, 25° et 50°, les raccords communs et les coins du culot, sur des distributions de démonstration archivées. Les sorties attendues sont schémas de blocs, maillages, métriques locales, rapport d'interfaces et zooms BF/culot. Positivité, bijections de raccords, absence d'effondrement et couverture du domaine doivent être contrôlées numériquement. Un dessin visuellement plausible ne passe pas le jalon. Si les limites prévues ne peuvent être respectées, le jalon échoue et le plan doit être révisé avant le développement des fonctionnalités dépendantes ; aucune tolérance n'est automatiquement relâchée.

## Parcours monoélément fin

Les noms de fixtures ci-dessous font partie de l'organisation à réaliser ; leurs formules et références exactes sont définies dans [geometry-verification.md](geometry-verification.md). Les configurations associées utilisent ces sources par un chemin relatif au fichier JSON et les limites définies dans [mesh-quality.md](mesh-quality.md).

```bash
build/install/bin/aero25d-mesh check --config tests/data/configs/V01.json --out work/check-v01 --json
build/install/bin/aero25d-mesh mesh --config tests/data/configs/V01.json --out work/mesh-v01 --json
build/install/bin/aero25d-mesh quality --bundle work/mesh-v01 --out work/audit-v01 --json
build/install/bin/aero25d-mesh export --bundle work/mesh-v01 --format vtk --out work/vtk-v01 --json
```

Attendus : code 0 pour chaque opération valide ; source archivée inchangée ; repères et c originale accessibles ; toutes les cellules/interfaces contrôlées ; cinq SVG après génération ; VTM/VTS lisibles dans un lecteur indépendant. Ouvrir `work/mesh-v01/figures/leading-edge.svg` et `trailing-edge.svg` pour l'inspection humaine, puis `work/vtk-v01/mesh.vtm` dans ParaView ou un lecteur VTK. Le rapport numérique reste la preuve des tolérances ; la vue ne la remplace pas.

## Parcours culot épais

```bash
build/install/bin/aero25d-mesh check --config tests/data/configs/V03.json --out work/check-v03 --json
build/install/bin/aero25d-mesh mesh --config tests/data/configs/V03.json --out work/mesh-v03 --json
build/install/bin/aero25d-mesh export --bundle work/mesh-v03 --format vtk --out work/vtk-v03 --json
```

Attendus : deux lèvres et culot distincts, topologie épaisse explicite, parois et sillage identifiés, raccords conformes utilisant les mêmes IDs globaux. La variante avec culot déjà fourni et celle avec fermeture ajoutée explicitement doivent produire les preuves de préservation/ajout correspondantes ; aucune génération en pointe n'est admissible.

## Flèche, inverse et fidélité

```bash
ctest --preset cpu-debug -L transforms --output-on-failure
ctest --preset cpu-debug -L geometry-fidelity --output-on-failure
```

Le premier label doit exercer 0°, 25°, 50°, translation globale non nulle sur les deux axes, échelle différente de 1 et Z non centré, sur un puis deux éléments. Vérifier W→N avant N→S, c originale inchangée et résidu maximal ≤10⁻¹² c. Vérifier séparément les axes Y et η, ainsi que le point au quart de corde. Ces tests ne calculent aucun moment.

Le second label doit comparer source/reconstruction à la référence originale dans S, puis transformation de cette référence à la reconstruction dans N. Le rayon et les angles de référence transformés doivent être utilisés : leur variation causée par la compression n'est pas un défaut. Les tableaux de distances comportent une borne d'erreur d'évaluateur suffisamment petite pour décider le budget, et jamais un simple maximum sur échantillonnage grossier.

## Familles et variation indépendante du domaine

```bash
build/install/bin/aero25d-mesh family --config tests/data/configs/V01.json --levels 0,1,2 --out work/family-v01 --json
build/install/bin/aero25d-mesh family --config tests/data/configs/V03.json --levels 0,1,2 --out work/family-v03 --json
build/install/bin/aero25d-mesh family --config tests/data/configs/V03.json --study domain --values 10,20 --out work/domain-v03 --json
build/install/bin/aero25d-mesh family --config tests/data/configs/V03.json --study parameter --vary mesh.first_height_over_c --values 0.00001,0.000005 --out work/height-v03 --json
```

Attendus : au moins trois niveaux avec même reconstruction et graphe topologique ; comptes et pas évoluant selon la règle quantitative du plan, rapports effectifs publiés par région. L'étude du domaine conserve les prescriptions de paroi et sillage proche. La dernière commande est une étude ciblée explicitement marquée diagnostic ; elle ne remplace pas les familles globales. Aucun nombre de cellules ou distance extérieure ne constitue une garantie sur Cx.

## Utilisation depuis Python et reproduction

```bash
PYTHONPATH=python python3 - <<'PY'
from aero25d_mesh import Client

client = Client(executable="build/install/bin/aero25d-mesh")
result = client.mesh(config="tests/data/configs/V01.json", out="work/python-v01")
assert result.exit_code == 0
assert result.report["valid"] is True
assert result.report["counts"]["cells"] == result.report["counts"]["checked_cells"]
replayed = client.reproduce(bundle=result.bundle, out="work/python-v01-replay")
assert replayed.exit_code == 0
PY
```

Comparer aux résultats CLI de la même configuration : connectivité/IDs identiques, écarts de coordonnées ≤10⁻¹² c et mêmes décisions de qualité. Les opérations algorithmiques doivent toutes provenir du moteur C++. Le wrapper lit le rapport et relaie les erreurs ; il ne recalculera pas un verdict favorable.

## Refus et défauts injectés

```bash
ctest --preset cpu-debug -L invalid-input --output-on-failure
ctest --preset cpu-debug -L ambiguous-features --output-on-failure
ctest --preset cpu-debug -L multielement --output-on-failure
ctest --preset cpu-debug -L injected-mesh-defects --output-on-failure
ctest --preset cpu-debug -L contracts --output-on-failure
```

Attendus : tous les défauts du corpus V-04 sont localisés ; V-05 exige une indication et refuse une indication contradictoire ; V-06 conserve les deux éléments et refuse la génération ; V-08 détecte cellules inversées/dégénérées, raccord absent/inversé, trou/chevauchement et étirement mal orienté. Les échecs laissent un rapport invalide et, si possible, les artefacts de diagnostic marqués ; aucun manifeste valide n'est publié. Les tests de contrat comparent codes, diagnostics, schémas et sorties CLI/Python.

## Preuves à archiver et portée du succès

Le dossier de qualification associe chaque FR à sa preuve selon [verification.md](verification.md) : sources et digests, configuration demandée/résolue, version/environnement, reconstruction, paramètres de topologie, tableaux de mesures et limites, rapports, figures et exports relus. La commande et ses sorties/code sont conservés, y compris en échec. Les temps et mémoires observés pourront être ajoutés seulement après mesure, avec contexte et sans extrapolation à la précision aérodynamique.

Le succès établit la validité géométrique et la qualité mesurée sur les cas définis. Il ne valide ni Euler/RANS, ni y+ réel, ni l'incertitude numérique future sur Cx.

## Campagnes de maillage (après J0 et parcours unitaires)

Les exemples sont des commandes futures, pas des essais effectués :

```bash
aero25d-mesh campaign check --file study.yaml --out work/inventory --json
aero25d-mesh campaign run --file study.json --out work/study --json
aero25d-mesh campaign run --file study.json --out work/study --resume --json
```

Le fichier porte profils/flèches/configurations/niveaux, un produit ou une liste explicite,
et on_failure obligatoire. Contrat : [campaign.md](contracts/campaign.md).
Depuis Python : `client.campaign_check(file="study.yaml", out="work/inventory")` puis
`client.campaign_run(file="study.json", out="work/study", resume=True)` pour une reprise.
Les fichiers relatifs se résolvent depuis la campagne ; aucune génération ne se fait au contrôle.
Les états, clés effectives et summary.csv permettent d’auditer doublons, échecs et reprise.

## Qualification des exports (obligatoire pour le produit qualifié)

Activer `AERO25D_EXPORT_QUALIFICATION=ON` avec VTK Python ou PyVista installé dans
l’environnement de tests ; exécuter le label CTest `export-qualification` sur VTP/VTM/VTS,
y compris après déplacement. Un lecteur absent empêche la qualification.
Ouvrir les ensembles fin/épais et contours seuls dans ParaView, sélectionner les rôles,
inspecter BA/deux lèvres et conserver version, logs, captures et conclusion.
Le produit s’exécute sans ces paquets avec BUILD_TESTING=OFF.

J0 reste limité aux premières tâches de [tasks.md](tasks.md) : cible core/démonstration,
références analytiques, contrôles indépendants et exports minimaux avec vues BA/lèvres.
Ni campagne, ni CLI complète, Python, reproduction ou évaluateurs généraux ne précèdent le go.

## R2 — portée de l’exécution et étapes conditionnelles

Objectif produit : [roadmap 2D/2.5D/2.75D](../../docs/product-roadmap.md). Les commandes
ci-dessus ne fournissent que le mailleur plan. Aucun code produit n’est encore présent et
aucune de ces commandes n’a été exécutée lors de la réconciliation. L’implémentation reste
non lancée. `geometry_mode=planar_2d` exige Λ=0 ; planar_25d admet 0°–50°. Les configurations
unitaires JSON ou YAML suivent le même schéma 1.2 ; les exemples JSON/YAML sont équivalents.
Une configuration conical_275d, taper_ratio ou cas de vol actif doit être refusée explicitement.
L’extension future inactive est signalée, sans champ aérodynamique produit.

Le premier groupe reste T001–T019/J0, en C++/CMake, sans Python/campagnes/stockage complet
ni G-FORM/JG-C. Ensuite les parcours plans ont leur acceptation T083 ; elle ne valide pas le
conique. Les tâches T084–T088 mènent à la revue scientifique G-FORM. T089–T093 (contrat puis
démonstrateur géométrique fin/épais et JG-C) attendent un go de formulation et J0 accepté.
Aucune commande conique n’est présentée comme disponible. Les futurs essais d’équations,
de Cp/Cf et d’efforts sont dans la roadmap, hors réalisation 001.

## Précisions R3 à appliquer lors de la réalisation

Ces parcours restent à implémenter. Les exemples JSON/YAML sont des demandes 1.2 et non
la preuve d'un run réussi. [L'échelle physique](contracts/physical-scale.md) se mesure dans
le contour original : mètres seuls suffisent, une indication contradictoire est refusée ;
en adimensionnel k=L/c_input, même si c_input≠1. Sans échelle, géométrie autorisée mais y+
unavailable. Flèche/normalisation ne redéfinissent jamais cette échelle.

`quality_profile` est la politique de génération et l'audit par défaut ; `audit_profiles`
demande plusieurs contrôles indépendants. Pour réauditer :
`aero25d-mesh quality --bundle run --audit-profile Q1 --out audit --json`, ou
`client.quality(bundle=run.bundle, audit_profile="Q1", out="audit")`.
La campagne expose mesh_artifact/audits/estimate/point_result par point : un même maillage
peut avoir un audit passé et un autre échoué sans que leurs verdicts soient écrasés.
`--resume --retry-failed` / `resume=True, retry_failed=True` crée de nouvelles tentatives
des échecs ; la reprise ordinaire peut relire leur échec avec les mêmes paramètres.
Le paramètre curvature_regularization_over_c a été retiré : le supprimer d'une demande
ancienne, sans le remplacer par une valeur prétendument active.

T001–T019 reste le premier groupe, aux critères J0 inchangés. La revue scientifique G-FORM
reste ouverte ; T095 clôt un bilan aussi après no_go sans cocher les tâches coniques non
applicables. Aucune commande de ce guide n'a été exécutée comme implémentation pendant R3.
