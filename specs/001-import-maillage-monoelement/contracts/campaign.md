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

La version `effective-mesh-v3` définit une sérialisation canonique : clés triées,
UTF-8, nombres binaire64 rendus avec aller-retour exact, −0 canonisé en 0, aucun NaN.
`mesh_key` est le SHA-256 de la source brute et des paramètres **résolus qui influencent
la géométrie ou la génération** : unités/échelle, indications/fermeture, reconstruction,
chaîne S/N/W/flèche, distributions, comptes F1, domaine, lissage et budgets géométriques,
plus versions des algorithmes et empreinte du binaire/options numériques.
Les ID, chemins d'origine, dates, incidence purement descriptive et `flow_estimate` en sont
exclus. Toute classification de champ est testée ; un champ futur inconnu est refusé.
Les valeurs demandées différentes donnant les mêmes valeurs effectives ont la même clé.
La source brute différente garde une clé différente, même pour des contours visuellement égaux.

`reuse_key` ajoute à `mesh_key` profil qualité et empreinte, schémas/règle de résolution,
environnement numérique exact, limites de ressources et options d'export demandées.
Une différence de ces éléments invalide la reprise certifiée ; il n'y a pas de compatibilité
floue de versions. Des limites de ressources différentes peuvent empêcher une exécution,
sans modifier le maillage physique et sans justifier une génération répétée dans une campagne.
Le calcul et la réutilisation de l'estimation facultative y+ sont traités par point après
maillage ; changer cette estimation ne génère pas de nouveaux nœuds.

Chaque `point_id` reste une ligne de l'inventaire. Des points ayant le même `mesh_key`
référencent un unique `mesh_case_id` et un bundle ; leur incidence et leurs diagnostics
de contexte sont conservés séparément. Le compteur d'appels de génération doit rester égal
au nombre de maillages effectifs réellement tentés, jamais au nombre d'incidences descriptives.

## Exécution, conservation et reprise

`execution.mode=sequential` et `execution.on_failure=continue|stop` sont obligatoires,
sans choix implicite. `campaign run --file study.yaml --out study --json` exécute les
maillages uniques dans l'ordre de leur première occurrence. Une seule génération est active.
Chaque cas publie son propre bundle par la procédure atomique unitaire. Après chaque point
terminé, l'état et le CSV sont écrits dans des fichiers temporaires voisins puis renommés ;
ils référencent uniquement des cas déjà publiés. L'état JSON fait autorité si une interruption
laisse le CSV à la révision précédente ; celui-ci est régénéré depuis l'état à la reprise.
Un journal `events.jsonl` conserve les transitions et les tentatives ; une ligne terminale
tronquée n'est pas un cas terminé. Aucun résultat antérieur n'est détruit par une reprise.

`--resume` exige un dossier existant de même campagne ; il ne se combine pas avec `--replace`.
Il vérifie les entrées actuelles, `reuse_key`, versions, état terminal `completed`, verdict
valide, digests, tailles, références et intégrité du bundle avant `reused`. Les points ayant
un maillage terminé partagé peuvent recevoir `reused` sans nouvelle génération. Les cas
`failed`, `running`, interrompus, incomplets ou corrompus sont retentés dans un nouveau
dossier d'essai ; les preuves anciennes restent archivées et la cause de non-reprise est écrite.
Un résultat compatible d'une autre campagne n'est pas recherché automatiquement.
Un seul écrivain est admis par dossier (verrou exclusif ; verrou persistant après crash
diagnostiqué, suppression seulement après constat de l'absence du processus propriétaire).

`continue` conserve l'échec et tente les points suivants. `stop` s'arrête après publication
de l'échec, inscrit tous les suivants `not_run`, puis retourne un code non nul.
Les doublons d'un cas échoué ne déclenchent pas une boucle de tentatives dans le même run :
ils portent le même échec ; seule une reprise explicite permet un nouvel essai.
Le statut global est `success` seulement si tous les points sont `completed|reused` valides,
ou tous `ready` pour `check`. Sinon `partial_failure`, `stopped` ou `invalid_request`.
Codes : 0 succès ; 2 demande de campagne invalide ; 6 cas en échec, arrêt ou inventaire non
prêt ; 7 erreur d'E/S/intégrité globale ; 8 limite de ressources globale ; 70 erreur interne.
Les codes unitaires précis restent dans les lignes, sans être masqués par le code global.

## Artefacts et accès Python

`campaign-resolved.json` archive la demande normalisée, les empreintes de toutes les entrées,
les points dans l'ordre, paramètres effectifs et clés, versions et substitutions. C'est une
photographie immuable par tentative ; une reprise crée une nouvelle photographie liée à la
précédente. `campaign-state.json` porte le statut courant et l'historique des tentatives.
Le tableau `summary.csv` a une ligne par point : `point_id,mesh_case_id,profile_id,sweep_deg,
configuration_id,level,incidence_deg,mesh_key,reuse_key,status,exit_code,bundle,diagnostic_count`.
Le JSON conserve en plus les diagnostics structurés et les causes de non-reprise.
Les deux documents utilisent l'enveloppe `campaign-state.schema.json` ; leurs champs `artifact`
et `phase` distinguent photographie résolue et état d'exécution.

`Client.campaign_check(file=..., out=...)` et
`Client.campaign_run(file=..., out=..., resume=False)` retournent un `CampaignResult`
(état, inventaire, CSV, chemins des cas), et la même erreur structurée si code non nul.
Un dictionnaire peut remplacer le fichier avec `base_dir` obligatoire ; il est sérialisé
en JSON sans modifier la base des chemins. Les méthodes ne réévaluent ni clés ni verdicts.

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
Les exports v1 de run sont fixes (bundle canonique, cinq SVG et ensemble VTP/VTM/VTS) ; leur
version entre dans reuse_key, sans option d’export cachée dans la demande de campagne.

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
Les résultats v1 ne sont pas repris comme v2 sans contrôle explicite de compatibilité.

Le futur DOE de résolution utilisera des **tuples de vol associés Mach/Re_c/T**, incidences,
flèches, paramètres indépendants d’effilement et réglages numériques. Il ne formera pas trois
listes Mach/Re/T croisées par défaut. Un futur cache de solution ajoutera ces entrées, modèle
physique/équations/SA-neg, conditions et versions du solveur ; un mesh_key n’est jamais une
clé de solution. Le contrat de ces champs reste futur, après décisions scientifiques et
thermodynamiques : ils ne sont pas ajoutés comme champs actifs au schéma courant.
