# Résultats, exports et reproduction

## Manifeste canonique

`manifest.json` contient au minimum les champs suivants ; leur présence est obligatoire dans un résultat final, même si certaines valeurs sont `null` après échec :

| Champ | Type / contenu |
| --- | --- |
| `schema_version` | Chaîne `1.2` pour nouvelles sorties ; lecture `1.0` explicitement migrée |
| `run_id`, `operation`, `status`, `valid`, `last_completed_stage` | Identité et état ; cohérents avec `quality/report.json` |
| `created_at_utc` | Date UTC ISO 8601, informative, exclue de la comparaison de reproductibilité |
| `generator` | Version sémantique, identifiant de révision si disponible, empreinte du binaire, mode compilation |
| `environment` | Système, architecture, compilateur/version, bibliothèque standard, bibliothèques liées, options numériques, nombre de threads |
| `source` | Chemin archivé relatif, digest SHA-256, unités déclarées, nombre d'éléments/points |
| `requested`, `resolved`, `geometry`, `quality` | Descripteurs chemin/digest vers les documents correspondants |
| `frames` | Références S/N/W, unités, Xref, c originale, Λ, translation/échelle globales et ordre des transformations |
| `topology` | Nom/version `thin` ou `thick`, schéma de blocs, paramètres de partition et version de générateur |
| `arrays` | Descripteurs des nœuds/indices avec type, dimensions, offsets, tailles et digest |
| `files` | Liste exhaustive des artefacts : rôle, chemin relatif, taille en octets, digest, statut de validité |
| `family` | `null` ou identité/type/niveau de l'étude, règles, reconstruction partagée et empreinte |

Les libellés définitifs de topologie et leur graphe sont ceux de [topology.md](../topology.md), et non des heuristiques du lecteur. `schema_version` identifie le format du bundle ; `topology.version` identifie son interprétation géométrique. Tous les fichiers référencés existent et sont vérifiés avant le marquage final valide. Un chemin contenant `..`, absolu ou sortant du bundle via un lien symbolique est refusé à la relecture.

## Tableaux et connectivité

- `nodes.f64le` : suite de N paires IEEE-754 binary64 `(ξ,z)` dans N, little endian. Le numéro de paire est l'ID global du nœud. `shape=[N,2]`, sans padding.
- `block_nodes.u64le` : tableaux d'IDs concaténés par ID de bloc croissant. Pour un bloc de `ni × nj` cellules : `shape=[nj+1,ni+1]`, i le plus rapide. Le manifeste donne l'offset et le nombre de valeurs de chaque bloc.
- `topology.json` : blocs et leur rôle, côtés indexés `i_min/i_max/j_min/j_max`, plages d'indices, frontières physiques et interfaces avec partenaires et orientation. Les directions logiques définies par la topologie imposent un Jacobien positif ; toute carte négative est refusée. Aucun retournement de connectivité ou champ de parité négative ne peut masquer une carte inversée.
- Ordre canonique positif d'une cellule : quatre nœuds du contour dans le sens trigonométrique en `(ξ,z)`. Les faces sont orientées selon ce contour ; leurs normales de sortie sont déduites, pas enregistrées avec une seconde convention indépendante.
- L'interface référence une unique séquence globale de nœuds, parcourue dans le sens canonique par un côté, dans le même sens ou le sens inverse par son partenaire. Même ID global signifie exactement la même coordonnée ; une comparaison à 10⁻¹² c est un contrôle d'intégrité additionnel, jamais une procédure de collage approximatif.

Le validateur contrôle dimensions, overflow, IDs dans les bornes, absence de cellules répétées ou superposées, couverture des côtés et bijections. Il contrôle aussi que les faces de deux blocs ne partagent pas une interface contradictoire. Aucun fichier de résolution ou champ d'écoulement n'est prévu.

## VTK sans dépendance d'exécution à VTK

Un export comporte `mesh.vtm` et un `block-XXX.vts` par bloc. Chaque VTS est un `StructuredGrid` XML 2D, d'étendue `0 ni 0 nj 0 0`, avec trois coordonnées `(ξ,0,z)` pour la visualisation ; il n'y a ni extrusion physique ni cellule volumique 3D. Les valeurs sont initialement écrites en ASCII avec `max_digits10` pour les binary64. Le nom du repère et ses unités figurent dans `FieldData` et le manifeste d'export.

`PointData` porte `global_node_id` ; `CellData` porte `global_cell_id`, `block_id`, les métriques sélectionnées et `valid`. Des nœuds peuvent être dupliqués par le format entre blocs, mais leur `global_node_id` et leurs coordonnées proviennent du même stockage canonique. `FieldData` indique également version du schéma, validité et référence aux métadonnées d'interface. La conservation complète des raccords reste dans le bundle canonique accompagnant l'export.

Le VTM référence les VTS par chemins relatifs et noms de blocs. Une relecture de qualification utilise un lecteur VTK indépendant uniquement comme dépendance de test ; le produit écrit les formats documentés lui-même. Une relecture par le propre exporteur seul ne constitue pas la preuve d'interopérabilité. Le contrôle compare nombre de cellules/nœuds par bloc, IDs, coordonnées et métriques.

