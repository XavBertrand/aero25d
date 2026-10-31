# Campagnes de maillage — contrat v1

État : conception du 2026-10-31, FR-031 à FR-040 ; aucun exécuteur implémenté.
Les schémas [campaign.schema.json](campaign.schema.json) et
[campaign-state.schema.json](campaign-state.schema.json) fixent les transports.

## Demande et développement

Une campagne UTF-8 JSON ou YAML contient `schema_version=1.2`, `kind=mesh_campaign`,
un `campaign_id`, des `profiles` (ID, chemin, unités et indications), des `configurations`
(ID et chemin vers une configuration unitaire JSON ou YAML), `execution` et `expansion`.
La configuration unitaire fournit les paramètres usuels du mailleur ; son `input`, ses
`units`, ses `hints` et sa `sweep_deg` sont explicitement remplacés par le profil et le point.
Si le profil omet les indications, celles de la configuration restent appliquées.
Toute substitution est inscrite dans demandé/résolu ; une indication incompatible est refusée.

Deux modes exclusifs :

- `cartesian` : listes non vides `profile_ids`, `sweeps_deg`, `configuration_ids`, `levels`.
  Produit explicite dans cet ordre, le niveau variant le plus vite. Aucune dimension implicite. Les point_id dérivés sont `p000000` puis l’indice ordinal décimal sur au moins six chiffres ; la liste équivalente reprend ces ID si leur égalité est testée.
- `points` : liste ordonnée de `{id, profile_id, sweep_deg, configuration_id, level,
  incidence_deg?}`. Chaque point est autonome ; aucune combinaison supplémentaire n'est créée.

Les niveaux sont des entiers ≥0 appliquant **F1** aux paramètres du niveau zéro de la
configuration, par le même résolveur que `family`. Une configuration isolée utilise `level=0`.
L'absence de `incidence_deg` reprend la métadonnée de la configuration, défaut 0°.
En mode cartésien les incidences ne constituent pas une dimension de génération.
Les ID de profils/configurations/points sont uniques. Les références inconnues, listes vides,
valeurs non finies, flèches hors [0°,50°] et tailles de produit débordantes sont refusées.

Les chemins `profiles[].input` et `configurations[].config` sont relatifs au **fichier de
campagne** ; les autres chemins d'une configuration référencée restent relatifs à cette
configuration. Aucun chemin ne dépend du cwd du processus après lecture. Les chemins de
sortie sont relatifs à la racine de résultat ; un ensemble déplacé reste relisible.
Les sources/configurations/profils qualité référencés sont copiés avec empreintes dans le
résultat. Une provenance absolue peut être informative mais n'est jamais requise pour reprendre.

YAML est converti vers le même modèle JSON, puis validé par le même contrat : un seul document,
clés chaînes uniques, scalaires JSON finis, séquences et mappings. Les scalaires entre guillemets restent des chaînes, sans conversion implicite. Tags personnalisés,
ancres/alias, clés de fusion, dates typées implicitement et clés non chaînes sont refusés avec
ligne/colonne ; booléens `true`/`false`, `null` et nombres suivent la grammaire JSON, sinon
chaînes. Pas de remplacement de variables, commande ou include exécutable. Le choix d'un
sous-ensemble explicite empêche une différence de résolution entre les deux syntaxes.

## Contrôle sans génération

`campaign check --file study.yaml --out inventory --json` lit et contrôle les sources,
identifications, reconstructions et paramètres nécessaires à la résolution, développe les
points et calcule leurs clés ; il n'appelle ni `generate` ni l'export de blocs maillés.
Il produit `campaign-resolved.json`, `campaign-state.json` et `summary.csv`, avec statuts
`ready`, `needs_input`, `unsupported` ou `failed`, et les diagnostics par point.
Tous les points sont inventoriés malgré une entrée invalide ; une erreur de grammaire du
document ou un produit trop grand peut empêcher le développement, ce qui est signalé.
`ready` signifie prêt à tenter la génération, jamais maillage valide ni Q1 satisfait.

## Identité effective et moteur unique

Le module `src/campaign/` orchestre les API C++ de lecture/résolution, F1, génération,
contrôle et export. Il ne recopie ni spline, ni topologie, ni loi de distribution, ni métrique.
La CLI et Python relaient cette même orchestration ; aucun exécuteur de campagne Python.

