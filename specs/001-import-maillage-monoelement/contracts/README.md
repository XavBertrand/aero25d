# Contrats d'interface v1

Ces contrats décrivent le comportement à construire. Ils ne sont pas une bibliothèque ou un exécutable disponible. Le schéma majeur `1` est obligatoire ; une clé inconnue est refusée, sauf dans un champ `extensions` explicitement réservé. Les fichiers JSON sont UTF-8, sans NaN/Infinity, avec nombres décimaux à point.

- [configuration.schema.json](configuration.schema.json) : configuration demandée ; les obligations dépendant d'une géométrie sont précisées ci-dessous.
- [report.schema.json](report.schema.json) : synthèse des contrôles, intervalles et diagnostics.
- [manifest.schema.json](manifest.schema.json) et [topology.schema.json](topology.schema.json) : résultats persistants et reproduction.
- [quality-profile.schema.json](quality-profile.schema.json) et [quality-profile-Q1.json](quality-profile-Q1.json) : profil initial figé par identifiant et limites proposées.
- [configuration.example.json](configuration.example.json) : configuration illustrative, sans résultat de génération revendiqué.
- [cli-python.md](cli-python.md) : commandes, Python, erreurs, estimation facultative de y+.
- [bundle.md](bundle.md) : formats persistants, export, reproduction et limites.
- [Modèle de données](../data-model.md) : sémantique des entités et transformations.

## Configuration et résolution

`input` est un chemin de fichier relatif au fichier de configuration, jamais au répertoire courant implicite. `units` est `m` ou `dimensionless` ; [physical-scale.md](physical-scale.md) fixe l’échelle originale : mètres ⇒ k=1 et indication facultative contrôlée à 1e-10 relatif ; adimensionnel ⇒ k=physical_chord_m/c_input si fourni. Aucune mise à l’échelle des nœuds. Sans échelle, la géométrie reste autorisée et y+ indisponible. Toutes les longueurs de configuration `*_over_c` sont rapportées à la corde originale c. Le champ `normalization` décrit la transformation globale N→W ; `translation` et `scale` sont dans les unités du fichier source, avec valeurs explicites si `mode=explicit`. `mode=chord` utilise s=c et une translation globale documentée dans la configuration résolue ; aucune normalisation par élément.

`mesh.topology=auto` ne choisit entre fin et épais que sur une identification géométrique non ambiguë ; une très petite épaisseur ne devient pas une pointe par seuil implicite. `base_intervals` est requis pour le culot épais et doit être `null` pour le cas fin. `surface_intervals` est le nombre total d'intervalles hors culot ; sa répartition entre blocs suit les distributions et contraintes de [topology.md](../topology.md). Les comptes de raccords sont dérivés par un solveur de contraintes entières déterministe ; un compte utilisateur incompatible est refusé, pas corrigé.

`normal_layers`, `first_height_over_c` et `normal_growth` imposent la couche proche paroi. `outer_layers` contrôle le raccordement extérieur ; il n'est pas confondu avec le nombre de couches proches. Si l'épaisseur totale et le domaine sont incompatibles, un diagnostic indique les valeurs demandées et la borne géométrique rencontrée. `leading_edge_concentration` et `trailing_edge_concentration` sont des facteurs sans dimension ≥1 du moniteur tangentiel documenté dans [geometry-verification.md](../geometry-verification.md) ; ils ne sont pas des angles.

Le profil qualité est un nom/version figé ou un document explicite complet, dont le digest et toutes les limites résolues sont archivés. `quality_profile` fixe la politique utilisée pendant la génération et l’audit par défaut ; `audit_profiles` peut demander plusieurs audits en lecture seule. [result-identities.md](result-identities.md) définit leurs clés et résultats indépendants ; la politique influençant le lissage appartient à mesh_key. `quality_profile` ne permet pas de supprimer les contrôles inconditionnels de topologie, positivité ou intégrité. L'absence de données d'écoulement (`flow_estimate=null`) autorise tous les parcours géométriques ; le rapport indique `y_plus.status=not_requested`.

Les champs obligatoires résolus, même lorsqu'un champ a un défaut documenté dans le schéma, sont tous écrits dans `resolved.json`. `requested.json` conserve les omissions utilisateur. Les champs dérivés contiennent leur provenance : par exemple `derived_from: family.rule.v1`, `profile: Q1`. Une incompatibilité est refusée avant génération si détectable à ce stade. La résolution ne peut pas changer un paramètre demandé pour satisfaire les tolérances.

## Extensions et compatibilité

