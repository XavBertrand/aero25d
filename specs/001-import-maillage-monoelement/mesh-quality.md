# Construction, qualité et familles — décisions D-03 et D-06

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

Statut : conception à réaliser et à éprouver. Aucun maillage produit. Ce document complète
[les topologies](topology.md) et [la géométrie](geometry-verification.md).

## Construction déterministe

1. Contrôler la source et construire une seule reconstruction certifiée dans S ; la transformer
   en N puis W. Construire un graphe de courbes de blocs, avec un propriétaire unique par arête
   partagée et un propriétaire unique par sommet commun. Déterminer les compatibilités des
   comptes avant d'allouer ; refuser les contraintes contradictoires en indiquant les partenaires.
2. Construire les courbes de paroi, rayons aux lèvres, ligne BA–amont, frontière extérieure,
   coupure fine ou limites du sillage épais. Les courbes de cœur jusqu'à Hcore et jusqu'à la fin
   du sillage proche sont immuables dans une série de domaines. Le culot est une vraie paroi.
3. Échantillonner les courbes une seule fois. Chaque bloc référence les mêmes identifiants
   globaux sur une interface ; il ne calcule pas sa propre copie. L'écriture VTK peut dupliquer
   les tuples par bloc mais fournit global_node_id et les mêmes valeurs à l'arrondi près.
4. Près des parois lisses, construire les premières rangées par tangentes/normales à la
   reconstruction, sans franchir la distance de sécurité aux normales voisines. Les rayons et
   secteurs aux lèvres sont ceux de topology.md : aucune extrusion indépendante du coin.
   Le cœur du sillage épais part du culot dans la direction normale à chaque face de culot.
5. Remplir chaque quadrilatère curviligne par interpolation transfinie de Coons, après
   construction de bandes de paroi fixées ; contrôler immédiatement le Jacobien de chaque
   cellule. Une interpolation de Coons n'assure pas à elle seule la positivité. Une construction
   initiale inversée est refusée, pas confiée à un lissage qui masquerait son défaut.
6. Lissage elliptique optionnel et déterministe : solution discrète de l'équation de grille
   α r_ii − 2β r_ij + γ r_jj = 0, avec α=|r_j|², β=r_i·r_j, γ=|r_i|²,
   par itérations de Gauss–Seidel sous-relaxées. Geler toutes les frontières, interfaces,
   caractéristiques, les deux premières rangées (pour les hauteurs et centres) et les lignes
   de cœur prescrites ; lisser uniquement les intérieurs admissibles. Les bandes de transition
   peuvent être traitées par sous-domaines logiques sans nouveaux blocs du graphe.
7. Essayer des facteurs de relaxation 1/2, 1/4, …, 2⁻²⁰ dans cet ordre pour une mise à jour ;
   n'accepter que si aires/Jacobiens restent positifs, les bornes dures restent satisfaites et
   le résidu décroît. Zéro mise à jour admissible : SMOOTHING_NOT_CONVERGED. Arrêt normal :
   déplacement max/c ≤ 10⁻¹⁰ ET résidu adimensionnel ≤ 10⁻⁸ pendant 5 balayages ; maximum
   5 000 balayages, sinon échec. Ce sont des limites de projet, pas des temps garantis.
8. Relancer tous les contrôles indépendamment de la réussite du générateur/lisseur. Un
   défaut critique interdit le statut valide ; les diagnostics restent exportables comme invalides.

## Distribution et contrôles utilisateur

Les comptes sont des nombres de cellules, donc N+1 sommets. La résolution tangentielle de
chaque branche est indépendante de Nnormal ; WU/WL et H partagent Nwake en i et les interfaces
radiales partagent Nnormal. Pour une face de culot non rectiligne, une normale par face est
nécessaire et l'admissibilité géométrique est vérifiée ; la qualification initiale ne généralise
pas les culots rentrants ou fortement courbes.