La séparation normative des clés, états et fichiers est fixée dans
[result-identities.md](result-identities.md) : point logique, entrées de génération,
maillage généré immuable, audits indépendants et estimations physiques distincts.
`effective-mesh-v3` inclut la politique qualité susceptible de modifier la génération,
selon `generation-quality-v1`. `audit_profiles` seul désigne les audits en lecture seule.
Il est interdit d'exclure globalement tous les profils qualité de mesh_key.

L'échelle est résolue par point selon [physical-scale.md](physical-scale.md), dans S original,
avant toute flèche/normalisation et avant reprise. En campagne, `profiles[].physical_chord_m`
peut surcharger explicitement l'indication de la configuration ; null l'annule. Omission la
conserve. Cette substitution, unités/indications/flèche et toutes les autres sont archivées.
Changer l'incidence descriptive ou flow_estimate ne multiplie pas les générations ; changer
un réglage de génération, y compris une limite consommée par le lissage, change mesh_key.

## Exécution, conservation et reprise

`execution.mode=sequential` et `execution.on_failure=continue|stop` sont obligatoires.
`campaign run --file study.yaml --out study --json` orchestre une seule génération active,
puis les audits requis et l'estimation de chaque point, dans l'ordre de première occurrence.
Un seul artefact de génération par mesh_key est tenté dans un run, avec résultats d'audits
et d'estimations référencés séparément ; aucun bundle partagé ne porte un verdict universel.
Les commits atomiques par couche, le verrou d'écrivain, les références et les règles de
reprise sont ceux de result-identities.md. Après chaque point terminal, publier état/CSV
et events.jsonl ; après crash l'état JSON fait autorité et le CSV est reconstruit.

`--resume` exige même racine/campagne et ne se combine pas avec --replace.
Il réutilise seulement les couches terminées, compatibles et intègres après revalidation
de chaque point. Un point est `reused` seulement si ses propres audits requis passent ;
un audit échoué mémorisé est relu comme échec. `--resume --retry-failed` demande explicitement
de nouvelles tentatives des couches échouées, sans écraser les anciennes. Les couches
incomplètes/corrompues sont retentées dans un nouveau dossier avec motif, jamais promues.
Changer d'audit ou d'estimation réutilise les nœuds compatibles mais recalcule la couche
concernée ; changer la politique de génération peut exiger un autre maillage.

`continue` tente le point suivant après conservation de l'échec ; `stop` marque les suivants
not_run. Un audit failed/uncertified échoue son point code 6 sans changer le maillage ni
un audit voisin. Une estimation indisponible faute d'échelle reste explicite et n'invente
aucune valeur ; elle n'invalide pas le parcours géométrique. Un échec de calcul de l'estimation
échoue le point. Le statut global success exige tous points completed/reused avec leurs
contrôles requis, ou tous ready en check. Sinon partial_failure/stopped/invalid_request.
Codes : 0 succès ; 2 demande globale invalide ; 6 point en échec/non prêt ou arrêt ;
7 E/S/intégrité globale ; 8 ressources globales ; 70 interne. Codes unitaires conservés.

## Artefacts et accès Python

`campaign-resolved.json` archive la demande, les sources, paramètres/règles résolus et clés
calculables à l'inventaire (mesh_digest, clés d'audit/estimation/résultat null avant génération).
Photographie immuable par tentative ; `campaign-state.json` garde l'état et l'historique.
Les deux utilisent campaign-state.schema.json, avec artifact/phase distincts et key_versions.
Les lignes portent mesh_artifact, audits[], estimate, point_result, physical_scale,
failed_stage et les identités conservées. Le bundle unitaire est facultatif et propre au point.
Les descripteurs et références typés suivent campaign-artifact.schema.json ; aucune référence
relative à un autre audit ne permet de remplacer son verdict.

Le CSV a une ligne par point, colonnes : `point_id,mesh_case_id,profile_id,sweep_deg,
configuration_id,level,incidence_deg,mesh_key,mesh_digest,reuse_key,status,exit_code,
mesh_artifact,audit_results,estimate_status,estimate_artifact,point_result,bundle,
failed_stage,diagnostic_count`. `audit_results` est une cellule JSON échappée CSV contenant
les audit_id/status/path ; les clés, digests et tentatives complets sont dans l'état JSON.
Les chemins de ces références ont path_base=campaign_root ; aucun cwd implicite.

