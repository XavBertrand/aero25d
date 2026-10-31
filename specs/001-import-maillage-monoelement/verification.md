# Vérification et preuves — plan d'acceptation

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

Statut : campagne à réaliser. Les seuls contrôles exécutés pendant la planification sont la
cohérence documentaire des fichiers/schémas et l'exercice arithmétique limité T-01 décrit dans
[coherence-review.md](coherence-review.md). Aucune ligne FR ci-dessous n'est déclarée satisfaite
par un logiciel existant.

## Oracles et organisation

Les formules exactes, provenance propre au projet, échantillonnage et mutations V-01 à V-09
sont définis dans [geometry-verification.md](geometry-verification.md). Les neuf cas de base
ne sont pas remplacés par des NACA/RAE téléchargés ; ces derniers restent la campagne future
scientifique. Utiliser un oracle analytique indépendant de la spline et du maillage pour
rayons, tangentes, positions, distances transformées. L'oracle d'indices/erreurs n'utilise pas
le lecteur qu'il teste. Les cellules injectées ont un oracle géométrique calculable directement.

Chaque test futur porte un nom stable ci-dessous ; ses sources sont prévues sous `tests/unit`,
`tests/topology_demo`, `tests/integration` ou `tests/contracts`. Aucun fichier de test n'est
créé à cette phase. Les labels CTest du guide regroupent ces tests, sans remplacer leurs critères.

Dossier de preuve futur : `evidence/<version>/<case>/<run_id>/`, contenant `source.dat`,
`fixture-oracle.json`, `requested.json`, `resolved.json`, `manifest.json`, `report.json`,
certificats bornés, tableaux locaux et figures. Les rapports référencent FR, cas, version du
profil Q1, formule analytique et entrée exacte. Les tables ci-dessous reprennent les critères
contractuels de la spécification, puis leur associent des essais concrets ; une statistique
agrégée seule ne suffit pas pour les contrôles exhaustifs.

## Traçabilité exhaustive