La redistribution de surface utilise une densité continue positive de longueur curviligne,
combinant un plancher, sqrt(c |κ|) et deux fonctions de concentration localisées BA/BF,
selon la formule exacte de geometry-verification.md §5 ; aucune régularisation ajoutée.
Les poids et largeurs sont configurables et conservés à travers la famille ; tous les angles
protégés restent des bornes de segments. Les paramètres effectifs et pas min/max sont publiés.
Une demande qui viole le budget reconstruction–segments du maillage est refusée avec la
résolution minimale suggérée, sans exécution automatique d'une nouvelle configuration.

Pour la bande de paroi : h_k=h1 g^(k−1), k=1..Nbl ;
Hbl=h1·expm1(Nbl log g)/expm1(log g), avec limite Hbl=Nbl h1 pour g=1.
La contrainte de croissance s'applique aux pas normaux, pas au rapport des longueurs totales.
La configuration canonique demande h1, Nbl et g : Hbl est résolu, pas fourni comme quatrième
contrainte indépendante. Si elle ne tient pas dans Hcore, échec. La bande suivante rejoint
Hcore puis l'extérieur avec distributions exponentielles segmentées dont premiers et derniers
pas sont raccordés ; résolution par recherche monotone, et refus si aucun raccord respecte
les bornes. La première distance de sillage depuis le culot vaut h1_base ; elle peut contraindre
la résolution de sillage demandée, sans imposer Nwake=Nnormal. Les configurations incompatibles
sont refusées avant génération.

Pour la première version, le culot utilise le même h1 par défaut que les autres parois ; une
surcharge base_first_height explicite est permise. Les paramètres de Hcore, de sillage proche
et de transition sont explicites dans les paramètres résolus. Aucune estimation de y+ n'est
requise pour calculer ces valeurs géométriques.

## Centre, distances et coins

Le centre d'une cellule est le centroïde d'aire de son quadrilatère à arêtes droites, calculé
par la formule polygonale signée avec sommation compensée. Il n'est ni l'intersection des
diagonales ni automatiquement la moyenne des quatre sommets.

Pour chaque face de paroi f : M_f est son milieu, t_f son unité tangentielle, n_f l'unité normale
vers le fluide déterminée par l'orientation ; d_center(f)=(C−M_f)·n_f et
h_first(f)=(M_opposite−M_f)·n_f. Enregistrer aussi les distances minimale/maximale des deux
sommets opposés sur n_f pour détecter une cellule très oblique. Exiger les trois distances
positives. Si une cellule touche deux faces de paroi à un coin, produire deux enregistrements ;
aucune moyenne des normales n'est appelée « normale du coin ». Le pied projeté hors de la face
est signalé ; l'orthogonalité utilise aussi l'écart tangentiel |(C−M_f)·t_f|.

La hauteur prescrite vise h_first(f) sur la normale de la face discrète ; la construction
utilise la normale de la reconstruction puis ajuste la rangée libre sous contraintes pour
respecter cette mesure. Si géométrie, budget et hauteur ne peuvent être satisfaits ensemble,
échec. Le diagnostic distingue angle de normale discrète/reconstruite et erreur de hauteur.

## Profil qualité initial Q1

Toutes les métriques sont calculées dans le repère normal N, après remise à l'échelle si le
stockage est W. Pour les critères dimensionnels on utilise c_original ; pour les critères de
forme on utilise les longueurs locales. Toutes les valeurs ci-dessous sont des choix initiaux
de projet, à justifier par les expériences E1–E5 du plan, pas des normes RANS. Elles sont figées
avec l'identifiant Q1 et ne sont jamais relâchées à l'exécution.

