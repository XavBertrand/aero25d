> Historique R2 ci-dessous, conservé avec ses correspondances. La révision courante est
> [R3 — correction des cinq écarts](#r3--correction-des-cinq-écarts) : contrats 1.2, clés v3
> et branches go/no_go remplacent les règles R2 correspondantes, sans effacer leur historique.

# Réconciliation R2 — inventaire, contradictions et correspondances

**Date** : 2026-10-31. **Autorité** : demande de révision globale des exigences après tâches.
Même fonctionnalité `001-import-maillage-monoelement`, aucune seconde spécification créée.
L’implémentation a été suspendue et n’a pas démarré pendant la réconciliation. Les documents
ne constituent ni acceptation scientifique ni validation de produit.

## Mécanisme Spec Kit effectivement utilisé

Version installée : **specify 0.15.1**, intégration Codex (métadonnées .specify/integration.json
et `specify --version` concordantes). Les compétences installées specify/plan/tasks/constitution
et analyze guident la révision **en place**, selon l’instruction utilisateur de réutilisation.
Le gabarit actif a été résolu par `specify preset resolve spec-template` et lu, sans recopier
un gabarit sur la spécification existante. setup-plan --json a conservé le plan existant ;
setup-tasks --json a résolu les mêmes chemins et le modèle actif. L’analyse utilise ensuite
check-prerequisites --json --require-tasks --include-tasks. feature.json est inchangé.
Le workflow complet installé enchaîne implement : il n’a donc pas été lancé. Les étapes
individuelles documentaires suffisent ici ; aucune commande d’implémentation n’est déclenchée.
Aucun .specify/extensions.yml trouvé : aucun hook avant/après les étapes concernées.

Les instructions applicables ont été recherchées jusqu’aux parents : aucun AGENTS.md trouvé.
Les compétences locales et la constitution sont les instructions de dépôt disponibles ;
les mécanismes .specify/scripts, modèles et workflow ont été inspectés et conservés.
La compétence plan demande une recherche déléguée ; deux agents ont effectué uniquement des
lectures (sources primaires et audit contrats), sans édition ni implémentation.

## Inventaire initial et impact du code/preuves

31 artefacts de fonctionnalité/constitution ont été inventoriés avec taille et SHA-256 avant
modification. Le tableau ci-dessous garde ces empreintes. Outillage supplémentaire inspecté :
compétences sous .agents/skills, scripts/templates/configuration/workflow sous .specify.
Aucun répertoire src/include/tests/python/evidence, CMakeLists de produit ou binaire/maillage
n’était présent. Les scripts Bash existants sont l’outillage Spec Kit, pas un mailleur réalisé ;
ils ne sont ni supprimés ni présentés comme validés par la réconciliation.
Les 83 tâches étaient toutes ouvertes. Les seules preuves présentes étaient documentaires
(checklists, rapport d’analyse précédent, T-01 arithmétique historique), sans qualification
logicielle ou scientifique du produit. Aucune case produit réussie ne pouvait être transférée.

| Artefact initial | Octets | SHA-256 avant R2 | État courant |
| --- | ---: | --- | --- |
| `.specify/memory/constitution.md` | 5919 | `7a202b29198edc6a6e5de675dbb728db682bc42497534ea29968d176c3c1b521` | révisé en place |
| `specs/001-import-maillage-monoelement/build-and-library.md` | 5526 | `d6e0d5513301ef7f82593c7c132a09de1442879fa67388221ea05c72bdb8e58c` | révisé en place |
| `specs/001-import-maillage-monoelement/checklists/plan-review.md` | 5308 | `e5554f57a7b2e5da19aeac7bb764536332d6172b336c28954495e08c06d85d57` | révisé en place |
| `specs/001-import-maillage-monoelement/checklists/requirements.md` | 5299 | `9ece8b1663fc942709c505187c7923ac554664181d3f6028f8a9adb6c6790b1d` | révisé en place |
| `specs/001-import-maillage-monoelement/coherence-review.md` | 5204 | `4cad2d6aede0c2ec177a22946c31b6d8fc31901cffc42b2c55b3a0ce256cfbf6` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/README.md` | 9519 | `9f66e1e678cf15d227877d59e17a372e241b97801a6fff1cc80e50efd257cbe1` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/bundle.md` | 11518 | `84d5139258f895af23d386e1d06151a35e5bcd190d5d4b4815f3ec338328ad2b` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/campaign-state.schema.json` | 6811 | `3523c246aa219debc58b57a5396d11318d32cf7d1e92e3bfed95af661cc4dbff` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/campaign.example.json` | 1146 | `07708e35ed59332b09962cf4dae5b83fc9307ce3f361e5ed82f869dfbce67928` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/campaign.example.yaml` | 869 | `e6320182cc8ef9338f498071642e35f56e9a6016565cd5cb735e5295d854d30b` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/campaign.md` | 11810 | `ac360f2d0bca242fb920b5b24d3944ea113aa565133c660f983543d97c5b798d` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/campaign.schema.json` | 5160 | `12cf9abebd28bac2f47bfd5d444cbc29685f8d961628937a1bf3c0c536555454` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/cli-python.md` | 9651 | `a718360fcfc5bc8a0aa2e127f4dec0cd43245d5b477dd022bcb05ee885ed3c64` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/configuration.example.json` | 871 | `8da653f1ea69b6c91e87398f4449131ee9d264c2e7a1d95e6a11d93b24f9f223` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/configuration.schema.json` | 9461 | `582185780c8386211c8c5fec7b058cc2402021aed98f87b26525b9854633412c` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/manifest.schema.json` | 6080 | `d49bbd78f559848b32556a266a074b91cb74639c28bdb638a835af098a4fec29` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/quality-profile-Q1.json` | 9759 | `d19e6062a01e8ce8dbeedbd7aaa85dee0d4127c0e8084198e35d7851778dfb71` | préservé |
| `specs/001-import-maillage-monoelement/contracts/quality-profile.schema.json` | 1746 | `ae90d9ad0b67a6f4e1e99dab2d57ddaec8fb1dab296c4bb48a821fd1dffa8452` | préservé |
| `specs/001-import-maillage-monoelement/contracts/report.schema.json` | 8273 | `973e2bdecf5dc1f14d4bfe57b9b4ad0e7ff0c35f5aecb86ee0df393a12c35c29` | révisé en place |
| `specs/001-import-maillage-monoelement/contracts/topology.schema.json` | 2780 | `d93a0aa8f9215d257404ada61c3ab4ef4756d6c66bc2a0089c8192039f6f469f` | révisé en place |
| `specs/001-import-maillage-monoelement/data-model.md` | 13269 | `52deb9436c9423ab1033bb27d9b56f6ea115b4f39b328cf688738cb36d9b7811` | révisé en place |
| `specs/001-import-maillage-monoelement/geometry-verification.md` | 30817 | `fa4100161758ccbbc108cb3aa50d90234d11067cd4400a059a7a2686ce18b8e3` | révisé en place |
| `specs/001-import-maillage-monoelement/mesh-quality.md` | 14754 | `a73ea5525edae246157b3ce4b9333f5762f196855673415cb3dc5f655232a363` | révisé en place |
| `specs/001-import-maillage-monoelement/plan.md` | 20675 | `09812c4e338d2a8b2e151cf43371952b91a0c09b5b81d637450762f9b9a5832a` | révisé en place |
| `specs/001-import-maillage-monoelement/quickstart.md` | 10710 | `84b0f7d4b632ead40057b4f99b45d83351003649bb258fcaa4fd98384ca88f39` | révisé en place |
| `specs/001-import-maillage-monoelement/research.md` | 12323 | `0668b70015ce7e714272ab1f0a8a913affbfea1898ce56716888fac29d608741` | révisé en place |
| `specs/001-import-maillage-monoelement/spec.md` | 55743 | `7377b426acce75c5d451bb767421412a509a5d8b470db4c2be1142590a51ae9c` | révisé en place |
| `specs/001-import-maillage-monoelement/tasks-analysis.md` | 13492 | `8dfa38c21e6122cfac4462706ef1d1d8a745c2ace12a48b07ee688619b8d4a2f` | révisé en place |
| `specs/001-import-maillage-monoelement/tasks.md` | 71079 | `ecf491ac3a7453aa409f328fdfab16ecb8fdfbe6981a833a6effa2606edc16f5` | révisé en place |
| `specs/001-import-maillage-monoelement/topology.md` | 21522 | `9a7aa709fdd3bfda6f7eb31e6439ba656cce60ca774c9aa06e1a8093da72afe7` | révisé en place |
| `specs/001-import-maillage-monoelement/verification.md` | 19427 | `88c282e51ca242051212b0db3dbd067f0c8897661d7bf5dd80622ef3e74c5734` | révisé en place |

Les nouveaux artefacts sont docs/product-roadmap.md (autorité produit), conical-study.md
(annexe scientifique de 001), ce registre, representation.schema.json (représentation plane)
et configuration.example.yaml (équivalent au JSON). Aucun contrat métrique conique détaillé
n’a été créé ; son chemin futur dans T089 reste volontairement inexistant.
Les snapshots techniques d’avant révision et scripts d’audit sont sous /tmp et ne sont ni
spécifications actives ni code produit. Les empreintes et correspondances utiles persistent ici.

## Contradictions et exigences manquantes à l’entrée

| ID | Gravité initiale | Artefacts | Constat / correction R2 |
| --- | --- | --- | --- |
| R-01 | HIGH | spec, constitution, plan | Objectif global resté 2.5D et sans autorité unique ; roadmap 20 PR, constitution 1.1.0 et références 2D/2.5D/2.75D ajoutées |
| R-02 | HIGH | modèle, topologie, stockage | Confusion possible entre indices, coordonnées planes, bases et métriques ; cinq couches séparées et portée plane explicite |
| R-03 | HIGH | research, tâches | Étude et décision coniques absentes ; corpus primaire, limites de preuve et porte G-FORM T088 avant T089–T093 |
| R-04 | HIGH | besoins futurs | Effilement dans pression locale non formulé, viscosité/SA-neg et fermeture transversale manquantes ; étude et FUT-02/03 explicités, aucun Euler qualifié RANS |
| R-05 | HIGH | références/résultats | Cp pariétal original et champ p dans plan physique confondables ; sorties séparées et intégrales coniques à dériver, q∞/c₀/axes/mesure Y fixés |
| R-06 | MEDIUM | configuration/CLI/Python | JSON seul pour configuration unitaire, modes implicites ; 1.1 explicite, YAML partagé T025 et refus actifs identifiés |
| R-07 | MEDIUM | manifeste/exports | Représentation plane et association de champ non explicites globalement ; descriptor plane-v1, point/cell/face, FP64 et tailles vérifiées |
| R-08 | MEDIUM | campagnes/cache | Version des modèles et distinction future clé de solution insuffisamment explicites ; effective-mesh-v2 et archive inactive signalée, incidence descriptive toujours dédupliquée |
| R-09 | MEDIUM | tâches/qualification | Aucun jalon conique ni dépendance scientifique ; US6, JG-C conditionnel et décisions séparées lot A/T083, deux lots/T095 |
| R-10 | MEDIUM | roadmap | Vérifications uniforme cartésien, limite 2.5D et séparation numérique/modèle absentes du programme conique ; FUT-01–07 et T094 ajoutés |
| R-11 | LOW | checklists/analysis | Comptes/états R1 devenus historiques ; revues R2 et analyse courante actualisées, historique daté conservé |

Les dépendances ouvertes sont des **décisions scientifiques explicites**, pas des exigences
oubliées. Aucun contrat métrique détaillé ne les remplace avant le go.

## Correspondance des exigences et critères

| Ancien identifiant | Identifiant actuel | Évolution / revalidation |
| --- | --- | --- |
| FR-001–008 | Identiques | Clauses préservées ; mapping/source line of flight complété par FR-047/048 |
| FR-009 | FR-009 | Formule conservée ; précision de portée planaire ajoutée, revalider modes/transformation dans V-11 |
| FR-010 | FR-010 | Exigence de section normale conservée pour 001 plan ; ne limite plus le produit final ni ne qualifie le conique |
| FR-011–021 | Identiques | Clauses/budgets préservés ; nouvelles références produit ne sont pas des résultats de solveur |
| FR-022 | FR-022 | Contrôles conservés ; volumes par unité d’extrusion explicitement plans, métriques coniques à dériver |
| FR-023–045 | Identiques | Clauses conservées ; modes/version/associations complétés par les nouvelles FR |
| Aucune | FR-046–060 | Produit/portée, cinq couches, refus, étude/décision, JG-C, restitution/références, champs, caches, validation future et traçabilité |
| SC-001–008 | Identiques | Critères antérieurs conservés, toujours non exécutés |
| Aucune | SC-009 | V-11 contrats/modes/représentations/associations/refus |
| Aucune | SC-010 | Revue G-FORM et corpus/dérivations, go ou no-go motivé |
| Aucune | SC-011 | JG-C conditionnel après go scientifique et J0 ; sinon bloqué |
| Aucune | SC-012 | Inventaire/correspondances et distinction 20 PR courant/futur |

Aucune exigence antérieure n’a été retirée ou différée hors de sa portée. Les 45 clauses
initiales restent intégralement présentes ; trois reçoivent une phrase de portée. Q1 et les
budgets H sont inchangés. Les activités d’équations restent futures depuis l’origine, désormais
visibles et étendues au 2.75D ; elles ne sont pas comptées comme FR réalisées par le mailleur.

## Correspondance exhaustive des anciennes tâches

Tous les IDs sont conservés. État initial de chaque ligne : **prévu, non exécuté, case ouverte**.
État actuel identique, explicité dans tasks.md. « Révisée » implique réexécuter l’ensemble des
critères R2 lors de la réalisation ; aucune preuve antérieure n’est présumée suffisante.
« Préservée » conserve résultat/dépendances/critères ; seul le statut a été rendu explicite.
Les exigences pertinentes et les fichiers exacts restent dans la tâche active ; le texte
ci-dessous indique précisément le changement, sans créer une deuxième liste active.

| Ancien ID | ID actif | Évolution | Changement / revalidation requise |
| --- | --- | --- | --- |
| T001 | T001 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T002 | T002 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T003 | T003 | révisée | Définir des types simples séparant source line of flight, LogicalTopology, PlanarPhysicalSection/View, transformations/bases et métriques euclidiennes ; aucun contrat métrique conique obligatoire. |
| T004 | T004 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T005 | T005 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T006 | T006 | révisée | Identifier explicitement planar-extrusion-v1 et S line of flight ; limiter cette transformation aux modes plans, préserver Y/η et les correspondances, sans transformation conique supposée. |
| T007 | T007 | révisée | Les tableaux de blocs/raccords constituent la topologie logique ; leurs courbes physiques sont celles de la représentation plane explicite, sans qualification conique. |
| T008 | T008 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T009 | T009 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T010 | T010 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T011 | T011 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T012 | T012 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T013 | T013 | révisée | Étiqueter ces mesures comme euclidiennes planes, avec association cellule/face ; ne pas en déduire un contrat ou une validité coniques. |
| T014 | T014 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T015 | T015 | révisée | Déclarer Float64 et représentation plane physique versus indices logiques, distinguer PointData/CellData et conserver associations/IDs ; aucun champ de pression fictif. |
| T016 | T016 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T017 | T017 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T018 | T018 | révisée | Le passage reste exclusivement plan et indépendant de G-FORM/JG-C ; ni étude conique ni qualification future ne conditionne ces neuf cas. |
| T019 | T019 | révisée | Le go ne porte que sur J0 plan ; conserver sa décision sans l’étendre au conique. Une future construction conique a son propre go scientifique et JG-C. |
| T020 | T020 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T021 | T021 | révisée | Préparer les cas schémas 1.1/lecture legacy 1.0, YAML unitaire équivalent, geometry_mode explicite, planar_2d avec Λ=0 et refus des champs de vol/effilement actifs. Ajouter representation.schema.json aux tests. |
| T022 | T022 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T023 | T023 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T024 | T024 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T025 | T025 | révisée | Introduire yaml-cpp 0.8.0 (source/SHA/notices) dans aero25d_document_io après J0 : lecteur JSON/YAML partagé, règles uniques et base des chemins. Valider geometry_mode (planar_2d ⇒ Λ=0), migration legacy archivée et refus UNSUPPORTED_GEOMETRY_MODE/UNSUPPORTED_FLOW_INPUT, sans ignorer de champ actif. |
| T026 | T026 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T027 | T027 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T028 | T028 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T029 | T029 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T030 | T030 | révisée | Exposer les correspondances en S original line of flight et N physique plan, sans calculer p/Cp ni confondre un mapping géométrique et une reconstruction de solution. |
| T031 | T031 | révisée | Exécuter aussi V-11 sur les configurations JSON/YAML, modèles 1.1/legacy, mappings et refus explicites ; aucune dépendance G-FORM. |
| T032 | T032 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T033 | T033 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T034 | T034 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T035 | T035 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T036 | T036 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T037 | T037 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T038 | T038 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T039 | T039 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T040 | T040 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T041 | T041 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T042 | T042 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T043 | T043 | révisée | Démontrer la séparation des vues LogicalTopology et PlanarPhysicalSection, le modèle/version de transformation et les associations, sans signature métrique conique anticipée. |
| T044 | T044 | révisée | Ajouter schémas de représentation 1.1, associations/tailles de champs, migration explicite des rôles legacy et refus de type physique non supporté. |
| T045 | T045 | révisée | Écrire représentation plane et fields/association/tuple_count dans le manifeste 1.1 ; conserver [N,2] FP64, topology_space logique et migration 1.0 tracée sans inférer de mode conique. |
| T046 | T046 | révisée | Décrire les tableaux de cellules/faces avec association, repère, unités et origine ; garder null motivé lorsque mode/représentation sont inconnus après échec. |
| T047 | T047 | révisée | Conserver le mapping de provenance line of flight pour chaque représentation et marquer les interpolations de visualisation ; pas de Cp/pression produit par ces exports. |
| T048 | T048 | révisée | Vérifier dtype Float64, tuples PointData/CellData selon association et étiquette plane_N/planar-euclidean-v1 ; ne pas présenter une vue logique comme surface physique. |
| T049 | T049 | révisée | Accepter les deux syntaxes de configuration avec le lecteur partagé et refuser explicitement mode conique/données de vol actives avant génération ; une extension inactive produit un diagnostic visible. |
| T050 | T050 | révisée | Comparer les modèles/versions de représentation et règles de migration ; aucun bundle legacy assimilé à un résultat conique ou réutilisé sans compatibilité. |
| T051 | T051 | révisée | Relayer syntaxe YAML, geometry_mode, refus actifs et diagnostic INACTIVE_EXTENSION sans supprimer de champ ni calculer une clé/restitution propre. |
| T052 | T052 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T053 | T053 | révisée | Comparer types Float64, nombre de tuples et associations PointData/CellData/face, représentations/bases et IDs. Ne jamais interpoler implicitement cellule→sommet pour faire passer la comparaison. |
| T054 | T054 | révisée | Inspecter aussi les associations et l’étiquette de section physique plane ; aucune vue physique conique n’est annoncée comme disponible. |
| T055 | T055 | révisée | Comparer JSON/YAML, modèles explicites, mapping S/N et refus coniques/vol actifs depuis CLI/Python ; inclure V-11 et les associations. |
| T056 | T056 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T057 | T057 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T058 | T058 | révisée | L’autonomie prouvée porte sur le mailleur plan, pas sur une capacité 2.75D ni un backend RANS/GPU livré. |
| T059 | T059 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T060 | T060 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T061 | T061 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T062 | T062 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T063 | T063 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T064 | T064 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T065 | T065 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T066 | T066 | révisée | Préparer V-11 et les mutations mode/transformation/version, champs de vol/effilement actifs refusés, extension inactive signalée ; aucun schéma métrique conique détaillé. |
| T067 | T067 | révisée | Réutiliser src/io/document_reader.cpp créé à T025, sans second parseur YAML ni dépendance Python ; valider le schéma de campagne et conserver la provenance des fichiers unitaires JSON/YAML. |
| T068 | T068 | révisée | Conserver et valider mode/représentation effectifs de chaque configuration ; ne pas supprimer les champs non supportés avant contrôle/développement. |
| T069 | T069 | révisée | Passer à effective-mesh-v2 : modèles/versions de représentation, transformations/bases et métriques planes effectives. À Λ=0 les deux modes plans ne partagent un maillage que si toute géométrie effective est identique ; conserver les modes demandés en provenance. Un mesh_key ne sert pas de cache de solution futur. |
| T070 | T070 | révisée | Conserver les refus actifs localisés et le diagnostic des archives futures inactives ; aucun champ Mach/Re/T/effilement silencieusement sans effet. |
| T071 | T071 | préservée | Critères/dépendances antérieurs conservés ; statut réel rendu explicite. |
| T072 | T072 | révisée | Muter séparément modèle, transformation, version de résolution et migration legacy ; aucune reprise incompatible. |
| T073 | T073 | révisée | Retourner l’enveloppe 1.1, codes unitaires spécifiques et diagnostic visible de non-exécution des extensions futures. |
| T074 | T074 | révisée | Parité des refus de modes/vol actifs et du diagnostic d’extension inactive ; aucun décodeur ou calcul de pression conique dans le wrapper. |
| T075 | T075 | révisée | Inclure lectures 1.0 explicitement migrées, représentation 1.1, modes plans à Λ=0, mutations de versions et refus des entrées coniques/vol actives. |
| T076 | T076 | révisée | Inclure la matrice de compatibilité R2 ; conserver la déduplication d’incidence descriptive, sans confondre clé de maillage et futur cas de vol/solution. |
| T077 | T077 | révisée | Contrôler associations et représentations des cas partagés, dtype FP64 et absence de champ aérodynamique/conique fictif. |
| T078 | T078 | révisée | Expliquer objectif produit 2D/2.5D/2.75D versus parcours de maillage plan disponibles, paramètres refusés et extension inactive ; les tuples de vol restent futurs. |
| T079 | T079 | révisée | Enregistrer les tests planaires V-11/representation/schema/parité, sans exiger la cible conique conditionnelle ; ses tests seront enregistrés dans T091. |
| T080 | T080 | révisée | Qualifier uniquement le lot A plan : FR-001–045 et exigences R2 planes applicables ; afficher séparément les exigences scientifiques FR-050–054/058 et leurs statuts bloqués/prévus, sans prétendre aux pressions ou au conique. |
| T081 | T081 | révisée | Les objectifs C++/CUDA/CPU FP64, 8/16 Go et coefficients sont référencés à la roadmap et ne sont pas présentés comme performances/capacités livrées. |
| T082 | T082 | révisée | Auditer 60 FR/12 SC et les 20 PR avec états courant/étude/futur ; conserver les correspondances R2, aucune exigence future déclarée vérifiée. |
| T083 | T083 | révisée | La décision accepte seulement le lot A **plan** ; publier séparément G-FORM/JG-C non exécutés/bloqués. La clôture globale de la fonctionnalité réconciliée attend T095. |

41 tâches ont une précision/révision sémantique R2 ; 42 gardent leurs critères antérieurs.
Aucune ancienne tâche n’est supprimée, remplacée par un autre ID ou cochée. Les travaux
supplémentaires sont :

| ID nouveau | Résultat | Dépendance de passage | Statut actuel |
| --- | --- | --- | --- |
| T084 | Corpus primaire audité | J0, T019 | Prévu, étude initiale seulement |
| T085 | Hypothèses/données géométriques indépendantes | T084 | Prévu |
| T086 | Fermeture transversale visqueuse/SA-neg | T084/T085 | Prévu, décision ouverte |
| T087 | Restitution, références/intégrales et validation | T085/T086 | Prévu |
| T088 | Go/no-go G-FORM | T084–T087 et revue scientifique | Ouvert, aucun go |
| T089 | Contrat métrique/géométrique et critères chiffrés | **T088 go**, J0 go, T043 | Bloqué ; contrat détaillé non créé |
| T090 | Oracles/cas coniques | T089 | Bloqué |
| T091 | Démonstrateur géométrique isolé | T089/T090, J0 go | Bloqué, non implémenté |
| T092 | Expérience fin/épais et limite c′→0 | T091 | Bloqué, non exécuté |
| T093 | JG-C géométrique | T092, G-FORM go, lecteur qualifié | Bloqué, non qualifié |
| T094 | Programme futur d’équations/campagnes | T088 go pour choix détaillés | Prévu, aucun code de solveur |
| T095 | Décisions des deux lots et preuves | T083/T088/T093/T094 | Prévu, pas d’acceptation globale anticipée |

T083 accepte seulement le lot A plan ; T095 ne pourra annoncer les deux lots satisfaits que
si leurs preuves existent. J0 n’attend aucune tâche nouvelle, ni la totalité CLI/Python/
campagnes/stockage/certification générale. Le sous-graphe conique dépend de J0 et de la décision,
jamais l’inverse. Les travaux futurs FUT-01–07 ont leurs propres spécifications à venir,
mais aucun autre tasks.md actif n’a été créé pendant cette réconciliation.

## Traçabilité des exigences produit

« Couverture » ci-dessous indique la portion de travail courante et l’étape future, pas
une capacité livrée. Toutes les étapes de résolution et de validation scientifique sont futures.

| PR | Portion de 001 / FR | Étape future requise | État produit |
| --- | --- | --- | --- |
| PR-001 | FR-014/030/046 | FUT-01/02/06 | Exigé, non livré |
| PR-002 | FR-048–053/059 | FUT-01/02/03 | Exigé, non livré |
| PR-003 | FR-001–013/047/054 | FUT-04 | Exigé, non livré |
| PR-004 | FR-005/009/015/046 | FUT-01–07, enveloppe qualifiée | Exigé, non livré |
| PR-005 | FR-019/020/050/051 | FUT-01/02/03 | Exigé, non livré |
| PR-006 | FR-050/051 | FUT-02/03 | Exigé, non livré |
| PR-007 | FR-055 | FUT-04 | Exigé, non livré |
| PR-008 | FR-054/055 | FUT-04 | Exigé, non livré |
| PR-009 | FR-047/054 | FUT-04 | Exigé, non livré |
| PR-010 | FR-006/047/055 | FUT-04, dérivation par mode | Exigé, non livré |
| PR-011 | FR-026/031–039/049 | FUT-05 | Exigé, non livré |
| PR-012 | FR-033–040/057 | FUT-05/06, cache de solution | Exigé, non livré |
| PR-013 | FR-026/038 | FUT-01/02/05, parcours de résolution | Exigé, non livré |
| PR-014 | FR-030/046 | FUT-01/06, GPU qualifié | Exigé, non livré |
| PR-015 | FR-025/046/058 | FUT-03/06 | Exigé, non livré |
| PR-016 | FR-027/028/041–044/056 | FUT-02/04, champs physiques | Exigé, non livré |
| PR-017 | FR-005/046 | FUT-07 | Exigé, non livré |
| PR-018 | FR-046/058 | Exclusion produit maintenue | Exigé, non livré |
| PR-019 | FR-050–053 | FUT-02/03 après G-FORM/JG-C | Exigé, non livré |
| PR-020 | FR-046/052/058–060 | Toutes étapes, états distincts | Exigé, non livré |

## Revue avant implémentation et premier groupe

Premier groupe techniquement prêt : **T001–T019**, critères J0 inchangés : neuf configurations,
V-03 fourni/complété, contrôles exhaustifs indépendants, budgets applicables et vues BA/lèvres.
Une revue de cette réconciliation et des contrats 1.1 est nécessaire avant leur implémentation ;
aucun lancement automatique. Le démarrage technique contrôlera les dépendances locales réelles.
La conformité du chemin plan ne réclame aucune acceptation scientifique préalable du conique.

Avant toute construction conique : revoir **G-FORM** avec compétences scientifiques adaptées,
notamment accès aux textes manquants, similitude visqueuse/SA-neg, section/bases/mesures,
restitution et intégrales. La métrique et ses seuils n’arrivent qu’après ce go. Les valeurs
exactes de la future enveloppe Reynolds/cas de vol, lois thermodynamiques/viscosité et budgets
GPU relèvent des futures étapes, sans chiffres ni garanties inventés dans cette révision.

La version 1.1.0 de constitution est une extension mineure de périmètre/gouvernance ; elle
ne relâche ni preuves ni tolérances. L’analyse courante est dans [tasks-analysis.md](tasks-analysis.md).

## Fichiers modifiés ou créés pendant R2

29 fichiers révisés et 5 nouveaux, tous documentaires ; deux fichiers Q1 conservés inchangés.

| Fichier | Action |
| --- | --- |
| `.specify/memory/constitution.md` | révisé |
| `docs/product-roadmap.md` | nouveau |
| `specs/001-import-maillage-monoelement/build-and-library.md` | révisé |
| `specs/001-import-maillage-monoelement/checklists/plan-review.md` | révisé |
| `specs/001-import-maillage-monoelement/checklists/requirements.md` | révisé |
| `specs/001-import-maillage-monoelement/coherence-review.md` | révisé |
| `specs/001-import-maillage-monoelement/conical-study.md` | nouveau |
| `specs/001-import-maillage-monoelement/contracts/README.md` | révisé |
| `specs/001-import-maillage-monoelement/contracts/bundle.md` | révisé |
| `specs/001-import-maillage-monoelement/contracts/campaign-state.schema.json` | révisé |
| `specs/001-import-maillage-monoelement/contracts/campaign.example.json` | révisé |
| `specs/001-import-maillage-monoelement/contracts/campaign.example.yaml` | révisé |
| `specs/001-import-maillage-monoelement/contracts/campaign.md` | révisé |
| `specs/001-import-maillage-monoelement/contracts/campaign.schema.json` | révisé |
| `specs/001-import-maillage-monoelement/contracts/cli-python.md` | révisé |
| `specs/001-import-maillage-monoelement/contracts/configuration.example.json` | révisé |
| `specs/001-import-maillage-monoelement/contracts/configuration.example.yaml` | nouveau |
| `specs/001-import-maillage-monoelement/contracts/configuration.schema.json` | révisé |
| `specs/001-import-maillage-monoelement/contracts/manifest.schema.json` | révisé |
| `specs/001-import-maillage-monoelement/contracts/report.schema.json` | révisé |
| `specs/001-import-maillage-monoelement/contracts/representation.schema.json` | nouveau |
| `specs/001-import-maillage-monoelement/contracts/topology.schema.json` | révisé |
| `specs/001-import-maillage-monoelement/data-model.md` | révisé |
| `specs/001-import-maillage-monoelement/geometry-verification.md` | révisé |
| `specs/001-import-maillage-monoelement/mesh-quality.md` | révisé |
| `specs/001-import-maillage-monoelement/plan.md` | révisé |
| `specs/001-import-maillage-monoelement/quickstart.md` | révisé |
| `specs/001-import-maillage-monoelement/reconciliation.md` | nouveau |
| `specs/001-import-maillage-monoelement/research.md` | révisé |
| `specs/001-import-maillage-monoelement/spec.md` | révisé |
| `specs/001-import-maillage-monoelement/tasks-analysis.md` | révisé |
| `specs/001-import-maillage-monoelement/tasks.md` | révisé |
| `specs/001-import-maillage-monoelement/topology.md` | révisé |
| `specs/001-import-maillage-monoelement/verification.md` | révisé |

## R3 — correction des cinq écarts

Date : 2026-10-31. Demande explicite de corrections **documentaires et contractuelles**
U1/I1/U2/U3/I2, suivies d'une nouvelle analyse Spec Kit en lecture seule. Aucun code produit
ni test d'acceptation n'existe/n'a été exécuté ; aucun go de J0 ou G-FORM n'est allégué.
Les outils d'audit rédactionnel sont restés dans /tmp. Constitution et fichiers Q1
conservés octet pour octet ; aucune évolution de principe ou de seuil qualité.

| Écart | Décision appliquée / correspondance des exigences existantes | Contrat autoritatif |
| --- | --- | --- |
| U1 | FR-020/023/024/029/033–040/057 conservées et précisées : point logique, entrées de génération, artefact généré, audits et estimations séparés ; politique qualité consommée par lissage/arrêt incluse dans mesh_key | contracts/result-identities.md, campaign.md, schémas campaign-state/artifact et report/manifest |
| I1 | FR-052/053/060 et SC-010/011 conservés : T095 clôt un bilan go ou no_go ; tâches T089–T094 non applicables après no_go, jamais réalisées artificiellement | tasks.md, plan.md, conical-study.md |
| U2 | FR-011/018/045 conservées : retirer curvature_regularization_over_c non consommé, avant T010 ; refus explicite toutes versions ; loi sqrt(c|κ|) justifiée par écart corde–arc et contrôlée indépendamment | geometry-verification.md §5/R3, configuration.schema.json |
| U3 | FR-006/007/019/020/029 et H-02 conservés : c_input dans S, mètres font autorité, indication cohérence seulement ; adimensionnel L/c_input ; tolérance relative1e-10/absolue0m et bornes archivées | contracts/physical-scale.md et physical-scale.schema.json |
| I2 | Aucun principe/exigence modifié : constitution 1.0.0 établie le 2026-10-30, 1.1.0 amendée le 2026-10-31 | plan.md / Constitution Check |

60 FR, 12 SC et T001–T095 conservés. Les compléments sont insérés sous les mêmes IDs,
sans seconde spécification. Contrats initiaux **non publiés** révisés 1.2 ; demandes 1.0/1.1
lisibles avec migrations explicites archivées, sans accepter le champ inutilisé supprimé ni
réutiliser aveuglément d'anciens verdicts. Nouvelles clés versionnées distinctes, sans cache
de solution. Les cinq nouvelles annexes sont des contrats de cette fonctionnalité.

### Correspondance R2 → R3 des tâches modifiées

Toutes ces tâches restent ouvertes/non exécutées ; leurs nouveaux critères devront être
vérifiés lors de leur future réalisation. Aucun ancien succès à transférer. Le tableau
R2 précédent conserve la correspondance antérieure ; aucune tâche n'est supprimée, remplacée
par un autre ID ou réaffectée à une nouvelle fonctionnalité.

| Ancienne tâche | Tâche courante | Critère précisé ou remplacé / revalidation nécessaire |
| --- | --- | --- |
| T010 | T010 | U2 : moniteur sans régularisation avant J0 ; échelle/sensibilité/coins/références ; preuves à produire |
| T021 | T021 | U2/U3/U1 : schémas 1.2, paramètre retiré, échelle et références typées ; preuves à produire |
| T025 | T025 | U3 : échelle S, tolérance/contradiction, archivage ; U2 : refus du champ retiré ; preuves à produire |
| T031 | T031 | Propagation des versions 1.2 dans l’acceptation import, mêmes critères ; preuves à produire |
| T034 | T034 | U2 : réutiliser moniteur J0 ; tests généraux de sensibilité/échelle et stabilité ; preuves à produire |
| T035 | T035 | U1 : politique consommée par lissage/arrêt dans projection de génération ; preuves à produire |
| T036 | T036 | U1 : audits multiples immuables et pass/fail coexistants ; preuves à produire |
| T037 | T037 | U3/U1 : conversion physique unique et estimations multiples séparées ; preuves à produire |
| T044 | T044 | U1/U3 : contrats de rapports/manifeste/références 1.2 ; preuves à produire |
| T045 | T045 | U1/U3 : versions de clés et instantanés distincts, échelle archivée ; preuves à produire |
| T046 | T046 | U1 : stockage des résultats distincts et matérialisation de bundle propre au point ; preuves à produire |
| T050 | T050 | U1/U2/U3 : --audit-profile, diagnostics d’échelle/champ retiré ; preuves à produire |
| T051 | T051 | U1/U3 : Python transmet paramètres/références sans calcul ; preuves à produire |
| T055 | T055 | U1/U2/U3 : parité CLI/Python, audits/estimations/échelle/champ retiré ; preuves à produire |
| T066 | T066 | U1/U3 : fixtures de contrats, partage, interruption et politique de génération ; preuves à produire |
| T068 | T068 | U3 : surcharge physique par profil, validation par point avant partage ; preuves à produire |
| T069 | T069 | U1 : versions/composition de toutes clés et classification des paramètres ; preuves à produire |
| T071 | T071 | U1 : remplace le bundle unique par génération/audits/estimations/points ; commits séparés ; preuves à produire |
| T072 | T072 | U1 : remplace la relance globale des échecs par reprise par couche, retry explicite et preuves immuables ; preuves à produire |
| T073 | T073 | U1 : CLI reprise/retry et références propres aux couches ; preuves à produire |
| T074 | T074 | U1/U3 : Python reprise/retry/échelle et résultats identiques ; preuves à produire |
| T075 | T075 | Propagation des versions et maintien de l’inventaire sans génération ; preuves à produire |
| T076 | T076 | U1/U3 : acceptation des six cas de partage/reprise/parité et échelles ; preuves à produire |
| T077 | T077 | U1 : export par contexte de point, même génération, pas de bundle/verdict universel ; preuves à produire |
| T088 | T088 | I1 : décision terminale go/no_go, raisons/preuves et non-applicabilité ; preuves à produire |
| T089 | T089 | I1 : reste conditionnelle go ; non applicable après no_go, jamais cochée réalisée ; preuves à produire |
| T090 | T090 | I1 : reste conditionnelle go ; non applicable après no_go, jamais cochée réalisée ; preuves à produire |
| T091 | T091 | I1 : reste conditionnelle go ; non applicable après no_go, jamais cochée réalisée ; preuves à produire |
| T092 | T092 | I1 : reste conditionnelle go ; non applicable après no_go, jamais cochée réalisée ; preuves à produire |
| T093 | T093 | I1 : JG-C seulement après go ; non applicable après no_go ; preuves à produire |
| T094 | T094 | I1 : programme détaillé seulement après go ; non applicable après no_go ; preuves à produire |
| T095 | T095 | I1 : dépendances alternatives explicites et bilan no_go atteignable sans T089–T094 ; preuves à produire |

Les 63 autres tâches sont préservées. Dans la branche no_go future, T089–T094 seront
annotées non applicables **à cette décision**, cases ouvertes ; actuellement G-FORM est
ouverte et elles restent bloquées. T095 : communs T083/T088 terminaux ; go ajoute résultats
T093/T094, no_go n'attend aucune tâche conditionnelle. Les deux graphes sont acycliques.
Une clôture study_completed_no_go est un bilan scientifique, pas une géométrie qualifiée.

### Vérifications documentaires exécutées avant analyse finale

- 10 schémas Draft 2020-12 valides, registre URN local sans réseau ; 29 cas synthétiques
  de forme positifs/négatifs (exemples, versions, champ supprimé, références/couches,
  verdicts, tolérance fixe, absence d'échelle), tous conformes aux attendus.
- Deux paires JSON/YAML équivalentes ; jsonschema 4.10.3 et PyYAML 6.0.1 utilisés pour
  l'audit seulement, sans ajout de dépendance produit.
- 72/72 FR/SC rattachés aux 95 tâches ; aucune case produit cochée ; dépendances
  acycliques en branches go/no_go, T095 atteignable dans chacune si ses préalables passent.
- T001–T019 restent fermées sur leur propre socle, sans dépendance tardive à Python,
  campagnes, stockage complet ou G-FORM. Liens locaux contrôlés.
- Constitution/Q1 inchangés par rapport au début R3 ; aucun fichier src/include/tests/
  python/evidence/CMakeLists.txt produit présent. Ces audits ne valident pas l'implémentation
  des clés, mesures, reprise, CLI/Python, VTK/ParaView ni la formulation conique.

L'invocation finale speckit-analyze utilise ensuite le mécanisme installé et rend son rapport
**sans écriture**. tasks-analysis.md est explicitement l'historique R2 ; aucun nouveau bilan
scientifique ni résultat logiciel n'est fabriqué pour supprimer les avertissements.

### Fichiers R3

35 fichiers documentaires/contractuels, dont 5 nouveaux.

| Fichier (relatif à specs/001-import-maillage-monoelement/) | Action |
| --- | --- |
| [checklists/plan-review.md](checklists/plan-review.md) | révisé |
| [checklists/requirements.md](checklists/requirements.md) | révisé |
| [coherence-review.md](coherence-review.md) | révisé |
| [conical-study.md](conical-study.md) | révisé |
| [contracts/README.md](contracts/README.md) | révisé |
| [contracts/bundle.md](contracts/bundle.md) | révisé |
| [contracts/campaign-artifact.schema.json](contracts/campaign-artifact.schema.json) | nouveau |
| [contracts/campaign-state.schema.json](contracts/campaign-state.schema.json) | révisé |
| [contracts/campaign.example.json](contracts/campaign.example.json) | révisé |
| [contracts/campaign.example.yaml](contracts/campaign.example.yaml) | révisé |
| [contracts/campaign.md](contracts/campaign.md) | révisé |
| [contracts/campaign.schema.json](contracts/campaign.schema.json) | révisé |
| [contracts/cli-python.md](contracts/cli-python.md) | révisé |
| [contracts/configuration.example.json](contracts/configuration.example.json) | révisé |
| [contracts/configuration.example.yaml](contracts/configuration.example.yaml) | révisé |
| [contracts/configuration.schema.json](contracts/configuration.schema.json) | révisé |
| [contracts/manifest.schema.json](contracts/manifest.schema.json) | révisé |
| [contracts/physical-scale.md](contracts/physical-scale.md) | nouveau |
| [contracts/physical-scale.schema.json](contracts/physical-scale.schema.json) | nouveau |
| [contracts/report.schema.json](contracts/report.schema.json) | révisé |
| [contracts/representation.schema.json](contracts/representation.schema.json) | révisé |
| [contracts/result-identities.md](contracts/result-identities.md) | nouveau |
| [contracts/scale-audit-cases.example.json](contracts/scale-audit-cases.example.json) | nouveau |
| [contracts/topology.schema.json](contracts/topology.schema.json) | révisé |
| [data-model.md](data-model.md) | révisé |
| [geometry-verification.md](geometry-verification.md) | révisé |
| [mesh-quality.md](mesh-quality.md) | révisé |
| [plan.md](plan.md) | révisé |
| [quickstart.md](quickstart.md) | révisé |
| [reconciliation.md](reconciliation.md) | révisé |
| [research.md](research.md) | révisé |
| [spec.md](spec.md) | révisé |
| [tasks-analysis.md](tasks-analysis.md) | révisé |
| [tasks.md](tasks.md) | révisé |
| [verification.md](verification.md) | révisé |

### Revue et premier groupe inchangés

Premier groupe prêt à exécuter après revue documentaire : **T001–T019** ; aucun démarrage
automatique. Passage : neuf V-01/02/03 × 0°/25°/50°, variantes V-03 fourni/complété,
positivité/intégrité/raccords/Q1 applicables, mesures indépendantes et vues BA/lèvres.
U2 ajoute les essais du moniteur à T010 dans ce socle, sans dépendance retardée.
Sur échec conserver preuves, revoir construction et rejouer, sans généralisation ni critères
relâchés. Revue préalable : séparation de clés/persistance et reprises, suppression du
paramètre/migration, échelle/tolérance et branches du bilan. G-FORM et faisabilité des cartes
planes restent des décisions/expériences prévues, jamais des capacités réalisées par R3.