## Figures

Les cinq SVG prévus sont `profile.svg`, `leading-edge.svg`, `trailing-edge.svg`, `wake.svg`, `global-mesh.svg`. Ils incluent titre, statut, unités, repère et légende. Les contours source, reconstruit et frontière maillée portent des styles distincts ; la présence d'un seul ou deux contours pendant `check` est explicitée. Les points/lèvres protégés, segments de culot ajoutés et défauts sont identifiables par ID. Les vues BA/BF/sillage portent une fenêtre de zoom archivée. Les défauts critiques utilisent des symboles lisibles sans dépendre exclusivement de la couleur.

Les SVG sont produits par un sérialiseur C++ minimal : segments, polylignes et texte échappé ; aucune dépendance à un moteur graphique ou Python pour le parcours autonome. Les courbes sont subdivisées avec le même contrôle de représentation géométrique que le reste des sorties ; leur figure ne sert pas d'évaluateur numérique.

## Reproduction et intégrité

Le dossier suffit à reproduire la génération avec la même version disponible localement, sans rechercher les fichiers d'origine. La commande `reproduce` vérifie les digests, recharge source et configuration résolue, puis crée un nouveau dossier. Elle n'utilise pas les nœuds existants comme entrée du mailleur. Les paramètres demandés sont conservés pour traçabilité ; la résolution déjà archivée est vérifiée, et une résolution différente entraîne un refus ou un résultat explicitement non comparable.

Les champs comparés sont : reconstruction et caractéristiques, IDs/connectivité/topologie identiques, paramètres résolus identiques, erreurs maximales des coordonnées ≤10⁻¹² c, mêmes décisions de qualité et mêmes contrôles couverts. Dates, chemins du dossier et run_id peuvent changer. Les versions/environnements différents sont signalés ; aucune identité bit à bit interplateforme n'est promise. Une divergence proche d'une limite doit être rapportée comme divergence de décision, pas masquée par une tolérance de reproduction.

Les empreintes utilisent SHA-256 via OpenSSL libcrypto ; elles prouvent l'identité des octets, sans revendication d'authentification de leur auteur. Les bibliothèques/version et licences sont inventoriées avec la version du générateur. Le profil qualité est archivé intégralement et son digest inclut les limites ; aucune résolution implicite d'un profil « courant » ne peut changer le verdict au rejeu.

Les contrats exécutables de forme sont [manifest.schema.json](manifest.schema.json) et [topology.schema.json](topology.schema.json). Le manifeste exige les références demandée/résolue et leurs digests, les repères et leur ordre, et les descripteurs typés des tableaux. Le document topologique définit les tableaux par bloc et les courbes d'interface partagées ; les contrôles sémantiques vérifient `shape=[nj+1,ni+1]`, couverture et correspondances. L'ID de nœud global, et non une proximité géométrique, définit le partage.

## VTP des contours et frontières — ajout du 2026-10-31

Le .vtp est du XML PolyData ASCII avec lignes (pas de faux quadrilatères pour des contours).
`contours-source.vtp` porte S ; `contours-reconstructed.vtp` porte N ; `boundaries.vtp` porte N
et les parois, champ lointain, sortie et interfaces. Une sortie W demande un fichier séparé,
explicitement nommé et adimensionnel. Une collection ne mélange jamais des repères sans
les déclarer ; toutes les positions visibles sont `(première_coordonnée,0,seconde_coordonnée)`.
Les blocs restent exclusivement des StructuredGrid VTS dans `mesh.vtm`.

Une ligne élémentaire par segment facilite la sélection des rôles et les liens de diagnostics.
`CellData` contient `export_object_id`, `role_code`, `representation_code`, `element_id`,
`source_segment_id`, `boundary_id`, `interface_id`, `face_id` et `valid`, selon applicabilité.
Les ID absents sont `Int64=-1` (ou `UInt64=0` si le tableau est non signé) avec masque `has_<id>=0` ; les ID présents utilisent Int64
si représentables, sinon UInt64 et un masque, sans troncature. `PointData` fournit les ID
source ou global_node_id et leurs masques ; aucun ID global n’est inventé pour une spline.
Le nombre d’IDs de diagnostic résolus est comparé au nombre d’IDs applicables, pas aux sentinelles.
Une interface est exportée une seule fois depuis la courbe canonique avec ses deux partenaires.

Codes de rôle : 1 extrados (`wall_upper`), 2 intrados (`wall_lower`), 3 culot (`wall_base`),
4 interface, 5 champ lointain, 6 sortie. Représentations : 1 source, 2 reconstruction,
3 frontière maillée, 4 interface canonique. Le culot absent du cas fin n’est jamais remplacé
par une ligne nulle. Le culot ajouté explicitement porte son origine distincte dans l’index.
`FieldData` conserve dictionnaires de codes, `frame`, `units`, `schema_version`, `valid`,
`run_id`, chaîne de transformations et chemin relatif de `export-index.json`.
Cet index associe export_object_id à provenance, frontière/interface et localisations du rapport.
`boundary_id` et `export_object_id` sont autorisés dans report.schema.json.

