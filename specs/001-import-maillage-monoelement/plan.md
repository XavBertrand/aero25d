# Implementation Plan: Import géométrique et maillage monoélément

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

**Branch**: `001-import-maillage-monoelement` (identifiant renvoyé par setup-plan ; aucune branche Git vérifiée/créée dans l'environnement) | **Date**: 2026-10-30 | **Spec**: [spec.md](spec.md)

**Input**: `specs/001-import-maillage-monoelement/spec.md`, plan du 2026-10-30 et compléments documentaires autorisés du 2026-10-31 (campagnes, VTP, qualification, ordonnancement J0).

## Summary

Construire un mailleur autonome CPU, utilisable depuis une commande et Python, qui contrôle
une source immuable, reconstruit sa géométrie, projette toute la section en flèche, génère
un maillage quadrilatéral structuré multibloc et en certifie les propriétés géométriques
mesurables. Le lecteur accepte plusieurs éléments ; le mailleur les refuse sans perte.
Aucun solveur, modèle de turbulence, infrastructure CFD générale ou dépendance CUDA.

Décision principale D-01 : **4 blocs pour un bord fin, 5 pour un bord épais**. Les deux blocs
de corps U/L partagent une interface au BA ; WU/WL séparent le sillage du corps. Le bord
épais ajoute H entre les deux lèvres et conserve la paroi du culot. Aux coins, les secteurs
latéraux ont de vraies cellules, pas des extrusions superposées ou des blocs écrasés.

Le premier jalon J0 devra démontrer les cartes et interfaces sur les références synthétiques
aux trois flèches avant le développement complet. La conception est décidée ; sa faisabilité
globale et les limites Q1 ne sont **pas encore vérifiées expérimentalement**.

## Technical Context

**Language/Version**: C++20, GCC≥12 de référence, Clang≥16 pour seconde compilation ; Python≥3.11.

**Primary Dependencies**: nlohmann/json 3.12.0 (MIT), OpenSSL≥3 libcrypto (Apache-2.0), sous-ensemble des prédicats de Shewchuk (domaine public), yaml-cpp 0.8.0 (MIT) pour le lecteur de documents JSON/YAML partagé, après J0 dès T025. Développement : CMake≥3.25/CTest (BSD-3-Clause), Ninja≥1.21 (Apache-2.0), Catch2 3.8.1 (BSL-1.0). Détails, raisons et notices dans [research.md](research.md).

**Storage**: Dossiers autonomes versionnés : source brute, configurations JSON/YAML normalisées selon le même schéma, tableaux binaires little-endian f64/u64, VTP/VTM/VTS ASCII, états de campagne JSON/CSV/journal et SVG. Aucune base de données ou service.

**Testing**: Catch2/CTest pour noyau ; unittest standard Python pour contrats/parité ; relecture indépendante obligatoire par VTK ou PyVista pour qualifier VTP/VTM/VTS, et ouverture ParaView tracée. Ces dépendances restent limitées aux tests/qualification. Tests d'acceptation demandés par la spécification, à réaliser ultérieurement.

**Target Platform**: Linux et WSL2, presets CMake compatibles CLion ; CPU seul sans toolkit CUDA. Les GPU RTX 4070 Ti Super 16 Go/A16-8Q 8 Go sont des contraintes futures d'exploitation, pas des performances revendiquées ici.

**Project Type**: Bibliothèque native, CLI et enveloppe Python mince utilisant le même moteur.

**Performance Goals**: Aucun temps de génération ni plafond de taille qualifié à ce stade. Mesurer durée, pic mémoire, nombre de cellules et coût des évaluateurs en E5 ; les budgets de ressources utilisateur sont des limites d'arrêt, pas des garanties de capacité.

**Constraints**: Source préservée ; pas de réparation, relaxation de tolérance ou baisse de résolution cachée ; binaire64 et modes numériques contrôlés ; aucune génération déléguée ; contrôles indépendants et exhaustifs ; contraintes géométriques originales et transformées distinctes.

**Scale/Scope**: Campagnes de maillage JSON/YAML séquentielles à points dédupliqués ; un seul contour par maillage, bords fins et épais ; 0°–50° ; quatre/cinq blocs, deux familles à au moins trois niveaux. Import multiélément sans génération ; étude conique et démonstration géométrique conditionnelle après G-FORM. Pas d’Euler/RANS, SA-neg, implicite, précision mixte, adaptation ou GUI complète.

## Constitution Check

La constitution a été établie en **1.0.0 le 2026-10-30**, puis amendée en
[1.1.0 le 2026-10-31](../../.specify/memory/constitution.md) via `speckit-constitution`.
Le premier établissement 1.0.0 a été réalisé à
partir du gabarit ; aucune ratification humaine antérieure n'est alléguée, aucun TODO restant.

| Gate | Avant recherche Phase 0 | Après conception Phase 1 / preuve documentaire |
| --- | --- | --- |
| I — Source et traçabilité | Conforme au contrat FR-001/004/007 | Données source/lexèmes immuables et transformations explicites, data-model et bundle |
| II — Conventions | Corrections H-03/H-05/H-07/H-08 effectuées | Repères S/N/W, inverse, Y/η et références transformées dans tous les contrats |
| III — États de preuve | Plan distinct de la réalisation | J0/E1–E5 marqués à exécuter ; aucune réussite maillage revendiquée |
| IV — Mesures et reproduction | 60 FR identifiées, dont les 45 antérieures conservées ; 20 PR classées courant/futur | verification.md et évaluateurs bornés ; manifestes et empreintes |
| V — Pas de données inventées | Objectifs distingués des résultats | Sources attribuées ; seule vérification arithmétique de formules effectuée, pas de benchmark |
| VI — Pas de relaxation | Budgets existants inchangés | Q1 figé par version, erreurs indéterminées bloquantes, sorties invalides |
| VII — Autonomie | Mailleur propre CPU | Dépendances génériques justifiées, aucune délégation CFD |
| VIII — Python | Moteur unique | Client subprocess/JSON, mêmes codes et données, pas d'algorithme Python |
| IX — CPU/CUDA proportionné | Aucun GPU nécessaire | Données planes contiguës ; étude conique, aucun solveur anticipé |
| X — Formulations/représentations | Roadmap 2D/2.5D/2.75D et cinq couches séparées | G-FORM ouvert bloque contrat métrique/construction coniques ; J0 indépendant |

Résultat R2 des gates : **conception documentaire conforme pour le chemin plan**, aucune dérogation. G-FORM est une porte scientifique ouverte ; les tâches coniques dépendantes sont explicitement bloquées, sans bloquer J0. Ce contrôle documentaire
ne constitue pas la validation numérique du produit. Les risques expérimentaux ne sont pas
reclassés en clarifications manquantes : leurs décisions et critères d'arrêt sont explicités.

## Project Structure

### Documentation (this feature)

```text
specs/001-import-maillage-monoelement/
├── spec.md                         # contrat comportemental corrigé, IDs conservés
├── checklists/requirements.md       # qualité documentaire de la spécification
├── plan.md                         # présent document
├── conical-study.md                # étude initiale, D-10/G-FORM ouverte
├── reconciliation.md               # inventaire et correspondances R2
├── research.md                     # Phase 0 et ajouts D-08/D-09, dépendances
├── topology.md                     # graphes, tables et coins 4/5 blocs
├── geometry-verification.md        # algorithmes, bornes et références V-01 à V-09
├── mesh-quality.md                 # construction, limites Q1 et famille F1
├── build-and-library.md            # presets et surface C++/vues de données
├── verification.md                 # FR → expériences/tests → preuves
├── coherence-review.md             # corrections et contrôle arithmétique limité
├── data-model.md                   # Phase 1
├── tasks.md                        # tâches ordonnées, J0 en premier
├── tasks-analysis.md               # analyse Spec Kit documentaire
├── contracts/                      # commandes, API, campagne, bundle et schémas JSON
└── quickstart.md                   # parcours futurs et vérification
```

Le découpage autorisé du 2026-10-31 est dans [tasks.md](tasks.md), avec le rapport de cohérence [tasks-analysis.md](tasks-analysis.md). Aucun travail produit n’est exécuté.

### Source Code (repository root)

Arborescence **prévue**, non créée à cette étape :

```text
CMakeLists.txt
CMakePresets.json
cmake/                             # options CPU, installation, vérification dépendances
include/aero25d/mesh/               # types de géométrie/blocs/rapports, vues contiguës
src/
├── geometry/                       # source, logique/physique plane, repères, arcs/évaluateurs
├── experiments/conical/            # seulement après go G-FORM, hors API produit plane
├── meshing/                        # squelette 4/5 blocs, distributions, TFI, lissage
├── quality/                        # métriques et intégrité indépendante du générateur
├── io/                             # JSON, tableaux, SHA256, VTK, SVG, reproduction
├── campaign/                       # orchestration partagée séquentielle, après J0
└── cli/                            # présentation seulement
python/aero25d_mesh/                # Client, résultats et exceptions, aucun calcul géométrique
tests/
├── qualification/                  # lecteur VTK/PyVista obligatoire et ouverture ParaView
├── unit/                           # géométrie, arithmétique, cellules et paramètres
├── topology_demo/                  # jalon J0 limité à la géométrie
├── integration/                    # V01 à V09, familles et reproduction
├── contracts/                      # CLI/Python, schémas et relecture VTK indépendante
└── data/{V01,…,V10,configs}/        # sources/formules/oracles figés
third_party/manifest.json           # versions exactes, licences, empreintes et notices
```

**Structure Decision** : modules géométrie, maillage, qualité et IO d’un noyau, orchestration de campagne séparée, pas de services ; qualité
réutilisable pour inspecter un bundle sans régénérer. L'exécutable et Python n'effectuent pas
leurs propres calculs de géométrie. Le futur solveur pourra consommer des vues non propriétaires
de coordonnées/connectivité et les normales/aires exposées ; pas d'allocateur GPU, de flux CUDA,
de modèle de turbulence ou d'interface abstraite de solveur créée maintenant.

## Décisions techniques et ordre de livraison futur

Les décisions D-01–09 du chemin plan ont des contrats concrets à qualifier. D-10 est une décision scientifique ouverte : aucun contrat métrique conique détaillé ne sera fixé avant son go.
Le détail des presets et des interfaces C++ est fixé dans [build-and-library.md](build-and-library.md) :

| Décision | Document faisant autorité | Résolution |
| --- | --- | --- |
| D-01 | topology.md | 4/5 blocs, tableaux exhaustifs, coordonnées d'interface uniques |
| D-02 | geometry-verification.md, mesh-quality.md | lecteur à états, prédicats exacts, spline clamped protégée, densité et TFI/lissage contraint |
| D-03 | mesh-quality.md | Q1, métriques/normalisations/limites/sévérités, gel après revue d'expériences |
| D-04 | geometry-verification.md, verification.md | références polynomiales propres, mutations exactes, oracles séparés et preuves |
| D-05 | data-model.md, contracts/, research.md | noyau C++, CLI, Python subprocess, JSON/binaires, VTK/SVG, presets CPU |
| D-06 | mesh-quality.md | F1, facteurs r=1, 2, 4 en subdivisions, lois exponentielles comparables, cœur gelé et domaine séparé |
| D-07 | research.md, contrats | y+ optionnel à partir de u_tau fourni et ν, aucun modèle empirique automatique |
| D-08 | contracts/campaign.md | JSON/YAML, expansion, séquence, clés effectives, progression et reprise stricte |
| D-09 | contracts/bundle.md | VTP par rôle, VTM/VTS, associations, qualification VTK/PyVista et ParaView |
| D-10 | conical-study.md, roadmap | Étude comparative initiale ; décision ouverte et bloquante avant métriques/construction coniques |

### D-08 — Campagnes et identité effective (complément)

Contrat [contracts/campaign.md](contracts/campaign.md), schémas de demande/état et exemples
JSON/YAML. La campagne développe profils/flèches/configurations/niveaux puis orchestre le
noyau et le résolveur F1 existants. Identités `mesh_key`/`reuse_key`, exécution séquentielle,
commits progressifs, reprise stricte, politique d’échec obligatoire et inventaire sans génération.
Le module C++ `src/campaign/` est partagé par la CLI et Python ; le lecteur JSON/YAML commun parse un sous-ensemble
JSON-compatible, sans ajout d’un mailleur Python. Les cas de vol restent une extension inactive.

### D-09 — Contours/frontières et qualification (complément)

VTP pour les contours, parois et interfaces avec rôles/IDs/repères/unités ; VTM/VTS pour
les blocs structurés. Contrat détaillé dans [contracts/bundle.md](contracts/bundle.md).
VTK ou PyVista obligatoire pour qualifier tous les formats ; ParaView doit être ouvert avec
preuves. Aucune de ces bibliothèques n’est liée au produit livré.

### J0 — Démonstration géométrique des topologies, premier jalon bloquant

Après le socle minimal CMake, coordonnées/blocs/prédicats, contrôles indépendants et générateurs de références,
réaliser uniquement squelette, coins, distributions et premier remplissage. Exercicer les neuf
combinaisons V-01/02/03 × 0°/25°/50°, avec culot fourni et explicitement complété. Avant
l'intégration complète CLI/Python, produire graphes, tableaux des raccords, min aires/Jacobiens,
figures BA/lèvres et tests du cœur gelé.

Critères : quatre ou cinq blocs attendus, aucune face/sommet effondré, interfaces bijectives
orientées, positivité de toutes les cellules, absence de chevauchement global et préservation
des caractéristiques dans les budgets. Vérifier aussi que les premières cellules peuvent
satisfaire Q1, surtout 30° de non-orthogonalité paroi et 2 % de hauteur. Si les cartes échouent,
**arrêter la généralisation** et revoir la construction/topologie avec preuves ; ne pas développer
les autres parcours en supposant le risque résolu. J0 n’exige pas une résolution aérodynamique et ne dépend ni de G-FORM ni de JG-C. Son succès ne qualifie aucune géométrie conique.

Périmètre exécutable : bibliothèque et cible de démonstration CTest sur adaptateur analytique
V-01/02/03, sans attente de CLI complète, Python, campagne, reproduction ou évaluateur général
Hausdorff/spline. Un oracle polynomial indépendant, bornes ciblées et contrôles algébriques
certifient les budgets de fidélité, caractéristiques et Q1 applicables à ces références.
Les métriques sont exhaustives par cellule/face/interface ; aucun coin n’est exclu.
Les VTP/VTM/VTS minimaux et SVG BA/lèvres servent à l’inspection. Ils ne dépendent pas du
bundle complet ni de la qualification VTK/ParaView, programmée après J0. Les limites E3 sur
culots 10⁻⁶ c et 10⁻¹⁰ c n’ajoutent pas d’obligation de succès aux neuf cas usuels.
Le résultat de J0 est un go/no-go conservé avec paramètres, versions, tableaux et preuves
négatives. Un no-go bloque toutes les phases de généralisation jusqu’à révision et réexécution.

### J1 — Géométrie et certification

Finaliser lecteur/provenance, indications, reconstruction, projection/inverse, distances et
rayons bornés, corpus V-04/05/06. Gel des oracles et Q1 après E1–E3, et revue de toute révision.
Parcours de contrôle utilisable indépendamment du générateur. Pas de rejet lexical abusif
pour une géométrie importable dont seul le maillage ou le rayon ne sont pas certifiables.

### J2 — Maillage et qualité complète

Compléter distributions, coins, transition, lissage optionnel et inspecteur indépendant.
Contrôles de toutes les cellules et interfaces, cas injectés, familles F1 et domaines séparés.
Un échec conserve un dossier de diagnostic invalide. La réussite de J0 ne dispense pas des
contrôles complets à tous les niveaux/demandes.

### J3 — Parcours, exports et reproduction

Finaliser configuration/rapport/bundle, cinq SVG, VTP/VTM/VTS, commandes, Client Python et campagnes de maillage ;
relecture indépendante obligatoire et ouverture ParaView, CPU sans CUDA/solveur installé et reproduction à version/environnement
figés. Exécuter la campagne FR complète avec preuves ; documenter seulement ensuite ce qui
est réellement vérifié et son enveloppe de validité.

Ces jalons cadrent les futurs travaux ; ils ne sont pas une liste de tâches exécutée.

## Risques, expériences et limites connues

| Expérience à réaliser | Risque / décision éprouvée | Critère pour poursuivre et preuves |
| --- | --- | --- |
| E1 / J0 | Injectivité globale des 4/5 blocs et coins épais, transition très étirée | Neuf cas, cartes locales/globales et tous les critères J0 ; revoir conception sur échec |
| E2 | Rayon BA et budgets spline/polygone/analytique après compression | Certificats d'intervalles pour distances/rayons/angles S/N/W, comparaison oracles exacts ; pas de certification par figure |
| E3 | Limites Q1, coût numérique, culot très mince et amplitudes de distribution | Balayages explicites h1/c de 10⁻⁴,10⁻⁵,10⁻⁶ ; g=1,05/1,15/1,25 ; culot 2b=0,02 puis 10⁻⁶ et 10⁻¹⁰. Attendre succès ou refus motivé selon les limites, jamais promettre succès partout ; gel Q1 par revue des distributions et cas négatifs |
| E4 | Cohérence F1 et indépendance du domaine | Deux familles aux trois niveaux, R=10c/20c à paramètres intérieurs fixes ; mêmes courbes/reconstruction et coordonnées de cœur identiques, pas décroissants ; limites toutes respectées pour qualifier la famille |
| E5 | Reproductibilité, formats et ressources | Relecture VTK indépendante, corruption de tailles/hashes, deux compilations de contrôle et deux exécutions même version/environnement ; temps/mémoire seulement mesurés et rapportés, aucun seuil de performance inventé |

Les entrées E3 extrêmes ne doivent pas toutes réussir : elles délimitent le domaine admissible
et prouvent le refus correct. Les cas obligatoires V-01 à V-03 doivent réussir pour qualifier
la fonctionnalité ; leur retrait nécessiterait une modification explicite du périmètre, interdite
comme simple moyen de faire passer la campagne.

Limites initiales : contour ordonné simple, branches sans repli, BA lisse identifiable, lèvres
convexes, culot droit (éventuellement incliné) compatible avec les secteurs. Culot courbe complexe,
forme rentrante ou données insuffisantes : import/inspection possibles si valides, maillage
refusé explicitement si la construction ne s'applique pas. La précision décimale perdue ne se
récupère pas par normalisation. L'évaluateur peut refuser faute de preuve ou de ressources.
La petite épaisseur ne déclenche jamais une conversion en pointe.

**Décisions techniques ouvertes** : G-FORM/D-10 reste ouvert pour le conique, sans clarification nécessaire au démarrage du chemin plan
pour produire des tâches. La suffisance de la construction 4/5 blocs, les paramètres de
transition compatibles pour les neuf cas, le besoin effectif du lissage et l’acceptabilité
numérique de Q1 restent à décider **sur preuves** de J0/E1–E3. Un échec implique une révision
de construction puis un nouveau J0 ; aucun go n’est anticipé. Le coût des évaluateurs,
l’enveloppe qualifiée et les versions exactes de l’environnement VTK/ParaView restent à mesurer
ou enregistrer dans les tâches de qualification. Les références analytiques et les limites proposées sont
fixées comme point de départ ; toute révision avant gel conserve historique et preuves négatives.
Les versions exactes de paquets système et empreintes vendored seront enregistrées au socle
futur, sans inventer ici une installation qui n'a pas eu lieu.

## Complexity Tracking

Aucune violation constitutionnelle à justifier. Les blocs de sillage supplémentaires ont une
raison géométrique et permettent des contrôles de résolution distincts ; pas d'infrastructure
pour des extensions non demandées. Les évaluateurs bornés sont nécessaires pour éviter une
fausse certification, même si leur coût expérimental reste inconnu.

## État de préparation à speckit-tasks

Phase 0 (recherche/décisions) et Phase 1 (modèle, contrats, guide) produites. Le contrôle
constitutionnel documentaire est conforme avant/après conception. Tâches générées sur la demande du 2026-10-31, avec J0/E1 avant la généralisation et expériences
séparées de l’acceptation. Le premier incrément est J0, puis US1/US2, US3/US4 et US5.
L’implémentation n’est pas lancée automatiquement.
Les hooks ont été contrôlés avant/après constitution et plan : aucun fichier extensions.yml,
donc aucun hook à exécuter. Le script setup-plan a résolu cette fonctionnalité via feature.json.

## Séparation des dépendances avant/après J0

Avant J0 : CMake C++20/CTest, Catch2 et prédicats robustes, sérialisation minimale JSON/XML/SVG,
références analytiques et contrôle numérique indépendant. Les générateurs/oracles de J0 sont en C++ ; le SHA-256 du dossier de preuve
est calculé par file(SHA256) CMake, sans interpréteur Python requis. Les cibles CLI/campagne et leurs
dépendances ne sont pas construites avant leur ajout ; le CMake minimal doit être utilisable
sans fichiers futurs ni cibles vides. Après J0 : OpenSSL dès la provenance US1 puis réutilisé pour les bundles, yaml-cpp dès T025 pour configurations et campagnes,
évaluateurs généraux et parcours complets. La qualification indépendante VTK/PyVista possède
une option CMake distincte `AERO25D_EXPORT_QUALIFICATION=ON`, qui échoue si son lecteur manque ;
les tests ordinaires/J0 ne requièrent pas cette option. Aucun skip ne vaut qualification.

## Réalisation en deux lots, sans seconde fonctionnalité

**Lot A plan** : T001–T083 révisées, acceptation T083 limitée aux capacités planes.
Les FR-001–045 sont conservées ; représentation plane et refus explicites des modes futurs
sont couverts avec FR-047–049/055–057/059. Les règles 1.2 des contrats et `effective-mesh-v3`
sont ajoutées avant réalisation ; lecture 1.0 par migration explicite, aucun conique implicite.

**Lot B étude/géométrie conique** : US6, T084–T094, décision T088, JG-C T093 ; synthèse
conditionnelle T095. Ordre : corpus → similitude/géométrie → fermeture visqueuse/SA-neg →
restitution/efforts/validation → **go G-FORM** → contrat géométrique/métrique → oracles →
construction expérimentale → JG-C. Le contrat métrique n’est pas détaillé aujourd’hui.
Un no-go conserve preuves/questions et laisse le lot B incomplet ; il n’annule pas une
acceptation plane réellement obtenue. Le graphe de tâches rend toutes ces dépendances explicites.

Les travaux futurs d’équations (uniforme cartésien, conservation/identités métriques, limite
2.5D, diffusion/turbulence, références indépendantes) appartiennent à FUT-01–07 de la roadmap.
Le champ `geometry_mode=conical_275d`, `taper_ratio` ou trois coordonnées ne constitue pas une
capacité disponible ; l’API produit le refuse jusqu’à une future spécification d’équations.

## Révision des entrées et dépendances

T025 introduit yaml-cpp dans un **lecteur C++ commun de documents** (`src/io/document_reader.cpp`,
cible `aero25d_document_io`) après J0. Configurations unitaires et campagnes JSON/YAML suivent
les mêmes règles de scalaires/erreurs et leurs schémas respectifs. T067 réutilise ce lecteur.
Le cœur géométrique et J0 n’exigent pas ce parseur ; aucune duplication Python/campagne.
Le mode plan garde `[N,2]` FP64 et métriques euclidiennes simples ; les couches source,
logique, physique, transformations et métriques sont identifiées sans infrastructure RANS générale.