| FR | Test prévu / cas | Méthode et critère mesurable | Preuves attendues |
| --- | --- | --- | --- |
| FR-001 | `geometry/import_counts` ; V-01/02/03/06 | Import de fixtures connues : Comptes, coordonnées et ordre de tous les éléments identiques aux données attendues | Sources, tableaux attendus/obtenus ; oracle et version du profil |
| FR-002 | `geometry/grammar_counts` ; V-04 | Variantes lexicales et fautes de comptage : Toutes les variantes autorisées passent ; chaque faute et donnée excédentaire est refusée | Fixtures, statuts et diagnostics ; oracle et version du profil |
| FR-003 | `geometry/invalid_contacts` ; V-04/06 | Injection de chaque défaut décrit : Tous les défauts sont localisés ; seule la répétition terminale autorisée est reconnue comme marqueur | Sources annotées, résultats de contrôle ; oracle et version du profil |
| FR-004 | `geometry/source_immutability` ; V-04 | Comparaison avant/après contrôle invalide : Source inchangée, zéro réparation cachée, cause et indices présents pour chaque défaut connu | Empreintes, diagnostics structurés et lisibles ; oracle et version du profil |
| FR-005 | `multielement/reject_meshing` ; V-06 | Import puis maillage de deux éléments : Deux éléments accessibles ; refus explicite ; zéro maillage partiel validé | Source, inventaire, statuts ; oracle et version du profil |
| FR-006 | `transforms/conventions_moment_axes` ; V-01/02/03 | Inspection de chaque cas valide : Toutes les conventions et références présentes et conformes à H-02 à H-05 | Métadonnées, figures annotées ; oracle et version du profil |
| FR-007 | `transforms/roundtrip_S_N_W` ; V-01/03/06 + T-01 | Aller-retour S→N→W→N→S avec translation, échelle, Z non centré et flèches 0°/25°/50° sur un et deux éléments : Erreur maximale ≤ 10⁻¹² c ; une transformation globale ; positions relatives restituées | Coordonnées et résidus, transformations ; oracle et version du profil |
| FR-008 | `ambiguous-features/hints` ; V-05 | Cas connus et ambigu résolu/incohérent : Tous les repères attendus trouvés ; zéro choix silencieux en ambiguïté ; indication incohérente refusée | Annotations attendues/obtenues, indications ; oracle et version du profil |
| FR-009 | `transforms/sweep_reference` ; V-01/02/03 + T-01 | Comparaison point par point à 0°, 25°, 50° : Écart à la formule et à son inverse ≤ 10⁻¹² c pour tous les points ; c inchangée | Tables de comparaison et résidus ; oracle et version du profil |
| FR-010 | `transforms/normal_section` ; V-01/02/03/06 | Inspection des dimensions et de l'extrusion : Deux coordonnées de calcul par sommet ; même direction d'extrusion pour tous les éléments ; transformation effective vérifiée même sans Mach renseigné | Métadonnées et coordonnées ; oracle et version du profil |
| FR-011 | `geometry-fidelity/radius_angles_shape` ; V-01/02/03 | Références analytiques et contrôle des courbures/caractéristiques : Budgets H-07/H-08 respectés ; zéro nouvelle intersection, zéro extremum/oscillation absent de la référence ; pas de pas plus grand dans une zone de forte courbure que dans une zone comparable moins courbe, hors concentration explicitement prescrite | Courbes et références dans S/N/W, rayons/angles transformés attendus et mesurés, espacements ; oracle et version du profil |
| FR-012 | `geometry-fidelity/bounded_distances` ; V-01/02/03 | Mesure bidirectionnelle des deux écarts : Chaque maximum respecte son budget ; méthode et incertitude de mesure présentes, références comparées transformées dans le même repère S/N/W | Cartes d'écart, maxima, tolérances ; oracle et version du profil |
| FR-013 | `geometry/closure_base` ; V-03/04/05 | Fermetures explicites/implicites et ouvertures suspectes : Culot fourni préservé ; culot ajouté seulement après confirmation ; tous les autres cas ambigus suspendus ; épaisseur conservée dans le budget caractéristique | Contours avant/après, segments ajoutés, confirmations ; oracle et version du profil |
| FR-014 | `topology-demo/quad_structured` ; V-01/02/03 | Inventaire et validation du maillage : Toutes les cellules quadrilatérales, indices structurés par bloc, frontières conformes au budget ; zéro appel à une résolution | Maillages, inventaire des types, trace du parcours ; oracle et version du profil |
| FR-015 | `topology-demo/thin_thick_sectors` ; V-01/03 + J0 | Cas fin/épais et inspection de topologie : Familles distinctes documentées ; culot/sillage explicites dans le cas épais ; zéro cellule effondrée dans le cas fin | Schémas de blocs, connectivités, rapport ; oracle et version du profil |
| FR-016 | `topology-demo/shared_interfaces` ; V-01/03/08 | Parcours exhaustif des raccords : Chaque face interne a exactement un partenaire ; correspondance bijective, orientations opposées et résidu géométrique ≤ 10⁻¹² c | Table des raccords, résidus et orientations ; oracle et version du profil |
| FR-017 | `meshing/region_coverage` ; V-01/02/03 | Inspection des régions : Chaque région applicable est identifiée, contient des cellules non dégénérées et possède ses mesures locales | Carte des régions, comptes et mesures ; oracle et version du profil |
| FR-018 | `meshing/control_parameters` ; V-07/08 | Variation séparée de chaque contrôle : Chaque paramètre valide est enregistré et a un effet mesuré sur sa grandeur cible ; chaque incompatibilité est refusée sans substitution cachée | Paramètres demandés/résolus, comparatifs ; oracle et version du profil |
| FR-019 | `meshing/wall_face_distances` ; V-01/03/08 | Mesure des premières cellules : Deux grandeurs distinctes présentes pour chaque face de paroi ; écart à la hauteur prescrite dans la tolérance de distribution du profil, explicitée avant génération | Cartes de hauteur et distance, limite appliquée ; oracle et version du profil |
| FR-020 | `contracts/optional_yplus` ; V-09 | Cas avec/sans données physiques : Toute estimation présente contient formule, entrées, hypothèses et avertissement ; aucune valeur garantie ; absence d'échelle physique signalée | Rapport d'estimation, métadonnées ; oracle et version du profil |
| FR-021 | `families/domain_independence` ; V-07 | Deux distances à paramètres intérieurs fixes : Distance mesurée conforme à la demande dans le budget du profil ; paramètres intérieurs prescrits inchangés ; aucune promesse sur Cx | Paramètres, cartes de distance, comparatif ; oracle et version du profil |
| FR-022 | `injected-mesh-defects/integrity` ; V-08 | Contrôle exhaustif et défauts injectés : Toutes les aires > 0 ; zéro cellule invalide, face mal orientée, défaut de fermeture ou raccord manquant ; chaque injection critique est détectée | Mesures par cellule/face, défauts attendus/observés ; oracle et version du profil |
| FR-023 | `injected-mesh-defects/quality_Q1` ; V-08 | Cas étiré aligné puis désaligné : Mesures et limites présentes pour toutes les cellules ; verdict conforme au profil et à l'orientation ; aucun rejet fondé uniquement sur le rapport d'aspect | Cartes, profil qualité et verdicts ; oracle et version du profil |
| FR-024 | `contracts/failure_artifacts` ; V-04/08/09 | Injection de défauts critiques et non critiques : Chaque défaut a localisation, mesure, limite et sévérité ; tout défaut critique entraîne l'échec et le marquage des sorties | Rapport, statuts commande/Python, sorties invalides ; oracle et version du profil |
| FR-025 | `families/global_F1` ; V-07 | Trois niveaux sur fin et épais : Même reconstruction, même graphe de blocs et frontières ; nombres de subdivisions strictement croissants dans chaque direction/région applicable ; pas maximaux décroissants et première hauteur non croissante ; rapports de raffinement publiés | Paramètres des niveaux, tableaux des pas/comptes, rapports ; oracle et version du profil |
| FR-026 | `contracts/python_cli_parity` ; V-09 | Parcours complet commande/Python : Toutes les opérations disponibles ; mêmes statuts, connectivité et mesures à la tolérance numérique de reproduction près ; zéro résolution lancée | Transcriptions et résultats comparés ; oracle et version du profil |
| FR-027 | `contracts/vtk_binary_roundtrip` ; V-09 | Relecture du VTK et de la représentation structurée : Tous les sommets/cellules exportés sont retrouvés ; blocs, frontières et interfaces conservés ; identifiants du rapport résolus à 100 % | Exports, résultats de relecture et correspondances ; oracle et version du profil |
| FR-028 | `contracts/report_svg_views` ; V-09 | Lecture du rapport et inspection des figures : Rapport lisible depuis Python ; cinq vues présentes, unités/repères/légendes présents ; trois contours superposables | Rapport, figures et résultat de lecture ; oracle et version du profil |
| FR-029 | `contracts/reproduction` ; V-09 | Deux générations dans le même environnement/version : Métadonnées complètes ; même connectivité et identifiants ; écart maximal des coordonnées ≤ 10⁻¹² c ; mêmes décisions de qualité | Sources, empreintes, manifestes et comparaison ; oracle et version du profil |
| FR-030 | `contracts/cpu_standalone` ; V-09 | Parcours dans un environnement CPU sans solveur externe : Import, génération, rapport et export aboutissent sans GPU ni exécutable/service CFD externe | Inventaire de dépendances et trace d'exécution ; oracle et version du profil |

