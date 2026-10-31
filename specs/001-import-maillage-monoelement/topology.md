# Décision D-01 — Topologies conformes monoélément

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

État : conception retenue le 2026-10-30, à démontrer au jalon J0. Aucun prototype,
maillage ni résultat de qualité n'a été produit pour cette décision. Les schémas sont
topologiques et ne constituent pas des maillages calculés.

## Notation commune

Toutes les coordonnées ci-dessous sont dans la section normale à l'extrusion,
éventuellement globalement normalisée selon le contrat de transformations. `A` est
le BA, `T+` la lèvre supérieure et `T−` la lèvre inférieure ; dans le cas fin les deux
lèvres ont la même position `T`. Les unités et la corde de référence originale ne
changent pas avec le choix de topologie.

Chaque bloc est un rectangle logique `[0,Ni] × [0,Nj]` de quadrilatères. Ses côtés
sont `W:i=0`, `E:i=Ni`, `S:j=0`, `N:j=Nj`. Un tableau de côté est lu suivant son
indice libre croissant, sans préjuger de l'orientation induite du bord de cellule.
Les cellules `(i,j),(i+1,j),(i+1,j+1),(i,j+1)` doivent avoir un jacobien positif dans
le dessin `(ξ,z)`. Le parcours induit d'un bloc positif est S croissant, E croissant,
N décroissant, W décroissant. Deux partenaires peuvent donc avoir le même ordre de
coordonnées et des orientations induites opposées.

Le registre de géométrie contient une seule courbe discrète par interface, des
identifiants globaux de sommets partagés et deux vues de cette courbe pour les blocs
incidents. Il ne génère jamais deux versions indépendantes d'une même interface.
Un éventuel écart de lecture est un diagnostic numérique ; aucune tolérance ne
transforme un trou, un recouvrement ou une correspondance manquante en interface.

## Bord de fuite fin : quatre blocs, famille C avec sillage coupé

Les blocs de corps `U` et `L` sont les deux moitiés de l'enroulement C, séparées en
amont au BA. Deux blocs de sillage `WU` et `WL` prolongent cet enroulement vers
l'aval et partagent la coupure de sillage. Ils permettent de régler le nombre de
couches normales indépendamment du nombre de stations longitudinales du sillage.

```text
                              frontière extérieure
                Oa ~~~~~~~~~ P+ =============================== D+
                 |      U    |               WU                 |
                 |           |                                  |
       interface |     A ====T--------------------------------- D0
         au BA   |      \    |               WL                 |
                 |      L    |                                  |
                Oa ~~~~~~~~~ P− =============================== D−
                              frontière extérieure

 A====T : contour supérieur et inférieur du solide, non représenté à l'échelle
 A→Oa  : interface amont ; T→P+ et T→P− : interfaces corps/sillage
 T→D0  : coupure de sillage interne, ni paroi ni frontière physique
 D−→D0→D+ : sortie aval ; Oa→P±→D± : champ lointain
```

Le dessin représente `Oa` deux fois pour la lisibilité : c'est un seul sommet et une
seule ligne `A→Oa` partagée, aucun segment de frontière extérieure doublé. Le solide
occupe la région entre les deux branches de `A` à `T`.

| Bloc | Dimensions | Direction i | Direction j | S | N | W | E |
| --- | --- | --- | --- | --- | --- | --- | --- |
| U | Nu × Nn | BA→lèvre supérieure sur la paroi | paroi→extérieur | paroi extrados | champ lointain Oa→P+ | interface A→Oa | interface T→P+ |
| L | Nl × Nn | lèvre inférieure→BA sur la paroi | paroi→extérieur | paroi intrados | champ lointain P−→Oa | interface T→P− | interface A→Oa |
| WU | Nw × Nn | proche→aval | coupure→extérieur supérieur | coupure T→D0 | champ lointain P+→D+ | interface T→P+ | sortie D0→D+ |
| WL | Nw × Nn | proche→aval | extérieur inférieur→coupure | champ lointain P−→D− | coupure T→D0 | interface P−→T | sortie D−→D0 |

`Nu`, `Nl` et `Nw` sont indépendants ; `Nn` est partagé par les deux côtés du corps
et les directions transversales du sillage. Le paramètre de résolution tangentielle
se répartit entre Nu/Nl ; leur égalité n'est pas exigée. Les stations du sillage
sont communes à WU/WL. Les paramètres incompatibles avec une interface conforme sont
refusés avant génération, jamais arrondis ou remplacés silencieusement.

