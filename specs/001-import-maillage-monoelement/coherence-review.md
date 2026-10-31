# Revue de cohérence — corrections et contrôle limité

Date : 2026-10-30. Cette revue concerne les documents et les formules ; aucun noyau C++,
mailleur, prototype topologique ou test logiciel de produit n'a été exécuté.

## Changements de spécification

- H-03 : quart de corde conservé, base globale et base d'extrusion distinctes ; Y et η ne
  coïncident qu'à Λ=0. Le contrat futur autorise un vecteur moment et une projection explicitement
  choisie. Aucun moment aérodynamique calculé.
- H-05 : chaîne S (source) → N (section normale, mêmes unités) → W (normalisée, adimensionnelle).
  Pour inverser : ξ=s*u+aξ, z=s*v+az, puis X=ξ/cosΛ+Xref, Z=z. L'ancien ordre était incorrect.
- H-07/H-08 : original comparé à original ; transformé comparé à la transformation de la même
  référence. Pour une carte A, κ_transformée=det(A)κ/||A*t||³ ; le rayon BA des fixtures à
  tangente verticale devient R/cosΛ. La compression voulue n'est pas un défaut géométrique.
- Matrice FR-007/011/012 et critères de preuve précisés ; FR-001 à FR-030 et V-01 à V-09 conservés.
- SC-001 précise les étapes : I-18/I-23 restent inspectables si lexicalement valides, sans faux
  certificat de rayon ou maillage. Ce n'est pas une réduction des défauts à couvrir.
- Gouvernance mise à jour après établissement de la constitution 1.0.0 sur autorisation courante.

## T-01 — Vérification arithmétique de l'ordre d'inversion

Calcul effectivement effectué avec Python standard binary64 pendant la rédaction :
source S=[(-2.3,4.7),(0.2,4.91),(3.4,5.2)], Xref=0.37,
translation dans N a=(2.1,-3.6), c=hypot(5.7,0.5). Pour chaque point et combinaison :
ξ=(X−Xref)*cosΛ ; u=(ξ−aξ)/s ; v=(Z−az)/s ;
ξ2=s*u+aξ ; z2=s*v+az ; X2=ξ2/cosΛ+Xref ; Z2=z2.
Mesure : max hypot(X2−X,Z2−Z)/c. Les assertions demandent ≤10⁻¹².

| Λ (degrés) | s (unités source) | Résidu maximal / c observé |
| --- | --- | --- |
| 0 | 0.4 | 1.7354651051267495e-16 |
| 0 | 7.0 | 1.7354651051267495e-16 |
| 25 | 0.4 | 1.5522471790568922e-16 |
| 25 | 7.0 | 1.5522471790568922e-16 |
| 50 | 0.4 | 2.1952090127776354e-16 |
| 50 | 7.0 | 2.1952090127776354e-16 |

Les six assertions passent pour ces données. Ce résultat vérifie l'exercice arithmétique et
l'ordre des formules, pas la future implémentation ni les bornes rigoureuses des évaluateurs.
La constante cos a ici été évaluée par `math.cos(math.radians(Λ))` ; la certification du noyau
utilisera les bornes décrites dans geometry-verification.md. Les entrées, formule et résidus
ci-dessus rendent ce petit contrôle reproductible sans le présenter comme un résultat de CFD.

Les preuves de produit restent à produire : transformations multiéléments, perte de précision,
unités et échelles physiques, cellules et raccords, limites Q1 et performances réelles.

## Revue du 2026-10-31 — préparation des tâches et analyse Spec Kit

Sur demande explicite : ajout US5/FR-031–045/SC-007–008/V-10, campagnes JSON/YAML et
contrats de clés/reprise, VTP par rôle et qualification VTK/PyVista/ParaView obligatoire.
Les 30 exigences initiales restent textuellement identiques ; aucun budget H/Q1 n’a été relâché.
La constitution reste inchangée. Les schémas d’indications acceptent désormais les positions
sur segments déjà décrites par le plan ; les diagnostics acceptent les identifiants d’export.

[tasks.md](tasks.md) contient 83 tâches, dont T001–T019 seulement pour le premier jalon J0.
Chaque tâche donne résultat, fichiers, dépendances, FR/SC et vérification/preuve attendue.
Le go/no-go bloque toute généralisation ; la preuve analytique ciblée de J0 ne qualifie pas
le lecteur ou les évaluateurs généraux. OpenSSL est ajouté après J0 dès la provenance US1,
puis réutilisé pour les bundles ; yaml-cpp reste dans l’orchestration de campagne après J0.

L’analyse [tasks-analysis.md](tasks-analysis.md) a relevé puis corrigé les incohérences de
contrats, de compte de cas et d’ordonnancement, sur autorisation de la demande courante.
Elle conserve les décisions expérimentales ouvertes sans prétendre les avoir tranchées.

Contrôles documentaires effectivement exécutés dans cet environnement : setup-tasks,
check-prerequisites avec tâches, unicité/préservation des 30 FR originales, 45 lignes dans
chacune des matrices de traçabilité, 83 tâches au format prescrit, dépendances acycliques,
53/53 FR/SC rattachées, sept schémas Draft 2020-12 validés avec jsonschema 4.10.3 et registre
local, trois exemples JSON conformes, égalité JSON/YAML avec PyYAML 6.0.1, cinq demandes
négatives rejetées par le schéma et liens Markdown locaux résolus. PyYAML n’est utilisé que
pour cet audit, sans ajout au produit. Les notices/format VTP nouveaux ont des sources primaires
citées dans research.md. Aucun hook enregistré (.specify/extensions.yml absent).