## Seuils et précision des preuves

- Tests géométriques : distances bidirectionnelles [L,U], rayon et angle comparés à la référence
  dans le même repère S/N/W ; le changement intentionnel sous cosΛ n'est jamais un défaut.
- Les tests de transformation couvrent translation horizontale/verticale, échelle ≠1,
  Z non centré, flèches 0°/25°/50° et deux éléments. Mesurer l'aller-retour dans les unités S.
- Test moments : même point quart de corde, eY=(0,1,0), eη=(sinΛ,cosΛ,0).
  À Λ=0° axes identiques ; à 25°/50° distincts. Un vecteur synthétique M=(1,0,0) a
  M·eY=0 et M·eη=sinΛ : c'est un test de métadonnées/projection, aucun calcul aérodynamique.
- J0 contrôle d'abord le squelette et toutes les cellules des topologies ; les tests complets
  ne sont déclarés réussis qu'après Q1 et fidélité. Aux coins, les mesures sont par face.
- Cas V-04 : I-18 (lèvres très proches) peut être importé ; I-23 (profil décimé) peut être
  inspectable sans certification du rayon. Attendre le refus ou statut à la bonne étape,
  jamais obliger le lecteur à déclarer ces entrées lexicalement invalides. SC-001 a été
  précisé sur ce point, couverture I-01 à I-24 intégralement conservée.
- Les comparateurs de qualité sont exercés à la limite, un ulp sous et au-dessus, avec
  incertitude qui croise la limite. Une indétermination ne devient pas un succès.
- Formats : relire avec un lecteur VTK indépendant en campagne ; vérifier les types de
  cellules, identifiants, coordonnées et interfaces par manifeste, pas seulement ouvrir une vue.
