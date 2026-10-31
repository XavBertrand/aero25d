# Feature Specification: Import géométrique et maillage monoélément

**Feature Branch**: Aucune branche créée ; métadonnées Git indisponibles dans cet environnement.

**Created**: 2026-10-30

**Status**: Réconciliation R2 du 2026-10-31 ; implémentation suspendue pendant cette révision. Aucun code produit ni qualification numérique présent à l’inventaire.

**Input**: Demande utilisateur du 2026-10-30 : préparer un solveur aérodynamique autonome, en spécifiant uniquement l'import géométrique et le maillage monoélément indépendant du solveur.

## Autorité et portée de cette révision

La [feuille de route produit](../../docs/product-roadmap.md) fait autorité sur l’objectif
final **2D/2.5D/2.75D**, la résolution compressible SA-neg, les sorties sectionnelles et
les campagnes de vol futures. Cette spécification reste la seule fonctionnalité active 001 :
mailleur plan et préparation scientifique/géométrique conique, sans solveur RANS livré.
Les exigences PR sont des objectifs produit ; les FR ci-dessous portent la réalisation
courante et les portes conditionnelles. Le [registre de réconciliation](reconciliation.md)
conserve inventaire, contradictions et correspondances des exigences/tâches antérieures.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Contrôler un profil sans lancer de calcul (Priority: P1)

Un aérodynamicien charge un fichier, contrôle les contours et leurs repères, identifie les singularités et comprend tout refus avant de demander un maillage.

**Why this priority**: La validité du contour conditionne tous les calculs futurs.

**Independent Test**: Importer les jeux valides, invalides et ambigus de la campagne ci-dessous ; comparer les coordonnées, les repères et les diagnostics aux résultats attendus, sans mailleur ni solveur.

**Acceptance Scenarios**:

1. **Given** un profil valide contenant des lignes vides et des commentaires autorisés, **When** il est chargé, **Then** tous les points et leur provenance sont conservés et les conventions sont accessibles.
2. **Given** un comptage erroné, une valeur non finie, un doublon interdit ou une intersection, **When** le contrôle est lancé, **Then** il échoue avec localisation et cause ; aucun contour corrigé n'est présenté comme valide.
3. **Given** un bord de fuite dont les lèvres sont ambiguës, **When** l'identification est demandée, **Then** les candidats et l'indication attendue sont fournis ; une indication explicite cohérente permet de poursuivre.
4. **Given** un fichier valide à deux éléments, **When** il est chargé puis soumis au mailleur, **Then** les deux éléments restent consultables et le maillage est refusé explicitement.

### User Story 2 - Obtenir un maillage contrôlé pour une section en flèche (Priority: P1)

L'utilisateur règle la géométrie de calcul et les distributions puis obtient un maillage et un rapport localisant ses éventuels défauts.

**Why this priority**: C'est le résultat principal de la fonctionnalité, préalable au futur RANS.

**Independent Test**: À partir de contours de référence validés, générer les cas fins et épais à trois flèches ; inspecter les transformations, les frontières, les blocs et toutes les mesures de qualité sans résolution.

**Acceptance Scenarios**:

1. **Given** un profil fin, **When** un maillage est demandé à 0°, 25° puis 50°, **Then** chaque résultat satisfait les exigences et utilise une topologie de famille C sans cellules effondrées.
2. **Given** un profil à culot épais, **When** il est maillé, **Then** les deux lèvres, la face de culot et le sillage sont représentés distinctement ; aucune conversion en pointe n'a lieu.
3. **Given** une configuration provoquant un défaut critique, **When** la génération est terminée ou interrompue, **Then** un échec explicite et un rapport exploitable sont retournés ; aucun résultat n'est déclaré utilisable.
4. **Given** un premier espacement normal demandé, **When** le maillage est contrôlé, **Then** la première hauteur et la distance du premier centre sont mesurées séparément ; un éventuel y+ est étiqueté comme estimation.

### User Story 3 - Inspecter, exporter et reproduire (Priority: P2)

Depuis une commande ou Python, l'utilisateur exécute le parcours complet et retrouve les mêmes données et conclusions de contrôle.

**Why this priority**: L'inspection et la reproductibilité rendent les résultats vérifiables et réutilisables.

**Independent Test**: Réaliser les deux parcours avec le même fichier et les mêmes paramètres résolus ; recharger les sorties et comparer les mesures et les identifiants.

**Acceptance Scenarios**:

1. **Given** une génération valide, **When** l'utilisateur exporte, **Then** il obtient un VTK lisible, les blocs et interfaces conservés, un rapport structuré, les métadonnées et les cinq vues demandées.
2. **Given** les métadonnées et la source archivées, **When** il relance dans la même version et le même environnement, **Then** la connectivité est identique et les coordonnées reproduites respectent la tolérance numérique archivée.
3. **Given** un refus d'import ou de maillage, **When** il passe de la commande à Python, **Then** le statut, la cause et les objets concernés concordent.

### User Story 4 - Préparer une étude de sensibilité (Priority: P2)

L'utilisateur génère au moins trois maillages comparables et fait varier séparément la frontière extérieure.

**Why this priority**: Une qualification future nécessite de dissocier raffinement et taille du domaine.

**Independent Test**: Sur un profil fin et un profil épais, comparer trois niveaux puis deux distances extérieures à paramètres intérieurs constants.

**Acceptance Scenarios**:

1. **Given** une configuration de référence, **When** trois niveaux sont demandés, **Then** la géométrie reconstruite et la topologie sont identiques ; le raffinement effectif de chaque région est documenté et contrôlé.
2. **Given** un niveau fixé, **When** seule la distance extérieure change, **Then** cette variation est identifiée séparément et ne modifie ni la géométrie ni les réglages prescrits de paroi, culot et sillage proche.

### User Story 5 - Piloter une campagne de maillage (Priority: P2)

L’utilisateur décrit une étude JSON ou YAML, contrôle son inventaire, puis exécute et reprend
les maillages depuis la commande ou Python sans gérer manuellement chaque dossier.

**Why this priority**: Les campagnes rendent les études de profils, flèches et résolutions
traçables et évitent des générations identiques pour des métadonnées sans effet géométrique.

**Independent Test**: Comparer les inventaires JSON/YAML et produit/liste explicites, exécuter
une campagne avec un échec injecté selon les deux politiques, interrompre puis reprendre,
modifier une entrée/version et vérifier les décisions de reprise et le nombre de générations.

**Acceptance Scenarios**:

1. **Given** deux profils, trois flèches et trois niveaux compatibles, **When** le produit explicite est contrôlé, **Then** 18 points sont inventoriés avec paramètres résolus, et aucun maillage n’est généré ; la liste équivalente fournit le même inventaire effectif.
2. **Given** des points répétés ne différant que par l’incidence descriptive, **When** la campagne est exécutée, **Then** un seul maillage est généré par clé effective et chaque point reste présent dans le tableau.
3. **Given** un cas défaillant entre deux cas valides, **When** `on_failure=continue` ou `stop`, **Then** le suivant est exécuté ou marqué non exécuté, les preuves sont conservées et le code global est non nul.
4. **Given** une campagne interrompue, **When** elle reprend, **Then** seuls les cas terminés, compatibles et intègres sont repris sans génération ; les autres sont retentés avec motif et historique.
5. **Given** les mêmes sources depuis un autre répertoire courant et via Python, **When** contrôle/exécution/reprise sont demandés, **Then** chemins, inventaire, codes et résultats concordent.
6. **Given** un champ d’extension future ou une demande active de vol, **When** le contrat est lu, **Then** l’extension reste non exécutée ou la demande est refusée ; aucune grandeur aérodynamique n’est fabriquée.

