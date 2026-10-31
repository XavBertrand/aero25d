# Specification Quality Checklist: Import géométrique et maillage monoélément

**Purpose**: Vérifier la complétude et la qualité de la spécification avant planification.
**Created**: 2026-10-30
**Feature**: [spec.md](../spec.md)

## Content Quality

- [x] CHK001 Aucun choix d'implémentation ajouté : algorithmes, architecture logicielle et contrats détaillés restent au plan.
- [x] CHK002 Valeur utilisateur explicite : contrôler, mailler, inspecter et reproduire sans résolution.
- [x] CHK003 Parcours lisibles par les parties prenantes ; les notions scientifiques nécessaires sont accompagnées de conventions.
- [x] CHK004 Toutes les sections obligatoires du modèle actif sont renseignées et dans leur ordre d'origine.

## Requirement Completeness

- [x] CHK005 Aucun marqueur de clarification non résolu.
- [x] CHK006 Exigences identifiées FR-001 à FR-060 (45 antérieures et 30 initiales conservées), testables par leur matrice d'acceptation.
- [x] CHK007 Résultats SC-001 à SC-012 mesurables : couverture, comptes, statuts, budgets et livrables.
- [x] CHK008 Critères de succès indépendants de la technologie d'implémentation.
- [x] CHK009 Scénarios d'acceptation définis pour les six parcours, y compris refus et reprise après indication.
- [x] CHK010 Cas limites lexicaux, géométriques, topologiques, numériques et de ressources recensés.
- [x] CHK011 Périmètre monoélément borné ; import multiélément autorisé et génération multiélément refusée explicitement.
- [x] CHK012 Hypothèses H-01 à H-09, dépendances et décisions D-01 à D-10 distinguées, avec G-FORM scientifique ouvert des exigences fermes.

## Feature Readiness

- [x] CHK013 Chaque FR dispose d'une méthode, d'un critère d'acceptation et de preuves à conserver dans la matrice.
- [x] CHK014 Les scénarios couvrent les parcours principaux depuis commande et Python, sans résoudre d'écoulement.
- [x] CHK015 Les résultats mesurables sont reliés à les cas V-01 à V-11 et V-C01/V-C03 conditionnels ; la vérification future peut établir succès ou échec.
- [x] CHK016 Aucun détail d'implémentation nouveau n'est imposé ; seules les contraintes techniques déjà demandées sont conservées comme contexte.

## Notes

Validation documentaire initiale du 2026-10-30, révisée R1 puis R2 le 2026-10-31 ; 16/16 points documentaires satisfaits. Cette checklist qualifie la spécification, pas un logiciel, un maillage déjà produit ou une précision aérodynamique. Aucun test numérique ni scénario d'acceptation logiciel n'a été exécuté à cette étape.

- Traçabilité documentaire contrôlée : 60 exigences et 60 lignes correspondantes dans « Matrice de vérification et preuves » ; critères de succès et campagne distincts.
- Fidélité et tolérances : « Les valeurs et conventions suivantes sont des choix de projet proposés, pas des normes scientifiques. » Les budgets H-07/H-08 sont chiffrés, dimensionnés par la corde originale et justifiés par leur objet. H-09 et D-03 exigent de fixer et justifier les limites opérationnelles de qualité au plan avant réalisation ; leur absence de valeurs définitives n'autorise aucun verdict numérique favorable.
- Contexte technique conservé à la demande de l'utilisateur : « noyau C++/CUDA, backend CPU de référence, usage courant Python » et cibles GPU figurent uniquement dans le contexte produit. Commande/Python, VTK et multibloc sont des contraintes de livrables explicites, pas des choix nouveaux de mise en œuvre.
- Limite scientifique explicite : « Ils ne prouvent ni convergence d'un futur solveur ni incertitude de 10⁻⁴ sur Cx. »
- Gouvernance : point initial traité le 2026-10-30 ; constitution 1.0.0 établie sur autorisation de la demande courante, sans approbation humaine antérieure alléguée. Aucun blocage fonctionnel pour J0 plan ; G-FORM ouvert bloque explicitement la construction conique.
- Aucun hook avant/après spécification : `.specify/extensions.yml` absent lors des contrôles. Lors de la seule rédaction initiale, aucun plan ni contrat/tâche n’avait encore été produit. Le plan et ses annexes ont suivi le 2026-10-30 ; les tâches sont produites sur autorisation du 2026-10-31. Aucun code produit.

Les éventuels points non cochés exigeraient une mise à jour avant la phase suivante. Les points cochés ne valent pas validation des hypothèses par l'utilisateur ni exécution des essais futurs.

Revue de cohérence du 2026-10-30 avant plan : H-03/H-05/H-07/H-08 et lignes FR-007/011/012 précisés, sans retrait d'exigence ni de cas. Checklist toujours documentaire ; les documents de conception produits ensuite relèvent du plan, séparé de la spécification.

Lors de la planification autorisée suivante, SC-001 a été précisé pour distinguer le refus à
l'étape appropriée des cas V-04 importables mais non certifiables/non maillables. I-01 à I-24,
FR-001 à FR-030 et V-01 à V-09 restent couverts, sans relaxation des budgets. Voir
[la revue du plan](plan-review.md) et [les corrections](../coherence-review.md).

Revue du 2026-10-31 : US5, FR-031–045, SC-007/008, V-10 et les contrats campagne/VTP
sont ajoutés ; les 30 FR originales restent textuellement conservées. La relecture de
qualification et ParaView sont obligatoires dans les futures preuves, sans dépendance runtime.
La conformité de cette checklist reste documentaire et ne coche aucune FR comme vérifiée.

## Revue R2 — portée et portes scientifiques

- [x] Le produit final 2D/2.5D/2.75D et ses 20 PR sont dans une roadmap d’autorité unique, référencée par la fonctionnalité ; aucun VLM/solveur d’aile requis.
- [x] 001 reste le mailleur plan et la préparation scientifique/géométrique conique conditionnelle, sans équations RANS ni pression fabriquée.
- [x] Les cinq couches source/logique/physique/bases/métriques sont distinguées, avec chemin plan simple et métriques coniques non figées avant G-FORM.
- [x] Les formulations transformée/Euler/RANS ont des sources primaires et limites d’accès/preuve explicites ; l’étude initiale ne prétend pas fermer la décision.
- [x] Références de coefficients et distinction Cp pariétal/champ p sur plan physique documentées ; intégrales coniques encore à dériver.
- [x] 45 FR/83 tâches antérieures ont une correspondance explicite ; tâches toujours ouvertes, statuts réels et revalidation notés.

Ces cases attestent uniquement le contrôle documentaire de la réconciliation. SC-010/011 et
les essais G-FORM/JG-C restent non exécutés ; aucun go scientifique n’est donné par cette checklist.
