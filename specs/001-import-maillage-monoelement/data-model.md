# Modèle de données — géométrie et maillage monoélément

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

Date : 2026-10-30. Contrats de conception uniquement ; aucune structure ci-dessous n'est encore implémentée. Les schémas de transport sont dans [contracts](contracts/README.md), les blocs dans [topology.md](topology.md), les évaluateurs dans [geometry-verification.md](geometry-verification.md).

## Identité et règles communes

Les identifiants locaux source sont des entiers à base 1 : élément, point, segment, ligne. Les identifiants de maillage sont des `uint64` à base 0 ; les indices `(i,j)` sont à base 0, sans cellule fantôme dans les fichiers. Un identifiant est stable dans tous les artefacts d'une génération ; aucune stabilité entre deux niveaux distincts n'est revendiquée. Les tables de correspondance d'interface, de provenance et d'export rendent ces relations explicites.

Une coordonnée originale possède son lexème décimal source et sa valeur calculable ; les octets du fichier restent la référence de provenance. Les calculs numériques sensibles utilisent le repère de travail global normalisé, sans réécriture de la source. Tous les nombres JSON doivent être finis ; une mesure non disponible est `null` avec un motif, jamais NaN ou un zéro de substitution. Unité, repère et normalisation sont obligatoires sur toute collection de coordonnées ou métriques dimensionnées.

## Entités