| Métrique | Définition | Limite Q1 et sévérité |
| --- | --- | --- |
| Topologie | Comptes, cycles de faces, unicité des partenaires, couverture, chevauchement de cellules non voisines | Tolérance zéro ; critique |
| Aire | Formule polygonale signée, somme compensée en coordonnées locales | A>0 impératif ; intervalle contenant zéro ⇒ non certifiable donc échec |
| Jacobien | Déterminants de la carte bilinéaire aux quatre coins, divisés par produit des deux longueurs locales | Jscaled>10⁻⁶ critique ; avertissement si <0,1 ; angle concave toujours critique |
| Dégénérescence | lmin/lmax et A/(lmax·lmin), avec min/max des quatre arêtes | lmin/lmax>10⁻¹⁰ et A/(lmax·lmin)>10⁻⁶ ; critique ; ne plafonne pas le rapport d'aspect à une valeur usuelle de grille isotrope |
| Fermeture | Norme de Σ_f n_f L_f / périmètre | ≤10⁻¹² ; critique ; graphe des sommets doit être exactement fermé indépendamment de ce résidu |
| Interface | Identifiants communs, sens de face opposés ; coordonnées uniques | Aucun trou/chevauchement autorisé ; résidu de relecture ≤10⁻¹² c ; critique |
| Orthogonalité paroi | angle entre C−M_f et n_f | ≤10° cible, avertissement au-delà ; >30° critique ; par face y compris au culot |
| Non-orthogonalité interne | angle aigu entre segment centroïdes et normale commune, valeur max des deux côtés | >45° avertissement, >80° critique |
| Variation normale | max(h_k/h_(k−1), inverse) hors saut déclaré de région | >1,25 avertissement ; >1,5 critique, y compris aux interfaces |
| Variation tangentielle | Ratio de longueurs successives dans une ligne et à son raccord | >1,5 avertissement ; >2 critique |
| Étirement | σmax/σmin du Jacobien local, orientation du vecteur singulier principal | >10³ avertissement informatif ; >10⁴ et angle avec tangente de paroi locale >15° critique dans la bande BL ; ailleurs >10⁶ critique. Un grand rapport aligné n'est pas rejeté sur sa seule valeur ; limite numérique de dégénérescence reste applicable |
| Fidélité | Bornes certifiées de distance, rayons/angles selon le repère | Limites H-07/H-08 ; dépassement ou certification impossible ⇒ critique pour le résultat qualifié |
| Première hauteur | |h_first−h_requested|/h_requested par face | ≤2 % ; critique si dépassé ; centres et projections mesurés séparément |
| Frontière extérieure | Écart des points à la courbe extérieure prescrite | ≤10⁻¹² c ; critique ; la distance physique minimale au contour est mesurée aussi |

Le défaut topologique ne devient jamais avertissement. Les indicateurs d'orthogonalité et
d'étirement ne sont pas des preuves d'erreur de discrétisation. Les sommets de coins ont
leurs angles de cellule publiés, jamais exclus du contrôle de Jacobien/positivité. Si une limite
est géométriquement incompatible avec une topologie, on révise la conception ou propose une
révision documentée de Q1, pas un traitement caché de faveur.

Précision numérique : orientation exacte pour les signes (méthode de geometry-verification.md),
intervalles arrondis vers l'extérieur pour aires/Jacobiens et erreurs géométriques sensibles.
Une borne qui croise une limite donne QUALITY_UNCERTIFIED ; ne pas arrondir pour faire passer.
Les résultats locaux, maxima, histogrammes et identifiants des pires cellules sont conservés.
Le contrôle d'intersection global utilise boîtes englobantes puis prédicats exacts ; contrôler
les seuls signes locaux n'exclurait pas un recouvrement de blocs éloignés.

## Familles globales F1

Figer une seule reconstruction certifiée, un graphe 4/5 blocs et les mêmes courbes de bord.
Niveaux ℓ=0,1,2 avec facteur r=2^ℓ. Multiplier par r les comptes de surface, culot, sillage
proche/lointain et tous les segments de direction normale, y compris bande BL et tampon.
Les singularités/coins et nombres de blocs ne changent pas.

Pour une bande normale de hauteur Hbl, écrire F(t)=(exp(βt)−1)/(exp(β)−1), β=Nbl0 log g0.
Au niveau ℓ, Nblℓ=r Nbl0 et y_k=Hbl F(k/Nblℓ). Ainsi :