Aucun code produit, fichier CMake de réalisation, fixture de maillage, test logiciel du
mailleur ou résultat aérodynamique n’a été créé/exécuté. Les scripts d’audit rédactionnel
utilisent /tmp ; ils ne constituent pas une implémentation de la fonctionnalité.

## Réconciliation R2 — objectif global et préparation conique

La demande suivante du 2026-10-31 révise les exigences, sans créer de fonctionnalité
concurrente. L’implémentation a été suspendue pendant la réconciliation ; l’inventaire n’a
trouvé aucun code/binaire/maillage de produit, seulement les documents et l’outillage Spec Kit.
Les 83 tâches antérieures étaient ouvertes : aucune réussite n’a été transférée ou invalidée
par suppression. Leur correspondance complète est dans [reconciliation.md](reconciliation.md).

La [roadmap d’autorité](../../docs/product-roadmap.md) fixe le produit final 2D/2.5D/2.75D
compressible SA-neg et ses sorties sectionnelles, sans VLM ni solveur d’aile. La constitution
est amendée 1.0.0→1.1.0 sur la demande courante, avec principe scientifique X, sans relâcher
les autres principes ni H/Q1. Les paragraphes R1 ci-dessus restent l’historique de la première
préparation ; ils ne décrivent pas le dernier nombre de tâches ou de versions.

FR-001–045 et SC-001–008 sont conservés ; FR-009/010/022 reçoivent une précision de portée
plane sans retirer les clauses précédentes. FR-046–060, SC-009–012, US6 et les cas V-11/
V-C01/V-C03 complètent 001. Les 95 tâches incluent T001–T083 préservées (41 révisées,
42 aux critères préservés) et T084–T095 nouvelles ; toutes restent non exécutées.

J0 conserve ses neuf configurations et son socle C++/CMake. Le chemin scientifique conique
est séparé : étude primaire initiale → dérivations/revue G-FORM T088 → contrat métrique T089
→ démonstrateur et JG-C T093. G-FORM reste ouvert ; aucun contrat métrique détaillé ni code
conique n’a été créé. Les références primaires et limites d’accès/preuve sont dans
[conical-study.md](conical-study.md), notamment la distinction Polytechnique 2.5D, Euler
conique et méthode sectionnelle DLR. Aucun calcul de pression ou résultat RANS n’est inventé.

Contrats 1.1 : représentation plane et associations explicites, coordonnées Float64, lecture
1.0 migrée avec règle/provenance, JSON/YAML communs dès T025, campagne réutilisant ce lecteur.
Les champs actifs de vol/conique sont refusés ; l’archive future inactive est signalée.
Les références de Cp/champ p/Cz/Cx/Cm sont fixées dans la roadmap, avec intégrales coniques
encore à dériver ; les futurs caches de solution restent distincts des clés de maillage.

Les mécanismes installés Spec Kit 0.15.1 (résolution de gabarit, setup-plan, setup-tasks et
check-prerequisites pour analyze) ont été exécutés en conservant les artefacts. Le workflow
complet, qui poursuivrait vers implement, n’a pas été lancé. Aucun hook enregistré.
Les audits documentaires contrôlent 60 FR/12 SC, matrices 60/60, 95 tâches/états/dépendances,
isolement de J0, blocage conique, huit schémas 2020-12, exemples/legacy, deux équivalences
JSON/YAML et 16 refus négatifs. Q1 est inchangé. Ces résultats ne constituent aucun test de
produit ni qualification scientifique. Voir [l’analyse R2](tasks-analysis.md).

## R3 — correction documentaire autorisée des cinq écarts

U1 : identités/résultats immuables par génération, audit, estimation et point ; la politique
qualité qui peut modifier les coordonnées est incluse dans la génération. I1 : T095 a une
branche bilan no_go indépendante des tâches go, laissées non applicables sans coche ;
G-FORM reste ouverte. U2 : paramètre inutilisé retiré avant T010, moniteur existant justifié
sans epsilon, essais d'échelle/sensibilité/protection. U3 : source-scale-v1 dans S,
mètres faisant autorité, adimensionnel L/c_input, tolérance1e-10/0m et conversion unique.
I2 : histoire de constitution corrigée seulement dans le plan, principes inchangés.

Le détail des correspondances et la liste des fichiers R3 sont dans
[reconciliation.md](reconciliation.md#r3--correction-des-cinq-écarts). Contrôles documentaires
exécutés : 10 schémas, 29 cas synthétiques de forme, deux équivalences JSON/YAML, conservation
60 FR/12 SC/95 tâches ouvertes, couverture72/72, deux graphes acycliques et clôturables,
liens locaux, constitution et Q1 inchangés. Aucun test du mailleur, aucune implémentation.
La nouvelle analyse Spec Kit qui suit est en lecture seule et rendue dans la réponse ;
le rapport R2 conservé n'est pas une acceptation des nouveaux critères R3.