- Reproduction : même version/environnement, identifiants/connectivité identiques et coordonnées
  ≤10⁻¹² c ; compilateurs/environnements différents sont comparés séparément sans prétendre
  à l'identité bit à bit. Une dérive du verdict près d'une limite doit être expliquée, pas cachée.

## Qualification documentaire et qualification produit

L'autorisation de générer des tâches repose sur la complétude des décisions et contrats, pas
sur une exécution de J0. La qualification du produit exige tous les SC de la spécification,
la campagne FR et les preuves ci-dessus. E1–E5 mesurent encore des inconnues. Aucun de ces
contrôles géométriques ne prouve l'incertitude aérodynamique de 10⁻⁴ sur Cx.

## Complément du 2026-10-31 — campagnes, VTP et J0

Chaque ligne suivante est **prévue, non exécutée**. Les noms de cas et tâches figurent dans
[tasks.md](tasks.md) ; une association documentaire ne vaut pas vérification.

| FR | Test prévu / méthode | Critère mesurable | Preuves attendues |
| --- | --- | --- | --- |
| FR-031 | JSON/YAML ; produit et points équivalents | 18 points attendus pour 2×3×3 ; même ordre et paramètres effectifs pour la liste équivalente ; modes contradictoires refusés | Demandes, inventaires comparés, diagnostics |
| FR-032 | Contrôle/inventaire seul avec cas invalide | Zéro appel à generate, zéro bloc exporté ; tous les points développables inventoriés avec causes ; ready ne signifie pas maillage valide | Compteurs d’appels, état résolu et tableau |
| FR-033 | Interruption avant/après publication d’un cas | Une génération active au maximum ; chaque cas publié conservé ; aucun cas interrompu marqué terminé | Journal, instantanés avant/après, bundles et empreintes |
| FR-034 | Reprise, puis mutations source/configuration/version/qualité et corruption | 100 % des cas terminés compatibles et intègres réutilisés ; zéro réutilisation incompatible ; tentatives invalides conservées | Clés, matrice de compatibilité, compteurs et historique |
| FR-035 | Un échec médian sous les deux politiques | continue tente le suivant ; stop le marque not_run ; code global non nul, échec et cause conservés | Rapports, journal et CSV des deux parcours |
| FR-036 | Comparaison manifeste/tableau/dossiers | Une ligne par point ; toutes les références et statuts concordent ; points et maillages uniques comptés séparément | Manifeste résolu, état, CSV et audit de références |
| FR-037 | Autre cwd puis déplacement des sorties | Mêmes entrées résolues et digests ; 100 % des liens requis restent relatifs et lisibles après déplacement | Arborescences, commandes et audit des chemins |
| FR-038 | Parcours commande/Python de campagne | Inventaires, clés, statuts, décisions de reprise et codes égaux ; aucun calcul de maillage dans Python | Transcriptions et comparaisons structurées |
| FR-039 | Extension réservée et demande active de vol | Extension archivée not_executed ; demande active refusée ; zéro solveur appelé et zéro résultat aérodynamique produit | Demandes, refus et inventaire des artefacts |
| FR-040 | Omissions/défauts équivalents et incidences différentes | Un appel de génération par clé effective tentée ; points tous conservés ; modification d’un paramètre géométrique change la clé | Vecteurs de clés, paramètres résolus et compteurs |
| FR-041 | VTP de contrôle et maillage fin/épais | Rôles applicables distincts, culot épais conservé, IDs résolus à 100 % vers diagnostics/provenance ; VTM/VTS structurés conservés | Exports relus et tables de correspondance |
| FR-042 | Inspection puis déplacement de l’ensemble exporté | Repères S/N/W et unités sans ambiguïté ; aucune référence requise absolue ; ensemble relu après déplacement | Métadonnées, audit de chemins et relecture |
| FR-043 | Lecteur indépendant VTK ou PyVista sur tous les formats | Comptes/types/IDs identiques ; résidus de coordonnées ≤10⁻¹² c ; qualification bloquée si lecteur absent ou test non exécuté ; produit utilisable sans ce lecteur | Versions de tests, logs et tableaux comparés |
| FR-044 | Ouverture ParaView des exports fin/épais et contrôle seul | VTM/VTS/VTP ouverts sans erreur ; rôles sélectionnables et repères inspectables ; zooms BA/lèvres avec version et conclusion | Journal d’ouverture, captures, état ParaView à chemins relatifs |
| FR-045 | J0 et revue sur neuf configurations, variantes de fermeture V-03 | Neuf succès de validité topologique et géométrique ciblée, limites Q1 applicables ; toutes cellules/interfaces contrôlées ; échec ⇒ arrêt avant généralisation | Rapport J0, mesures locales, vues BA/lèvres, décision go/no-go signée par auteur et datée |

