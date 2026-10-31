# Commande autonome et interface Python

## Commandes

Nom d'exécutable prévu : `aero25d-mesh`. Toute commande accepte `--help`, `--out DIR` et `--replace` (refus d'écrasement par défaut). `--json` impose une unique synthèse JSON sur stdout, sans préambule ; les messages de progression et diagnostics lisibles vont sur stderr. Le code de sortie est celui du rapport. La bibliothèque ne dépend pas de cette présentation.

| Parcours | Commande contractuelle | Résultat |
| --- | --- | --- |
| Charger, contrôler, reconstruire et visualiser | `aero25d-mesh check --config config.json --out check --json` | Source, géométrie, repères, diagnostics, figures de contours ; aucun maillage requis |
| Générer et contrôler | `aero25d-mesh mesh --config config.json --out run --json` | Dossier canonique complet ; rapport qualité ; cinq vues |
| Réinspecter la qualité | `aero25d-mesh quality --bundle run --audit-profile Q1 --out audit --json` | Validation d'intégrité puis mesures exhaustives et rapport ; entrée immuable |
| Exporter | `aero25d-mesh export --bundle run --format vtk --out vtk-out --json` | VTP/VTM/VTS et métadonnées de correspondance ; entrée immuable |
| Famille globale | `aero25d-mesh family --config config.json --levels 0,1,2 --out family --json` | Reconstruction commune, niveaux et comparaison des distributions |
| Variation ciblée | `aero25d-mesh family --config config.json --study parameter --vary mesh.first_height_over_c --values 0.00001,0.000005 --out height-study --json` | Étude de diagnostic, jamais étiquetée convergence globale |
| Variation du domaine | `aero25d-mesh family --config config.json --study domain --values 10,20 --out domain-study --json` | Même géométrie et mêmes réglages intérieurs ; frontière éloignée seule modifiée |
| Reproduire | `aero25d-mesh reproduce --bundle run --out replay --json` | Nouvelle génération et comparaison des données, connectivités et décisions |

`check` fait le contrôle géométrique et la reconstruction avec la même configuration que le maillage. Un fichier multiélément peut être contrôlé et transformé ; il ne produit jamais un maillage partiel. `quality` utilise par défaut le profil archivé ; `--audit-profile NOM_OU_FICHIER` (Python `client.quality(..., audit_profile=...)`) demande un nouvel audit immuable sous sa propre audit_key. Le profil de génération et le verdict initial restent archivés. Même un audit passé ne remplace pas un échec sous d'autres paramètres. `export` refuse par défaut un bundle invalide ; `--diagnostic-invalid` permet uniquement un export clairement marqué `valid=0` et `INVALID` dans les titres/métadonnées.

Les chemins passés en arguments sont résolus relativement au répertoire courant ; les références internes à une configuration sont relatives à celle-ci. Les chemins archivés dans le bundle sont relatifs au bundle et ne peuvent pas en sortir. Une commande n'exécute jamais de shell ou de programme utilisateur issu d'une configuration.

## Codes de sortie et diagnostics

| Code | Signification | Exemples de codes structurés |
| --- | --- | --- |
| 0 | Opération réussie et contrôles requis satisfaits | `CHECK_PASSED`, `MESH_VALID` |
| 2 | Configuration/arguments invalides | `CONFIG_SCHEMA`, `PARAMETER_INCOMPATIBLE`, `UNKNOWN_KEY` |
| 3 | Données géométriques invalides | `COUNT_MISMATCH`, `NONFINITE_COORDINATE`, `SELF_INTERSECTION`, `ZERO_CHORD` |
| 4 | Indication ou précision source nécessaire | `AMBIGUOUS_FEATURE`, `CLOSURE_CONFIRMATION_REQUIRED`, `GEOMETRY_NOT_CERTIFIABLE` |
| 5 | Capacité hors périmètre/topologie non admissible | `MULTIELEMENT_UNSUPPORTED`, `TOPOLOGY_NOT_ADMISSIBLE` |
| 6 | Génération ou qualité en échec | `GENERATION_INVERTED`, `QUALITY_CRITICAL`, `SMOOTHING_NOT_CONVERGED` |
| 7 | Entrée/sortie ou intégrité | `IO_ERROR`, `DIGEST_MISMATCH`, `BUNDLE_SCHEMA`, `OUTPUT_EXISTS` |
| 8 | Limite de ressources contrôlée | `CELL_LIMIT`, `MEMORY_LIMIT`, `SIZE_OVERFLOW` |
| 70 | Erreur interne imprévue | `INTERNAL_ERROR` |

