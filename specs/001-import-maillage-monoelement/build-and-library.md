# Construction et bibliothèque — contrat de conception

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

Aucun CMakeLists, preset ou en-tête applicatif n'est créé pendant la planification. Le
présent document fixe le comportement attendu de ces futurs fichiers, sans développer
une infrastructure de solveur.

## CMake et CLion

Projet CMake≥3.25, `LANGUAGES CXX` exclusivement. C++20 sans extensions GNU obligatoires.
Cibles : `aero25d_mesh_core`, `aero25d-mesh`, et suites de tests si `BUILD_TESTING=ON`.
La bibliothèque ne lie ni Python ni CUDA. `find_package(OpenSSL 3 REQUIRED COMPONENTS Crypto)`
sert à SHA-256 ; JSON et Catch2 sont résolus depuis source locale figée ou préfixe installé,
sans FetchContent réseau implicite. Le mode `BUILD_TESTING=OFF` n’exige ni Catch2, ni VTK/PyVista, ni ParaView.
Le jalon initial construit seulement le noyau utile et `aero25d-topology-demo` (test), sans
cible CLI tant que ses sources ne sont pas disponibles. Les références/oracles sont C++ et
les empreintes de preuve viennent de CMake ; aucun interpréteur Python requis à J0. OpenSSL et yaml-cpp sont ajoutés
après J0, respectivement à la provenance/SHA-256 dès US1 et au lecteur partagé
`aero25d_document_io` dès T025. La campagne réutilise ce lecteur ; le noyau géométrique
ne lie pas yaml-cpp. Le produit plan complet exige JSON et YAML pour configurations/campagnes.
`AERO25D_EXPORT_QUALIFICATION=ON` en développement requiert explicitement un lecteur Python
VTK ou PyVista configuré et échoue à la configuration s’il manque. ParaView est vérifié dans
l’acceptation dédiée ; son absence laisse la qualification incomplète.

| Preset (configure/build/test de même nom) | Valeurs fixes |
| --- | --- |
| cpu-debug | Ninja ; build/cpu-debug ; Debug ; BUILD_TESTING=ON ; diagnostics et assertions |
| cpu-release | Ninja ; build/cpu-release ; Release ; BUILD_TESTING=ON ; limites et critères inchangés |
| cpu-sanitize | Hérite cpu-debug ; build/cpu-sanitize ; AddressSanitizer/UBSan ; pas de mesure de performance comparée à Release |

`CMakePresets.json` utilise le schéma 6, cmakeMinimumRequired 3.25.0 ; les buildPresets et
testPresets indiquent leur configurePreset ; seuls les testPresets portent `output.outputOnFailure=true`. `CMakeUserPresets.json`
non versionné choisit les chemins de paquets/compilateurs locaux sans modifier les limites.
Dans CLion, choisir le profil WSL/Linux et importer ces presets ; aucun chemin Windows n'est
encodé dans les fichiers partagés. Installer sous le préfixe CMake `bin`, `lib`, `include`
et les fichiers Python purs ; le guide utilise aussi PYTHONPATH=python pour le développement.

Les builds de référence refusent `-ffast-math`. Les prédicats et intervalles sont compilés avec
arrondi compatible et sans contraction non contrôlée (`-ffp-contract=off` pour GCC/Clang).
Archiver les flags complets et tester que les protections de filtrage restent correctes.
Une compilation qui ne respecte pas leurs hypothèses ne peut produire un résultat certifié.
Pas de test de présence CUDA, ni option qui l'active implicitement si un toolkit est trouvé.

## Surface C++ minimale

API conceptuelle sous `aero25d::mesh` : `read_geometry`, `validate_geometry`,
`resolve_features`, `reconstruct`, `transform_geometry`, `generate`, `inspect_quality`,
`export_bundle`, `generate_family`. Elles prennent des types de configuration/rapport
versionnés décrits dans data-model.md ; elles n'écrivent pas sur stdout et ne quittent pas
le processus. Les erreurs prévisibles retournent un résultat typé avec diagnostics ; les
exceptions imprévues sont traduites au bord CLI en INTERNAL_ERROR, sans statut valide.

`Geometry` possède source/provenance et reconstruction ; `FrameChain` est immuable ;
`Mesh` possède coordonnées, tables de blocs, faces et raccords ; `MeshView` fournit des
vues constantes non propriétaires sur ces tableaux et les métadonnées de repère. Chaque
vue indique compte, stride, dtype, unités et durée de vie liée au propriétaire. Aucune
méthode ne retourne un pointeur GPU ou ne suppose une mémoire unifiée.

Pour préparer la consommation CPU/CUDA ultérieure : coordonnées ξ et z et connectivités sont
contiguës, indices uint64 côté fichier/API, blocs sans cellules fantômes, faces orientées
et interfaces une à une. Les tailles peuvent être converties en indices plus compacts par
un futur consommateur après contrôle de débordement, sans changer les données canoniques.
Aucune structure d'état conservatif, résidu, flux, matrice implicite ou turbulence n'est incluse.
Le tableau de coordonnées sur disque est intercalé [N,2] ; une vue CPU stridée peut l'exposer
sans imposer ce layout à un futur kernel CUDA. L'ajout futur d'un upload sera une autre fonction.

La CLI est l'unique propriétaire des parcours fichiers/exit codes. Python l'appelle par
processus structuré ; une liaison native future pourra exposer la même API sans réécrire
les algorithmes. Les algorithmes de vérification restent indépendants du succès de generate.

## API de campagne

La cible d’orchestration expose `resolve_campaign`, `check_campaign`, `run_campaign` et
`resume_campaign`, sous `aero25d::mesh::campaign`, avec les types du modèle de données.
Elle appelle le noyau et les fonctions de résolution F1, sans copier les algorithmes.
Elle gère uniquement entrées JSON/YAML, expansion, clés, ordonnancement séquentiel, preuves
progressives et reprise. Les codes globaux et états sont ceux de contracts/campaign.md.
CLI et Python n’ajoutent pas de boucle de génération indépendante.

## R2 — surfaces de données et limites de construction

Séparer `LogicalTopology` et `PlanarPhysicalSection/View`, conserver la source line of flight,
le modèle de transformation et les métriques planes ; ne pas ajouter d’interface métrique
conique obligatoire à MeshView. T003/T006/T007/T013 se limitent à ces types simples et aux
contrôles de J0. `src/experiments/conical/` et sa cible de test sont uniquement créés après
T088 go et T089 contrat dérivé, hors des commandes produit planes. Aucun fichier de cette
arborescence n’existe ou n’est requis pour configurer J0. Le solveur C++/CUDA/CPU FP64 est
un objectif futur de la roadmap, pas une cible de construction dans 001.
