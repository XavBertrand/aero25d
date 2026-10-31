# Recherche et décisions — Phase 0

> Réconciliation R2 : objectif produit 2D/2.5D/2.75D dans la
> [feuille de route](../../docs/product-roadmap.md). Ce document décrit le chemin plan
> de 001 ; la géométrie conique attend [G-FORM](conical-study.md), sans RANS livré.

Date : 2026-10-30. Statut : décisions de conception, aucune preuve de fonctionnement du
mailleur. Recherche répartie entre géométrie/évaluateurs, topologies et contrats ; consolidation
ci-dessous. Les sources citées étayent les formats/licences ou méthodes, pas une réussite
sur nos géométries. Les interfaces planes sont décidées ; la formulation conique D-10 reste scientifiquement ouverte et bloquante pour sa construction.

## R-01 — Noyau et construction (D-05)

**Décision** : C++20, bibliothèque `aero25d_mesh_core`, exécutable `aero25d-mesh`, CMake ≥3.25,
Ninja, GCC ≥12 de référence et Clang ≥16 comme seconde compilation de contrôle. Presets schéma 6 :
`cpu-debug`, `cpu-release`, `cpu-sanitize`, répertoires `build/<preset>`, tests via CTest.
Linux/WSL, aucun langage CUDA activé dans CMake et aucune détection de toolkit requise.

**Raison** : un seul algorithme, bibliothèque testable sans CLI, utilisation CLion des presets
standard, tableaux contigus transférables ultérieurement. Référence binaire64, ordre déterministe,
pas de fast-math, contraction FMA désactivée dans les prédicats/évaluateurs si leur preuve l'exige.