| Interface | Vue 1, paramètre k | Vue 2, paramètre | Correspondance | Orientations induites |
| --- | --- | --- | --- | --- |
| IF-LE | U.W, j=k | L.E, j=k | k→k ; 0≤k≤Nn | opposées |
| IF-U | U.E, j=k | WU.W, j=k | k→k ; 0≤k≤Nn | opposées |
| IF-L | L.W, j=k | WL.W, j=Nn−k | k→Nn−k | opposées |
| IF-WAKE | WU.S, i=k | WL.N, i=k | k→k ; 0≤k≤Nw | opposées |

### BA et pointe

Le BA est un point ordinaire de deux côtés de paroi, relié à l'amont par une
interface de longueur strictement positive. Ni le côté au BA ni une ligne de
cellules ne s'effondre en un point. Les deux branches reconstruites y partagent une
tangente et une normale cohérentes, avec rayon identifiable strictement positif.
Les premières lignes normales des deux blocs utilisent cette même normale et les
mêmes coordonnées sur IF-LE.

À la pointe, IF-U part suivant la normale extérieure unilatérale de l'extrados,
IF-L suivant celle de l'intrados, et IF-WAKE suivant une direction aval appartenant
au secteur fluide restant. Contrairement à un découpage en deux seuls blocs C,
quatre secteurs de cellules se rencontrent au sommet T. Pour un bord fin à angle
solide fini, les secteurs U/L sont proches de 90° et les deux secteurs de sillage
partagent le reste de l'angle fluide. Une pointe à angle nul n'impose pas
algébriquement une cellule effondrée : les quatre secteurs peuvent rester
strictement positifs. La démonstration J0 devra néanmoins inclure une telle limite
si sa prise en charge est revendiquée ; elle n'est pas déclarée qualifiée ici.

Les deux occurrences logiques de T ont le même identifiant global de sommet ; cela
n'autorise jamais une face à deux sommets identiques. La topologie fine est
construite directement avec quatre blocs ; aucun bloc de culot n'est créé puis
écrasé pour l'obtenir.

## Bord de fuite épais : cinq blocs, famille C–H avec culot

`U`, `L`, `WU` et `WL` gardent leur rôle. Un cinquième bloc `H` remplit le sillage
central, de la face de culot jusqu'à la sortie. Les lèvres restent deux points
distincts ; leurs positions, angles et épaisseur sont des contraintes de géométrie.

```text
                              frontière extérieure
                Oa ~~~~~~~~~ P+ =============================== D+
                 |      U    |               WU                 |
                 |           |                                  |
                 |     A ====T+-------------------------------- Dcu
       interface |     solide |                H                 |
         au BA   |            | culot                            |
                 |      =====T−-------------------------------- Dcl
                 |      L    |               WL                 |
                Oa ~~~~~~~~~ P− =============================== D−
                              frontière extérieure

 A→T+ : extrados ; T−→A : intrados ; T−→T+ : culot physique
 T+→Dcu et T−→Dcl : deux interfaces internes distinctes du sillage central
 T±→P± : interfaces corps/sillage latéral, non confondues avec les précédentes
```

| Bloc | Dimensions | Direction i | Direction j | S | N | W | E |
| --- | --- | --- | --- | --- | --- | --- | --- |
| U | Nu × Nn | A→T+ | paroi→extérieur | extrados | champ lointain Oa→P+ | interface A→Oa | interface T+→P+ |
| L | Nl × Nn | T−→A | paroi→extérieur | intrados | champ lointain P−→Oa | interface T−→P− | interface A→Oa |
| WU | Nw × Nn | proche→aval | interface centrale haute→extérieur supérieur | interface T+→Dcu | champ lointain P+→D+ | interface T+→P+ | sortie Dcu→D+ |
| WL | Nw × Nn | proche→aval | extérieur inférieur→interface centrale basse | champ lointain P−→D− | interface T−→Dcl | interface P−→T− | sortie D−→Dcl |
| H | Nw × Nb | culot→aval | lèvre basse→lèvre haute | interface T−→Dcl | interface T+→Dcu | paroi culot T−→T+ | sortie Dcl→Dcu |

| Interface | Vue 1 | Vue 2 | Correspondance des paramètres croissants | Orientations induites |
| --- | --- | --- | --- | --- |
| IF-LE | U.W, j=k | L.E, j=k | k→k | opposées |
| IF-U | U.E, j=k | WU.W, j=k | k→k | opposées |
| IF-L | L.W, j=k | WL.W, j=Nn−k | k→Nn−k | opposées |
| IF-CU | WU.S, i=k | H.N, i=k | k→k ; 0≤k≤Nw | opposées |
| IF-CL | WL.N, i=k | H.S, i=k | k→k ; 0≤k≤Nw | opposées |

