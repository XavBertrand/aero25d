# Specification Analysis Report — réconciliation R2

**Date** : 2026-10-31. **Mécanisme** : speckit-analyze installé (Spec Kit 0.15.1), après
révision en place par specify/plan/tasks et amendement séparé de constitution. Le script
check-prerequisites --json --require-tasks --include-tasks a été exécuté une fois pour cette
analyse et a résolu la fonctionnalité 001 et tasks.md existants. Aucune autre spécification
ni liste active n’est créée. Les corrections documentaires sont explicitement autorisées
par l’utilisateur ; aucune réalisation de produit n’est autorisée par ce rapport.

## Résultat

Les contradictions de cadrage sont corrigées dans les documents. Les 60 FR et 12 SC ont des
tâches avec critères/preuves ; les 20 PR du produit ont une affectation courant/étude/futur,
sans prétendre les réaliser dans 001. **Aucun test de produit, maillage ni résultat RANS
n’est présent ou exécuté**. La décision de formulation G-FORM reste ouverte et bloque les
seules tâches de contrat métrique/construction coniques T089–T093 ; elle ne bloque pas J0.

## Constats et état après corrections

| ID | Catégorie | Gravité initiale | Localisation | Résumé | Correction / état |
| --- | --- | --- | --- | --- | --- |
| C1 | Cohérence de portée | HIGH | spec:objectif produit, constitution, roadmap | Produit global limité implicitement au 2.5D | Roadmap PR-001–020, constitution 1.1.0 et références explicites 2D/2.5D/2.75D ; résolu |
| C2 | Séparation de livraison | HIGH | spec:US6/FR-046–060, plan:lots | Risque d’élargir 001 au solveur RANS complet | Lot A plan, lot B étude/géométrie conditionnelle, FUT-01–07 pour équations/solution ; résolu |
| C3 | Données / terminologie | HIGH | data-model, representation.schema, bundle | Topologie, physique, bases et métriques confondables | Cinq couches séparées ; chemin plan [N,2] FP64 simple, contrat métrique conique non créé ; résolu |
| C4 | Sous-spécification scientifique | HIGH | conical-study, T084–T093 | Pas de décision sur similitude, viscosité/SA-neg et restitution | Sources primaires annotées, limites d’accès/preuve, G-FORM ouverte et dépendances go explicites ; cadrage résolu, décision scientifique non prise |
| C5 | Références de sortie | HIGH | roadmap:coefficients, conical-study:restitution | Cp original et champ p, Y/η et mesure d’efforts ambigus | q∞ total/c₀/axes vent/quart de corde/Y et unité Y fixés ; intégrales/mapping coniques à dériver après étude ; références résolues |
| I1 | Contrats | MEDIUM | configuration.schema, README, CLI/Python, T021/T025/T067 | JSON unitaire seul, modes non discriminés | JSON/YAML partagé, modes plans explicites 1.1, refus du conique/vol actif, migration 1.0 tracée ; résolu |
| I2 | Champs et exports | MEDIUM | manifest.schema, bundle, T045/T048/T053 | Association et représentation physique insuffisamment explicites | Descripteurs point/cell/face, tailles/origines, Float64 et vue plane explicite, contrôle indépendant ; résolu |
| I3 | Campagnes/reprise | MEDIUM | campaign, T069/T072/T075 | Modèles/versions et futur cache de solution trop implicites | effective-mesh-v2, entrées effectives, déduplication incidence, extensions inactives signalées et solution future séparée ; résolu |
| I4 | Ordonnancement | HIGH | tasks:J0/G-FORM/JG-C | Aucun jalon conique ni dépendance scientifique explicite | J0 T001–T019 inchangé dans sa portée ; T088 go avant T089–T093, jamais l’inverse ; résolu |
| I5 | Programme de preuve | MEDIUM | verification:FUT, T094 | Uniforme cartésien, limite 2.5D et séparation numérique/modèle non affectés | FUT-01–07 et programme futur T094 ; aucun test d’équations présenté comme réalisé ; résolu |
| T1 | Traçabilité / statut | MEDIUM | reconciliation, tasks:statuts, T083/T095 | Pas de correspondance après élargissement des exigences | Tous T001–T083 conservés, 41 révisés/42 préservés, T084–T095 ajoutés, toutes cases ouvertes ; décisions de lots distinctes ; résolu |

