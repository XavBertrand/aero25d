# Géométrie, évaluateurs et références de vérification

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

Date : 2026-10-30. Document de conception associé au [plan](plan.md), décisions D-02 et D-04. Aucun algorithme, prototype ou essai décrit ici n'a été exécuté. Les valeurs ci-dessous sont des contrats proposés pour l'implémentation et les essais futurs, sans résultat numérique revendiqué.

## 1. Lecture, provenance et arithmétique

Le lecteur C++ conserve les octets source, leur empreinte, chaque lexème décimal, le numéro de ligne (base 1), le numéro d'élément/point source (base 1) et les coordonnées converties. Il effectue une lecture par états : compte d'éléments, compte de points, exactement ce nombre de paires, élément suivant, fin. Il retire uniquement le suffixe `#`, ignore les lignes devenues vides et exige une conversion complète de chaque lexème. Les entiers sont contrôlés avant allocation ; dépassement d'entier ou multiplication d'une taille provoque un refus. La mémoire demandée est estimée avant génération, jamais réduite automatiquement.

Les lexèmes restent immuables. Avant identification de la corde, une précondition globale utilise le centre de la boîte englobante et sa plus grande dimension non nulle. Après identification, les calculs sensibles sont effectués dans un repère de travail global q=(p−o)/c, avec origine proche de la géométrie (quart de corde) et c original strictement positif ; pour l'import multiélément, même origine et même échelle pour tous les éléments. Cette précondition numérique interne est distincte de la normalisation de sortie utilisateur et possède sa transformation dans les métadonnées. Aucune rotation de la corde n'est appliquée implicitement.