Une modification de sens, d'unité, d'ordre de tableau ou de topologie identifiée incrémente le schéma majeur. Une extension facultative additive incrémente la version mineure. Un lecteur refuse les majeures inconnues ; il ne déduit pas la version de l'extension du fichier. Les schémas JSON couvrent la forme ; les invariants de géométrie, raccord, unités, distribution et intégrité sont contrôlés par la bibliothèque C++.

## Paramètres complémentaires Q1/F1

`surface_intervals=Nu+Nl` ; `upper_surface_intervals` fixe Nu et le reste fixe Nl. Si omis, la répartition dérivée déterministe est archivée. `normal_layers=Nbl` compte les couches de la bande de paroi ; `outer_layers` compte les couches supplémentaires, d'où `Nnormal=Nbl+outer_layers`. La répartition de ces dernières entre transition au cœur et tampon extérieur est résolue et figée pour la famille. `base_first_height_over_c` surcharge explicitement h1 au culot, sinon h1 est partagé. `core_height_over_c` définit Hcore (défaut 0,5) ; `near_wake_length_over_c` définit Lnear (défaut 2). `near_wake_intervals` est inclus dans `wake_intervals` ; il ne s'ajoute pas à celui-ci. Exiger `0<near_wake_intervals<wake_intervals` et `Lnear<Lwake` ; les contradictions sont localisées.

Les paramètres `mesh.density` donnent poids de courbure et largeurs de concentration ; les amplitudes A_BA/A_BF du moniteur sont `leading_edge_concentration−1` et `trailing_edge_concentration−1` (un facteur 1 ajoute donc une amplitude nulle). Leurs fonctions exactes sont décrites dans geometry-verification.md et les valeurs résolues sont archivées. Le profil de qualité initial est [Q1](quality-profile-Q1.json), conforme à [quality-profile.schema.json](quality-profile.schema.json).

Une famille globale prend les paramètres du niveau zéro et applique F1 de mesh-quality.md : r=2^ℓ, tous les comptes multipliés par r, distributions cumulatives et Hbl fixés, `gℓ=g0^(1/r)` et `h1ℓ=h10*expm1(log(g0)/r)/expm1(log(g0))` (limite h10/r pour g0=1). Il est interdit de remplacer cette règle par h10/r lorsque g0≠1. Des surcharges par niveau définissent une étude de diagnostic, pas F1. `resolved.json` conserve les comptes de chaque segment, les valeurs dérivées et leur règle.

Les métriques certifiées portent `lower_bound`, `upper_bound` et `comparator`, en plus d'une valeur représentative. Un succès à seuil supérieur exige `upper_bound≤limit` (ou strictement < selon comparateur) ; à seuil inférieur, `lower_bound≥limit` (ou >). Un intervalle chevauchant la limite est non certifiable, donc ne peut être converti en succès par arrondi.

Le champ `frame=N` du profil Q1 désigne le repère par défaut des métriques du maillage. Il ne supprime pas les contrôles géométriques dans S, N et W : les métriques de fidélité/rayon/angle sont répétées par repère avec la référence attendue de ce repère ; pour W, les budgets dimensionnels sont convertis par s. Chaque entrée de rapport conserve son propre `frame`. Une erreur dans S ne peut donc pas être masquée par une compression dans N.

Les chaînes `condition` de Q1 sont des identifiants de prédicats enregistrés, pas du code ni des expressions évaluées arbitrairement. Les deux conditions initiales reconnues sont exactement `inside_boundary_layer AND principal_axis_to_wall_tangent_deg>15` et `outside_boundary_layer` ; une condition inconnue est refusée lors du chargement du profil. La cohérence sémantique impose notamment `manifest.valid=report.valid`, les mêmes opération/statut/stade, un résultat valide sans défaut critique, et des références/digests correspondant aux fichiers réels ; un JSON conforme au schéma ne suffit pas à prouver ces propriétés.

## Compléments du 2026-10-31

- [campaign.md](campaign.md), [campaign.schema.json](campaign.schema.json) et
  [campaign-state.schema.json](campaign-state.schema.json) : demande, résolution et état
  progressif, contrôle sans génération, reprise et clés effectives. Enveloppes distinctes
  des rapports/manifeste unitaires ; leur version 1.2 reste propre à chaque type, avec lecture 1.0 explicitement migrée.
- [campaign.example.json](campaign.example.json) et [campaign.example.yaml](campaign.example.yaml) :
  demandes équivalentes, chemins vers les futures fixtures, sans résultat d’exécution.