| Entité | Contenu essentiel | Invariants et relations |
| --- | --- | --- |
| `SourceDocument` | Octets archivés, URI d'origine informative, SHA-256, format/version, unités déclarées, lexèmes, lignes et comptes | Fichier immuable ; digest calculé sur les octets, avant toute interprétation |
| `SourceElement` | `element_id`, liste ordonnée de `SourcePoint`, compte déclaré, marqueur éventuel de fermeture | Tous les éléments lus ; refus de génération si nombre différent de 1 |
| `SourcePoint` | `point_id`, `line`, deux lexèmes X/Z, valeurs, indices source | Aucun point supprimé de l'archive ; répétition terminale autorisée représentée explicitement |
| `WorkingContour` | Références des points source, permutation/sens de parcours, segments, fermeture, segments ajoutés | Orientation de travail documentée, pas de réparation cachée ; segment ajouté identifié et confirmé |
| `FeatureSet` | BA, lèvres supérieure/inférieure, pointe éventuelle, intervalles extrados/intrados/culot, sommets protégés | Toute caractéristique référence point/paramètre source ; résultats automatiques et indications distincts ; ambiguïté ne vaut pas sélection |
| `UserHints` | Source des indications, indices des repères, type de BF explicite, confirmation du culot ajouté | Ne peut déroger aux contrôles d'intégrité ; paramètres archivés dans demandé et résolu |
| `FrameChain` | Repères source S, section normale N, travail W ; flèche, Xref, translation globale, échelle positive, matrices et inverse | Ordre direct S→N→W ; inverse W→N→S ; une seule chaîne pour tous les éléments |
| `ReferenceGeometry` | c originale, projection de corde X, quart de corde en S/N/W, unités et échelle physique facultative | La compression en flèche ne remplace jamais c originale |
| `AxisMetadata` | Base globale X/Y/Z, base ξ/η/z, conventions incidence et moment futur | Y et η sont distincts ; aucune valeur de force ou moment dans cette fonctionnalité |
| `Reconstruction` | Segments paramétriques, sommets protégés, continuités, paramètres d'interpolation, version et empreinte | Construite une fois pour une famille ; géométrie commune à tous les niveaux |
| `GeometryEvidence` | Distances encadrées, précision des évaluateurs, rayon/angles et leur statut, références comparées, diagnostics | Original comparé à original ; transformé comparé à transformation attendue ; non certifiable explicitement indiqué |
| `RequestedConfiguration` | Entrées utilisateur, paramètres exprimés avec unités/conventions, champs omis connus | Immuable ; inconnus/refusés localisés par pointeur JSON |
| `ResolvedConfiguration` | Valeurs résolues exhaustives, profil de qualité versionné, subdivisions, distributions, limites, dépendances et paramètres facultatifs | Aucune relaxation automatique ; toute valeur dérivée porte la règle utilisée ; incompatibilité provoque échec |
| `InterfaceCurve` | ID, ordre canonique, tableau unique de `global_node_id`, extrémités et courbe de référence | Coordonnées générées une seule fois ; chaque bloc référence ces mêmes nœuds |
| `MeshNodes` | Table globale `(ξ,z)` en N et représentation W si nécessaire, identifiants | Doubles finis ; aucun stockage géométrique indépendant d'une interface dans deux blocs |
| `StructuredBlock` | `block_id`, rôle, `(ni,nj)` cellules, table `(ni+1,nj+1)` d'IDs globaux, orientation logique positive | Toutes les cellules quadrilatérales ; Jacobien logique positif obligatoire ; aucune inversion masquée par réordonnancement |
| `BoundaryPatch` | ID, bloc, côté, plage d'indices inclusive des nœuds, type physique/raccord, orientation | Toute face extérieure appartient exactement à une frontière physique ; chaque face intérieure à un raccord |
| `InterfacePair` | ID, côtés partenaires, tailles, orientation `same/reversed`, permutation, ID de courbe commune | Bijection un à un des nœuds et faces, sens des normales opposés ; aucun trou admis par tolérance |
| `CellMetrics` | ID global cellule, bloc `(i,j)`, aire, centroïde, fermeture, étirement, orientation et métriques de qualité | Contrôle exhaustif ; unités et profil de limites connus ; centre géométrique du quadrilatère, pas moyenne implicite des sommets |
| `WallFaceMetrics` | ID face, normale de la face, hauteur première cellule, distance normale du centroïde, mesures aux coins | Deux faces incidentes au coin possèdent chacune leurs mesures ; pas de normale unique inventée au sommet |
| `QualityReport` | Statut, compteurs, minima/maxima/quantiles, tableaux locaux, défauts, limites et méthodes | Un seul défaut critique invalide le résultat ; les agrégats ne remplacent pas les données locales |
| `Diagnostic` | Code stable, sévérité, message, localisation source/maillage/paramètre, mesure, limite et motif | Les codes servent au traitement automatique ; texte lisible ; localisation partielle permise seulement si objet inexistant/non lisible |
| `ReproductionManifest` | Version, environnement, empreintes, configurations, chaîne de transformation, fichiers et formats, statuts | Validation des tailles/digests avant chargement ; chemin relatif contenu dans le dossier ; aucune référence absolue requise pour reproduire |
| `MeshFamily` | Configuration racine, reconstruction commune, niveaux, facteurs mesurés, variantes ciblées ou domaine | Les trois expériences ont des types distincts ; une variante ciblée n'est pas étiquetée famille de convergence |

## Repères, unités et moment futur

S est le plan source `(X,Z)` à Y=0, dans les unités déclarées `m` ou `dimensionless`. N est le plan normal `(ξ,z)` dans les mêmes unités : `ξ=(X−Xref)cosΛ`, `z=Z`. W est le repère de travail adimensionnel global : `(u,v)=((ξ−tξ)/s,(z−tz)/s)`, avec `s>0` exprimé dans les unités de S/N. Sans normalisation demandée pour les sorties, W reste un repère interne de calcul archivé ; les sorties utilisateur N ne sont pas confondues avec W.

L'inverse commence par `ξ=s*u+tξ`, `z=s*v+tz`, puis `X=ξ/cosΛ+Xref`, `Z=z`. Les matrices et leurs paramètres sont archivés pour vérifier la composition, sans prendre une matrice recalculée arrondie comme unique vérité. Le test d'aller-retour est mesuré relativement à c originale, pas relativement aux coordonnées absolues.