Aucune incohérence documentaire CRITICAL restante détectée. Les sujets scientifiques suivants
ne sont pas reclassés en réussite : ils sont ouvertement bloquants pour leur sous-graphe.

## Décisions encore ouvertes et revues nécessaires

| ID | Décision / risque | Travaux | Effet sur le démarrage |
| --- | --- | --- | --- |
| O1 | Faisabilité globale des cartes planes 4/5 blocs et paramètres/transition compatibles Q1 | T016–T019, E1 ; éventuelle révision puis nouveau J0 | À démontrer par le premier groupe, pas présumée |
| O2 | Similitude conique, section physique, coordonnées/bases, données/pentes et restrictions | T084/T085/T088 | Bloque T089–T093 tant que G-FORM n’a pas de go |
| O3 | Dérivées transversales, contraintes/énergie, viscosité/Reynolds local, distance paroi et tous termes SA-neg | T086/T088 ; textes primaires manquants à acquérir/revoir | Aucune preuve Euler ne ferme cette décision RANS |
| O4 | Restitution Cp original/champ p, intégrales par unité Y et limite vers l’extrusion | T087/T088, puis critères T089 | Références de sortie fixées ; calcul conique toujours non dérivé/implémenté |
| O5 | Métriques/bornes et programme de vérification du conique | T089 après go, T090–T094 | Contrat détaillé volontairement différé ; aucune singularité tolérée par défaut |
| O6 | Environnement réel et accès aux références | T001/T002, T052/T054, T084 | Vérifications locales/source à leur phase ; lecteur/ParaView requis pour qualification, pas pour J0 |

Avant implémentation : revoir la réconciliation et contrats 1.1/migration/refus/associations.
Avant construction conique : revue scientifique identifiée G-FORM, pas un simple accord sur
un champ taper_ratio. Les lois thermodynamiques/viscosité des futurs cas de vol, enveloppe
Reynolds exacte et budgets CUDA seront fixés dans leurs futurs travaux, sans inventions ici.

## Couverture FR et SC

« Oui » signifie **planifié avec tâche**, jamais vérifié. Les tâches listées privilégient les
travaux spécifiques plutôt que T080/T082/T095 qui auditent l’ensemble. La matrice contractuelle
complète et ses preuves attendues sont dans spec.md et verification.md (60 lignes chacune).

