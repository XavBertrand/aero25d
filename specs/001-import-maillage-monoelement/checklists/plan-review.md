# Revue documentaire du plan

**Date** : 2026-10-30. **Plan** : [plan.md](../plan.md).
Cette revue initiale qualifie les documents au 2026-10-30 ; le complément du 2026-10-31 figure ci-dessous. Les expériences restent à réaliser.

- [x] Constitution 1.0.0 établie via le mécanisme demandé sur autorisation courante, sans antériorité d'approbation inventée.
- [x] Spécification et checklist lues ; ordre inverse S/N/W et distinction Y/η corrigés ; références transformées explicites.
- [x] FR-001 à FR-030 et V-01 à V-09 conservés ; matrice complète vers tests et preuves.
- [x] Phase 0 recherche et décisions D-01 à D-07 documentées, sans choix fonctionnel laissé implicite.
- [x] Phase 1 modèle, contrats JSON/CLI/Python/bundle et guide de vérification produits.
- [x] Quatre/cinq blocs fixés ; rôles, indices, frontières, partenaires, inversions et coordonnées partagées documentés.
- [x] Coins du culot construits par secteurs distincts et carte locale à déterminant positif ; positivité globale explicitement à démontrer.
- [x] Reconstruction, distances, courbures et erreurs d'évaluation définies ; forme inconnue et refus de certification distingués.
- [x] Centre d'aire, distances par face de paroi, métriques Q1, limites, sévérités et gel avant qualification décrits.
- [x] Famille F1 quantitative et variation ciblée/domaine indépendantes distinguées ; aucun lien promis entre cellules et précision Cx.
- [x] Dépendances, versions cibles, licences et raison d'utilisation documentées ; aucun mailleur/service CFD délégué.
- [x] Construction CPU sans CUDA, presets CMake/CLion et interfaces C++/Python proportionnées définis.
- [x] Échecs, artefacts invalides, ressources et reproduction spécifiés ; y+ facultatif sans blocage géométrique.
- [x] J0/E1 placé avant généralisation ; risques E1–E5 associés à des critères d'arrêt et preuves.
- [x] États prévu/implémenté/vérifié/validé distingués ; aucun prototype, test de produit ou performance inventé.
- [x] Contrôle constitutionnel documentaire avant/après conception conforme ; aucun hook enregistré ; aucune tâche ni implémentation déclenchée lors de cette planification du 2026-10-30.

Les contrôles automatiques effectués portent seulement sur la syntaxe JSON, la cohérence
structurelle des schémas locaux, les liens documentaires, la couverture des 30 FR et l'absence
de marqueurs de gabarit/clarifications. T-01 vérifie seulement l'arithmétique de six compositions
sur des données déclarées. Aucun maillage n'existe pour attester les cases d'acceptation produit.

Les limites Q1 ont le statut initial_design. Avant qualification, E1–E5 fourniront les données
nécessaires à leur revue et gel ; tout changement de valeur ou de méthode doit être explicite
et versionné, sans diminuer la couverture d'acceptation.

Contrôle historique du 2026-10-30 : sept JSON syntaxiquement lisibles, références `$ref` locales résolues,
liens Markdown locaux existants, 30 FR dans la liste, la matrice de spécification et la matrice
de vérification. Configuration exemple et Q1 contrôlés localement pour les types, champs,
limites et références employés. Dans l’environnement de cette revue initiale, le paquet `jsonschema` n’était pas disponible : aucune validation
par un moteur JSON Schema 2020-12 externe n'est revendiquée. Aucun `tasks.md`, source
produit ou fichier CMake de réalisation n’avait été créé à cette date. Aucun hook avant/après enregistré.

## Complément documentaire R1 du 2026-10-31 — historique des tâches autorisées

- [x] Les 30 FR originales sont textuellement conservées ; 45 FR et 8 SC ont une couverture de tâches, sans prétendre à leur vérification produit.
- [x] Campagnes JSON/YAML, expansion explicite, séquence/progression, reprise, échec et clés effectives ont un contrat commun CLI/Python, sans algorithme de maillage dupliqué.
- [x] VTP par rôle et correspondances de diagnostics ajoutés ; VTM/VTS structurés conservés ; lecteur VTK/PyVista et ouverture ParaView obligatoires uniquement en qualification.
- [x] T001–T019 forment le socle J0 indépendant des parcours complets, avec contrôles nécessaires à sa validité, preuves et arrêt sur no-go.
- [x] 83 tâches au format prescrit, dépendances/FR/SC/vérifications/preuves explicites ; expériences distinctes des acceptations et neuf tâches marquées parallélisables.
- [x] Analyse Spec Kit effectuée après génération, corrections documentaires appliquées sur autorisation courante ; risques techniques restant à décider sur preuves signalés.
- [x] Sept schémas validés par jsonschema 4.10.3, références locales contrôlées, trois exemples conformes, JSON/YAML équivalents et cinq mutations de demande invalides rejetées.
- [x] Aucun hook à exécuter ; aucune implémentation ni qualification numérique du mailleur lancée.

Le contrôle courant complète la revue historique ci-dessus : jsonschema est disponible dans
cet environnement d’audit, et tasks.md existe désormais. Ces cases attestent la rédaction
et les contrôles documentaires réellement effectués, jamais la réussite des essais du produit.
Voir [tasks-analysis.md](../tasks-analysis.md) pour la matrice détaillée et les décisions ouvertes.

## Réconciliation R2 — revue du plan courant

- [x] Constitution amendée 1.0.0→1.1.0, principes conservés, principe X et roadmap globale ajoutés ; changement autorisé par la demande courante.
- [x] Même spécification/feature.json ; scripts installés setup-plan/setup-tasks utilisés sans copie destructive ni seconde fonctionnalité.
- [x] 60 FR, 12 SC et 20 PR classées par lot/étape, 95 tâches dont T001–T083 conservées ; correspondances explicites.
- [x] J0 plan T001–T019 indépendant de G-FORM/JG-C ; aucun résultat de topologie conique qualifié par anticipation.
- [x] Décision conique ouverte ; T089–T093 bloquées avant T088 go et critères détaillés à figer ensuite.
- [x] Contrats 1.2 et migration 1.0 explicites, JSON/YAML communs ; YAML partagé dès T025, réutilisé en campagne.
- [x] Modes actifs de vol/conique refusés, extensions archivées signalées ; clés effectives incluent modèles/transformations/versions.
- [x] Associations des champs, Float64, représentations physiques/logiques et lecture indépendante obligatoire spécifiées.
- [x] Inventaire initial sans code produit ni preuve de maillage, aucune suppression de code ni implémentation lancée.

La revue documentaire n’accepte ni formulation, ni maillage, ni solution. L’analyse courante
et ses contrôles exécutés sont dans tasks-analysis.md ; l’historique R1 ci-dessus reste daté.

## Révision R3 — compléments à relire avant réalisation

Les cinq écarts U1/I1/U2/U3/I2 sont traités dans les contrats 1.2 et reconciliation.md :
clés génération/audits/estimations/points, bilan go/no_go exécutable, paramètre de
régularisation retiré, échelle originale unique et historique constitution corrigé dans
le plan. Les 60 FR, 12 SC et T001–T095 sont conservés ; les essais R3 de verification.md
restent **prévus et non exécutés**. Cette note n'ajoute aucune case de réussite produit
et ne ferme pas G-FORM. La constitution et les limites Q1 sont inchangées pendant R3.