V-10 comprend V-01/V-02 × 3 flèches × 3 niveaux compatibles (18 points), leur liste équivalente, et des points V-03 avec configuration épaisse distincte,
un jeu de doublons d’incidence, une erreur médiane et des interruptions avant/après commit.
Le contrôleur de qualification vérifie aussi JSON/YAML, champs inconnus, IDs dupliqués,
produit débordant, changement de cwd et déplacement de dossier. La matrice de reprise mute
séparément source, paramètres effectifs, version binaire/algorithmes, environnement,
profil qualité et intégrité ; une modification d’incidence seule conserve le maillage.

J0 est une preuve ciblée sur contour analytique : pas de fausse certification du lecteur
ou de la spline générale. Son oracle polynomial et les contrôles algébriques bornés précèdent
les évaluateurs avancés. Les neuf cas et les deux fermetures V-03 sont obligatoires ; les
très petits culots E3 sont des limites avec succès ou refus motivé. Tous les critères locaux
et globaux de validité et Q1 applicables sont exigés, sans exclusion des coins.
Une impossibilité non décidée numériquement donne no-go, jamais un passage par inspection visuelle.

La qualification des exports exige VTK ou PyVista, ainsi qu’une ouverture ParaView versionnée.
L’absence du lecteur ou de ParaView empêche d’attribuer le statut qualifié, et doit être
rapportée comme essai non exécuté/bloqué, pas comme réussite par skip. J0 n’attend pas cette
qualification d’interopérabilité : ses exports minimaux servent à l’inspection et seront
qualifiés ensuite par V-09. `BUILD_TESTING=OFF` n’a aucune dépendance VTK/PyVista/ParaView.

## Réconciliation R2 — matrice complémentaire et portée

Les 45 lignes antérieures restent applicables au **chemin plan** ; FR-009/010/022 sont
précisées dans la spécification sans modifier les valeurs H/Q1. Les 15 exigences suivantes
complètent les preuves de 001. Aucune n’est cochée sur la seule présence d’un document/test.

| FR | Méthode | Critère mesurable | Preuves attendues |
| --- | --- | --- | --- |
| FR-046 | Audit produit/fonctionnalité | 20 PR reliées à étapes courantes/futures et statuts explicites | Roadmap, matrice PR/FR et analyse |
| FR-047 | V-11, source/physique inverse | Source/digests inchangés ; correspondances planes ≤10⁻¹² c | Cartes/IDs et résidus |
| FR-048 | Inspection types et représentations V-11 | Cinq couches distinctes ; aucun champ métrique conique obligatoire dans le chemin plan | Schémas, vues et tests natifs |
| FR-049 | Demandes négatives V-11 | 100 % des modes/champs actifs non supportés refusés et localisés ; extension inactive signalée | Demandes, diagnostics et parité CLI/Python |
| FR-050 | Revue du corpus primaire | Trois familles comparées, sources/accès/pages et limites de preuve explicités | Bibliographie annotée et tableau comparatif |
| FR-051 | Revue G-FORM | Toutes rubriques de décision renseignées par dérivation ou limitation motivée ; données géométriques déterminantes | Dérivations, contrôles de cohérence et décision |
| FR-052 | Audit du graphe et de la porte | Aucune tâche de construction/métrique conique admissible sans go G-FORM ; aucun contrat figé avant celui-ci | Graphe, état de décision et liste des tâches bloquées |
| FR-053 | JG-C conditionnel V-C01/V-C03 | Après go : deux géométries fin/épais à pente non nulle ; raccords exacts, bornes métriques positives et limites figées après décision ; sinon bloqué | Oracles, certificats, vues et verdict JG-C |
| FR-054 | Revue restitution + correspondances géométriques | Deux livrables de pression distincts ; mapping et erreurs géométriques vérifiés, aucun champ p/Cp inventé | Note de dérivation, mappings et limites de portée |
| FR-055 | Revue références et dérivation par mode | Axes/q∞/c₀/quart de corde/Y/mesure définis ; intégrales coniques encore soumises à G-FORM | Contrat de références et essais analytiques futurs identifiés |
| FR-056 | Relecture VTP/VTS/canonique V-11 | 100 % tableaux dimensionnés selon association, Float64 de coordonnées, représentation et repères explicites | Exports relus et comptages d’associations |
| FR-057 | Mutations de clés V-10/V-11 | Aucune reprise après changement pertinent ; incidence descriptive ne multiplie pas les maillages | Vecteurs de clés, résolution versionnée et matrice de mutations |
| FR-058 | Audit feuille de route et validation conique | Uniforme/métriques/limite/références/erreurs séparées affectés à travaux futurs ; aucun statut livré | Table des étapes futures et critères |
| FR-059 | Audit dépendances puis J0 | Aucun chemin de J0 vers G-FORM/JG-C ; neuf configurations et critères J0 conservés | Graphe et preuves J0 futures |
| FR-060 | Audit avant/après réconciliation | 45 FR/83 tâches antérieures toutes retrouvées ou explicitement mappées ; aucun statut obsolète coché | Inventaire/historique, correspondances et analyse |

