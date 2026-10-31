# Feuille de route produit Aero25D — autorité de cadrage

**Révision** : 2026-10-31, réconciliation R2. **État** : exigences et dépendances, aucune
capacité implémentée ou validée. Cette feuille de route complète la constitution 1.1.0 et
la seule [spécification active 001](../specs/001-import-maillage-monoelement/spec.md).
Elle ne constitue pas une seconde spécification de fonctionnalité ni une liste de travaux
RANS autorisés à l’exécution. Les exigences utilisateur courantes font autorité en cas de conflit.

## Produit final exigé

| ID | Exigence produit | Livraison / preuve future requise |
| --- | --- | --- |
| PR-001 | Solveur sectionnel autonome, maillage et résolution propres, sans solveur CFD externe | Parcours installé complet avec inventaire des dépendances |
| PR-002 | Modes 2D, 2.5D avec flèche et 2.75D avec flèche/effilement | Équations et hypothèses identifiées par mode ; vérification des limites entre modes |
| PR-003 | Entrée et référence de restitution : profil original en section line of flight | Octets/provenance conservés et correspondances vérifiées vers chaque représentation |
| PR-004 | Domaine cible initial M=0,2–0,89, Reynolds de plusieurs millions à plusieurs dizaines de millions, Λ=0°–50°, attaché/faiblement décollé, monoélément fin/épais | Catalogue de cas avec valeurs exactes et enveloppe effectivement qualifiée, sans extrapolation |
| PR-005 | RANS compressible pleinement turbulent, SA-neg, intégration jusqu’à la paroi | Équations visqueuses/énergie/SA-neg vérifiées, conditions aux limites et distance paroi ; hypothèses coniques explicites |
| PR-006 | L’effilement intervient dans les équations locales déterminant les pressions | Comparaisons Cp à géométrie/flèche/vol contrôlés ; une correction de coefficients intégrés ne suffit pas |
| PR-007 | Coefficients **sectionnels** Cz, Cx, Cm | Dérivation des efforts et moments par mode avec mesure sectionnelle et signes vérifiés |
| PR-008 | Distributions pariétales p, Cp et frottement/Cf sur extrados, intrados et culot | Données localisées sur la géométrie originale, composantes tangentielles/bases et normalisation indiquées |
| PR-009 | Pressions restituées dans la section originale line of flight | Distinguer valeurs pariétales et champ p dans le plan physique demandé ; similitude/reconstruction et erreurs vérifiées |
| PR-010 | Références cohérentes : axes vent, q∞ total, corde originale, quart de corde, axe Y global et mesure par unité de Y | Contrat ci-dessous ; aucune identification de Y et η hors Λ=0 |
| PR-011 | Campagnes JSON/YAML selon un même schéma versionné : produit cartésien explicite ou liste de cas | Profils, tuples de vol (Mach, Reynolds, température), incidences, flèches, données d’effilement et réglages numériques, avec contrôles de cohérence |
| PR-012 | Sauvegarde progressive, politique explicite d’échec et reprise après contrôle de compatibilité | Clés distinctes géométrie/maillage et solution ; entrées effectives, transformations, équations, versions et intégrité prises en compte |
| PR-013 | CLI et Python utilisables au quotidien sur un noyau unique | Statuts/rapports et résultats concordants, aucune logique scientifique dupliquée |
| PR-014 | C++/CUDA sous Linux/WSL2, CLion, CPU FP64 de référence ; RTX 4070 Ti Super 16 Go et A16-8Q 8 Go partagés | Budgets mémoire mesurés, comparaisons CPU/GPU et modes de précision explicités ; aucune performance présumée |
| PR-015 | Fiabilité Cz/Cm prioritaire, puis incertitude numérique Cx d’ordre 10⁻⁴ sur le domaine qualifié | Études de convergence/itérations/domaine et erreurs quantifiées ; validation expérimentale et erreur de modèle traitées séparément |
| PR-016 | VTM/VTS structurés, VTP contours/frontières, FP64, IDs/repères/unités/chemins relatifs et association des champs explicite | Raccords/orientations canoniques conservés, lecteur indépendant obligatoire en qualification, ouverture ParaView |
| PR-017 | Extension future au multiélément | Lecteur actuel conservant tous les éléments ; futur maillage/résolution/qualification spécifiques |
| PR-018 | Aucun VLM, couplage à un solveur d’aile ni performance globale d’aile finie requis | Pas de dépendance ou critère d’acceptation introduisant ces capacités |
| PR-019 | Décision scientifique avant construction de la géométrie/métrique conique | Porte G-FORM décrite ci-dessous ; résultat Euler jamais présenté comme preuve RANS |
| PR-020 | Séparer capacités finales, réalisation courante, études et travaux futurs | Statuts et traçabilité FR/tâches ; aucun champ réservé ni troisième coordonnée présenté comme mode 2.75D livré |