La résolution Nb du culot est indépendante de Nn et Nu/Nl. Nw est partagé par les
trois blocs de sillage ; l'indépendance entre résolution normale du corps Nn et
résolution longitudinale Nw est conservée. Le premier espacement longitudinal de H
au culot est contraint par la première hauteur de paroi demandée. Il fixe les
premiers espacements des deux interfaces IF-CU/CL, sans imposer à tout le sillage
la même loi géométrique de croissance que celle des couches de corps.

### Construction locale aux lèvres : trois secteurs, pas deux extrusions superposées

À la lèvre haute, `ns` est la normale extérieure unilatérale de l'extrados et `nb`
la normale extérieure du culot ; les deux sont calculées dans le repère transformé.
IF-U démarre suivant ns ; IF-CU démarre suivant nb. Le bloc WU remplit le secteur
entre ces deux rayons, avec deux arêtes de longueurs strictement positives. Le bloc
U occupe le secteur côté extrados et H celui côté culot. À un coin solide proche de
90°, trois cellules partagent donc un angle fluide proche de 270°, chacune avec un
secteur proche de 90°. À la lèvre basse, la construction est symétrique dans son
principe, avec le sens logique de WL décrit dans la table.

La construction n'applique pas une normale unique au sommet et ne fait pas se
croiser des bandes extrudées sur les deux faces. Elle utilise des normales par
face et un bloc supplémentaire de transition qui occupe la région manquante.
Pour les premières arêtes, longueurs et directions sont imposées par les hauteurs
requises sur les faces incidentes ; la connexion aval est interpolée avec tangentes
initiales imposées, puis les premières cellules sont vérifiées face par face.
Aucune moyenne des normales ne sert à déplacer les lèvres ou à arrondir le coin.

Localement, pour la lèvre haute, les directions logiques de WU sont nb pour i et ns
pour j. Le déterminant `det(nb,ns)` doit être strictement positif. Dans H,
`det(nb,tculot)` doit être positif avec tculot orienté T−→T+ ; dans U, la tangente
extrados et ns donnent une base directe. Ces conditions fournissent des secteurs
locaux non effondrés. Une petite plage de coin de WU peut être construite
explicitement par `P(i,j)=T+ + a(i) nb + b(j) ns`, où a et b sont des suites
strictement croissantes commençant à zéro. Chaque cellule de cette plage a l'aire
`Δa Δb det(nb,ns)>0`. Sa première colonne est le rayon IF-U et sa première ligne
le rayon IF-CU ; les étendues sont bornées avant la première courbure ou perte de
dégagement des interfaces. Pour WL, écrire localement
`P(i,Nn−k)=T− + a(i) nb + b(k) ns_bas`, avec le signe logique inversé en j.
Cette plage est un parallélogramme logique réel, pas un éventail de quadrilatères
ayant deux sommets confondus. Si cette plage imposée et les premières bandes du
corps/culot ne peuvent être raccordées avec les hauteurs demandées, la configuration
est refusée. Le reste des lignes s'adapte par interpolation contrainte ; les
coordonnées de cette plage restent gelées pendant le lissage.

Ce calcul local ne constitue pas une preuve du jacobien de tout le bloc : toutes
les cellules et les extrema du jacobien bilinéaire doivent encore être contrôlés
après construction et après chaque lissage.

Limites : culot rentrant, normales qui rendent le secteur latéral nul ou inversé,
plusieurs coins intermédiaires non protégés, base fortement courbe dont les normales
se croisent dans la bande imposée ou absence de dégagement suffisant entraînent un
refus explicite. Le premier chemin vise un culot rectiligne déjà fourni ou ajouté
explicitement, éventuellement incliné mais de normale orientée vers l'aval et de
lèvres convexes. Une base courbe ne sera acceptée qu'après démonstration d'une carte
injective respectant les mêmes contraintes, sans être réputée couverte par le seul
cas droit.

## Construction globale et domaine extérieur

Les coordonnées de la section transformée sont normalisées pour les calculs
sensibles. Noter la boîte du contour `[xmin,xmax]×[zmin,zmax]`. Une réalisation
simple de la frontière extérieure utilise :

- un sommet amont `Oa=(xmin−R, zA)` ;
- `zhaut=zmax+R`, `zbas=zmin−R` ;
- `P+=T+ + λ+ ns+`, où `λ+=(zhaut−zT+)/ns+_z`, et
  `P−=T− + λ− ns−`, où `λ−=(zbas−zT−)/ns−_z` ;