Une opération peut réussir avec avertissements, mais jamais avec défaut critique. Un échec conserve autant que possible la source, les paramètres, les diagnostics et les figures disponibles dans un dossier `.invalid-<run_id>`. `last_completed_stage` évite de présenter des vérifications non exécutées comme satisfaites. Si le disque ou le chemin ne permet aucun fichier, le rapport minimal JSON est fourni sur stdout et la cause sur stderr. Une interruption brutale extérieure au processus ne peut garantir un rapport ; aucun manifeste valide n'est alors publié.

Le statut `valid` est relatif à l'opération : un contrôle géométrique valide ne signifie pas qu'un maillage existe. `operation`, `last_completed_stage` et l'inventaire des artefacts rendent cette différence explicite. Les codes sont stables ; le texte du message n'est pas une API de reconnaissance d'erreur.

## Python initial

Paquet prévu : `aero25d_mesh`, Python ≥3.11. Il utilise uniquement la bibliothèque standard pour valider les arguments simples, créer des répertoires temporaires contrôlés, appeler le binaire par liste d'arguments avec `shell=False`, lire JSON et exposer les résultats. Il n'implémente aucune géométrie, aucune distribution et aucun contrôle qualité numérique.

```python
from aero25d_mesh import Client, MeshError

client = Client(executable="/absolute/path/aero25d-mesh")
checked = client.check(config="tests/data/configs/V01.json", out="work/check")
run = client.mesh(config="tests/data/configs/V01.json", out="work/run")
report = client.quality(bundle=run.bundle, out="work/audit")
export = client.export(bundle=run.bundle, format="vtk", out="work/vtk")
family = client.family(config="tests/data/configs/V01.json", levels=[0, 1, 2], out="work/family")
replay = client.reproduce(bundle=run.bundle, out="work/replay")
```

Chaque méthode retourne un `Result` contenant `report: dict`, `bundle: pathlib.Path | None`, `artifacts: dict[str, pathlib.Path]`, `exit_code: int`, `stdout`, `stderr`. Le rapport est chargé sans perte ; les grands tableaux sont exposés par chemins et lecteurs séquentiels (`iter_cell_metrics`, `iter_wall_faces`) plutôt que copiés dans chaque résultat. Les tableaux binaires peuvent être lus par `array`/`memoryview` ; NumPy n'est pas une dépendance obligatoire. Le client sait prendre un fichier JSON ou un dictionnaire de configuration ; un dictionnaire impose `base_dir` pour ses chemins relatifs, évitant un changement implicite de résolution.

Pour les codes non nuls, il lève `MeshError` avec `exit_code`, `report`, `diagnostic_bundle`, `stderr`. `raise_on_error=False` retourne le même `Result` pour inspection ; il ne modifie pas le résultat C++. Les sous-classes `ConfigurationError`, `GeometryError`, `NeedsInputError`, `UnsupportedError`, `GenerationError`, `IntegrityError`, `ResourceError` suivent les codes. Un binaire absent, une sortie JSON mal formée ou un signal externe sont des erreurs de transport distinctes (`TransportError`), sans inventer de rapport qualité.

### Comparaison avec une liaison native

L'appel structuré de l'exécutable satisfait l'ensemble des parcours, partage exactement le moteur C++, isole les pannes et évite une chaîne ABI/compilation Python supplémentaire. Son coût de lancement et la lecture des fichiers restent à mesurer ; aucune performance n'est présumée. Une liaison native permettrait accès direct aux tableaux et appels répétés moins coûteux mais ajouterait empaquetage, durée de vie des objets et compatibilité ABI. Elle est différée jusqu'à mesure d'un besoin. La bibliothèque C++ ne prend aucune dépendance Python, afin de garder cette évolution possible sans réécrire les algorithmes.

## Estimation facultative de y+