## Références d’entrée, de pression et de coefficients

S désigne la section line of flight originale à la station Y₀, dans le plan (X,Z).
La corde c₀ est la longueur BA–milieu des lèvres dans S ; les axes source ne sont pas tournés
silencieusement. Le point de moment est BA+0,25(BF−BA), à Y₀. Le vecteur moment physique
reste disponible ; le coefficient Cm demandé est sa projection sur **+Y global**.
L’axe d’extrusion η=(sinΛ,cosΛ,0) constitue une autre direction, jamais un renommage de Y.

Dans S, e_c est l’unité BA→BF et e_n sa perpendiculaire dirigée vers le haut dans le dessin
XZ. Pour la convention d’incidence de H-04 : e_D=cosα e_c+sinα e_n suit la vitesse amont,
e_L=−sinα e_c+cosα e_n. Cx est positif selon e_D, Cz selon e_L ; le signe de Cm suit +Y
(cabreur selon H-03). q∞=ρ∞‖U∞‖²/2 utilise la vitesse **totale** et les conditions physiques
du cas de vol, pas seulement sa composante normale à l’extrusion. Cp=(p−p∞)/q∞.

La mesure sectionnelle de référence est la force/moment **par unité de Y global**, à Y₀ :
Cz=(dF/dY·e_L)/(q∞c₀), Cx=(dF/dY·e_D)/(q∞c₀),
Cm=(dM/dY·e_Y)/(q∞c₀²). Une sortie par unité de η ou avec autre axe de moment doit être
étiquetée séparément et convertie par une dérivation vérifiée. Les intégrales physiques et
leur mesure doivent être dérivées pour le mode conique ; aucun facteur cosΛ ou de similitude
du cas extrudé n’est repris automatiquement. Ces formules fixent des **définitions de sortie**,
pas une recette d’intégration conique déjà établie.

Deux livrables de pression restent distincts : les valeurs pariétales associées aux paramètres
et points du contour original (y compris le culot), et le champ scalaire p sur un **plan physique**
line of flight demandé. Le second nécessite une reconstruction spatiale justifiée par la
formulation ; changer les noms ou les abscisses d’une solution de section ne la démontre pas.
Le frottement est la traction pariétale tangentielle physique ; Cf_vector=τwall,t/q∞, avec
bases/composantes et signes déclarés pour toute projection. Les champs aux cellules, sommets
et faces ont des associations explicites ; une interpolation
de visualisation ne remplace ni valeur conservative ni pression de paroi.

## Frontière de la fonctionnalité courante 001

001 réalise à terme l’import/contrôle, reconstruction, transformations et maillage **plan**
monoélément, qualité, familles, exports, CLI/Python et campagnes de maillage. Elle réalise aussi
une étude et une décision sur les besoins coniques, puis une démonstration géométrique JG-C
**conditionnelle** à cette décision. Les inconnues scientifiques et leur résolution font partie
des travaux ; les équations RANS, coefficients et champs d’écoulement restent futurs.
Le produit courant refuse les demandes de vol ou de mode conique actif non supporté.