- [bundle.md](bundle.md) : VTP ajouté aux VTM/VTS ; rôles, IDs, repères, unités et relecture
  VTK/PyVista obligatoire en qualification, ainsi qu’ouverture ParaView.

Les indications `leading_edge_point`, `upper_lip_point`, `lower_lip_point` acceptent un indice
entier source ou `{segment_id,t}` avec t∈[0,1], conformément à geometry-verification.md.
La localisation d’un diagnostic peut inclure boundary_id/export_object_id/representation.
Ces ajouts interviennent avant toute réalisation et n’allègent aucune règle existante.

Les références croisées des schémas utilisent les URN `$id` enregistrés dans un registre
local. Le validateur ne doit pas tenter de les résoudre par réseau. La validation documentaire
emploie les ressources locales exactes. Avant première réalisation, le jeu initial est désormais révisé en `1.2` par R2 ; dès le premier format publié, la règle majeure/mineure ci-dessus
s’applique aux évolutions. Aucun bundle `1.0` antérieurement distribué n’est allégué.

## R2 — versions, modes et lecteur JSON/YAML partagé

La [roadmap produit](../../../docs/product-roadmap.md) couvre 2D/2.5D/2.75D ; les contrats
présents sont ceux du **produit plan**, pas le contrat métrique conique futur.
Les configurations et campagnes acceptent JSON **ou YAML** avec le même schéma versionné
par type d’opération. YAML utilise les règles de scalaires/séquences/mappings définies dans
campaign.md ; un lecteur C++ partagé les applique dès T025, et T067 le réutilise.
Les chemins restent relatifs au document propriétaire, y compris une configuration YAML.
[configuration.example.yaml](configuration.example.yaml) équivaut à l’exemple JSON.

Les nouvelles demandes/sorties portent 1.2 ; Q1 reste à son schéma 1.0, inchangé. Les lectures
1.0 préservent l’original et archivent la règle `legacy-planar-extrusion-1-to-1.2` : mode
`planar_25d`, représentation plane, champs associés d’après leurs rôles canoniques connus.
Un rôle/champ inconnu n’est pas deviné. Les différences de règle/version invalident la reprise
certifiée ; une lecture legacy n’est pas une autorisation de réutiliser aveuglément le résultat.

En configuration 1.2, `geometry_mode` est obligatoire : `planar_2d` impose sweep_deg=0 ;
`planar_25d` admet [0°,50°] et son cas limite 0°. Ces noms identifient les géométries, pas
un solveur déjà présent. `conical_275d`/taper_ratio/pentes actives et Mach/Reynolds/température
sont non supportés : diagnostic explicite et pointeur du paramètre avant exécution, aucune
ignorance silencieuse. Les clés inconnues restent refusées. L’estimation flow_estimate déjà
prévue n’est pas un cas de vol résolu et conserve ses statuts d’estimation.

[representation.schema.json](representation.schema.json) décrit uniquement la représentation
plane explicite. Manifeste/topologie/rapport 1.2 exposent représentation, espace logique et
associations des tableaux ; aucun contrat métrique conique détaillé n’est publié. Les bornes
et invariants sémantiques restent contrôlés par C++, au-delà de la seule forme JSON Schema.

## R3 — corrections U1/I1/U2/U3/I2 avant première publication

Le jeu initial **non publié** est révisé en 1.2 (1.0/1.1 conservés en lecture avec migration
archivée, sans reprise aveugle). Les références URN majeures restent 1 ; chaque schéma
énumère ses versions. Q1 reste inchangé en 1.0. La migration de demande 1.1 vers 1.2 archive
`request-1.1-to-1.2-r3`, résout audit_profiles=[quality_profile] si omis et source-scale-v1.
Le champ inutilisé curvature_regularization_over_c est retiré avant publication : refus
REMOVED_PARAMETER/code 2 dans toutes les versions, sans l'ignorer dans une migration.
Les migrations ne réécrivent pas les octets ni les verdicts antérieurs.

Nouvelles annexes normatives, sans seconde spécification :
[result-identities.md](result-identities.md), [physical-scale.md](physical-scale.md),
[campaign-artifact.schema.json](campaign-artifact.schema.json),
[physical-scale.schema.json](physical-scale.schema.json).
[scale-audit-cases.example.json](scale-audit-cases.example.json) décrit des essais prévus,
pas une demande unitaire ni des résultats mesurés. Les contraintes d'égalité de clés,
d'unités, digests, verdicts et références restent des validations sémantiques à implémenter.