| Exigence | Tâche ? | Tâches spécifiques prévues | État réel |
| --- | --- | --- | --- |
| FR-001 | Oui | T020, T022, T031 | Prévu, non exécuté |
| FR-002 | Oui | T020, T022, T031 | Prévu, non exécuté |
| FR-003 | Oui | T005, T012, T020, T023, T027, T031 | Prévu, non exécuté |
| FR-004 | Oui | T020, T022, T023, T030, T031, T040, T045, T049 | Prévu, non exécuté |
| FR-005 | Oui | T020, T022, T023, T030, T031, T033 | Prévu, non exécuté |
| FR-006 | Oui | T003, T004, T006, T018, T021, T024, T025, T031, T046 | Prévu, non exécuté |
| FR-007 | Oui | T005, T006, T022, T025, T031, T045, T050 | Prévu, non exécuté |
| FR-008 | Oui | T004, T020, T021, T024, T030, T031 | Prévu, non exécuté |
| FR-009 | Oui | T005, T006, T018, T025, T031 | Prévu, non exécuté |
| FR-010 | Oui | T003, T006, T025, T031 | Prévu, non exécuté |
| FR-011 | Oui | T004, T010, T014, T026, …, T033, T034, T041, T060 | Prévu, non exécuté |
| FR-012 | Oui | T014, T027, T028, T029, T031, T034, T036, T041, T046 | Prévu, non exécuté |
| FR-013 | Oui | T004, T009, T014, T017, …, T031, T033, T038, T041 | Prévu, non exécuté |
| FR-014 | Oui | T003, T007, T008, T011, …, T035, T041, T043, T048 | Prévu, non exécuté |
| FR-015 | Oui | T008, T009, T011, T016, T018, T033, T041 | Prévu, non exécuté |
| FR-016 | Oui | T003, T007, T008, T009, …, T041, T043, T048, T053 | Prévu, non exécuté |
| FR-017 | Oui | T007, T008, T009, T011, T033, T034, T041 | Prévu, non exécuté |
| FR-018 | Oui | T010, T011, T016, T017, …, T060, T061, T063, T064 | Prévu, non exécuté |
| FR-019 | Oui | T010, T013, T018, T032, …, T041, T042, T046, T064 | Prévu, non exécuté |
| FR-020 | Oui | T037, T043, T046, T055 | Prévu, non exécuté |
| FR-021 | Oui | T010, T017, T034, T059, T061, T062, T063, T064, T065 | Prévu, non exécuté |
| FR-022 | Oui | T005, T007, T012, T013, …, T041, T042, T064, T079 | Prévu, non exécuté |
| FR-023 | Oui | T013, T016, T018, T032, …, T039, T041, T042, T064 | Prévu, non exécuté |
| FR-024 | Oui | T003, T012, T013, T021, …, T046, T049, T051, T055 | Prévu, non exécuté |
| FR-025 | Oui | T059, T060, T061, T062, T063, T064, T065 | Prévu, non exécuté |
| FR-026 | Oui | T021, T030, T040, T044, …, T065, T073, T074, T079 | Prévu, non exécuté |
| FR-027 | Oui | T015, T044, T047, T048, T053, T055, T056, T058, T064 | Prévu, non exécuté |
| FR-028 | Oui | T015, T030, T044, T046, …, T054, T055, T057, T058 | Prévu, non exécuté |
| FR-029 | Oui | T002, T039, T044, T045, …, T062, T065, T079, T081 | Prévu, non exécuté |
| FR-030 | Oui | T001, T002, T025, T037, …, T067, T077, T079, T081 | Prévu, non exécuté |
| FR-031 | Oui | T066, T067, T068, T070, T073, T075, T078 | Prévu, non exécuté |
| FR-032 | Oui | T066, T068, T070, T073, T074, T075, T078 | Prévu, non exécuté |
| FR-033 | Oui | T071, T072, T073, T076, T078 | Prévu, non exécuté |
| FR-034 | Oui | T066, T069, T072, T073, T074, T076, T078 | Prévu, non exécuté |
| FR-035 | Oui | T066, T071, T073, T076, T078 | Prévu, non exécuté |
| FR-036 | Oui | T066, T068, T069, T070, …, T075, T076, T077, T078 | Prévu, non exécuté |
| FR-037 | Oui | T066, T067, T068, T070, T072, T073, T075, T077, T078 | Prévu, non exécuté |
| FR-038 | Oui | T074, T075, T076, T077, T078 | Prévu, non exécuté |
| FR-039 | Oui | T066, T067, T069, T070, T073, T075, T078, T081 | Prévu, non exécuté |
| FR-040 | Oui | T066, T068, T069, T070, …, T075, T076, T077, T078 | Prévu, non exécuté |
| FR-041 | Oui | T015, T030, T044, T047, T048, T053, T054, T055, T077 | Prévu, non exécuté |
| FR-042 | Oui | T015, T030, T044, T047, T048, T053, T054, T055, T077 | Prévu, non exécuté |
| FR-043 | Oui | T052, T053, T055, T058, T077, T079, T081 | Prévu, non exécuté |
| FR-044 | Oui | T054, T055, T058 | Prévu, non exécuté |
| FR-045 | Oui | T001, T002, T004, T005, …, T017, T018, T019, T083 | Prévu, non exécuté |
| FR-046 | Oui | T058, T078, T081, T083, T084, T094 | Prévu, non exécuté |
| FR-047 | Oui | T003, T006, T015, T025, …, T090, T091, T092, T093 | Prévu, non exécuté |
| FR-048 | Oui | T003, T006, T007, T013, …, T085, T089, T091, T093 | Prévu, non exécuté |
| FR-049 | Oui | T021, T025, T031, T044, …, T075, T076, T078, T079 | Prévu, non exécuté |
| FR-050 | Oui | T084, T086, T088 | Prévu, non exécuté |
| FR-051 | Oui | T085, T086, T087, T088, T089, T090, T092, T094 | Prévu, non exécuté |
| FR-052 | Oui | T083, T085, T086, T088, T089, T090, T091, T093 | Conditionnel / G-FORM ouvert |
| FR-053 | Oui | T089, T090, T091, T092, T093 | Conditionnel / G-FORM ouvert |
| FR-054 | Oui | T030, T047, T087, T088, T090, T092, T093, T094 | Prévu, non exécuté |
| FR-055 | Oui | T006, T087, T088, T094 | Prévu, non exécuté |
| FR-056 | Oui | T013, T015, T021, T043, …, T089, T091, T092, T093 | Prévu, non exécuté |
| FR-057 | Oui | T021, T025, T031, T044, …, T076, T077, T078, T094 | Prévu, non exécuté |
| FR-058 | Oui | T078, T081, T086, T087, T088, T094 | Prévu, non exécuté |
| FR-059 | Oui | T003, T006, T007, T013, …, T058, T079, T083, T093 | Prévu, non exécuté |
| FR-060 | Oui | T019, T083, T084 | Prévu, non exécuté |
| SC-001 | Oui | T020, T031, T083 | Prévu, non exécuté |
| SC-002 | Oui | T018, T041, T043, T083 | Prévu, non exécuté |
| SC-003 | Oui | T020, T024, T031, T083 | Prévu, non exécuté |
| SC-004 | Oui | T059, T064, T065, T083 | Prévu, non exécuté |
| SC-005 | Oui | T055, T058, T064, T065, T083 | Prévu, non exécuté |
| SC-006 | Oui | T032, T042, T043, T055, T083 | Prévu, non exécuté |
| SC-007 | Oui | T066, T075, T076, T077, T078, T083 | Prévu, non exécuté |
| SC-008 | Oui | T053, T054, T055, T058, T077, T083 | Prévu, non exécuté |
| SC-009 | Oui | T031, T053, T055, T075, T083 | Prévu, non exécuté |
| SC-010 | Oui | T084, T088 | Prévu, non exécuté |
| SC-011 | Oui | T089, T090, T093 | Conditionnel / G-FORM ouvert |
| SC-012 | Oui | T083 | Prévu, non exécuté |