`inspection.vtm` peut regrouper seulement les VTP de même repère N ; `mesh.vtm` conserve
les blocs VTS seuls. Tous les chemins VTM, index et manifestes sont relatifs au fichier qui
les référence, contenus dans l’ensemble exporté. Le manifeste canonique inventorie VTP,
index, VTM et VTS avec leurs empreintes ; aucun chemin absolu n’est nécessaire après déplacement.
Un parcours `check` exporte les contours disponibles sans inventer de frontière maillée.
Un export invalide sur demande explicite conserve `valid=0`/`INVALID` partout.

La qualification impose un lecteur VTK ou PyVista indépendant : relire les trois formats,
comparer types, dimensions/comptes, rôles, IDs, correspondances et coordonnées ≤10⁻¹² c,
puis déplacer l’ensemble et le relire. L’absence du lecteur bloque la qualification et n’est
pas un succès par skip. Une vérification d’ouverture ParaView sur fin/épais et contrôle seul
conserve version, commandes/manipulations, logs et captures globales/BA/lèvres avec sélection
des rôles. Le fichier d’état ParaView, s’il est archivé, utilise des chemins relatifs.
Le lecteur, PyVista éventuel et ParaView restent des dépendances de tests, jamais d’exécution.

## R2 — représentation physique, logique et association des champs

Le bundle 1.2 distingue le contour line of flight S, les indices/blocs/raccords logiques,
la représentation plane physique N, les transformations/bases et métriques euclidiennes.
`representation` satisfait representation.schema.json ; `topology_space=logical_i_j` ne
prétend pas décrire une surface physique. Le tableau actuel reste `[N,2]` Float64 `(ξ,z)` ;
le plongement VTK `(ξ,0,z)` est étiqueté **plane_N / planar-euclidean-v1**, pas une extrusion
physique 3D ni une représentation conique. Les coordonnées ASCII conservent max_digits10 et
le DataArray déclare Float64. Une future surface courbe doit conserver son immersion
physique, son mapping et son étiquette ; une vue logique/dépliée distincte ne la remplace pas.

`arrays[].association` vaut point pour global_nodes, logical_node pour block_node_ids,
interface_node pour interface_node_ids. Chaque champ du tableau `fields` porte nom, fichier,
association point/cell/face, nombre de tuples/composantes, repère, unités et origine.
Une interpolation pour affichage porte origin=interpolated_for_visualization et
source_association ; elle ne remplace pas les données mesurées. Le contrôle C++ compare
les nombres de tuples aux entités canoniques, pas seulement aux dimensions XML déclarées.
`quality/cells.jsonl` et ses métriques sont aux cellules ; `wall_faces.jsonl` aux faces.
PointData et CellData restent distincts lors de la relecture indépendante, sans conversion
implicite cellule→sommet. Les métadonnées d’association figurent dans le manifeste et l’index.

Un lecteur 1.0 conserve les octets et étiquette sa migration plane avec la règle archivée ;
il ne déduit jamais un mode conique d’une troisième coordonnée ou d’un nom. Le modèle
métrique conique détaillé reste bloqué par G-FORM et sera versionné après T088/T089, sans
modifier par anticipation le layout du chemin plan. VTK/PyVista et ParaView restent obligatoires
pour qualifier les exports, uniquement en environnement de tests.

## R3 — fichiers partagés et verdicts propres aux opérations

Le bundle unitaire autonome conserve son contrat manifest.valid=report.valid. Plusieurs
points peuvent partager **les octets générés** via mesh_artifact, sans partager un verdict
qualité unique ; voir [result-identities.md](result-identities.md). Les données canoniques
nœuds/topologie/source et génération sont immuables. Audits et estimations possèdent chacun
une clé/version, des paramètres et des fichiers immuables ; cells/wall_faces de qualité ne
sont pas écrasés par les estimations physiques. `result_records` inventorie leurs copies
locales avec clé, version, chemin, taille et digest dans chaque manifeste/rapport 1.2.
Une matérialisation autonome inclut ses copies locales et n'exige aucun chemin externe.
Les exports portent le contexte d'audit/point et le statut de cette opération, pas un valid
universel sur le maillage. Des audits contradictoires peuvent donc coexister sans mutation.
Dans le rapport agrégé d'un point à plusieurs audits, quality_profile/profile_sha256 sont
null ; result_records référence chacun des rapports d'audit, qui nomme son profil exact.
Avec un seul audit ces champs peuvent désigner ce profil. La politique de génération reste
dans generation-resolved.json, indépendamment du profil nommé dans un rapport d'audit.

`physical_scale` dans manifeste et rapport (null seulement avant résolution impossible)
archive la politique source-scale-v1 et ses mesures ; resolved.json porte le même objet.
[physical-scale.md](physical-scale.md) fixe mètres/adimensionnel, tolérance dimensionnée
et conversion unique W→N→m. `quality/report.json.y_plus` ne contient aucune min/max/valeur
si le statut est unavailable/not_requested/failed. Les instantanés d'estimation conservent
ν, u_tau, distance/version, échelle et hypothèses ; ils ne sont pas des champs de solution.