`Client.campaign_check(file=..., out=...)` et
`Client.campaign_run(file=..., out=..., resume=False, retry_failed=False)` exposent
CampaignResult avec inventaire et références de chaque couche. retry_failed exige resume.
Un dictionnaire impose base_dir ; mêmes clés/statuts/diagnostics, sans boucle ni calcul Python.
L'erreur globale structurée conserve les résultats partiels et les échecs locaux.

## Extension future réservée

Seul `kind=mesh_campaign` est accepté. `extensions.future_flight` peut archiver un objet
ininterprété, explicitement `not_executed`, hors clés effectives. Un autre `kind`, des champs
de vol actifs ou une demande de résolution provoquent `UNSUPPORTED_CAMPAIGN_KIND`/code 2.
Aucun cas Mach/Reynolds, force, moment, coefficient, polar ou champ fictif n'est produit.
L'estimation y+ déjà prévue reste une estimation géométrique facultative, jamais une résolution.

## Précisions d’enveloppe et fixtures de qualification

Un document impossible à parser/valider produit `request=null`, `status=invalid_request`,
`valid=false`, diagnostics et inventaire vide si le développement est impossible. Les octets
originaux sont conservés dans les entrées archivées si lisibles ; l’enveloppe n’exige pas qu’une
demande invalide satisfasse le schéma de demande. `future_flight_status` vaut `not_provided`
ou `not_executed` selon la présence de l’extension ; il ne signifie jamais calcul effectué.
`valid=true` sur `phase=inventory` signifie contrôle terminé sans erreur, sans maillage valide.
Les exports de run restent fixes (cinq SVG, ensemble VTP/VTM/VTS et données canoniques
partagées avec résultats de point) ; le bundle autonome peut être matérialisé sans
régénérer. Leur version entre dans reuse_key, sans option d’export cachée.

Le produit de 18 points V-10 emploie **V-01 et V-02**, trois flèches et niveaux 0/1/2,
avec une configuration fine commune. Les essais incluant **V-03** emploient des points explicites
et leur configuration épaisse compatible, ou un produit séparé épais. Ainsi aucune demande
`base_intervals=null` n’est appliquée involontairement à un culot épais, et les incompatibilités
ne sont jamais corrigées silencieusement pour atteindre le compte de 18.

## R2 — modes, cache et extension de vol

Le lecteur commun JSON/YAML de configuration est réutilisé ; les contrats du lot plan ne
rendent disponible aucun cas de vol ni mode conique. Les fichiers référencés en configuration
1.2 portent geometry_mode ; un mode conique ou champ actif Mach/Reynolds/température/effilement
est refusé explicitement, pas supprimé avant le calcul de clé. `extensions.future_flight`
reste une archive inactive avec `future_flight_status=not_executed` **et diagnostic
INACTIVE_EXTENSION**. Elle ne peut jamais activer une résolution ou rendre une requête active
valide. Une clé active hors schéma produit un refus localisé.

`effective-mesh-v3` inclut le modèle de représentation physique, les transformations/bases
et leurs versions effectives, métriques planes et versions de résolution, avec la source,
les paramètres et versions déjà prévus. Deux demandes planar_2d et planar_25d à flèche nulle
peuvent partager la géométrie seulement si leurs données effectives sont identiques ; le mode
demandé reste en provenance. L’incidence sans effet géométrique ne multiplie pas les maillages.
Les clés legacy effective-mesh-v1/v2 ne sont pas reprises comme effective-mesh-v3 sans recalcul et contrôles explicites de compatibilité.

Le futur DOE de résolution utilisera des **tuples de vol associés Mach/Re_c/T**, incidences,
flèches, paramètres indépendants d’effilement et réglages numériques. Il ne formera pas trois
listes Mach/Re/T croisées par défaut. Un futur cache de solution ajoutera ces entrées, modèle
physique/équations/SA-neg, conditions et versions du solveur ; un mesh_key n’est jamais une
clé de solution. Le contrat de ces champs reste futur, après décisions scientifiques et
thermodynamiques : ils ne sont pas ajoutés comme champs actifs au schéma courant.
