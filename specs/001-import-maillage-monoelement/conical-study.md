# Étude préalable des formulations sectionnelles coniques — D-10 ouverte

**Réconciliation R2, 2026-10-31.** Annexe de [001](spec.md) et de la
[feuille de route produit](../../docs/product-roadmap.md). État : étude bibliographique
initiale et questions de dérivation identifiées ; **G-FORM non accepté**. Aucun prototype,
contrat métrique conique détaillé ni calcul de pression n’est livré. Les tâches T084–T088
complètent les dérivations et leur revue ; T089–T093 restent conditionnelles à leur go.

## Sources primaires et accès effectif

| Réf. | Source et accès au 2026-10-31 | Portée exploitable et limites |
| --- | --- | --- |
| C-01 | R. C. Lock, *An equivalence law relating three- and two-dimensional pressure distributions*, ARC R&M 3346, travail de mai 1962, publication souvent citée 1964 ; [notice Cranfield](https://reports.aerade.cranfield.ac.uk/handle/1826.2/3927) | Notice/résumé consultés, document référencé ; transformation de distributions, pas preuve de notre RANS conique |
| C-02 | Z.-M. Xu, Z.-H. Han, W.-P. Song, *An improved 2.75D method relating pressure distributions of 2D airfoils and 3D wings*, AST 128 (2022), 107789, [DOI](https://doi.org/10.1016/j.ast.2022.107789) | Résumé/introduction indexés ; accès direct éditeur refusé (403) ; méthode de transformation, équations complètes à acquérir |
| C-03 | E. M. Lee, J. T. Batina, *Conical Euler simulation of wing rock for a delta wing planform*, Journal of Aircraft 28 (1991), [NASA NTRS](https://ntrs.nasa.gov/citations/19910048913) | Notice/résumé, aucun téléchargement NTRS ; Euler conique supersonique, pas RANS turbulent transsonique |
| C-04 | T. Streit, G. Wichmann, F. von Knoblauch zu Hatzbach, R. L. Campbell, *Implications of Conical Flow for Laminar Wing Design and Analysis*, AIAA 2011-3808, [DOI](https://doi.org/10.2514/6.2011-3808) | Référence/résumé identifiés ; texte intégral non obtenu, dérivations à vérifier avant choix |
| C-05 | T. Streit, S. Wedler, M. Kruse, *DLR natural and hybrid transonic laminar wing design incorporating new methodologies*, Aeronautical Journal 119(1221), 1303–1326 (2015), DOI 10.1017/S0001924000011283 ; [PDF complet](https://www.aerosociety.com/media/3576/4311.pdf) | Texte consulté, notamment §2.1.2 p.1308 (page PDF 6) ; méthode sectionnelle dans FLOWer, sans preuve de notre SA-neg ou de nos blocs |
| C-06 | S. Bourgault-Côté, S. Ghasemi, A. Mosahebi, É. Laurendeau, *Extension of a Two-Dimensional Navier–Stokes Solver for Infinite Swept Flow*, AIAA Journal 55(2), 662–667 (2017), [PolyPublie](https://publications.polymtl.ca/36528/), [DOI](https://doi.org/10.2514/1.J055139) | Métadonnées institutionnelles/éditeur ; texte non archivé dans PolyPublie, accès institutionnel intermittent ; travail confirmé de Polytechnique Montréal pour **2.5D extrudé**, pas attribué au conique |
| C-07 | S. R. Allmaras, F. T. Johnson, P. R. Spalart, *Modifications and Clarifications for the Implementation of the Spalart–Allmaras Turbulence Model*, ICCFD7-1902 (2012), [texte des auteurs](https://www.iccfd.org/iccfd7/assets/pdf/papers/ICCFD7-1902_paper.pdf), [TMR SA-neg](https://tmbwg.github.io/turbmodels/spalart.html) | Texte et définition TMR consultés ; modèle compressible/branche négative, aucune fermeture conique fournie automatiquement |

La recherche a identifié un travail Polytechnique pertinent sur l’extrusion en flèche.
Elle n’a pas identifié de formulation RANS conique de Polytechnique dans le corpus consulté ;
cette limite d’accès/recherche ne constitue pas une affirmation d’inexistence. T084 doit
compléter le corpus, vérifier les textes manquants et consigner auteurs/pages/équations.

## Comparaison des trois voies

| Voie | Où intervient l’effilement ? | Ce que cette voie peut étayer | Ce qui manque pour l’objectif produit |
| --- | --- | --- | --- |
| Transformation de résultats 2D | Conditions équivalentes, transformation de géométrie ou distributions après calcul | Estimation/contre-épreuve de Cp sous hypothèses d’équivalence (C-01/C-02) | Ne suffit pas à faire intervenir l’effilement dans les équations locales de pression |
| Formulation Euler conique | Réduction des équations inviscides par fermeture suivant une similitude | Champ inviscide réduit, exemple C-03 | Diffusion visqueuse, énergie dissipative, frottement et SA-neg absents ; domaine de C-03 distinct du nôtre |
| Formulation visqueuse/RANS conique | Géométrie et fermeture transversale dans le problème sectionnel résolu | Candidat à étudier pour PR-005/006 ; méthode sectionnelle C-04/C-05 | Dérivation propre des termes moléculaires/turbulents, restrictions et vérification SA-neg nécessaires |

Dans C-05, les conditions imposent des valeurs suivant des rayons issus de l’intersection
BA/BF, avec une section centrale et des sections latérales. Le proche champ utilise l’apex
géométrique ; le champ lointain relaxe ces conditions vers une périodicité extrudée. L’article
rapporte des comparaisons de distributions de pression. Cette transition est une hypothèse
à examiner, pas une règle adoptée silencieusement. Le contexte laminaire/transition de cette
publication ne démontre pas notre RANS pleinement turbulent ni l’aptitude des topologies 4/5
blocs. Aucun chiffre de performance ou accord quantitatif n’est repris de ses figures.

La décision d’orientation est de conserver l’objectif d’un calcul **local** sensible à
l’effilement et d’étudier une formulation visqueuse conique. Elle n’est pas un go de
formulation : aucune fermeture particulière n’est encore retenue et Euler n’est pas un repli
permettant d’annoncer la capacité SA-neg.

## Géométrie candidate et données indépendantes (proposition de projet)

Premier domaine proposé : tronçon trapézoïdal, profils exactement semblables, orientation
constante (aucune variation locale de vrillage), sans dièdre dans la première étude, station
de référence Y₀, corde positive sur un intervalle local déclaré. Les effets d’emplanture,
de saumon, ruptures de planform, changement de forme, apex proche et décollement fortement
tridimensionnel restent hors de l’hypothèse initiale. Aucun intervalle quantitatif de pentes
n’est déclaré qualifié avant JG-C et les futurs essais d’équations.

La paramétrisation candidate A utilise le profil original S à Y₀, c₀, la position d’une ligne
de référence (quart de corde), sa pente locale X′ref et la pente de corde c′, dans les unités
physiques cohérentes. Les lois candidates sont c(Y)=c₀+c′(Y−Y₀) et
Xref(Y)=Xref₀+X′ref(Y−Y₀), Zref constant. L’orientation de S reste celle de la source.
Une famille de contours semblables s’obtient en appliquant le facteur c(Y)/c₀ aux écarts
au point de référence, puis la translation de cette ligne. Il s’agit d’une description
géométrique à étudier, **pas** de coordonnées de calcul ou de métriques déjà décidées.
X′ref et une flèche de cette même ligne ne sont pas deux degrés de liberté indépendants.

L’alternative B fournit deux stations Y, leurs cordes et positions de ligne de référence,
permettant de dériver les pentes. Le taper ratio c₂/c₁ sans séparation des stations et
position de référence est insuffisant. A et B sont des modes d’entrée alternatifs ; s’ils
sont fournis ensemble, vérifier leurs résidus avec limites explicites et refuser une
incohérence. Une flèche BA/BF/ligne moyenne doit être nommée et convertie à partir de ces
données, sans les confondre avec Λ du cas extrudé.

La limite c′=0 doit retrouver l’extrusion avec la pente de ligne de référence conservée.
Le calcul de cette limite doit éviter une division numérique par un apex qui part à l’infini.
Cela ne préjuge pas du choix de section conique (plane, normale, autre surface) ni de sa
représentation numérique, décisions soumises à G-FORM.

## Obstacle visqueux/turbulent à résoudre

**Inférence dimensionnelle de projet, à dériver et vérifier** : une homothétie géométrique
ne rend pas automatiquement RANS auto-similaire à viscosité moléculaire et état amont fixés.
À vitesse et viscosité fixées, le Reynolds fondé sur la longueur varie avec la corde ; les
échelles convectives et diffusives changent différemment. Il faut annoncer toute approximation
de Reynolds local, viscosité ou exposant de similitude, sans ajustement physique implicite.

La revue doit dériver les dérivées transversales premières **et secondes** des grandeurs
physiques et des bases ; celles de densité, vitesse, énergie/température et viscosités ;
les gradients entrant dans les contraintes et flux thermiques ; la distance physique à
la surface solide (distincte d’une simple distance dans une coupe). Pour SA-neg, traiter
la variable transportée, production/destruction/diffusion, branche négative et conditions
aux limites dans les mêmes hypothèses. C-07 et la TMR définissent le modèle de référence,
mais ne déterminent pas ces choix coniques. Les éventuelles conditions relaxées du champ
lointain et la fermeture au sillage/culot épais doivent être explicites.

## Séparation des données et chemin plan

1. **Source** : contour line of flight, ordre/lexèmes/IDs/repères S conservés.
2. **Logique** : indices (i,j), blocs, régions et raccords, sans interprétation physique universelle.
3. **Physique** : immersion de la section choisie, coordonnées et normales dans l’espace physique.
4. **Transformations/bases** : application directe, inverse/mapping et variations des bases.
5. **Métriques/variations** : mesures et dérivées nécessaires aux équations retenues.

Le chemin plan garde ses deux coordonnées FP64, métriques euclidiennes et bases constantes.
Les tables 4/5 blocs sont à éprouver au J0 **plan** ; leur réutilisation conique est seulement
une hypothèse à tester après G-FORM. Pas d’interface abstraite générale de solveur ni tableau
métrique conique obligatoire dans les premiers en-têtes. Aucun ajout de taper_ratio ou d’un
troisième nombre par sommet ne satisfait le point 5 ou ne constitue une implémentation 2.75D.

## Restitution et efforts : décisions à dériver

Les définitions de sortie de [la feuille de route](../../docs/product-roadmap.md) fixent
q∞ total, c₀ originale, axes vent, quart de corde, moment sur +Y global et mesure par unité
de Y. L’étude doit dériver comment le modèle conique fournit les intégrales physiques
correspondantes, incluant normales, mesure de surface, pression et frottement ; elle ne
reprend pas automatiquement les facteurs d’extrusion. Aucune performance d’aile finie.

La table pariétale Cp(S) conserve les branches originales et leurs correspondances de
paramètres/IDs. Le champ p sur un plan line of flight demande une reconstruction selon la
similitude choisie et une intersection avec le plan demandé. La pression est scalaire ; les
vecteurs de frottement changent de base physique. Une inversion de la seule abscisse ne
vérifie ni ces opérations ni leur normalisation. JG-C teste seulement les correspondances
géométriques et un **scalaire analytique de test explicitement synthétique**, jamais une
pression calculée ou une capacité aérodynamique.

## Porte scientifique G-FORM — état ouvert

| Rubrique obligatoire | Sortie attendue avant go | État courant |
| --- | --- | --- |
| Similitude et domaine | Lois/exposants et approximations, restrictions mesurables | Candidat géométrique ; dérivation ouverte |
| Section/coordonnées | Section physique choisie et relation à S, bases et inverse | Non choisi |
| Données de géométrie | Paramétrisation indépendante, pentes, station et contrôles | A/B proposés, pas de schéma conique figé |
| Fermeture transversale | Dérivées premières/secondes et dépendance des bases | À dériver |
| Visqueux et turbulence | Contraintes, chaleur, viscosité, Reynolds local, distance paroi et SA-neg | À dériver/revoir |
| Métriques | Quantités nécessaires et identités, calcul/bornes et limite d’extrusion | Contrat détaillé bloqué avant décision |
| Conditions et limites | Parois, culot, sillage, champ lointain et éventuelle relaxation | À justifier |
| Restitution et coefficients | Mappings/mesure sectionnelle et intégrales physiques | Références fixées, dérivations ouvertes |
| Validation | Oracles, limites vers 2.5D, cas indépendants, erreurs séparées | Stratégie ci-dessous, seuils spécifiques après choix |

Le go exige toutes rubriques soutenues par dérivation et revue scientifique identifiée,
avec sources/pages/équations vérifiables. Un texte non obtenu reste une limite de preuve ;
un no-go conserve le diagnostic et les travaux suivants. La présence de cette note ne ferme
aucune de ces rubriques. T089–T093 ne commencent pas avant un go explicite de T088.

## Qualification géométrique et future vérification des équations

JG-C compare fin/épais, pente non nulle puis séquence tendant vers zéro : mapping/inverse,
sources inchangées, raccords/orientations, métriques non singulières, reconstruction de la
section line of flight et scalaire analytique synthétique. Les critères chiffrés, unités et
incertitudes sont dérivés de la formulation puis figés dans T089 avant l’acceptation ; ne
pas leur substituer les seules aires du maillage plan. Un échec reste distinct du go J0.

Les futurs travaux FUT-02/03 devront vérifier les métriques/identités et la conservation,
un écoulement uniforme **cartésien** représenté dans une base éventuellement variable,
la limite 2.5D, des solutions manufacturées avec diffusion et transport turbulent, et des
références indépendantes. Comparer séparément erreurs d’itération/maillage/domaine et erreur
de modèle conique ; comparer Cp/Cf et efforts avant toute revendication sur Cx≈10⁻⁴.
Ces essais d’équations ne sont pas exécutés ni implémentés dans 001.

## R3 — issue de la revue et bilan conditionnel

La décision G-FORM reste **OUVERTE**. T088 peut produire go ou no_go motivé ; une revue
non terminée ne produit aucun verdict. Après go, T089–T094 sont applicables, JG-C qualifie
seulement la géométrie si ses preuves passent. Après no_go, ces tâches sont non applicables
à cette décision (cases ouvertes, raisons conservées), et T095 peut clore le bilan
scientifique avec T083/T088, sans attendre leur exécution. Conserver résultats T084–T088,
limites de validité, obstacles et travaux non exécutés ; ce bilan n'est pas un JG-C réussi.
Une nouvelle étude/décision versionnée peut rouvrir ultérieurement la branche, sans effacer
le no-go précédent. Aucun contrat métrique détaillé ni capacité RANS n'est créé ici.