Le parcours géométrique n'a besoin d'aucune donnée d'écoulement. Le mode initial `supplied_friction_velocity` demande une viscosité cinématique ν>0 et une vitesse de frottement estimée par l’utilisateur `u_tau>0`. L’échelle est résolue dans S selon [physical-scale.md](physical-scale.md) : mètres directement, ou k=L/c_input. Sans échelle, retour y_plus.status=unavailable avec PHYSICAL_SCALE_UNAVAILABLE, aucune valeur physique et géométrie autorisée. Il fournit, pour chaque face de paroi, `y_plus_estimate=u_tau*d_center_m/ν`. Le rapport conserve valeurs, unités, origine des données, distance réellement mesurée et hypothèse de `u_tau` uniforme si une unique valeur est fournie. Une valeur uniforme ne prédit pas le frottement local, le stagnation ou le culot.

Une corrélation de plaque plane n'est pas retenue initialement : elle ajouterait une hypothèse de frottement local inadaptée au BA, au culot et aux gradients de pression sans être nécessaire à FR-020. Le mode fourni par l'utilisateur résout entièrement ce contrat facultatif, sans décision de corrélation en suspens. Aucun Mach/Reynolds n'est déduit des seules coordonnées. Toute valeur calculée est `status=estimate`, jamais `validated` ou `guaranteed` ; l'objectif futur y+≈0,5–1 ne constitue pas un critère garanti de génération.

## Campagnes

Contrat complet et enveloppes distinctes : [campaign.md](campaign.md).

```text
aero25d-mesh campaign check --file study.yaml --out inventory --json
aero25d-mesh campaign run --file study.json --out study --json
aero25d-mesh campaign run --file study.json --out study --resume --json
```

La sortie JSON unique est l’enveloppe campaign-state.schema.json, sans la faire passer pour
un rapport unitaire report.schema.json. Les bundles des cas conservent leurs contrats unitaires
et leurs codes précis. Le code global est 0/2/6/7/8/70 suivant campaign.md.
`Client.campaign_check` et `Client.campaign_run(resume=...)` exposent le même état dans
`CampaignResult`, avec inventaire, summary et chemins des cas. `MeshError` conserve l’état
sur échec global. Ni CLI ni wrapper ne dupliquent la boucle de l’orchestrateur C++.

## R2 — modes, YAML et références produit

Les arguments `--config` et les méthodes de configuration Python acceptent JSON/YAML selon
configuration.schema.json ; le même lecteur partagé et base_dir s’appliquent. Aucune opération
solve/conical n’est ajoutée. Une demande conical_275d ou géométrie d’effilement active retourne
`UNSUPPORTED_GEOMETRY_MODE` (code unitaire 5) ; des champs de vol actifs donnent
`UNSUPPORTED_FLOW_INPUT` (code unitaire 5). Un paramètre inconnu reste code 2. En campagne,
une demande globale invalide donne code 2 ; les codes unitaires restent dans les lignes.
Ces diagnostics sont identiques depuis Python. Une extension inactive est signalée avec
son statut, sans résolution. planar_2d avec Λ≠0 donne PARAMETER_INCOMPATIBLE/code 2.

Le discriminant doit être contrôlé avant le rejet générique du schéma pour donner le motif
spécifique ; sa présence ne permet jamais de franchir une validation refusée. La lecture
legacy, les versions de transformation/représentation et les associations sont conservées
par Result et CampaignResult. Les références des futurs Cp, champs de pression et Cz/Cx/Cm
sont fixées dans la [roadmap](../../../docs/product-roadmap.md), sans résultat produit dans 001.

## R3 — résultats distincts et parité

Les configurations peuvent demander audit_profiles ; Result.report/result_records et
CampaignResult.points exposent les références typées mesh/audit/estimate/point (contrat
[result-identities.md](result-identities.md)), sans les fusionner en un verdict de maillage
universel. Les tableaux y+ restent propres à leur estimate_key, jamais ajoutés en écrasant
wall_faces d'un autre audit. `campaign run --resume --retry-failed` correspond exactement
à `Client.campaign_run(..., resume=True, retry_failed=True)` ; sans resume, code 2.
Un échec d'audit/estimation garde ses preuves et les références des autres couches.
`PHYSICAL_CHORD_MISMATCH`/`REMOVED_PARAMETER` donnent code 2 ;
`PHYSICAL_SCALE_UNCERTIFIED` donne code 4, avec mêmes pointeurs et contexte Python/CLI.