- les normales de lèvre satisfont `ns+_z>0`, `ns−_z<0` et les λ sont positifs ;
  les interfaces IF-U/L sont exactement les rayons T±→P± ;
- une sortie verticale `ξout=xmax+Lout`, avec `D±=(ξout,zmax+R/zmin−R)` ;
- deux quarts d'ellipse de Oa à P+ et de P− à Oa, tangents verticalement en Oa et
  horizontalement en P±, prolongés par les segments horizontaux P±→D±.

Ici R est une extension de boîte en amont et verticalement, pas la distance minimale
universelle de chaque cellule à la paroi ; cette définition doit être affichée dans
la configuration et le rapport. Lout est l'extension aval au-delà de xmax. Les
points D0 du cas fin, ou Dcu/Dcl du cas épais, se trouvent sur la sortie et conservent
l'ordre vertical de la coupure ou des interfaces centrales. Les surfaces externes
ne sont pas assimilées à une garantie d'erreur de Cx.

La courbe IF-LE part selon la normale commune au BA et arrive en Oa sans retour ni
intersection. IF-U/L sont les rayons rectilignes suivant les normales unilatérales
des lèvres : agrandir R les prolonge sans courbure ni déplacement de leur portion
intérieure. Les largeurs des quarts d'ellipse sont indépendamment
`P+_ξ−Oa_ξ` et `P−_ξ−Oa_ξ`, strictement positives. Les deux rayons doivent rester
hors du solide et sans intersection avec le squelette. Il faut également
`ξout>max(P+_ξ,P−_ξ)` avec une marge strictement positive compatible avec le profil
qualité et les cellules de sillage. Si R augmente sans reculer assez la sortie,
la configuration peut donc devenir incompatible ; aucun R ou Lout n'est ajusté
silencieusement. Les cas `ns+_z≤0`, `ns−_z≥0`, de faible composante verticale
empêchant ce dégagement ou d'ordre extérieur inversé sont refusés comme non pris
en charge par cette réalisation du domaine. La coupure fine et les
deux interfaces centrales épaisses partent dans des directions aval admissibles,
puis rejoignent la sortie ; dans le cas épais, les premières bandes de H sont les
translations du culot suivant nb. Toutes ces courbes font partie du squelette
partagé, construit avant les intérieurs des blocs. Toute impossibilité de respecter
simultanément tangentes, ordre, dégagement et qualité est un échec explicite.

Une initialisation par interpolation transfinie contrainte remplit chaque rectangle
logique avec ces côtés fixés. Les premières bandes normales et les transitions de
coins sont construites avant l'interpolation du reste ; un lissage ne peut ni
déplacer le contour ni écraser ces bandes ni désapparier une interface. Il ne peut
rendre acceptable un squelette qui s'intersecte. Les critères d'arrêt et limites de
qualité sont définis dans le plan et son profil qualité commun.

### Variation indépendante du champ lointain

Les courbes et lignes d'indices du noyau sont archivées : `j≤Jn_core` dans U/L ;
dans WU la région `i≤Iw_near ET j≤Jn_core` ; dans WL la région
`i≤Iw_near ET j≥Nn−Jn_core`. Leurs intersections possèdent des coordonnées uniques
et fixes. Pour H, toutes les coordonnées `i≤Iw_near` restent fixes. Si seule R
varie à sortie aval constante, les lignes intérieures de sillage peuvent être
gelées sur toute leur longueur ; si la sortie recule, seuls les intervalles
`i≤Iw_near` sont contractuellement conservés, pas les anciennes coordonnées de sortie. Changer R ou l'extension aval reconstruit seulement la portion extérieure,
avec une transition après les courbes gelées. Les premières hauteurs, la géométrie,
la résolution du culot et les stations prescrites du sillage proche sont inchangées.

La zone de gel doit elle-même être admissible, de dimensions strictement positives,
et ne pas dépasser le domaine demandé. R trop petit, sortie en amont de la fin de
sillage proche ou impossibilité de raccord sans violation du profil qualité sont
refusés ; le noyau n'est pas déplacé pour faire passer la configuration. Les
coordonnées du noyau comparées entre deux tailles de domaine doivent être
identiques dans la même génération de référence, pas seulement « proches ».

## Domaine d'admissibilité initial et refus