### User Story 6 - Préparer une géométrie conique sur une formulation justifiée (Priority: P2)

L’utilisateur veut savoir quelles données et hypothèses permettent d’introduire l’effilement
dans le calcul local futur de pression. Il obtient une étude comparative, une décision
scientifique revue, puis seulement si celle-ci autorise la construction une démonstration
**géométrique** conique distincte du maillage plan. Il ne reçoit aucun faux résultat RANS.

**Why this priority**: Éviter de figer des métriques ou de généraliser une extrusion avant
la dérivation des hypothèses visqueuses et turbulentes ; limiter les reprises sans retarder J0.

**Independent Test**: Auditer la décision G-FORM et ses dérivations. Après go scientifique,
éprouver JG-C sur fin/épais à pente de corde non nulle et sa limite nulle, avec oracles
indépendants et restitution géométrique. Aucun calcul de pression n’est exigé de ce jalon.

**Acceptance Scenarios**:

1. **Given** les sources primaires 2D transformé/Euler/RANS coniques et Polytechnique Montréal, **When** la revue est faite, **Then** chaque affirmation porte son niveau de preuve et une preuve Euler n’est pas qualifiée RANS/SA-neg.
2. **Given** un tronçon trapézoïdal à profils semblables sans variation locale de vrillage, **When** les entrées sont étudiées, **Then** station, corde et pentes locales sont déterminables ; un taper ratio seul est insuffisant, les entrées redondantes sont vérifiées.
3. **Given** une décision de formulation encore ouverte ou refusée, **When** les tâches de construction conique sont examinées, **Then** elles restent bloquées et aucun contrat métrique détaillé n’est figé ; J0 plan conserve ses propres critères et dépendances.
4. **Given** G-FORM et J0 acceptés, **When** JG-C est exécuté, **Then** correspondances source/physique, raccords, métriques non singulières et limite d’extrusion sont contrôlés sur fin/épais ; aucune qualification RANS ne lui est attribuée.
5. **Given** une demande conique ou des données de vol actives adressées au produit plan, **When** le contrat est validé, **Then** un refus explicite les identifie ; aucune donnée n’est acceptée silencieusement sans effet.

### Edge Cases

- Fichier vide ou tronqué ; entier nul, négatif ou non entier ; trop ou trop peu de points/éléments ; données résiduelles après le dernier élément ; ligne de coordonnées mal formée.
- NaN, infini, corde nulle, points tous alignés, segments de longueur nulle, répétition intérieure d'un point, croisement ou contact non adjacent ; intersections entre éléments signalées aussi à l'import.
- Contour inversé ; premier point arbitraire ; fermeture explicite avec point terminal répété ; bord de fuite fin sans point terminal répété ; deux lèvres distinctes ; culot déjà fourni avec points intermédiaires.
- Ouverture ailleurs qu'au bord de fuite, grand segment de fermeture suspect, lèvres indiscernables à la tolérance choisie : diagnostic et indication/correction externe, jamais fermeture arbitraire.
- Géométrie peu échantillonnée au bord d'attaque : ne pas annoncer une fidélité de rayon que les données ne permettent pas d'établir.
- Paramètres négatifs ou non finis, nombre de couches non entier, croissance incompatible avec le domaine, limite de qualité trop restrictive, ressources insuffisantes : refus explicite avec paramètres concernés ; aucun allègement silencieux.
- Très fort étirement aligné avec la couche limite ; désalignement d'un étirement similaire ; raccord inversé, décalé ou manquant ; cellule de très petite aire positive mais numériquement dégénérée.
- Flèche hors [0°, 50°], mode planar_2d avec flèche non nulle, demande conical_275d active ou champs Mach/Reynolds/température/effilement : refus explicite dans le produit plan. L’incidence reste une métadonnée de maillage, pas une campagne de résolution.

## Requirements *(mandatory)*

### Functional Requirements

Les exigences FR ci-dessous sont fermes. Les conventions détaillées H sont des hypothèses explicites de spécification, révisables avant planification ; les choix D restent au plan. Chaque FR possède une ligne de vérification dans la matrice, dont les critères sont contractuels.

#### Import et intégrité

- **FR-001** : Lire la structure « nombre d'éléments, puis pour chacun nombre de points et paires X Z », sans perdre aucun élément ni point déclaré.
- **FR-002** : Accepter les lignes vides et les commentaires documentés par H-01 ; exiger des comptes entiers strictement positifs et exactement deux nombres par ligne de coordonnées ; rejeter toute incohérence de comptage et donnée supplémentaire non commentée.
- **FR-003** : Contrôler valeurs finies, corde non nulle, doublons, segments nuls, auto-intersections et contacts non adjacents ; contrôler aussi les intersections entre contours d'éléments. Distinguer la répétition terminale autorisée des doublons invalides.
- **FR-004** : Ne jamais réparer silencieusement une géométrie invalide. Fournir statut, cause, élément, point(s)/segment(s) et ligne(s) source quand disponibles ; conserver l'entrée même en échec.
- **FR-005** : Lire et contrôler un fichier multiélément complet ; refuser explicitement toute demande de maillage s'il contient plusieurs éléments, sans produire de maillage monoélément partiel.

#### Géométrie et section normale

- **FR-006** : Exposer et archiver axes, unités, corde originale, origine, sens des contours, flèche, incidence et référence du moment selon les conventions explicites H-02 à H-05.
- **FR-007** : Conserver les coordonnées originales, l'ordre source et toutes les transformations avec leur inverse. Toute normalisation est optionnelle, explicite et globale pour tous les éléments ; aucune normalisation élément par élément.
- **FR-008** : Identifier bord d'attaque, extrados, intrados, lèvres et éventuel culot ; exposer les ambiguïtés, permettre des indications utilisateur traçables et les vérifier avant de poursuivre. Une indication ne doit pas dispenser des contrôles de validité.
- **FR-009** : Appliquer à tous les points la transformation ξ = (X − Xref) cos(Λ), z = Z, pour Λ dans [0°, 50°], avant l'éventuelle normalisation. À 0°, retrouver X − Xref et Z. La corde de référence reste celle de la section originale. Précision R2 : cette formule décrit la représentation plane extrudée ; elle ne définit pas la transformation conique future.
- **FR-010** : Produire un maillage 2D dans la section normale à une direction horizontale commune d'extrusion ; conserver cette direction comme métadonnée pour un futur écoulement à trois composantes. Une correction du Mach ne remplace pas la transformation géométrique. Précision R2 : exigence de maillage plan de 001, distincte de l’objectif produit 2.75D et du jalon conditionnel JG-C.

#### Reconstruction du contour