**Alternatives** : un noyau Python dupliquerait la future logique C++ ; CUDA maintenant ajouterait
une dépendance inutile ; pas de framework général de solveur, de plugins ni de partitionnement GPU.
Les règles de version 6 sont documentées par [CMake 3.25](https://cmake.org/cmake/help/v3.25/manual/cmake-presets.7.html).

## R-02 — Python par processus structuré (D-05)

**Décision** : Python ≥3.11, uniquement bibliothèque standard pour l'enveloppe initiale.
`subprocess.run` avec liste d'arguments, sans shell ; JSON de configuration/résultat, fichiers
binaires pour les gros tableaux. API Client décrite dans les contrats, aucune logique géométrique.

**Raison** : satisfait les parcours ordinaires, conserve les diagnostics et isole les échecs natifs ;
pas de matrice d'ABI Python/C++, pas de dépendance NumPy obligatoire. Le coût de lancement et
les copies seront mesurés ; aucune latence n'est annoncée.

**Alternative** : liaison native pybind11 (BSD-3-Clause) utile pour échanges fréquents en mémoire,
mais reportée tant qu'un besoin mesuré ne la justifie pas. Une future liaison appellera la même
bibliothèque ; elle ne modifiera pas le modèle de données. Aucune dépendance pybind11 installée ici.
[Documentation officielle pybind11](https://pybind11.readthedocs.io/en/stable/intro.html).

## R-03 — Persistance et inspection (D-05)

**Décision** : JSON versionné pour configuration, manifeste, repères et rapport ; tableaux
little-endian f64/u64 avec descripteurs de dimensions/tailles/empreintes. Canonique : une table
unique de sommets et indices par bloc. VTK XML ASCII `.vts` par bloc dans un `.vtm`, `.vtp` pour contours/frontières, plus manifeste
d'interfaces et global_node_id. Écriture C++ spécifique au sous-ensemble choisi ; pas de VTK
à l'exécution. Cinq figures SVG générées par le noyau d'export C++, avec échantillonnage seulement
pour affichage ; leurs points ne servent pas à certifier la fidélité.

**Raison** : simple à relire, structure multibloc conservée sans énorme bibliothèque de rendu.
Le [format XML VTK officiel](https://docs.vtk.org/en/v9.6.1/vtk_file_formats/vtkxml_file_format.html)
définit StructuredGrid ; le [lecteur multibloc officiel](https://vtk.org/doc/nightly/html/classvtkXMLMultiBlockDataReader.html)
permet la relecture indépendante des exports. Les raccords conservatifs sont notre contrat,
pas une propriété déduite du fichier VTK seul.

**Alternatives** : HDF5 et CGNS utiles plus tard mais trop de complexité initiale ; JSON pour tous
les nombres du maillage trop volumineux ; capture graphique externe non reproductible écartée.

## R-04 — Géométrie certifiable (D-02/D-04)

**Décision** : provenance immuable, normalisation globale pour les calculs, orientations adaptatives
exactes sur les nombres représentés, arcs cubiques paramétriques clamped avec coins protégés,
certification par enveloppes Bézier et intervalles, redistribution par densité de courbure.
Voir [geometry-verification.md](geometry-verification.md) pour les décisions complètes, cas et limites.

**Raison** : distingue modification polygonale, fidélité analytique et forme réelle inconnue ;
la géométrie sous-échantillonnée ou indécidable ne reçoit pas un faux certificat.

**Alternatives** : interpolation en Z(X) singulière au BA ; spline libre sans contrôle d'oscillation ;
polyligne présentée comme rayon lisse ; epsilon unique pour tous les contacts. Écartées.
Les [prédicats de Shewchuk](https://www.cs.cmu.edu/~quake/robust.html) sont une source primaire
avec code de prédicats dans le domaine public, distinct du logiciel Triangle.

## R-05 — Topologies explicites (D-01)

**Décision** : bord fin : quatre blocs U/L de corps et WU/WL de sillage ; bord épais : cinq,
ajout H entre les limites de sillage, avec face de culot réelle. Table exhaustive des arêtes,
indexation et construction des coins dans [topology.md](topology.md).

**Raison** : les blocs de sillage dissocient Nwake de Nnormal. Aux lèvres épaisses, un secteur
extérieur distinct occupe l'espace entre normale de branche et normale du culot ; pas de
croisement de deux extrusions ni de cellule obtenue par effondrement d'un culot.

**Alternatives** : C à deux blocs / C–H à trois blocs : mêmes indices normaux/sillage imposés,
coin à deux secteurs souvent très oblique, rejetées pour l'interface utilisateur souhaitée.
Un réseau arbitraire de nombreux blocs sans démonstration préalable n'apporte aucune garantie.

## R-06 — Génération, contrôle et familles (D-03/D-06)

**Décision** : courbes et bandes proches paroi imposées, interpolation transfinie, lissage elliptique
optionnel sous contraintes, contrôle indépendant exhaustif. Profil Q1 et règle de famille F1,
avec limites et échecs dans [mesh-quality.md](mesh-quality.md). Les limites sont des choix de
projet proposés et révisables seulement par changement documenté avant gel des références.

**Raison** : ne pas confondre validité topologique, robustesse numérique, indicateur de qualité
et validation RANS. Geler le cœur permet une variation indépendante du domaine.

**Alternatives** : lisser puis accepter toute aire positive ; ratio d'aspect seul ; trois nombres
de cellules sans règles de raffinement ; déplacer la frontière et régénérer silencieusement
la paroi. Écartées.

## R-07 — Estimation facultative y+ (D-07)

**Décision** : utiliser uniquement un u_tau fourni par l'utilisateur et une viscosité cinématique ν,
plus l'échelle physique pour la distance du centre ; y+_estimé=u_tau d_center/ν, par face de paroi.
Absence de ces données : estimation absente, génération géométrique autorisée. Exiger u_tau et ν
finis positifs et déclarer leur origine. La première version utilise un u_tau scalaire uniforme, déclaré comme hypothèse ; les valeurs locales ne sont pas déduites des coordonnées.

**Raison** : une vitesse de frottement supposée ne garantit ni cisaillement ni y+ réel avant résolution.
**Alternatives** : corrélation plaque plane automatique ou estimation à partir du seul Reynolds :
non retenues initialement, car elles ajouteraient un modèle empirique et des limites de validité.
Aucune valeur de Mach/Reynolds n'est nécessaire à cette fonctionnalité géométrique.

## R-08 — Dépendances et licences

Les versions ci-dessous sont des cibles de construction, pas un inventaire de paquets déjà
installés. Avant réalisation, les sources choisies seront archivées avec version exacte,
SHA-256 et notices dans `third_party/manifest.json` ; aucun téléchargement automatique en
configuration normale. Prévoir des caches/vendeurs locaux et une installation par paquets.
Les correctifs de sécurité des bibliothèques système restent autorisés et sont enregistrés.

| Dépendance | Version cible / licence | Nécessité et portée |
| --- | --- | --- |
| nlohmann/json | 3.12.0 ; MIT | Bibliothèque générique pour lire/écrire le contrat JSON, erreurs structurées ; évite un parseur JSON maison. Vendored ou paquet exact, pas de réseau runtime. |
| OpenSSL libcrypto | ≥3.0, version exacte archivée ; Apache-2.0 | SHA-256 des sources/tableaux pour intégrité et reproduction ; seule la cible Crypto est liée, aucun TLS/réseau. Évite une implémentation cryptographique propre. |
| Prédicats de Shewchuk | predicates.c publication 1996, copie exacte et empreinte à archiver ; domaine public indiqué par l'auteur | Sous-ensemble orient2d/expansions, adaptation namespace/compilation revue ; aucun mailleur externe. |
| CMake/CTest | ≥3.25 ; BSD-3-Clause | Construction, installation et orchestration des tests ; uniquement développement. |
| Ninja | ≥1.11 ; Apache-2.0 | Générateur de construction choisi pour CLion/CI ; développement seulement. |
| GCC et libstdc++ / Clang alternatif | GCC≥12 ; GPL-3.0 avec GCC Runtime Library Exception ; Clang≥16 Apache-2.0 avec LLVM exception | Compilateur standard ; conserver les notices runtime applicables à la distribution, pas de code de compilateur embarqué dans le noyau. |
| Python standard | ≥3.11 ; licence PSF et notices associées | Enveloppe et tests de contrat via unittest ; aucun paquet de calcul requis. |
| Catch2 | 3.8.1 ; BSL-1.0 | Assertions, cas paramétrés et diagnostics C++ ; seulement avec BUILD_TESTING=ON. |
| yaml-cpp | 0.8.0 cible ; MIT | Parse configurations unitaires et campagnes YAML dans le lecteur C++ de documents partagé, après J0/T025 ; conversion au modèle JSON commun. Aucun moteur Python ni parseur YAML maison ; source locale figée avec empreinte/notice. |
| VTK Python (ou PyVista et VTK) / ParaView | versions exactes de campagne ; BSD-3-Clause, MIT pour PyVista et notices tierces | Un lecteur VTK ou PyVista est obligatoire en qualification des exports ; ouverture ParaView obligatoire. Dépendances des tests uniquement, jamais du mailleur livré. |
| jsonschema Python | version exacte de qualification à figer ; MIT | Validation indépendante JSON Schema 2020-12 des contrats, tests/documentation uniquement. |

Sources primaires des licences : [nlohmann](https://github.com/nlohmann/json),
[OpenSSL](https://github.com/openssl/openssl), [CMake](https://cmake.org/licensing/),
[Ninja](https://github.com/ninja-build/ninja/blob/master/COPYING),
[GCC exception](https://www.gnu.org/licenses/gcc-exception-3.1.html),
[LLVM](https://llvm.org/docs/DeveloperPolicy.html#license),
[Python](https://www.python.org/about/legal/), [Catch2](https://github.com/catchorg/Catch2),
[VTK](https://vtk.org/about/). Les notices exactes des artefacts distribués seront vérifiées
avant redistribution ; ce plan n'affirme aucune distribution déjà qualifiée.

## R-09 — YAML, orchestration et exports supplémentaires (D-08/D-09)

Choisir yaml-cpp 0.8.0, [version publiée](https://github.com/jbeder/yaml-cpp/releases/tag/0.8.0)
avec [licence MIT](https://github.com/jbeder/yaml-cpp/blob/0.8.0/LICENSE).
C’est une cible figée à archiver, pas une affirmation sur la dernière version disponible.
La bibliothèque standard C++ ne parse pas YAML ; une dépendance générique évite un parseur
complexe propre et maintient la parité CLI/Python par le même modèle C++.
L’alternative PyYAML imposerait Python au parcours autonome ou deux parseurs ; un simple
convertisseur externe contredirait l’accès équivalent et les chemins/règles uniques.
La portée est limitée au lecteur de documents partagé après J0/T025 ; le noyau géométrique n’en dépend pas et la campagne réutilise ce lecteur.
Le sous-ensemble YAML contractuel est contrôlé au-dessus de la bibliothèque, sans supposer
que ses conversions implicites ont la sémantique JSON souhaitée.

Le contrat VTP utilise PolyData/Lines selon les [formats XML officiels VTK](https://docs.vtk.org/en/latest/vtk_file_formats/vtkxml_file_format.html).
L’écriture reste un sous-ensemble XML C++ ; le lecteur indépendant est obligatoire dans
les tests de qualification et l’ouverture ParaView conserve sa propre preuve. Le choix
VTK direct minimise les dépendances de test ; PyVista est une alternative de qualification,
avec version et licence enregistrées si retenue, sans ajouter PyVista au produit.

## R-10 — Cadrage scientifique et décision D-10 ouverte

La [note conique](conical-study.md) constitue l’étude primaire initiale. Elle compare
transformations 2D, Euler conique et visqueux/RANS conique, avec accès effectif et limites de
chaque source. Le travail Polytechnique identifié concerne l’extrusion en flèche 2.5D ; il
n’est pas une preuve RANS conique. Le candidat visqueux impose une dérivation des variations
transversales, viscosité/Reynolds, énergie, distance paroi et SA-neg. Une similitude géométrique
ne suffit pas à les fermer. Les pentes locales et la station de référence remplacent un
simple taper ratio non déterminant ; les entrées redondantes seront contrôlées.

Décision actuelle : conserver le calcul local sensible à l’effilement comme objectif produit ;
ne retenir encore ni coordonnées physiques, ni fermeture, ni contrat métrique coniques.
La décision T088 doit permettre ou refuser explicitement la suite T089–T093. Les définitions
sectionnelles et la restitution line of flight sont dans la roadmap, avec intégrales à dériver.
Les approches d’équivalence restent références de comparaison, Euler une vérification
inviscide séparée ; aucune ne remplace le RANS compressible pleinement turbulent SA-neg exigé.

## R3 — décisions contractuelles ciblées (aucune extension scientifique)

U2 : retirer curvature_regularization_over_c non consommé par le moniteur retenu ; la loi
sqrt(c|κ|), sa raison géométrique (écart corde–arc) et ses limites sont explicitées dans
geometry-verification.md §5/R3. Les arcs réguliers ont κ finie ; les angles sont séparés,
κ=0 vaut zéro, l'erreur de quadrature est contrôlée sans régulariser la forme.
U3 : source-scale-v1 mesure c_input dans S, mètres faisant autorité ou conversion L/c_input.
Tolérance fixe 1e-10 relative et 0 m absolue, archivée avec bornes ; aucun facteur de flèche
n'est une échelle physique. U1 : la politique qualité consommée par la génération appartient
à mesh_key ; audits et estimations ont leurs propres clés/résultats immuables. Ces décisions
sont mathématiques/contractuelles locales ; elles ne tranchent aucun point de G-FORM.