Cible initiale : un contour simple, deux branches de profil ordonnées, monotones en
ξ hors voisinage lisse du BA, une épaisseur positive dans l'intérieur de la corde,
un BA régulier et deux lèvres identifiables. Cambrure et épaisseur peuvent être
asymétriques. Les transformations à 0°, 25° et 50° sont appliquées à toute la
référence avant construction ; aucun seuil d'angle ou rayon original n'est appliqué
sans transformation aux coins ou au BA comprimés.

Les indications utilisateur peuvent lever l'identification ambiguë de ces
caractéristiques ; elles n'autorisent jamais une carte inversée, un segment nul ou
une intersection. Les formes rentrantes, surplombs importants, nez non régulier,
base à plusieurs coins non traités, peau repliée, passages trop étroits pour les
hauteurs prescrites ou une topologie à jacobien non positif ne sont pas acceptées
par cette construction initiale. Le diagnostic distingue `GEOMETRY_INVALID`,
`TOPOLOGY_NOT_ADMISSIBLE`, `PARAMETER_INCOMPATIBLE` et `QUALITY_CRITICAL`. Une
absence de prise en charge n'est pas présentée comme une invalidité du profil réel.

Aucune plage numérique générale de cambrure, épaisseur ou angle n'est déclarée
qualifiée par ce document. Les neuf cas V-01/02/03 aux trois flèches doivent passer
J0 et la campagne de qualité avant toute revendication de couverture de ces cas.
Les limites supplémentaires révélées par J0 doivent être consignées ; elles ne
peuvent être remplacées par des résultats inventés ou par un relâchement automatique.

## Pourquoi cette décision et comment la vérifier

Le découpage minimal en deux blocs C fins ou trois blocs C–H épais a été écarté : il
couple le nombre de couches autour du corps au nombre de stations longitudinales
du sillage et impose des secteurs très obtus aux coins de culot. Les quatre/cinq
blocs retenus permettent de séparer les contrôles et de conserver des cartes locales
régulières aux lèvres. Une infrastructure générale de blocs arbitraires, un maillage
non conforme et des raccords mortier ne sont pas requis ici.

Le jalon J0 doit réaliser le squelette et un maillage purement géométrique pour les
neuf cas de base, vérifier toutes les bijections, signes et minima de jacobiens,
les faces et la fermeture, puis agrandir le domaine en comparant le noyau gelé.
Les figures des lèvres et du BA doivent montrer les sommets et les secteurs de
cellules. Ajouter un cas à culot mince mais non nul et un cas de coin incompatible
vérifie le refus sans conversion en pointe. Aucun prototype de ce type n'est livré
à l'étape du plan.

La présence d'une coupure C raccordée un à un dans le sillage est documentée par
la ressource primaire [NASA/TMR, maillages NACA 0012](https://tmbwg.github.io/turbmodels/naca0012_grids.html).
La séparation géométrique corps/sillage pour bord épais est également décrite dans
[NASA NTRS, Navier-Stokes analysis of blunt trailing edge airfoils](https://ntrs.nasa.gov/citations/19920039522).
Ces sources motivent les familles de topologie ; elles ne valident pas notre
construction locale à quatre/cinq blocs, qui est une décision propre au projet et
reste à démontrer.

## Précision de l’ordonnancement J0 — 2026-10-31

J0 reçoit les courbes analytiques directement via un adaptateur réservé aux tests. Les types,
courbes partagées, cartes, distributions et contrôleurs utilisés seront ceux du noyau ; il
n’y a pas de second algorithme de maillage de démonstration. Les preuves de fidélité et de
caractéristiques utilisent un oracle polynomial indépendant avec bornes ciblées ; les
évaluateurs généraux des contours importés viennent après le go. Les VTP de contours et
frontières et VTM/VTS des blocs permettent l’inspection avec zooms BA et deux lèvres.
L’interopérabilité obligatoire VTK/ParaView est qualifiée ensuite, sans retarder J0.
Les très petits culots E3 peuvent être refusés avec preuve ; V-03 usuel (0,02 c) reste obligatoire.
Sur no-go, conserver mesures/artefacts, diagnostiquer et revoir les cartes, puis rejouer les
neuf configurations et les deux fermetures V-03 avant généralisation.

## R2 — portée des décisions 4/5 blocs

Les tables ci-dessus décrivent la topologie logique et sa **construction plane** candidate
pour J0, sans qualification conique. La géométrie physique, les bases et métriques restent
séparées des indices/interfaces. Même si un graphe est réutilisable après G-FORM, ses cartes,
normales/mesures et raccords devront être rééprouvés à JG-C. J0 ne dépend de cette étude ni
de sa réussite ; JG-C dépend du go J0 et du go scientifique, jamais l’inverse.