Les 20 PR ne sont pas des exigences toutes livrées par le mailleur : leur matrice dans
reconciliation.md indique les portions courantes et les futures étapes FUT. La traçabilité
est 20/20, sans taux de réalisation produit revendiqué.

## Constitution et tâches non rattachées

Constitution 1.1.0 : principes I–IX conservés, principe X ajouté. Modification mineure autorisée
par la réconciliation, avant cette analyse ; aucune dilution de seuil ou principe pour accepter
un cas. I/II : source/provenance, conventions et cinq couches ; III/IV/V : statuts/preuves et
sources limitées explicitement ; VI : H/Q1 inchangés et no-go bloquants ; VII/VIII : produit
autonome, noyau unique CLI/Python et lecteur de qualification seulement en tests ; IX/X :
CPU plan sans infrastructure RANS, géométrie conique après décision et J0 indépendant.
Aucune violation constitutionnelle détectée. Aucune tâche sans FR/SC ; aucune exigence
antérieure disparue. Les recouvrements entre FR de représentation et d’export sont des
spécialisations traçables, pas des exigences concurrentes à supprimer.

## Contrôles effectivement exécutés

- Spec Kit 0.15.1 et intégration Codex identifiés ; modèle actif résolu/lu ; setup-plan
  conserve le plan, setup-tasks résout les chemins, check-prerequisites retrouve tâches.