- gℓ=g0^(1/r), strictement rapproché de 1 ;
- h1ℓ=h10·expm1(log(g0)/r)/expm1(log(g0)), avec h10/r si g0=1 ;
- le nombre de couches est multiplié par r et Hbl inchangé ;
- les maxima de pas sont décroissants, sans prétendre à un facteur spatial exactement 2 partout.

Cette règle est également utilisée pour les segments exponentiels du sillage et des tampons ;
les distributions tangentielles conservent leur fonction cumulative et sont échantillonnées
à r fois plus de subdivisions. On publie les rapports effectifs, par région/direction. Une
famille ne résout donc pas séparément une valeur de h1ℓ contradictoire ; la demande canonique
porte sur le niveau zéro et la règle F1. Une surcharge par niveau la classe comme diagnostic,
pas comme famille globale. Si un niveau n'atteint pas le budget de fidélité, toute la famille
est signalée incomplète ; aucun niveau inférieur n'est omis silencieusement.

Exemple de paramètres destinés à E1, à ajuster explicitement avant gel des fixtures :
Nu=Nl=192, Nbl0=48, g0=1,15, h10=10⁻⁵ c, Nnormal0=96 dont 48 couches BL,
Nwake0=192 (96 proche, 96 prolongement), Nbase0=24 si épais ; Hcore=0,5 c,
Lnear=2 c, Lwake=20 c, R=10 c. Chaque segment de transition doit être résolu de façon
compatible avec Q1 ; ces valeurs sont des entrées d'expérience, pas des cas déjà réussis.
Nnormal0 compte les couches totales, dont les segments hors BL sont résolus et publiés,
sans imposer un Hcore incompatible avec les pas. Les ajustements avant gel sont versionnés.

## Diagnostics et domaine extérieur

Une série de diagnostic ne change qu'un paramètre nommé (h1, croissance, Nsurface, culot,
sillage…) ; elle publie tous les paramètres résolus et les conséquences de compatibilité.
Elle ne se présente pas comme une famille de convergence. Une contradiction provoque un refus.

Pour la variation extérieure, geler coordonnées et distributions de surface, toutes les
rangées jusqu'à Hcore et le sillage jusqu'à Lnear ; seuls le tampon hors cœur et le prolongement
lointain sont recalculés. Conserver les comptes du cœur ; augmenter explicitement les comptes
de tampon si nécessaire pour satisfaire Q1 (dans une nouvelle configuration résolue, jamais
par relaxation d'une limite). Les rayons de domaine R=10 c et 20 c servent de premiers essais,
avec Lwake fixé à une valeur compatible géométriquement avec les deux domaines. Comparer
l'identité des courbes/lignes gelées et les paramètres, pas seulement les comptes globaux.
Aucune distance extérieure ou quantité de cellules ne garantit la précision sur Cx.

## Application au jalon initial — 2026-10-31

J0 utilise les contrôleurs indépendants de positivité, dégénérescence, fermeture, interfaces,
recouvrement global, centroïdes/hauteurs par face et métriques Q1 applicables. La fidélité
est bornée par l’oracle analytique ciblé, sans attendre les évaluateurs généraux de spline.
Aucune exemption de coin ni simple positivité d’aire ne suffit au go. Les premières cellules
respectent les limites de 30° et 2 %, les autres limites critiques Q1 s’appliquent à leurs
régions respectives. Les distributions proposées ci-dessus restent des entrées à éprouver,
pas une référence réussie. Un ajustement explicite de comptes/distributions avant gel est
conservé avec ses échecs ; il n’autorise pas de relâchement automatique des limites.

## R2 — validité plane et métriques coniques

Q1 conserve exactement ses limites et son domaine plan ; « volume par unité d’extrusion »
n’est pas une mesure conique par défaut. La qualité de J0 ne valide ni les métriques d’un
modèle conique ni une équation RANS. T089 devra dériver et figer contrôles de non-singularité,
orientations, mesures et erreurs après G-FORM, avant JG-C ; des tableaux de métriques détaillés
ne sont pas introduits dans le noyau plan par anticipation. Les associations cellule/face
restent explicites dans le canonique et les exports.