- **FR-011** : Distinguer points source et sommets du maillage ; reconstruire et redistribuer en préservant le rayon du bord d'attaque identifiable, les angles, les lèvres, l'épaisseur et la face de culot, sans oscillations ou intersections artificielles ; concentrer la distribution dans les fortes courbures.
- **FR-012** : Mesurer séparément les écarts source–reconstruction et reconstruction–frontière maillée, dans les repères original et transformé ; publier tolérances, méthode, couverture de mesure et erreur résiduelle. Ne pas prétendre reconstruire la forme réelle entre points non renseignés.
- **FR-013** : Accepter une fermeture explicite ou un contour implicitement fermé suivant H-06. Compléter seulement un bord de fuite épais identifié et confirmé selon cette convention ; conserver le culot fourni, refuser ou suspendre toute autre ouverture ambiguë. Ne jamais transformer silencieusement un culot en pointe.

#### Maillage indépendant

- **FR-014** : Générer sans résolution un maillage conforme au contour, uniquement quadrilatéral et structuré multibloc, prévu pour une résolution jusqu'à la paroi.
- **FR-015** : Distinguer la famille C pour bord de fuite fin et une famille multibloc représentant culot et sillage pour bord épais, de type C–H ou équivalent justifié au plan ; ne pas créer le cas fin par effondrement non contrôlé de cellules du cas épais.
- **FR-016** : Conserver blocs, indices, frontières physiques et interfaces ; chaque interface interne doit posséder un raccord univoque, une géométrie coïncidente et des orientations compatibles avec une future discrétisation conservative.
- **FR-017** : Résoudre et identifier séparément bord d'attaque, couches extrados/intrados, bord de fuite, culot lorsqu'il existe, sillage proche et raccordement au champ lointain.
- **FR-018** : Permettre le contrôle explicite de résolution tangentielle, concentrations aux deux bords, premier espacement normal, nombre de couches, croissance normale, résolution du culot, longueur/résolution du sillage, distance extérieure et niveau de raffinement ; valider les incompatibilités, sans remplacement silencieux des valeurs.
- **FR-019** : Distinguer la hauteur de la première cellule, mesurée de la paroi à sa face opposée suivant la normale locale, et la distance normale du premier centre à la paroi. Exposer les valeurs demandées et mesurées ; ne pas supposer que la seconde vaut exactement la moitié de la première.
- **FR-020** : Si une estimation de y+ est fournie, publier formule, hypothèses, grandeurs physiques et emplacement du premier centre utilisés ; la désigner explicitement comme estimation avant résolution. L'objectif futur est de l'ordre de 0,5 à 1, sans garantie du y+ réel.
- **FR-021** : Permettre une variation indépendante de la distance extérieure, sans valeur fixe présentée comme garantie de précision aérodynamique.

#### Qualité et sensibilité