Le quart de corde est un point de référence global en S à Y=0 ; ses projections N/W sont enregistrées. `moment_contract` contient `reference_point`, `global_basis=[eX,eY,eZ]`, `extrusion_basis=[eξ,eη,ez]` et `projection_axes={pitch_global:eY, extrusion:eη}`. À flèche non nulle `M·eY` et `M·eη` sont des projections différentes. Le futur solveur pourra fournir `M_global` et déclarer sa projection choisie ; aucune composante de moment calculée, unité de moment ou normalisation de coefficient n'est produite ici.

## Organisation physique du résultat

Un dossier de résultat est autonome :

```text
run/
  manifest.json                 # état, versions, unités, références et fichiers
  source/profile.dat            # octets originaux
  requested.json                # configuration soumise
  resolved.json                 # configuration exhaustive
  geometry.json                 # repères, caractéristiques, reconstruction, provenance
  mesh/topology.json            # blocs, côtés, raccords et IDs
  mesh/nodes.f64le              # ξ,z de chaque ID global, sans en-tête
  mesh/block_nodes.u64le        # tables des blocs concaténées, i le plus rapide
  quality/report.json           # synthèse et diagnostics
  quality/cells.jsonl            # une mesure locale par cellule
  quality/wall_faces.jsonl       # une mesure locale par face de paroi
  figures/{profile,leading-edge,trailing-edge,wake,global-mesh}.svg
  vtk/contours-source.vtp       # S, provenance des points/segments
  vtk/contours-reconstructed.vtp # N (W sur demande explicite)
  vtk/boundaries.vtp            # N, parois et interfaces par rôle
  vtk/export-index.json         # IDs d’objets et liens de diagnostic
  vtk/mesh.vtm                  # si export demandé
  vtk/block-000.vts ...
```

Le manifeste décrit pour chaque tableau `dtype`, `endianness`, `shape`, `offset_bytes`, `size_bytes`, `sha256`. Le produit des dimensions et les calculs de tailles sont contrôlés avant allocation. Les offsets sont en octets ; le premier nœud d'un bloc est à `(0,0)` et i varie le plus vite. Un seul `nodes.f64le` est source des coordonnées d'interface. Les champs auxiliaires en VTK sont dérivés de ce canonique, jamais utilisés pour reconstituer un raccord par rapprochement géométrique.

## États et transitions

```text
received → parsed → geometry_valid → features_resolved → reconstructed
         → transformed → topology_ready → generated → quality_passed → committed_valid
```

Toute étape peut produire `failed`, avec `last_completed_stage`, un rapport et les artefacts de diagnostic disponibles. `needs_input` est distinct d'une géométrie invalide ; il correspond notamment aux indications manquantes. Un import multiélément peut atteindre `geometry_valid` mais échoue en `unsupported` lors de la génération. `not_checked` ne signifie jamais `valid`.

Les sorties sont préparées dans un répertoire temporaire voisin. Le manifeste `valid=true` n'est publié par renommage atomique qu'après toutes les vérifications requises. En échec, le dossier est publié sous le nom demandé avec suffixe `.invalid-<run_id>` ; son manifeste porte `valid=false`, un statut explicite et le dernier stade terminé. Il ne contient pas de manifeste de maillage valide ni de fichier VTK sans marquage. Une commande ne remplace jamais un résultat préexistant sans une option explicite `--replace` ; cette option ne relâche aucune exigence de qualité.

## Entités supplémentaires — campagne et visualisation (2026-10-31)