La conversion est binaire64, avec arithmétique d'expansions pour les différences/orientations difficiles. Les tests d'orientation utilisent un filtre d'erreur puis `orient2d` adaptatif exact sur les coordonnées flottantes représentées, sans epsilon de colinéarité. Pour la validité topologique source, les prédicats portent sur les valeurs source représentées : une translation est évaluée en expansions et une mise à l'échelle par puissance de deux, quand nécessaire, ne doit perdre aucun bit ni provoquer de sous-flux. Un signe calculé uniquement sur des coordonnées normalisées arrondies n'est pas un certificat du signe source. Cette précision exacte concerne les données représentées, pas les réels décimaux idéaux. La méthode et ses conditions d'arrondi sont documentées par [Shewchuk, prédicats géométriques robustes](https://www.cs.cmu.edu/~quake/robust.html). Son code de prédicats est explicitement publié dans le domaine public ; seule la portion nécessaire est envisagée, avec provenance et tests, sans intégrer Triangle ni un mailleur externe.

Les calculs de distances/curvatures à borner utilisent des intervalles binaire64 à arrondi extérieur ; les options de compilation excluent `fast-math` et les transformations qui invalident les bornes. Une bibliothèque d'intervalles n'est pas indispensable : le sous-ensemble +, −, ×, /, racine et `nextafter` nécessaire aux courbes polynomiales est limité et testé. Les certificats ne supposent pas une garantie universelle en ulp de la bibliothèque mathématique système. Sur Λ∈[0°,50°], convertir les degrés en radians avec un intervalle de π, puis borner sin et cos par développements de Taylor jusqu'au degré 22 (degré 21 pour les termes impairs de sin), avec leur reste de Lagrange et arrondis extérieurs. L'intervalle de π est figé par des bornes décimales rationnelles documentées ; l'enveloppe trigonométrique et la valeur binaire64 choisie sont archivées. Une approximation libm peut servir à proposer la valeur centrale, mais le certificat utilise l'enveloppe indépendante. Les bornes de vérification intègrent le reste, la conversion d'angle et chaque opération. Pour sqrt, encadrer la proposition binaire64 par des voisins `nextafter`, vérifier par carrés bornés que les extrémités enferment l'argument positif, puis élargir au besoin ; la proposition libm seule ne constitue pas une preuve.

### Grandes coordonnées et détails très petits

Le déplacement de l'origine ne recrée pas des chiffres perdus lors de la conversion. Pour une entrée décalée de 10¹⁶ avec des écarts de 10⁻⁶, binaire64 peut confondre les points : le lecteur compare les valeurs décimales canoniques (signe, significande entière, exposant ; `1`, `1.0` et `1e0` sont équivalents) et les valeurs converties, puis refuse `PRECISION_LOSS` si deux coordonnées décimalement distinctes deviennent indiscernables. Il ne les classe pas comme doublons intentionnels et ne les fusionne pas. La conversion décimale vers binaire64 doit être correctement arrondie, avec un intervalle conservateur d'au plus un demi-ulp autour de la valeur choisie, élargi pour les cas limites et sous-normaux. Cette garantie est vérifiée sur les lexèmes canoniques et des cas de référence ; la simple utilisation d'une routine sans contrat d'arrondi documenté ne suffit pas. Une estimation conservatrice de l'erreur de conversion, ramenée à c, est comparée aux budgets demandés ; si elle ne permet pas de trancher, statut `UNCERTIFIABLE_PRECISION`.

Les petits espacements/épaisseurs représentables restent distincts. Aucun seuil de proximité ne transforme deux lèvres en une pointe. Une épaisseur trop petite pour les distributions ou les limites de dégénérescence peut être refusée par le mailleur, tout en restant importable et inspectable. Les tests de contact topologique sont séparés des avertissements de quasi-contact : un segment voisin sans intersection n'est pas déclaré intersectant à cause de sa seule distance.

## 2. Fermeture, intersections et repères

La répétition terminale exactement égale au premier point est un marqueur autorisé : l'indice source est conservé et une correspondance le relie au premier sommet de travail. Les autres doublons exacts, segments de longueur nulle et retours sur un segment adjacent sont invalides. Sans répétition terminale, la fermeture implicite fait partie du contour à examiner ; elle n'est pas ajoutée comme une correction cachée.

Les segments sont filtrés par boîtes englobantes puis testés avec orientations exactes, projection ordonnée pour colinéarité et appartenance aux segments. Une rencontre de segments consécutifs n'est autorisée qu'à leur sommet commun. Croisement propre, recouvrement colinéaire, contact tangent non adjacent et contact entre éléments sont diagnostiqués séparément. Un balayage spatial/BVH accélère les candidats mais ne décide jamais de la validité. Les arcs reconstruits sont contrôlés séparément par subdivision Bézier bornée ; une paire non séparée après la limite de précision constitue un refus de certification, jamais une absence d'intersection présumée.

L'aire polygonale signée indique le sens. Le contour de travail peut être inversé pour être trigonométrique dans (X,Z) ; les indices source et l'action d'inversion restent conservés. Pour un contour reconnu, l'extrados est la branche supérieure et l'intrados la branche inférieure entre BA et lèvres. La désignation n'est pas obtenue par le seul signe de Z : un décalage vertical ne doit rien changer.

### Identification automatique, confiance et indications

L'identification utilise une liste de candidats, pas une réduction aveugle à un unique min/max de X : support amont pour le BA, régions aval, ruptures de tangente et segment de culot possible. Les voisins sont examinés à plusieurs tailles de fenêtre (3, 5, 9 points distincts), dans le repère normalisé. Les contraintes sont : BA commun aux deux branches ; lèvres distinctes ou pointe explicitement classée ; branches simples sans croisement ; orientation et parcours cohérents ; culot disjoint des branches hors lèvres. Un seul ensemble satisfaisant toutes ces contraintes peut être accepté automatiquement.

Au BA lisse, le minimum de X est recherché dans une région candidate ; une abscisse égale à plusieurs points, plusieurs régions de courbure plausible ou une discordance entre fenêtres produit `AMBIGUOUS_FEATURE`. Un rayon estimable n'est pas une preuve qu'un point est le BA physique. Les indications peuvent donner des indices source ou des positions sur segments source avec paramètre [0,1], le type fin/épais, les lèvres, les coins protégés et la confirmation du culot ajouté. Leur interprétation figure dans `resolved.json`. Elles ne permettent ni croisement, ni suppression de points, ni déplacement imposé d'une lèvre. Une indication incohérente échoue avec la contrainte violée.

Le culot ajouté est une droite entre les deux lèvres reconnues seulement lorsque `hints.confirm_straight_base_closure=true` a été explicitement fourni. Les segments déjà présents restent la référence. Une fermeture ailleurs qu'au culot ou un raccord suspect n'est pas réparé automatiquement.

## 3. Reconstruction retenue

### Courbes et caractéristiques protégées

La première version utilise des splines cubiques paramétriques interpolantes, exprimées ensuite en segments de Bézier cubiques pour les évaluateurs. Le paramètre initial est la longueur cumulée des cordes, normalisée par arc. Les angles protégés découpent les arcs ; le culot rectiligne reste une droite exacte. Les points caractéristiques restent des nœuds imposés, pas des points libres d'un ajustement.

Pour assurer le passage et la tangente au BA identifié, les arcs intrados–BA et BA–extrados sont raccordés au BA avec une tangente commune explicitement construite. Chaque arc est C² à ses nœuds internes ; le raccord au BA est C¹ et ses deux courbures unilatérales sont évaluées et publiées. Aucune continuité C² globale n'est revendiquée au BA si elle n'est pas satisfaite. Les dérivées terminales sont estimées par un ajustement local cubique sur 5 points, contrôlé par comparaison sur 7 et 9 points quand disponibles. Au minimum horizontal identifié et confirmé, la tangente est verticale dans le repère source ; sa norme paramétrique provient de l'ajustement. Un profil orienté différemment exige la convention de BA correspondante et ne subit pas une rotation cachée.

Les arcs se raccordent aux coins protégés avec deux tangentes distinctes ; ils ne lissent jamais un angle du culot. Les conditions aux extrémités sont documentées ; les différences entre les estimations 5/7/9 points donnent un indicateur de sensibilité des tangentes, pas une erreur scientifique garantie.

### Certification de forme

Le choix d'une spline cubique ne garantit pas l'absence d'oscillation. Après construction, chaque arc est contrôlé avant redistribution :

1. Les dérivées polynomiales sont converties en Bernstein ; subdivision et isolation des racines déterminent les extrema de X et Z ainsi que les signes de courbure sur les intervalles où une référence les impose.
2. Aucun nouvel extremum ne peut apparaître en dehors d'une région d'extremum reconnue dans les données ou dans la référence analytique de qualification. Les subdivisions incertaines ne sont pas acceptées comme monotones.
3. Les arcs doivent être réguliers : norme de la dérivée strictement positive avec borne inférieure certifiée ; les contrôles de distance et d'absence d'intersection sont exécutés.
4. Les budgets de H-07/H-08 doivent être respectés. Si les données sont trop pauvres ou la spline oscille, `RECONSTRUCTION_UNCERTIFIED` est retourné. Aucun passage automatique à une polyligne, suppression de points ou relâchement des seuils n'est autorisé.

Ce choix privilégie un comportement vérifiable sur les profils de qualification. Une reconstruction plus générale pourra faire l'objet d'une évolution explicite ; elle n'est pas dissimulée derrière un repli automatique.

### Trois notions de fidélité

- **Fidélité polygonale** : distance de Hausdorff entre polyligne source et spline dans le repère original, budget 10⁻⁴ c. Ce contrat mesure une modification des données.
- **Fidélité à une référence analytique** : les points source sont échantillonnés sur une courbe connue ; la reconstruction et la paroi maillée sont comparées à cette courbe, et rayon/angles sont contrôlés. Le défaut de la polyligne source elle-même est mesuré séparément.
- **Forme réelle inconnue** : aucune donnée entre deux points ne fournit une courbe physique unique. On peut certifier le contrat polygonal et publier une sensibilité de rayon ; on ne peut certifier le rayon réel d'un profil peu échantillonné.

## 4. Distances et courbures avec bornes

### Distance maximale, sans certification par une grille arbitraire

Pour un segment Bézier de degré n, comparer ses points de contrôle Pi aux contrôles Li de la droite paramétrée entre les deux extrémités. La différence est un Bézier de contrôles Pi−Li ; par convexité, sa norme est majorée par max ‖Pi−Li‖. Après subdivision de de Casteljau, cette borne fournit une polyligne Aδ avec dH(A,Aδ)≤δ. C'est une borne de distance pour les deux ensembles par correspondance paramétrique, pas seulement la distance des contrôles à la droite infinie. La propriété d'enveloppe convexe et les subdivisions sont décrites par la [documentation algorithmique du projet bezier](https://bezier.readthedocs.io/en/stable/algorithms/geometric-curve-intersection.html) ; aucune dépendance à ce paquet n'est introduite.

Les distances point–segment sont évaluées avec intervalles et projection bornée dans [0,1]. Pour une direction polyligne Aδ→Bδ, un sous-segment source S est couvert par son milieu m et rayon r=longueur(S)/2 ; la fonction distance à Bδ étant 1-lipschitzienne, `dist(m,Bδ)−r` et `dist(m,Bδ)+r` bornent les distances sur S. Les distances échantillonnées fournissent aussi des minorants du maximum. Une file prioritaire subdivise les sous-segments dont le majorant peut encore dominer ; les deux directions sont traitées. Une simple comparaison des seuls sommets des polylignes est interdite : le maximum dirigé peut être intérieur à un segment.

Les bornes Hausdorff obtenues pour les polylignes sont élargies de δA+δB et des intervalles d'arrondi. Le résultat est un intervalle [L,U] contenant la distance maximale des courbes. Pour un seuil T : succès seulement si U≤T ; échec si L>T ; sinon subdivision jusqu'à une largeur ≤min(T/100,10⁻⁸ c). Si l'intervalle chevauche encore T, le verdict est indéterminé et la génération n'est pas déclarée valide. Limites initiales : 2×10⁶ sous-intervalles et profondeur 60 ; dépassement = `EVALUATOR_LIMIT`, pas tolérance agrandie. Ces limites limitent les ressources, pas la précision revendiquée.

Les évaluateurs de qualification sont distincts des algorithmes générant les nœuds. Les bornes de rayon/angle ne sont pas déduites d'un nombre fixe de points de visualisation. Une erreur à 10⁻¹² c pour une caractéristique ou un aller-retour est testée directement par des opérations bornées sur les points concernés ; on ne force pas l'évaluateur global de Hausdorff à cette précision sans nécessité.

### Courbure et rayon au BA

Sur chaque côté régulier, κ=(x′z″−z′x″)/(x′²+z′²)^(3/2), R=1/|κ|. Les dérivées sont exactes pour la spline représentée, évaluées avec intervalles. Pour les extrema sur une région, isoler les racines du numérateur polynomial de la dérivée de κ² puis borner chaque intervalle ; une boîte contenant x′²+z′²=0 est un défaut de régularité. Le calcul au BA utilise les deux limites unilatérales ; sur les références analytiques, chacune doit être à ≤1 % du rayon de référence et l'écart relatif entre elles ≤1 %. Une boîte ne permettant pas de décider avant la limite de ressources retourne « non certifiable ».

La largeur d'intervalle visée pour le rayon analytique est 0,01 % du rayon de référence, soit cent fois plus fine que le budget de 1 %. Pour les angles : intervalle de largeur ≤0,001° avant comparaison à 0,1°. Les critères angulaires utilisent des produits scalaires, déterminants et normes bornés, avec comparaison à des seuils sin/cos calculés par le même évaluateur à reste borné ; aucun `atan2` libm n'intervient dans le certificat. Un angle affiché par `atan2` reste une valeur descriptive accompagnée de son intervalle, obtenu par inversion monotone des bornes trigonométriques. Les simples différences finies de courbure ne servent pas de certificat.

Sur données inconnues, calculer des ajustements circulaires locaux sur fenêtres 5/7/9 et un ajustement polynomial local ; publier les rayons et leur dispersion. Une dispersion >1 % ou un conditionnement ne permettant pas de décider entraîne `radius_status=uncertifiable`, sans prétendre que 1 % borne l'erreur physique. Une paroi conforme aux points peut rester inspectable, mais le cas ne passe pas une qualification exigeant un rayon connu.

### Transformation de flèche

Pour A=diag(cosΛ,1), rΛ=A(r−(Xref,0)), le rayon de référence est celui de rΛ, pas celui de r. Si t est la tangente unitaire source, κΛ=det(A)κ/‖At‖³ ; cette relation est obtenue directement par la formule de courbure. Au BA des références ci-dessous, t est vertical : RΛ=R/cosΛ. Les angles transformés sont calculés entre les tangentes A t1 et A t2. Une normalisation par une longueur L divise tous les rayons par L et laisse les angles inchangés. Le test contrôle à la fois la courbe transformée attendue, les rayons transformés et la bonne remise en unités ; il n'interdit pas la compression intentionnelle de X.

## 5. Redistribution tangentielle

Après reconstruction figée, intégrer l'abscisse curviligne et une densité positive sur chaque arc. Le moniteur initial est `w(s)=1 + A_LE exp(−(s_LE/σ_LE)²) + A_TE exp(−(s_TE/σ_TE)²) + A_k sqrt(c |κ(s)|)`, avec amplitudes non négatives, largeurs positives et valeurs demandées/résolues archivées. Les distances s_LE et s_TE sont mesurées le long de l'arc, pas sur X. La pondération de courbure agit aussi à forte courbure hors BA/BF ; les coins sont des extrémités imposées et ne sont pas traités comme une courbure infinie.

Les nœuds équidistribuent l'intégrale de w. Les quadratures/inversions adaptatives possèdent une borne d'erreur cumulée ≤10⁻¹⁰ c sur l'abscisse curviligne, plus stricte que le budget paroi 10⁻⁵ c. Les nombres d'intervalles de chaque arête sont résolus une fois par le graphe de contraintes des interfaces ; aucune génération indépendante de deux côtés d'un raccord. Des nœuds peuvent être ajoutés seulement dans le cadre d'une politique de résolution explicite ; si la résolution prescrite ne permet pas de respecter le budget, échec et proposition lisible, sans augmentation silencieuse.

L'intégrale cumulée et sa fonction inverse sont conservées comme règle de distribution de famille. Le maillage de paroi reste polygonal ; sa distance à la spline est certifiée avec l'évaluateur précédent. Les figures n'ont pas de rôle de certification numérique.

## 6. Références analytiques exactes retenues

Toutes les formules de cette section sont des fixtures synthétiques définies pour le projet, sans téléchargement de contour ni licence de base de données tierce. Elles qualifient la géométrie ; elles ne sont pas des profils aérodynamiquement validés.

Avec a=0,16, t∈[−1,1] :

`X(t)=t² ; Z(t)=a t(1−t²) + b t³ + m t²(1−t²)`.

Le parcours va de la lèvre inférieure t=−1, par BA t=0, à la lèvre supérieure t=1 ; les valeurs m et b déterminent les trois familles. BA=(0,0), BF=(1,0), c=1 et quart de corde=(0,25,0). Le rayon BA exact est R=a²/2=0,0128 puisque r′(0)=(0,a), r″(0)=(2,2m). Le terme de cambrure ne change pas ce rayon. Les tangentes aux lèvres ont pentes : supérieure `−a+1,5b−m`, inférieure `a−1,5b−m`. L'épaisseur BF vaut 2b ; le cas b=0 se ferme en pointe, avec angle fini. Le cas b>0 se ferme par une droite verticale entre les deux lèvres.

Le contour fourni emploie t_k=−1+2k/2048 pour k=0…2048. Le point terminal répète le point initial uniquement lorsque b=0 ; le lecteur conserve ce marqueur. Dans la variante épaisse à culot fourni, ajouter 31 points intérieurs régulièrement espacés sur le culot, puis le marqueur terminal. Dans la variante épaisse à culot à compléter, conserver seulement les 2049 points de l'arc et la confirmation explicite de fermeture. Écrire les décimaux avec précision suffisante à la relecture binaire64 et archiver les valeurs réelles attendues/arrondies. Ce choix de 2048 intervalles est une résolution de fixture proposée, pas une preuve que les budgets sont satisfaits ; l'oracle analytique et ses bornes doivent le vérifier.

| Cas | Définition précise et variantes | Contrôles et preuves attendus |
| --- | --- | --- |
| V-01 | a=0,16 ; b=m=0. Profil symétrique à pointe. Λ=0°,25°,50°. Sens direct/inversé et départ cyclique différent. | Points/repères attendus, R=0,0128 source, RΛ=0,0128/cosΛ ; angles de pointe issus des deux tangentes ; certificats de distance, maillages et cartes qualité. |
| V-02 | a=0,16 ; b=0 ; m=0,08. Cambrure moyenne m X(1−X), maximum 0,02. Même échantillonnage et trois flèches. | Distinction extrados/intrados hors symétrie, même rayon BA ; pentes −0,24 et +0,08 aux lèvres ; courbes/rayons transformés contrôlés. |
| V-03 | a=0,16 ; b=0,01 ; m=0. Culot de hauteur 0,02 ; pentes ±0,145 en valeur absolue. Culot fourni/ajouté, Λ=0°,25°,50°. | Épaisseur et face préservées ; angles calculés entre tangente de branche et verticale ; preuves locales des coins et du sillage ; trois flèches et deux modes de fermeture. |
| V-04 | Mutations déterministes de V-01/V-03, identifiées I-01 à I-24 ci-dessous. | Table de statuts/indices/lignes attendus et observés ; empreinte source avant/après ; aucune réparation. |
| V-05 | Ellipse C(θ)=(0,5+0,5 cosθ ; 0,1 sinθ), θ_k=2πk/192 ; aucun coin ne distingue deux lèvres. Trois essais : sans indication, indications BA=θπ et lèvres θ=±π/12, puis indication BA égale à une lèvre. | Sans indication : ambiguïté. Avec indications cohérentes : contrôle/identification et reconstruction peuvent poursuivre ; l'arc aval est identifié comme culot fourni courbe, sans remplacement par droite. Indication contradictoire : refus. La compatibilité de ce culot courbe avec le premier mailleur doit être traitée explicitement comme acceptation géométrique ou refus topologique ; V-05 teste l'import/identification, sans exiger un maillage de l'ellipse. |
| V-06 | Deux contours : V-01 ; 0,3×V-03+(1,2 ; 0,2). Variante intersectante : second contour V-01 déplacé de (0,5 ; 0). | Comptage complet, transformations globales et rapports de positions conservés ; refus multielement au maillage ; intersections interéléments dans la variante chevauchante. |
| V-07 | V-01 et V-03 à 25°, trois niveaux conformément au plan ; deux distances extérieures au niveau médian, autres paramètres prescrits constants. | Graphe de blocs/contour identiques entre niveaux, pas et comptes par région, première hauteur, croissance, culot/sillage, frontières extérieures ; aucune conclusion sur Cx. |
| V-08 | Mutations du maillage et des interfaces décrites ci-dessous ; au moins une par type. | Défauts attendus/observés, localisations, bornes et sévérités ; statut invalide et zéro export utilisable par défaut. |
| V-09 | Exécuter les deux parcours commande/Python sur V-01 et V-03, puis les refus V-04/V-06 ; CPU sans CUDA ni solveur installé. | Contrats, exports/relectures, cinq vues, manifestes de reproduction, comparaison des statuts, connectivités et coordonnées ≤10⁻¹² c. |

### Corpus invalide V-04 : mutations prescrites

Chaque mutation part d'une source valide explicitement identifiée ; une seule faute est introduite sauf indication contraire. Le générateur futur des fixtures écrit le fichier et un manifeste des lignes/indices attendus, sans calculer l'attendu avec le lecteur testé.

| ID | Mutation | Attendu |
| --- | --- | --- |
| I-01 | Fichier vide | Compte d'éléments manquant |
| I-02 | Nombre d'éléments 0, puis −1, puis 1,5 : trois variantes | Entier strictement positif requis |
| I-03 | Nombre de points 0, −1, 3,5 : trois variantes | Entier strictement positif requis |
| I-04 | Compte de points augmenté de 1 | Fin prématurée ou compte incohérent |
| I-05 | Compte de points diminué de 1 | Données résiduelles |
| I-06 | Compte d'éléments augmenté de 1 | Élément manquant |
| I-07 | Ligne de coordonnées avec un nombre puis trois nombres | Arity incorrecte |
| I-08 | Lexème `1.2oops`, puis séparateur décimal virgule | Nombre invalide |
| I-09 | Un X=NaN puis Z=+Inf puis X=−Inf | Valeur non finie |
| I-10 | Point consécutif répété au milieu de l'extrados | Doublon/segment nul avec deux indices |
| I-11 | Répétition d'un point non adjacent | Contact/doublon intérieur |
| I-12 | Deux sommets permutés créant un croisement propre | Intersection et deux segments |
| I-13 | Parcours colinéaire revenant en arrière | Recouvrement de segments |
| I-14 | Sommet placé exactement sur un segment non adjacent | Contact non adjacent |
| I-15 | Trois points distincts colinéaires comme élément | Aire/corde ou géométrie dégénérée |
| I-16 | Tous les points égaux | Corde nulle, doublons |
| I-17 | Retrait d'un arc au BA, ouverture manifeste hors BF | Fermeture suspecte/ambiguë, maillage bloqué |
| I-18 | Deux lèvres distinctes à distance 10⁻¹⁰ c | Aucune fusion ; import conservé ; refus éventuel motivé du maillage |
| I-19 | Source translatée de 10¹⁶ avec détails 10⁻⁶ perdus en conversion | Precision loss explicitement diagnostiquée |
| I-20 | Entier de taille supérieure à uint64 et taille d'allocation débordante | Refus avant allocation |
| I-21 | Deux éléments tangents en un point | Contact interéléments |
| I-22 | Deux éléments avec segments superposés | Recouvrement interéléments |
| I-23 | V-01 décimé en ne conservant que t={−1 ; −0,5 ; 0 ; 0,5 ; 1} | Rayon réel non certifiable ; aucun succès de qualification du rayon |
| I-24 | Λ=−1°, 50,01°, NaN ; h1≤0 ; couches non entières ; croissance non finie | Paramètre invalide localisé, aucune substitution |

Le manifeste V-04 distingue obligatoirement l'étape contrôlée : lecture, validité géométrique, certification de reconstruction ou paramètres de génération. Les mutations I-18 et I-23 ne sont pas des erreurs de lecture : I-18 doit rester importable avec ses deux lèvres distinctes, puis peut échouer explicitement au maillage ; I-23 peut rester inspectable mais échoue à la certification de rayon. I-24 échoue lors de la validation des paramètres concernés. Une absence de certification n'est jamais présentée comme un rejet lexical de la source. Chaque variante possède un statut et une étape attendus ; une campagne ne peut se déclarer réussie par le seul comptage de refus de parseur.

Les variantes lexicales positives utilisent commentaires en tête/après valeurs, lignes vides et notation scientifique sur une source inchangée. Le point terminal répété autorisé fait partie des contrôles positifs, pas d'I-10.

### Défauts injectés V-08

- Permuter deux sommets d'une cellule : croisement et orientation invalide.
- Écraser deux sommets adjacents : arête nulle et dégénérescence.
- Déplacer un coin pour aire négative, puis pour Jacobien bilinéaire négatif en un coin même si l'aire totale reste positive.
- Déplacer une face interne sans déplacer son partenaire ; casser une correspondance d'indice ; répéter un partenaire ; supprimer un partenaire ; inverser l'orientation ; créer un trou et un chevauchement dans deux variantes distinctes.
- Fournir une cellule d'aire positive trop faible par rapport aux espacements locaux pour la limite de dégénérescence du profil.
- Construire des rectangles de rapports 10³ et 10⁵ avec direction longue tangentielle puis normale/désalignée : contrôler le classement contextualisé, sans seuil de rejet d'aspect seul.
- Injecter des sauts d'espacement, de l'obliquité et une première hauteur incompatible avec la demande juste de part et d'autre de chaque limite du profil qualité.
- Déplacer un point de paroi hors budget, arrondir un coin de culot et supprimer sa face dans trois variantes.

Les valeurs à la limite, un ulp sous et un ulp au-dessus sont prévues pour les comparateurs numériques. Aucun de ces fichiers n'a été créé ou exécuté pendant la rédaction du plan.

## 7. Preuves et limites

Pour chaque cas : source originale, formule/version du générateur de fixture, paramètres, précision d'écriture, transformée analytique, oracle d'indices, configuration demandée/résolue, profils de limites, intervalles [L,U], statistiques d'évaluation, statuts, cartes locales et figures. Le rapport doit distinguer « implémenté », « vérifié sur données/oracle » et « validé pour un usage ».

Les risques encore expérimentaux sont : comportement des splines aux zones très sous-échantillonnées ; sensibilité du rayon BA aux jets clamped ; coût de la certification Hausdorff près d'un seuil ; préservation de la forme pour contours bruyants et très petites épaisseurs ; compatibilité des topologies avec les neuf configurations de qualification. Aucun succès de prototype ou de performance n'est supposé. Un échec doit produire les preuves qui permettent une révision explicite de conception, pas déclencher une relaxation automatique des tolérances.

## Séparation J0 / évaluateurs généraux — 2026-10-31

Pour J0 uniquement, les références polynomiales ci-dessus servent directement de contours.
Un oracle de test C++ indépendant vérifie positions, tangentes/rayons transformés et distances
polynôme–segments par subdivision/borne de dérivées. Il produit des bornes et refuse une
indétermination ; un simple échantillonnage graphique ne certifie pas un budget.
Ce socle ciblé suffit aux contrôles de validité de la démonstration sans attendre le lecteur,
la reconstruction clamped ni l’évaluateur général Hausdorff/Bézier des sections 1–5.
J0 ne démontre pas la conformité de ces algorithmes généraux ; E2/J1 et les essais US1
restent nécessaires pour FR-011/012. Les transformations, blocs et contrôles de cellules
proviennent déjà du noyau commun, sans mailleur spécifique à la campagne ou à Python.

## R2 — source line of flight et futur conique

S est la section originale line of flight ; les formules de projection/compression et leurs
oracles ci-dessus ne s’appliquent qu’au modèle `planar-extrusion-v1`. La topologie logique
n’est pas une géométrie physique. Les données coniques candidates et pentes/lois de similitude
sont étudiées dans conical-study.md, sans figer leur métrique ici. Après G-FORM seulement,
JG-C vérifiera mapping/inverse, correspondances et limite c′→0 ; les erreurs/rayons du chemin
plan ne seront pas transposés à une surface physique différente sans dérivation.

## R3 / U2 — moniteur sans paramètre de régularisation (décision avant T010)

`mesh.density.curvature_regularization_over_c` est retiré du contrat initial non publié,
sans remplacement ni valeur implicite. Sa présence, même dans une demande legacy, produit
`REMOVED_PARAMETER` (code 2, pointeur exact) avant résolution ; il n'est jamais accepté sans effet.
Le moniteur du §5 est conservé **sans régularisation**. Sur chaque arc reconstruit régulier
la vitesse paramétrique est bornée loin de zéro et la courbure est finie. Aux inflexions,
`sqrt(c |κ|)` vaut zéro et reste continue ; l'intégration adaptative borne son erreur en
séparant les zéros de courbure si nécessaire, sans epsilon géométrique. Aux angles protégés,
on sépare les arcs et conserve exactement leur sommet et leurs comptes compatibles ; on
n'intègre aucune impulsion de courbure et on ne lisse pas le coin. Un arc non régulier est refusé.

L'effet géométrique retenu est une densité croissante en courbure, avec plancher unitaire
sur les parties droites : à fortes courbures, le poids en racine carrée suit la dépendance
en sqrt(|κ|) du nombre de segments nécessaire pour borner l'écart corde–arc
(écart local ≈|κ| h²/8). Ce choix ne modifie ni rayon ni angle de la reconstruction,
et ne garantit pas à lui seul le budget de fidélité, qui reste contrôlé indépendamment.
Il n'est pas ajusté pour obtenir un succès des seules fixtures.

`A_k=curvature_weight≥0` ; A_LE/A_TE et σ_LE/σ_TE restent ceux des contrats.
Dans N, `c=c_original` en unités source, κ est en inverse de ces unités, et
`σ=width_over_c*c_original`. Dans W, employer simultanément `c_W=c_original/s`,
`κ_W=s κ_N`, `σ_W=σ_N/s` et les distances curvilignes W. Le moniteur est adimensionnel.
Une homothétie λ>0 conserve c|κ|, les distances/largeurs et la cumulative normalisée ;
translations/rotations rigides n'affectent pas cette distribution. La compression de flèche
est anisotrope : recalculer κ et l'abscisse sur la courbe transformée (formule du §4),
sans supposer la distribution invariante en Λ ni prendre la corde comprimée comme c.

Essais à réaliser : T010 sur V-01/02/03 avant J0, T034 sur reconstructions générales :
homothéties λ=0,1/1/10, calcul N versus W, poids A_k=0/1/4 avec courbure variable
(sensibilité des positions/pas, sans changer les caractéristiques), inflexion et partie droite,
coins protégés et stabilité des quadratures aux trois flèches. T021/T025 refusent le champ
retiré ; aucune exigence n'est vérifiée par cette seule décision documentaire.