- **FR-022** : Contrôler toutes les cellules et faces : aires signées strictement positives (volumes par unité d'extrusion), absence de dégénérescence et d'auto-intersection, orientation et fermeture ; contrôler toutes les interfaces. Précision R2 : les aires/volumes par unité d’extrusion sont ceux de la représentation plane ; les mesures physiques coniques restent à dériver.
- **FR-023** : Mesurer orthogonalité, variations d'espacement, étirement et son orientation, et fidélité au contour ; appliquer un profil de limites explicites. Ne pas rejeter un grand rapport d'aspect uniquement sur sa valeur sans examiner son alignement avec la couche limite.
- **FR-024** : Produire un rapport localisant chaque défaut et sa sévérité, avec mesure et limite appliquée ; retourner un échec explicite sur tout défaut critique. Des sorties de diagnostic peuvent subsister mais doivent porter le statut invalide.
- **FR-025** : Générer au moins trois niveaux cohérents de raffinement sur une même reconstruction et une même topologie ; maîtriser et publier les évolutions tangentielles, normales, du culot et du sillage. Maintenir des règles comparables de concentration et ne pas utiliser la distance extérieure comme raffinement.

#### Parcours et livrables

- **FR-026** : Donner accès depuis une commande et Python aux mêmes opérations : charger, contrôler, inspecter les contours original/reconstruit, paramétrer, mailler, exporter, consulter la qualité et générer une famille ; fournir des résultats et statuts concordants sans lancer de solveur.
- **FR-027** : Exporter un maillage VTK visualisable et une représentation conservant les blocs, leurs frontières et interfaces, avec correspondance des identifiants du rapport. Le VTK seul ne doit pas faire perdre la structure interne.
- **FR-028** : Fournir un rapport structuré lisible depuis Python et les figures du profil, du bord d'attaque, du bord de fuite/culot, du sillage et du maillage global, avec repères, unités et légende identifiables ; permettre la superposition source/reconstruction/frontière maillée.
- **FR-029** : Archiver source et empreinte, version du générateur, environnement pertinent, paramètres demandés/résolus, indications, conventions, transformations, tolérances, profil qualité et mesures ; permettre une reproduction contrôlée selon la matrice.
- **FR-030** : Permettre l'utilisation autonome du mailleur sur CPU, sans dépendance d'exécution à un solveur CFD externe ni exigence de GPU pour cette fonctionnalité.

#### Campagnes et qualification des exports — ajouts du 2026-10-31

- **FR-031** : Décrire une campagne JSON ou YAML par listes de profils, flèches et configurations/niveaux ; choisir explicitement entre produit cartésien de dimensions déclarées et liste de points explicites, sans combinaison implicite.
- **FR-032** : Contrôler et inventorier tous les points résolus d’une campagne sans génération de maillage ; publier les incompatibilités et statuts de préparation sans déclarer les maillages valides.
- **FR-033** : Exécuter initialement les cas séquentiellement et enregistrer progressivement chaque résultat et ses preuves ; une interruption ne doit pas invalider les cas déjà terminés ni publier un cas incomplet comme terminé.
- **FR-034** : Reprendre uniquement les cas terminés valides dont les entrées effectives, versions, environnement numérique et artefacts intègres sont compatibles selon une règle explicite ; conserver les tentatives et motifs de non-reprise.
- **FR-035** : Exiger un paramètre explicite de poursuite ou d’arrêt sur échec ; conserver l’échec, le statut des points non exécutés et un code global non nul si la campagne n’est pas entièrement satisfaite.
- **FR-036** : Publier un manifeste résolu et un tableau récapitulatif avec une ligne par point demandé, ses paramètres, identifiants, statut, diagnostics et liens vers les résultats ; conserver la distinction entre point demandé et maillage unique.
- **FR-037** : Résoudre les chemins de profils et configurations relativement au fichier de campagne ; conserver des références relatives transportables dans les ensembles de résultats et une provenance vérifiable.
- **FR-038** : Donner depuis Python accès au contrôle, à l’inventaire, à l’exécution et à la reprise des campagnes avec les mêmes résultats, codes et règles que la commande ; appeler la même orchestration et le même noyau de maillage.
- **FR-039** : Réserver une extension versionnée aux futures campagnes de vol sans les exécuter ici ; refuser une demande de résolution ou un type actif non pris en charge, et ne produire aucun résultat aérodynamique simulé.
- **FR-040** : Identifier les maillages identiques par les entrées géométriques et paramètres de génération effectifs, avec versions explicites ; mutualiser leur génération et conserver chaque point logique. Une incidence sans effet géométrique ne doit pas déclencher un nouveau maillage. La campagne ne doit dupliquer aucun algorithme du mailleur.
- **FR-041** : Exporter en .vtp les contours et frontières, avec rôles distincts pour extrados, intrados, culot et interfaces ; fournir des identifiants reliant les objets exportés, leur provenance et les diagnostics. Conserver VTM/VTS pour les blocs structurés.
- **FR-042** : Inscrire dans les exports les repères et unités explicites, la représentation source/reconstruction/frontière maillée, et utiliser des chemins relatifs dans les ensembles de fichiers ; préserver leur lecture après déplacement.
- **FR-043** : Rendre obligatoire une relecture indépendante par VTK ou PyVista dans la qualification des exports VTP/VTM/VTS ; comparer comptes, types, identifiants et coordonnées avec les données canoniques. Cette dépendance de test ne doit pas être requise pour exécuter le produit.
- **FR-044** : Vérifier l’ouverture des exports dans ParaView et conserver version, procédure, vues des blocs, contours et frontières, et conclusion d’inspection. Une ouverture seule ne remplace aucun contrôle numérique.
- **FR-045** : Placer avant toute généralisation un jalon J0 démontrant V-01/02/03 × 0°/25°/50°, avec mesures exhaustives de validité, interfaces et vues détaillées du BA et des lèvres. Sur échec, conserver les preuves, diagnostiquer et revoir la construction avant de poursuivre, sans relaxation silencieuse. Les culots extrêmement minces restent des tests de limites pouvant être refusés explicitement.

#### Réconciliation produit et préparation conique — R2

- **FR-046** : Référencer une feuille de route produit unique couvrant le solveur sectionnel autonome 2D/2.5D/2.75D, SA-neg, sorties et campagnes futures ; distinguer ces exigences du mailleur courant, sans VLM ni solveur d’aile requis.
- **FR-047** : Conserver le contour original line of flight comme entrée et référence de restitution, avec correspondances et provenance explicites entre source, reconstruction et représentation physique de calcul.
- **FR-048** : Séparer contour source, topologie logique (indices/blocs/raccords), section physique, transformations/bases et métriques/variations ; conserver un chemin plan simple, sans contrat métrique conique anticipé.
- **FR-049** : Identifier explicitement les modes de géométrie pris en charge et refuser les demandes coniques et champs de vol/effilement actifs non supportés. Un champ réservé est signalé non exécuté ; aucune absence d’effet n’est silencieuse.
- **FR-050** : Produire une étude ciblée de sources primaires comparant transformations de résultats 2D, Euler conique et visqueux/RANS conique, incluant les travaux pertinents de Polytechnique Montréal ; viser l’effet de l’effilement sur la pression locale, sans confondre preuves Euler et RANS.
- **FR-051** : Établir une décision de formulation précisant similitude, section/coordonnées, géométrie et pentes, dérivées transversales, viscosité/turbulence SA-neg, métriques nécessaires, restrictions, restitution line of flight, efforts et validation. Étudier d’abord un tronçon trapézoïdal à profils semblables sans variation locale de vrillage ; refuser taper ratio seul ou redondances incohérentes.
- **FR-052** : Faire dépendre toute construction et tout contrat métrique coniques détaillés d’une décision scientifique acceptée ; conserver les inconnues et no-go explicites. Ne pas présenter trois coordonnées ou un champ taper_ratio comme mode 2.75D implémenté.
- **FR-053** : Prévoir un jalon géométrique conique JG-C distinct, après J0 et décision de formulation : fin/épais à effilement non nul, correspondances, raccords, métriques non singulières, restitution géométrique line of flight et limite d’extrusion à pente de corde nulle ; aucune résolution RANS dans ce jalon.
- **FR-054** : Distinguer dans les besoins futurs Cp pariétal sur extrados/intrados/culot en coordonnées originales et champ de pression sur un plan physique line of flight ; documenter et vérifier toute reconstruction par similitude. Dans 001, vérifier seulement les correspondances géométriques, sans fabriquer de pressions.
- **FR-055** : Fixer les références des futurs Cz/Cx/Cm sectionnels (axes vent, q∞ total, corde originale, point/axe de moment, mesure par unité de Y global) et conserver Y distinct de η ; exiger une dérivation spécifique des intégrales coniques, sans reprendre automatiquement les facteurs extrudés.
- **FR-056** : Distinguer explicitement association cellule/sommet/face, topologie et représentation physique dans le canonique et les exports FP64. Une future surface physique conique ou courbe ne doit jamais être exportée comme grille plane sans indication de représentation et mapping.
- **FR-057** : Versionner les règles de transformation/représentation et de compatibilité ; fonder caches/reprises sur entrées effectives et versions pertinentes, sans nom de profil seul. Distinguer cache de maillage et futur cache de solution dépendant des cas de vol et du modèle d’équations.
- **FR-058** : Rendre visibles les futurs travaux d’équations coniques : métriques, préservation de l’uniforme cartésien, limite 2.5D, références indépendantes et séparation des erreurs numérique/de modèle ; les maintenir hors de l’implémentation courante du mailleur.
- **FR-059** : Préserver J0 plan comme premier jalon, indépendant de la décision et de la qualification coniques, avec ses neuf configurations et contrôles nécessaires. Une acceptation du parcours plan ne doit pas être présentée comme acceptation du conique.
- **FR-060** : Conserver identifiants et correspondances des exigences/tâches révisées ou différées, avec statut réel et revalidation nécessaire ; suspendre l’implémentation pendant la réconciliation, conserver tout code/preuve existant sans le déclarer validé.

### Conventions proposées et tolérances

Les valeurs et conventions suivantes sont des choix de projet proposés, pas des normes scientifiques. Elles rendent les tests définissables sans choisir d'algorithme. Le plan doit les reprendre ou documenter et justifier toute révision dans la spécification avant de figer les tests.

- **H-01 — Grammaire** : texte décimal à point, notation scientifique acceptée ; séparateurs blancs ; `#` introduit un commentaire jusqu'à la fin de ligne, y compris après une valeur. Aucun autre commentaire implicite. Les lignes vides/commentées ne comptent pas comme données.
- **H-02 — Axes et unités** : X horizontal vers l'aval du vol, Z vertical vers le haut ; Y complète le trièdre direct (X,Y,Z). Les unités ne sont pas déduites des valeurs : l'utilisateur déclare mètres ou coordonnées adimensionnelles ; par défaut, coordonnées adimensionnelles signalées comme telles. Une estimation physique de y+ nécessite une échelle physique explicite. L'import ne tourne pas automatiquement le profil pour aligner sa corde.
- **H-03 — Références** : BA est le bord d'attaque identifié ; BF est le milieu des deux lèvres (confondues pour une pointe). La corde originale c est la longueur du segment BA–BF dans le plan XZ ; sa projection X est aussi conservée, sans confusion avec c. Le quart de corde est BA + 0,25(BF − BA) ; Xref est sa coordonnée X. Le point de référence futur des moments reste ce quart de corde. Les axes global +Y et d'extrusion +η = sin(Λ)eX + cos(Λ)eY sont deux métadonnées distinctes : ils coïncident seulement à flèche nulle. Le moment de tangage global est la projection d'un futur vecteur moment sur +Y (positif cabreur à flèche nulle) ; une projection sur +η doit être désignée séparément et ne lui est pas identique à flèche non nulle. Le contrat futur conserve vecteur, base, point de référence et axe de projection choisi. Aucune force ni aucun moment n'est calculé ici.
- **H-04 — Orientation et incidence** : les contours de travail sont parcourus dans le sens trigonométrique dans le dessin (X horizontal, Z vertical), intérieur à gauche ; l'ordre original reste conservé. Extrados/intrados sont les deux branches entre BA et les lèvres, supérieure/inférieure dans le repère documenté ; un cas ambigu exige une indication. L'incidence α est l'angle entre la corde originale orientée BA→BF et la vitesse amont, positif lorsque la composante de cette vitesse se dirige au-dessus de la corde dans la section XZ. La convention concerne la section originale ; aucune incidence normale équivalente n'est inventée par le mailleur.
- **H-05 — Flèche et normalisation** : η suit sin(Λ)eX + cos(Λ)eY ; ξ suit cos(Λ)eX − sin(Λ)eY ; z suit eZ. Λ est en degrés, positif lorsque l'extrusion vers +Y se déplace vers l'aval ; la référence au quart de corde ne change pas la direction d'invariance. La projection de la section fournie à Y=0 est exactement celle de FR-009. Sans normalisation, z=Z ; toute translation verticale ultérieure est une opération globale séparée et archivée. Une normalisation éventuelle utilise une seule translation et une seule échelle positive pour tous les éléments, jamais la corde comprimée par cos(Λ) comme nouvelle référence utilisateur. Les repères intermédiaires sont S=(X,Z), section source à Y=0, N=(ξ,z), section normale projetée, puis W=(u,v), repère global de travail. S et N ont les unités déclarées de la source. W=((ξ−aξ)/s,(z−az)/s), avec une seule translation (aξ,az) dans les unités de N et une échelle s>0 dans ces mêmes unités, est adimensionnel. Le repère N est conservé même sans normalisation. L'inverse applique strictement l'ordre inverse : d'abord ξ=s u+aξ et z=s v+az si W existe, puis X=ξ/cos(Λ)+Xref et Z=z ; cos(Λ) reste non nul sur le domaine accepté. La précondition numérique éventuelle dans S est séparée de ces transformations physiques et elle aussi archivée. Un aller-retour doit couvrir translation horizontale/verticale, changement d'échelle, flèche non nulle et Z non centré.
- **H-06 — Fermeture** : le contour source suit des points successifs du périmètre, départ et sens libres. Le dernier point peut répéter exactement le premier pour déclarer une fermeture ; ce marqueur ne crée pas de segment nul de travail et sa prise en compte est signalée. Sans répétition, la fermeture implicite est présentée dans l'inspection. Deux extrémités reconnues comme lèvres distinctes peuvent être reliées par un culot rectiligne, avec confirmation explicite de cette convention dans les paramètres ; ce segment est marqué « ajouté ». Un culot déjà décrit n'est pas remplacé. Une ouverture suspecte ou une identification non univoque bloque le maillage jusqu'à indication cohérente ou correction du fichier. La proximité numérique seule ne décide jamais de supprimer une épaisseur.
- **H-07 — Budgets géométriques proposés** : c désigne toujours la corde originale. Pour les cas de qualification bien résolus, budget par défaut de distance maximale bidirectionnelle source polygonale–reconstruction : 10⁻⁴ c ; reconstruction–frontière maillée : 10⁻⁵ c ; déplacement d'un point caractéristique identifié et aller-retour des transformations : 10⁻¹² c. Les deux premières valeurs bornent des modifications géométriques distinctes et ne donnent aucune borne sur Cx. La dernière est un budget de cohérence numérique, choisi très inférieur aux budgets de reconstruction, non une tolérance de réparation. Dans S, la reconstruction se compare à la référence originale. Dans N, elle se compare à l'application de la même projection à cette référence, et dans W à sa projection puis normalisation. Les distances et budgets sont exprimés dans le même repère, avec conversion par s pour W ; c reste la corde originale. Les comparaisons ne portent jamais sur une référence originale non transformée face à un contour transformé. Une configuration peut imposer des budgets plus stricts ; si le budget est incompatible avec les données, il faut échouer ou demander une modification explicite.
- **H-08 — Rayon, angles et bruit numérique** : sur une référence analytique de rayon connu, variation relative du rayon au BA ≤ 1 % ; les sommets anguleux et leur raccord de culot sont des caractéristiques imposées, avec écart angulaire ≤ 0,1°. Ces budgets de qualification visent à révéler arrondi et émoussement, sans équivalence aérodynamique revendiquée. Ces limites s'appliquent à la référence et à la reconstruction dans un même repère : dans S à la référence originale, dans N à la référence transformée, dans W à cette dernière normalisée. La compression horizontale cos(Λ) change volontairement les rayons et les angles ; on vérifie leur valeur transformée attendue, et non leur invariance. Sur données non analytiques, publier l'estimation et sa sensibilité au voisinage d'évaluation ; déclarer « non certifiable » si les données ne permettent pas de vérifier le rayon. Le plan fixe les méthodes d'évaluation et les bornes d'erreur de mesure, assez fines pour trancher chaque budget. Aucun point proche mais distinct ne doit être fusionné automatiquement.
- **H-09 — Limites de qualité** : positivité et intégrité topologique sont inconditionnelles ; le profil qualité doit aussi contenir des limites numériques finies pour dégénérescence, orthogonalité, variation d'espacement, étirement avec orientation et raccords. Le plan choisit leurs définitions et valeurs par une étude géométrique documentée, pas par une prétendue norme universelle. La tolérance de raccord et de fermeture normalisée par c ne peut dépasser 10⁻¹² sur les cas de qualification. Les interfaces ont une correspondance de sommets et faces un à un ; une tolérance ne permet ni trou ni chevauchement topologique. Tout profil utilisé est archivé avant génération. Les limites ne peuvent être relâchées automatiquement pour faire réussir un cas.

### Matrice de vérification et preuves

Chaque ligne indique la méthode et le critère mesurable de l'exigence correspondante. « Tous » signifie 100 % des objets ou cas visés. Les résultats constituent des preuves à produire lors de la future réalisation, et non des essais déjà exécutés. Les budgets H-07/H-08 et les limites du profil H-09 sont inclus dans les critères. Une mesure sans limite résolue ou sans précision suffisante ne vaut pas acceptation.

| ID | Méthode de vérification | Critère d'acceptation | Preuves à conserver |
| --- | --- | --- | --- |
| FR-001 | Import de fixtures connues | Comptes, coordonnées et ordre de tous les éléments identiques aux données attendues | Sources, tableaux attendus/obtenus |
| FR-002 | Variantes lexicales et fautes de comptage | Toutes les variantes autorisées passent ; chaque faute et donnée excédentaire est refusée | Fixtures, statuts et diagnostics |
| FR-003 | Injection de chaque défaut décrit | Tous les défauts sont localisés ; seule la répétition terminale autorisée est reconnue comme marqueur | Sources annotées, résultats de contrôle |
| FR-004 | Comparaison avant/après contrôle invalide | Source inchangée, zéro réparation cachée, cause et indices présents pour chaque défaut connu | Empreintes, diagnostics structurés et lisibles |
| FR-005 | Import puis maillage de deux éléments | Deux éléments accessibles ; refus explicite ; zéro maillage partiel validé | Source, inventaire, statuts |
| FR-006 | Inspection de chaque cas valide | Toutes les conventions et références présentes et conformes à H-02 à H-05 | Métadonnées, figures annotées |
| FR-007 | Aller-retour S→N→W→N→S avec translation, échelle, Z non centré et flèches 0°/25°/50° sur un et deux éléments | Erreur maximale ≤ 10⁻¹² c ; une transformation globale ; positions relatives restituées | Coordonnées et résidus, transformations |
| FR-008 | Cas connus et ambigu résolu/incohérent | Tous les repères attendus trouvés ; zéro choix silencieux en ambiguïté ; indication incohérente refusée | Annotations attendues/obtenues, indications |
| FR-009 | Comparaison point par point à 0°, 25°, 50° | Écart à la formule et à son inverse ≤ 10⁻¹² c pour tous les points ; c inchangée | Tables de comparaison et résidus |
| FR-010 | Inspection des dimensions et de l'extrusion | Deux coordonnées de calcul par sommet ; même direction d'extrusion pour tous les éléments ; transformation effective vérifiée même sans Mach renseigné | Métadonnées et coordonnées |
| FR-011 | Références analytiques et contrôle des courbures/caractéristiques | Budgets H-07/H-08 respectés ; zéro nouvelle intersection, zéro extremum/oscillation absent de la référence ; pas de pas plus grand dans une zone de forte courbure que dans une zone comparable moins courbe, hors concentration explicitement prescrite | Courbes et références dans S/N/W, rayons/angles transformés attendus et mesurés, espacements |
| FR-012 | Mesure bidirectionnelle des deux écarts | Chaque maximum respecte son budget ; méthode et incertitude de mesure présentes, références comparées transformées dans le même repère S/N/W | Cartes d'écart, maxima, tolérances |
| FR-013 | Fermetures explicites/implicites et ouvertures suspectes | Culot fourni préservé ; culot ajouté seulement après confirmation ; tous les autres cas ambigus suspendus ; épaisseur conservée dans le budget caractéristique | Contours avant/après, segments ajoutés, confirmations |
| FR-014 | Inventaire et validation du maillage | Toutes les cellules quadrilatérales, indices structurés par bloc, frontières conformes au budget ; zéro appel à une résolution | Maillages, inventaire des types, trace du parcours |
| FR-015 | Cas fin/épais et inspection de topologie | Familles distinctes documentées ; culot/sillage explicites dans le cas épais ; zéro cellule effondrée dans le cas fin | Schémas de blocs, connectivités, rapport |
| FR-016 | Parcours exhaustif des raccords | Chaque face interne a exactement un partenaire ; correspondance bijective, orientations opposées et résidu géométrique ≤ 10⁻¹² c | Table des raccords, résidus et orientations |
| FR-017 | Inspection des régions | Chaque région applicable est identifiée, contient des cellules non dégénérées et possède ses mesures locales | Carte des régions, comptes et mesures |
| FR-018 | Variation séparée de chaque contrôle | Chaque paramètre valide est enregistré et a un effet mesuré sur sa grandeur cible ; chaque incompatibilité est refusée sans substitution cachée | Paramètres demandés/résolus, comparatifs |
| FR-019 | Mesure des premières cellules | Deux grandeurs distinctes présentes pour chaque face de paroi ; écart à la hauteur prescrite dans la tolérance de distribution du profil, explicitée avant génération | Cartes de hauteur et distance, limite appliquée |
| FR-020 | Cas avec/sans données physiques | Toute estimation présente contient formule, entrées, hypothèses et avertissement ; aucune valeur garantie ; absence d'échelle physique signalée | Rapport d'estimation, métadonnées |
| FR-021 | Deux distances à paramètres intérieurs fixes | Distance mesurée conforme à la demande dans le budget du profil ; paramètres intérieurs prescrits inchangés ; aucune promesse sur Cx | Paramètres, cartes de distance, comparatif |
| FR-022 | Contrôle exhaustif et défauts injectés | Toutes les aires > 0 ; zéro cellule invalide, face mal orientée, défaut de fermeture ou raccord manquant ; chaque injection critique est détectée | Mesures par cellule/face, défauts attendus/observés |
| FR-023 | Cas étiré aligné puis désaligné | Mesures et limites présentes pour toutes les cellules ; verdict conforme au profil et à l'orientation ; aucun rejet fondé uniquement sur le rapport d'aspect | Cartes, profil qualité et verdicts |
| FR-024 | Injection de défauts critiques et non critiques | Chaque défaut a localisation, mesure, limite et sévérité ; tout défaut critique entraîne l'échec et le marquage des sorties | Rapport, statuts commande/Python, sorties invalides |
| FR-025 | Trois niveaux sur fin et épais | Même reconstruction, même graphe de blocs et frontières ; nombres de subdivisions strictement croissants dans chaque direction/région applicable ; pas maximaux décroissants et première hauteur non croissante ; rapports de raffinement publiés | Paramètres des niveaux, tableaux des pas/comptes, rapports |
| FR-026 | Parcours complet commande/Python | Toutes les opérations disponibles ; mêmes statuts, connectivité et mesures à la tolérance numérique de reproduction près ; zéro résolution lancée | Transcriptions et résultats comparés |
| FR-027 | Relecture du VTK et de la représentation structurée | Tous les sommets/cellules exportés sont retrouvés ; blocs, frontières et interfaces conservés ; identifiants du rapport résolus à 100 % | Exports, résultats de relecture et correspondances |
| FR-028 | Lecture du rapport et inspection des figures | Rapport lisible depuis Python ; cinq vues présentes, unités/repères/légendes présents ; trois contours superposables | Rapport, figures et résultat de lecture |
| FR-029 | Deux générations dans le même environnement/version | Métadonnées complètes ; même connectivité et identifiants ; écart maximal des coordonnées ≤ 10⁻¹² c ; mêmes décisions de qualité | Sources, empreintes, manifestes et comparaison |
| FR-030 | Parcours dans un environnement CPU sans solveur externe | Import, génération, rapport et export aboutissent sans GPU ni exécutable/service CFD externe | Inventaire de dépendances et trace d'exécution |

| FR | Méthode | Critère d’acceptation | Preuves attendues |
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

| FR | Méthode | Critère | Preuves |
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

### Campagne d'acceptation à préparer

| Cas | Contenu et variantes | Exigences principalement exercées |
| --- | --- | --- |
| V-01 | Profil symétrique analytique à bord fin, rayon de BA connu ; Λ=0°, 25°, 50° | FR-006 à FR-017, FR-022/023 |
| V-02 | Profil cambré connu à bord fin ; Λ=0°, 25°, 50° | Identification non symétrique, reconstruction et maillage |
| V-03 | Profil à culot épais connu, angles et épaisseur de référence ; Λ=0°, 25°, 50° ; culot fourni puis ajouté explicitement | FR-008, FR-011 à FR-019 |
| V-04 | Corpus contenant chaque erreur lexicale, de comptage et géométrique des Edge Cases, avec lignes et indices attendus | FR-001 à FR-004, FR-024 |
| V-05 | Identification ambiguë des lèvres/fermeture ; sans indication, avec indication valide puis contradictoire | FR-008, FR-013 |
| V-06 | Deux éléments décalés et d'échelles différentes, puis intersection interéléments | FR-003, FR-005, FR-007, FR-009 |
| V-07 | Famille à trois niveaux pour V-01 et V-03 à 25° ; deux distances extérieures au niveau intermédiaire | FR-018 à FR-025 |
| V-08 | Cellules/raccords volontairement invalides, étirements alignés et désalignés ; paramètres incompatibles | FR-018, FR-022 à FR-024 |
| V-09 | Parcours commande/Python, VTP/VTM/VTS relus obligatoirement avec VTK ou PyVista, ouverture ParaView, reproduction V-01/V-03 et refus V-04/V-06 ; CPU seul | FR-026 à FR-030, FR-041 à FR-044 |
| V-11 | Modes/JSON-YAML, séparation des représentations, contrats 1.2/legacy, associations et refus actifs | FR-047–049, FR-056/057 |
| V-C01/V-C03 | Géométrie conique conditionnelle fin/épais, pentes non nulles puis limite nulle, après G-FORM ; hors du produit plan | FR-051–054 |
| V-10 | Campagnes JSON/YAML, produit/liste, inventaire, exécution séquentielle, échec, interruption/reprise et déduplication | FR-031 à FR-040 |

Les références, leurs éventuelles licences, paramètres, résultats attendus et méthodes de mesure doivent être figés avant les tests d'acceptation. Le choix précis des contours et des évaluateurs relève de D-04 ; il ne réduit pas cette couverture. Les preuves sont regroupées par cas, version, paramètres et identifiant FR pour permettre l'audit d'un succès comme d'un échec. La campagne complète combine inspection humaine des figures et contrôles numériques exhaustifs, sans jugement visuel seul pour les limites numériques.

### Périmètre de J0 et qualification complète

J0 utilise les références analytiques V-01/02/03 directement, avec un adaptateur de contour
réservé à la démonstration. Il vérifie les neuf configurations et, pour V-03, les deux modes
culot fourni/complété explicitement. Il ne dépend ni du lecteur général achevé, ni de toute
la CLI, de Python, des campagnes, de la reproduction ou des évaluateurs généraux de spline.
Les contrôles indépendants de cellules, interfaces, recouvrement, caractéristiques et fidélité
à l’oracle analytique restent obligatoires, avec bornes d’erreur ciblées et limites Q1
applicables. J0 n’attribue pas à lui seul le statut qualifié du produit ni FR-011/012 complets.
Les preuves comprennent mesures par cellule/face/interface, tableaux de coins et vues détaillées
aux deux lèvres et au BA ; les distributions et seuils utilisés sont archivés avant verdict.
Les culots 10⁻⁶ c et 10⁻¹⁰ c restent hors des neuf cas usuels obligatoires.

### Key Entities *(include if feature involves data)*

- **Source géométrique** : contenu original, empreinte, unités déclarées, éléments, points ordonnés et lignes source.
- **Géométrie contrôlée** : diagnostics, repères identifiés, indications utilisateur, convention de fermeture et segments ajoutés, sans écraser la source.
- **Transformation** : repère original/normalisé/normal à l'extrusion, corde et références originales, flèche, transformation globale et inverse.
- **Contour reconstruit** : géométrie de référence pour tous les niveaux, caractéristiques protégées, écarts et limites d'évaluation.
- **Configuration de génération** : contrôles demandés et résolus, tolérances, profil qualité, domaine et provenance/version.
- **Maillage multibloc** : sommets, cellules, régions, frontières, blocs et raccords orientés avec identifiants stables dans les livrables d'une génération.
- **Rapport qualité** : statut global, métriques locales et agrégées, limites, défauts et localisations, estimations explicitement qualifiées.
- **Campagne de maillage** : demande JSON/YAML, développement explicite, points logiques et clés de maillages effectifs, état progressif, tentatives et manifeste/tableau de reprise.
- **Objet de visualisation** : représentation, rôle, repère/unité, ID d’export et correspondances source/frontière/interface/diagnostic.
- **Famille et dossier de preuves** : niveaux, variations indépendantes de domaine, manifestes, exports et figures, comparaisons et traçabilité FR/cas.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001** : 100 % des fichiers invalides du corpus V-04 sont refusés à l’étape appropriée avec les localisations attendues ; les cas importables mais non certifiables ou non maillables reçoivent le statut attendu sans fausse invalidité lexicale ; 100 % des variantes valides sont chargées sans perte de données. V-06 conserve les éléments et refuse tout maillage multiélément.
- **SC-002** : Les neuf configurations de base V-01 à V-03 aux trois flèches produisent des maillages valides, sans défaut critique et dans les budgets géométriques documentés ; toutes les cellules et interfaces sont contrôlées.
- **SC-003** : V-05 bloque tout choix ambigu non renseigné et aboutit avec une indication valide ; zéro géométrie invalide n'est réparée silencieusement.
- **SC-004** : Deux familles, fine et épaisse, comportent chacune au moins trois niveaux respectant FR-025 ; une variation séparée sur deux distances extérieures respecte FR-021.
- **SC-005** : Chaque résultat valide du parcours complet possède le dossier de sortie complet et les cinq vues ; sa reproduction satisfait FR-029. L'utilisateur peut retrouver chaque défaut signalé dans le maillage ou le contour par son identifiant.
- **SC-006** : Tous les défauts critiques injectés de V-08 provoquent un échec explicite ; les deux parcours utilisateur donnent les mêmes conclusions. Aucune exécution d'acceptation ne lance de résolution aérodynamique.

- **SC-007** : V-10 satisfait FR-031 à FR-040 : inventaires équivalents, zéro génération au contrôle, une seule génération par maillage effectif, politiques d’échec et reprise vérifiées après mutations et interruption.
- **SC-008** : Les VTP/VTM/VTS de V-09 passent la relecture indépendante obligatoire et l’ouverture ParaView ; 100 % des IDs applicables sont résolus et les liens restent lisibles après déplacement. Le produit fonctionne sans VTK/PyVista installé.

- **SC-009** : V-11 vérifie la séparation source/logique/physique/bases/métriques et les contrats de modes, JSON/YAML, associations, versions et refus explicites ; les données planes restent FP64 sans dépendance conique.
- **SC-010** : L’étude et la revue G-FORM couvrent toutes les rubriques FR-050/051, avec sources primaires, hypothèses et limites de preuve ; décision acceptée ou no-go motivé, aucune construction détaillée avant go.
- **SC-011** : Après go G-FORM et J0, JG-C contrôle les deux cas coniques géométriques, mapping line of flight et limite de pente nulle avec tolérances dérivées/figées ; tant que le go manque, statut bloqué, jamais qualifié.
- **SC-012** : Inventaire et correspondances conservent les 45 FR et 83 tâches antérieures, leurs statuts et les revalidations ; roadmap distingue 20 exigences produit et toutes capacités non livrées.

Ces résultats prouvent une validité géométrique et une qualité mesurée selon les profils et cas qualifiés. Ils ne prouvent ni convergence d'un futur solveur ni incertitude de 10⁻⁴ sur Cx.

## Assumptions

### Hypothèses et dépendances

- H-01 à H-09 fixent les conventions proposées pour les détails non imposés par la demande. Elles sont identifiées comme hypothèses, sans rouvrir les décisions déjà fournies (monoélément, section normale, autonomie, familles topologiques, interfaces et sorties).
- Les utilisateurs sont des aérodynamiciens capables de fournir une échelle physique et des indications de repères lorsque nécessaire ; l'entrée ne contient que des points de périmètre ordonnés, pas un nuage de points à trier.
- La fonctionnalité n'exige ni réseau, ni données d'écoulement, ni licence d'un solveur externe. Une visualisation externe du VTK est possible, sans interface graphique complète à construire.
- La connaissance physique du rayon réel dépend de l'échantillonnage fourni ; une mesure de conformité aux données n'est pas une certification de la géométrie physique.
- Les budgets géométriques choisis sont des contrats de contrôle révisables et justifiés par les défauts qu'ils visent à détecter. Leur pertinence aérodynamique sera étudiée ultérieurement, indépendamment de leur respect numérique ici.

### Décisions réservées au plan

- **D-01** : Nombre de blocs et raccords exacts pour les familles fine/épaisse ; justification géométrique de C–H ou de l'équivalent, comportement aux lèvres et jonctions.
- **D-02** : Reconstruction, protection des angles, identification automatique, redistribution, génération et lissage ; traitement explicite des transitions et configurations irréalisables, sans modifier le contrat géométrique.
- **D-03** : Définitions opérationnelles et limites du profil qualité, tolérances de distribution et de distance extérieure, précision des évaluateurs et stratégie de détection des dégénérescences. Le plan doit les justifier sur les géométries cibles et figer des valeurs numériques avant réalisation ; aucun contrôle sans seuil ne permet une acceptation.
- **D-04** : Jeux de référence exacts, mesures de rayon/courbure/distance, corpus de défauts et organisation des preuves ; vérifier les budgets proposés, et modifier explicitement cette spécification si une incompatibilité est démontrée.
- **D-05** : Forme des commandes et interfaces Python, organisation des données, formats internes et variante VTK, schémas de rapport, rendu des figures, dépendances et stratégie de reproduction.
- **D-06** : Règles quantitatives des niveaux de raffinement, maintien des concentrations, première hauteur et nombre de couches, raccordement du domaine extérieur ; aucune exigence d'imbrication exacte des sommets entre niveaux.
- **D-07** : Formule éventuelle d'estimation de y+, grandeurs physiques nécessaires, limitations et ressources ; aucune promesse de performance non mesurée à ce stade.

- **D-08** : Contrat de campagne, formats JSON/YAML, identité effective et compatibilité de reprise ; règles détaillées dans contracts/campaign.md.
- **D-09** : VTP par rôle et correspondances de diagnostics, qualification indépendante et ouverture ParaView ; règles détaillées dans contracts/bundle.md.

### Questions réellement bloquantes et état du dépôt

Aucune question fonctionnelle supplémentaire n’est nécessaire pour préparer J0 plan après revue : les décisions de périmètre sont fournies. G-FORM reste une décision scientifique ouverte, bloquant le contrat métrique et la construction coniques ; elle n’est pas réputée résolue par les hypothèses planes. Les ambiguïtés d'une géométrie particulière doivent être traitées à l'utilisation suivant FR-008/013, pas par une question générique préalable.

La constitution `.specify/memory/constitution.md`, initialement un gabarit, a été établie en version 1.0.0 le 2026-10-30 par le mécanisme `speckit-constitution`, sur autorisation explicite de la demande de planification courante. Il ne s'agit pas d'une approbation humaine antérieure. Le point de gouvernance signalé à la rédaction initiale est ainsi traité. Aucun `AGENTS.md` applicable ni `.specify/extensions.yml` n'a été trouvé ; aucun hook n'est donc exécuté. Le modèle actif a été résolu par `specify preset resolve spec-template` vers `.specify/templates/spec-template.md`. La numérotation configurée est séquentielle ; cette fonctionnalité est la première.

### Objectif produit et feuille de route faisant autorité

Le contrat global est dans [docs/product-roadmap.md](../../docs/product-roadmap.md) :
solveur sectionnel autonome **2D/2.5D/2.75D** compressible pleinement turbulent SA-neg,
Cz/Cx/Cm, pression/frottement, restitution line of flight, campagnes CLI/Python.
Domaine M=0,2–0,89, Reynolds de plusieurs millions à plusieurs dizaines de millions,
Λ=0°–50°, attaché/faiblement décollé, monoélément fin/épais puis multiélément futur.
Cz/Cm priment ; la cible d’incertitude numérique Cx≈10⁻⁴ est distincte de l’accord expérimental.
C++/CUDA Linux/WSL2/CLion, référence CPU FP64, usage quotidien Python, RTX 4070 Ti Super
16 Go et A16-8Q 8 Go partagés sont les cibles produit. Aucun VLM ni solveur d’aile requis.

001 reste le mailleur plan et ses parcours, plus étude G-FORM et jalon géométrique conique
conditionnel JG-C ; aucune équation Euler/RANS/SA-neg, pression ni coefficient n’est implémenté
ici. Les futurs travaux, y compris vérification conique de l’uniforme et de la limite 2.5D,
ont des dépendances dans la feuille de route. [conical-study.md](conical-study.md) est une
annexe scientifique de cette spécification, pas une fonctionnalité concurrente.

### Historique des corrections de cohérence

Le 2026-10-30, à la demande de préparation du plan : H-03 sépare Y et η sans déplacer le quart de corde ; H-05 définit S/N/W et inverse d'abord la normalisation ; H-07/H-08 imposent des comparaisons à la référence dans le même repère, sans interdire la variation volontaire des rayons/angles sous compression. FR-007/009/011/012 conservent leurs identifiants et leur couverture, avec précisions de leurs preuves. Les trente exigences et V-01 à V-09 sont conservés. Constitution établie sur autorisation courante. Les budgets chiffrés ne sont pas relâchés.

### Compléments autorisés du 2026-10-31

FR-001 à FR-030 sont conservées sans suppression ; FR-031 à FR-045, US5, V-10 et
SC-007/008 complètent leur couverture. FR-027/028 restent applicables aux nouveaux VTP.
La dépendance YAML générique est justifiée au plan/research ; VTK/PyVista est obligatoire
uniquement pour la qualification, et ParaView fait l’objet d’une vérification d’ouverture.
Les campagnes courantes portent sur le maillage ; aucune campagne de vol ni résolution
n’entre dans le périmètre. Les tâches sont préparées sur autorisation, sans réalisation.

### Réconciliation R2 — correspondance et état

FR-001–045 et SC-001–008 sont conservés ; FR-009/010/022 sont précisés comme planaires
sans retirer leurs critères. FR-046–060, SC-009–012 et US6 complètent le cadrage. Les écarts,
anciennes/nouvelles tâches et revalidations sont dans reconciliation.md. Aucune FR n’est
déclarée satisfaite par la seule présence de ces documents. La constitution 1.1.0 ajoute
le cadre produit et la porte scientifique sans changer les budgets plans.