- Inventaire initial de 31 artefacts avec tailles/SHA-256, instructions et scripts inspectés ;
  aucune source/binaire/fixture/preuve de produit. feature.json inchangé, une seule fonctionnalité.
- 60 FR/12 SC uniques, matrices spec/verification 60/60 ; les 45 clauses antérieures restent
  présentes, seules FR-009/010/022 sont complétées pour leur portée plane. Q1 est byte-identique.
- 95 tâches au format checklist, IDs et labels de phase conformes, statut/vérification/preuve
  pour chacune ; 83 IDs préservés, aucune case cochée, aucune dépendance cyclique/future.
- Analyse du graphe : ancêtres de T019 limités à T001–T018 ; T089–T093 ont T088 en ancêtre,
  et T089 exige les go G-FORM/J0. Aucun contrat conique détaillé créé avant la décision.
- Huit schémas passent Draft 2020-12 et références URN/locales résolues sans réseau ; trois
  exemples JSON et deux exemples legacy conformes, deux paires JSON/YAML égales, descripteur
  plan et deux modes à flèche nulle conformes ; 16 mutations négatives refusées par schémas.
  Validateur jsonschema 4.10.3, PyYAML 6.0.1 pour l’audit documentaire seulement.
- Liens Markdown locaux contrôlés, anciens/nouveaux fichiers et exigences/tâches mappés.
- Aucun extensions.yml : aucun hook before/after constitution/specify/plan/tasks/analyze à lancer.

Ces tests portent exclusivement sur les **documents et formes de contrats**. Les codes CLI,
parité Python, interprétation YAML par le futur C++, intégrité des bundles, maillages et VTK/
ParaView ne sont pas exécutés. Leurs contrôles sont des tâches, pas des preuves produites.
La lecture documentaire de sources primaires ne constitue pas une validation de formulation.

## Comptages, tests indépendants et prochaines actions

| Groupe | Tâches | Critère indépendant futur |
| --- | ---: | --- |
| Socle/J0 | 19 | Neuf configurations, variantes V-03, Q1 applicable, contrôles et vues BA/lèvres ; go T019 |
| US1 | 12 | T031, corpus import/fidélité et contrats/modes V-11, sans génération |
| US2 | 12 | T041–T043, neuf maillages usuels complets et défauts V-08 |
| US3 | 15 | T053–T055/T058, exports et associations relus, ParaView, parité/reproduction CPU |
| US4 | 7 | T064/T065, F1 à trois niveaux, domaine et règles communes |
| US5 | 13 | T075–T078, campagne JSON/YAML, séquence/reprise/échecs et clés effectives |
| US6 | 11 | G-FORM T088 puis JG-C T093 conditionnel, sans résolution |
| Transversal | 6 | Acceptation plane T083, puis revue séparée des deux lots T095 |
| **Total** | **95** | **Aucune tâche produit exécutée** |

51 IMP, 13 ESS, 6 EXP, 25 ACC. Neuf tâches [P] préservées ; exemples par parcours et
conditions de disponibilité des fichiers dans tasks.md. Le premier incrément est J0,
le MVP natif plan US1/US2 reste distinct des parcours complets et de la préparation conique.

Métriques : 60 FR + 12 SC = **72 exigences/critères**, couverture documentaire 72/72,
95 tâches rattachées, 20/20 PR affectées à courant/futur. Ambiguïtés de portée non traitées :
0 ; duplications concurrentes : 0 ; incohérences documentaires CRITICAL restantes : 0.
Les décisions scientifiques O2–O5 sont **ouvertes et bloquantes pour le conique**.

Premier groupe prêt après revue : **T001–T019 uniquement**, avec les critères du passage J0
dans tasks.md. Une future demande explicite peut autoriser ce groupe. Aucune commande
speckit-implement ni workflow complet n’a été exécuté ; aucune implémentation n’est lancée.