J0 reste le premier jalon : V-01/02/03 × 0°/25°/50°, topologies planes 4/5 blocs,
contrôles indépendants et exports minimaux. J0 ne dépend ni de G-FORM, ni de JG-C, ni
des campagnes/Python/stockage complets. Le succès de J0 ne qualifie aucun mode conique.

## Dépendances et étapes futures

| Étape | Contenu | Dépendances / passage | Statut réel |
| --- | --- | --- | --- |
| J0 puis J1–J3 / 001 | Mailleur plan et parcours de maillage complets | Critères existants, sans relaxation | Prévu, aucun code ni essai produit |
| G-FORM / 001 | Étude transformations 2D / Euler conique / RANS conique, hypothèses et décision D-10 | Sources primaires, dérivations et revue scientifique T084–T088 | Étude initiale documentaire ; décision ouverte |
| JG-C / 001 | Démonstrateur géométrique isolé fin/épais à pente de corde non nulle | **J0 et G-FORM acceptés**, critères/métriques dérivés puis figés T089–T093 | Bloqué scientifiquement, non construit ; ne rend pas le mode produit disponible |
| FUT-01 | Équations compressibles et SA-neg 2D/2.5D, discrétisation et référence CPU FP64 | Mailleur plan qualifié ; spécification propre de résolution et thermodynamique | Futur, non autorisé ici |
| FUT-02 | Équations visqueuses/RANS coniques et SA-neg adaptées, flux et conditions aux limites | G-FORM, JG-C, FUT-01 et contrat métrique dérivé/versionné | Futur, aucun RANS conique revendiqué |
| FUT-03 | Vérification des équations coniques | FUT-02 : métriques, écoulement uniforme cartésien, limite de pente nulle vers 2.5D, références indépendantes et solutions manufacturées | Futur ; erreurs numériques séparées des hypothèses de modèle |
| FUT-04 | Pression/frottement, forces/moments et restitution line of flight | Dérivation par mode, quadratures et mappings vérifiés ; FUT-01/02 selon mode | Futur, aucun coefficient calculé |
| FUT-05 | Campagnes de résolution et DOE complets | Tuple Mach/Re_c/T cohérent avec échelle physique, thermodynamique/transport ; incidences/flèches/effilement et solveur de chaque mode | Futur, même schéma JSON/YAML par type d’opération |
| FUT-06 | CUDA et qualification scientifique du domaine | CPU FP64 vérifié, budgets 8/16 Go mesurés, convergence et comparaisons indépendantes | Futur, cible Cx non démontrée |
| FUT-07 | Extension multiélément | Mailleur/résolution/qualifications dédiés, sans perte de sources | Futur |

Les références de validation externes peuvent provenir de calculs indépendants/expériences,
sans devenir des dépendances d’exécution du produit. L’utilisation de travaux VLM dans la
bibliographie ne crée aucune exigence de couplage VLM dans cette feuille de route.

## Porte G-FORM et premier périmètre conique étudié

Le candidat est un tronçon trapézoïdal à profils **semblables**, sans variation locale de
vrillage, avec station Y₀, corde positive, pente de corde et pente/position d’une ligne de
référence explicites. La note [d’étude](../specs/001-import-maillage-monoelement/conical-study.md)
précise les hypothèses à éprouver, la paramétrisation candidate et les limites. Un taper ratio
isolé ne détermine ni pentes locales ni station de référence. Deux cordes et leur séparation
peuvent être une entrée alternative, mais ne sont pas exigées avec les pentes sans contrôle
de cohérence. Aucun contrat métrique conique détaillé n’est figé avant G-FORM.

La décision doit couvrir similitude, coordonnées/section physique, géométrie, dérivées
transversales, viscosité/turbulence/SA-neg, métriques et variations, restrictions, restitution
line of flight, intégration sectionnelle et validation. Si un volet n’est pas soutenu, le
résultat est une restriction ou un no-go explicite avec recherches suivantes ; il n’est pas
remplacé par une formulation Euler présentée comme RANS. G-FORM reste ouvert à cette révision.