V-11 couvre modèles explicites 1.2 et lecture 1.0, configurations JSON/YAML, représentation
physique/logique, correspondances line of flight, associations point/cell/face et champs
actifs non supportés. T021/T025/T031, T044–T055 et T066–T077 l’intègrent aux acceptations
planes existantes, sans dépendance conique. Les mutations incluent chaînes de transformations,
modèles/versions effectifs, migration et corruption ; le nom du profil ne suffit jamais.

G-FORM (T088) est une acceptation scientifique, pas un résultat numérique. La note initiale
conical-study.md n’est pas un go : dérivations complètes et revue identifiée sont requises.
JG-C (T093) dépend de T088 **avec go**, T019 **avec go J0**, et du contrat/critères T089.
Ce contrat fixe après décision les seuils chiffrés, unités et bornes d’incertitude des mesures
coniques ; aucun seuil provisoire sans dérivation ne permet de lancer la construction.
Les cas V-C01/V-C03 doivent couvrir profil fin et épais à pente non nulle, une séquence
c′→0, des entrées/apex incompatibles, raccords et métriques non singulières, restitution de
la géométrie S et d’un scalaire analytique explicitement synthétique. Ce scalaire ne sera
jamais nommé pression/Cp calculé. Relecture VTK/PyVista et ouverture ParaView des vues physiques
et logiques restent requises. Une vue physique éventuellement courbe n’est pas aplatie sans
étiquette ni mapping. Un no-go conserve mesures et acquis plans séparés.

La **qualification du lot A** T083 porte le mailleur plan ; T095 audite séparément lot A et
lot B d’étude/géométrie. Si G-FORM/JG-C manque, le lot B reste non vérifié/bloqué même si le
lot plan est accepté. Ni verdict ne vaut capacité de résolution 2.75D. Aucun code ni preuve
numérique de produit n’est présent à la date de réconciliation.

### Travaux futurs de vérification des équations (hors implémentation 001)

| Étape roadmap | Essai requis | Ce que la preuve devra distinguer |
| --- | --- | --- |
| FUT-01 | SA-neg compressible 2D/2.5D, référence CPU FP64, cas analytiques/manufacturés et indépendants | Équations, discrétisation et convergence, sans réattribuer un succès géométrique comme preuve RANS |
| FUT-02/03 | Identités métriques et mesures physiques/conservation | Compatibilité de la géométrie, métriques et équations, pas seulement Jacobien plan positif |
| FUT-03 | Écoulement uniforme cartésien en base variable | Composantes transformées cohérentes et résidu numérique ; aucun uniforme fictif à composantes curvilignes imposées constantes |
| FUT-03 | Limite c′→0 vers 2.5D | Équations, conditions aux limites et restitution, pas seulement convergence des coordonnées |
| FUT-03 | Solutions manufacturées comprenant diffusion/énergie et transport turbulent | Erreurs/ordres attendus, branches SA-neg et hypothèses de fermeture vérifiées séparément |
| FUT-03/04 | Références indépendantes Cp/Cf/efforts, restitution dans S et champ p sur plan physique | Écart numérique, modèle conique et erreurs expérimentales distincts ; accords locaux avant coefficients |
| FUT-04/06 | Études de maillage, itération et domaine puis comparaison CPU/GPU | Cible d’incertitude Cx≈10⁻⁴ sur seuls cas qualifiés ; aucune garantie d’accord expérimental |

Les PR de la roadmap sont classées courant/partiel/futur dans reconciliation.md. Elles ont
une étape de vérification visible sans être déclarées toutes implémentées par les 95 tâches.