| Entité | Contenu | Invariant |
| --- | --- | --- |
| `CampaignRequest` | Document original JSON/YAML, base des chemins, profils, configurations, développement et politique | Une seule syntaxe normalisée et aucune génération au contrôle |
| `CampaignPoint` | ID logique, profil/flèche/configuration/niveau/incidence, paramètres demandés/résolus et diagnostics | Une ligne par demande même si plusieurs lignes partagent un maillage |
| `EffectiveMeshCase` | mesh_case_id, mesh_key, reuse_key, bundle et versions des règles | Une génération par identité effective tentée dans une exécution ; incidence descriptive exclue |
| `CampaignSnapshot` | Demande normalisée, substitutions, sources/digests, points et compatibilité | Photographie résolue immuable par tentative ; références relatives |
| `CampaignState` | ready/pending/running/completed/reused/failed/not_run, compteurs et tentatives | Publication progressive ; completed exige un bundle valide et intègre ; un échec n’est pas repris comme succès |
| `ExportObject` | export_object_id, représentation, rôle, repère, unités, provenance et IDs canonique/source | Tous les objets exportés sont adressables ; pas de faux global_node_id sur un point source |

`CampaignSnapshot` et `CampaignState` utilisent campaign-state.schema.json avec les champs
`artifact`/`phase`. La campagne conserve `requested-campaign.{json|yaml}`, des instantanés
`attempts/<id>/campaign-resolved.json`, l’état courant, events.jsonl, summary.csv et
`cases/<mesh_case_id>/attempts/<id>/`. Les entrées archivées sont dans `inputs/` ; les chemins
internes sont relatifs au dossier qui les contient. L’état fait autorité sur le CSV après crash.
Les états `ready` et `reused` sont distincts de la validité d’une géométrie ; la phase/opération
et le verdict du bundle restent nécessaires. Un champ futur de vol est archivé not_executed.

## Séparation R2 des représentations et associations

| Couche | Représentation courante | Préparation conique / statut |
| --- | --- | --- |
| `OriginalFlightSection` | SourceDocument/SourceElement en S, contour original line of flight, identités et provenance | Reste référence d’entrée/restitution, aucune source remplacée |
| `LogicalTopology` | StructuredBlock et InterfacePair : (i,j), IDs, partenaires, régions et orientations | Graphe candidat à rééprouver ; 4/5 blocs non qualifiés pour le conique |
| `PlanarPhysicalSection` | MeshNodes `[N,2]` FP64 en N, plongement visuel (ξ,0,z) explicitement plan | Future immersion physique à décider après G-FORM, aucun `[N,3]` ajouté aujourd’hui |
| `TransformAndBasis` | FrameChain S→N→W et inverse, bases globales/extrusion distinctes, modèle/version | Future application/bases variables et dérivées à dériver avant contrat |
| `PlanarMetricEvidence` | Aires/longueurs/centroïdes et métriques euclidiennes du chemin plan | Métriques et variations d’équations coniques non figées avant T088 |

Les en-têtes courants peuvent être simples types/vues ; aucune hiérarchie abstraite générale
d’équations n’est nécessaire. La représentation 1.2 porte `planar-euclidean-v1`, mode demandé,
source line of flight, espace logique, plan N et versions de transformations/métriques.
Les tableaux de nœuds/indices restent ceux du chemin plan ; la description empêche de les
présenter comme métriques coniques. Un futur format physique distinct requerra sa propre
version et son mapping, après décision.

Les champs ont `association=point|cell|face`, `tuple_count`, `components`, repère/unités et
origine. `quality/cells.jsonl` porte des mesures aux cellules, `wall_faces.jsonl` aux faces,
les coordonnées sont aux sommets ; une interpolation vers les sommets est étiquetée et
n’altère pas l’association d’origine. Les indices logiques ont leur association distincte.
Les futurs Cp pariétaux dans S et p dans un plan physique ne sont pas deux noms d’un même
champ. Aucune entité de solution/champ p n’est produite par 001.

La lecture legacy 1.0 est interprétée par la règle `legacy-planar-extrusion-1-to-1.2` et
archivée comme provenance, avec mode `planar_25d` (y compris Λ=0, limite admise). Les nouvelles
sorties sont 1.2 ; aucun champ métrique futur n’est deviné depuis des coordonnées ou un nom.
