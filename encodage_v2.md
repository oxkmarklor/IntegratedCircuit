# Les encodages

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$, pour la culture, je précise que le terme _bit_ vient de la contraction de _binary digit_.
Un champ binaire est quant à lui composé d'un ensemble de bit, c'est-à-dire d'un ensemble de $0$ et de $1$.
Ce sont ces champs binaires qui nous permettent de représenter des nombres dans un ordinateur, et par extension de faire des calculs ainsi que de mémoriser des résultats.

Il y a cependant de nombreuses façons de représenter des nombres dans un champ binaire, nous parlons alors de l'__encodage__ des nombres.
Tous les __encodages__ ne se valent pas car il y en a des plus adaptés/efficaces que d'autres en fonction des besoins.
Par exemple, pour représenter des nombres entiers naturels dans un champ de $N$ bits, nous pourrions simplement calculer la somme des bits qui compose le champ.
Cet __encodage__ ne remplit cependant aucun cahier des charges, au risque d'être donc sous optimal pour bien des situations.

Dans ce premier chapitre, nous allons expliqués comment fonctionnent deux des __encodages__ les plus connus : le __binary unsigned__ ainsi que la __virgule flottante__.
Ces __encodages__ sont néanmoins intrinséquement liés au concept de la __notation positionnelle__, d'où le fait que ce sujet soit abordé dans le chapitre suivant.

## La notation positionnelle

La __notation positionnelle__ est une sorte d'__encodage__ qui permet de représenter les nombres de façon générique sous n'importe quel base numérique.
Veuillez noté que la __notation positionnelle__ n'est pas la seule manière qui nous est donné pour représenter des nombres, il existe aussi la __notation scientifique__ qui, comme son nom l'indique, est principalement utilisée dans le monde des sciences ; le document consacre plus loin un chapitre entier à la __notation scientifique__. 

Dans notre quotidien, lorsque nous manipulons des nombres (en base décimale) nous les représentons communément en __notation positionnelle__, c'est-à-dire de la sorte : $11.75$.
En l'occurrence, il s'avère que chacun des chiffres qui compose ce nombre est le facteur d'une puissance $10_{10}$, le nombre sous-jacent ayant pour valeur la somme de ces produits.

// illu ou explication indice






/*
 2- la notation positionnelle dans le cas d'un nombre entier naturel (103, admettons) écrit en base 10
 3- illustration de la façon dont s'écrit un nombre en base 10
 4- la notation positionnelle dans les cas des nombres à virgule (103.375, par exemple) écrit en base 10 + référence pour la partie entière
 5- illustration de la façon dont s'écrit la partie décimale d'un tel nombre (inclure explication/illustration pour la partie entière, comme vu précédemment?)
 6- la propriété élémentaire que confère la notation positionnelle à l'ensemble des bases numériques
 7- exemple de cette propriété rattacher au cas des nombres écrits en base 10 (103, 103.375, ou les deux?)
 
*/

//

## L'encodage Binary Unsigned

Le __Binary Unsigned__ est un encodage permettant de coder des nombres entiers naturels par le biais de la notation positionnelle, exactement comme le fait la base décimale pour représenter des nombres.
Par exemple, en décimale chacun des chiffres qui compose la valeur $103$ est le facteur d'une puissance de $10_{10}$, la représentation du nombre sous-jacent se calcule comme la somme de ces produits.
Veuillez noter que la syntaxe suivante $X_{10}$ spécifie la base numérique dans laquelle le nombre $X$ est écrit, $10$ pour la base décimale et $2$ pour la base binaire.
Cette syntaxe ne sera utilisée que lorsqu'il y aura une ambiguïté dans l'interprétation d'un nombre composé uniquement de $0$ et de $1$.

$$103 \ = \left(1 \times 10_{10}^{\quad 2} + 0 \times 10_{10}^{\quad 1} + 3 \times 10_{10}^{\quad 0}\right)$$

Le chiffre des unités $3$ est facteur de $10_{10}$ à la puissance $0$ ; le chiffre des dizaines $0$ est facteur de $10_{10}$ à la puissance $1$, tandis que le chiffre des centaines $1$ est le facteur de $10_{10}$ à la puissance $2$.
Remarquez que les puissances de $10_{10}$ dont les chiffres sont facteurs, multiplient leurs valeurs par dix en passant du chiffre des unités, aux dizaines, puis des dizaines, aux centaines, etc. ; cependant, sachez que tout ce que nous venons de voir n'est pas propre à la base décimale.

La base binaire permet également de représenter des nombres d'une façon équivalente, et ce, grâce à l'encodage __Binary Unsigned__.
Avec cet encodage, chaque bit d'un nombre est le facteur d'une puissance de $2$ qui est positive ou nul, la valeur du nombre ainsi représenté se calcule par le biais de la somme de ces produits.
En outre, ces nombres ne codent pas de signe $\pm$ sur un bit, ce qui explique le fait que l'encodage __Binary Unsigned__ ne permette que le codage d'entiers naturels. 

$$103 \ = \ 1100111_2 \ = \left(1 \times 2^6 + 1 \times 2^5 + 0 \times 2^4 + 0 \times 2^3 + 1 \times 2^2 + 1 \times 2^1 + 1 \times 2^0\right)$$

Remarquez que les bits à $0$ n'affectent pas la valeur du nombre, c'est pourquoi vous entendrez souvent dire (y compris dans ce document) que la valeur d'un nombre codé en __Binary Unsigned__ se calcule comme la somme des puissances de $2$ dont le bit est à $1$.

En outre, chaque bit au sein d'un champ binaire a un positionnement qui lui est propre mais il n'existe pas de nom pour ces positions, dans les faits nous ne parlons pas de bit des unités, des dizaines, et ainsi de suite mais plutôt du _poids_ d'un bit.
Le poids d'un bit fait référence à la puissance de $2$ dont ce bit est facteur, plus un bit est facteur d'une grande puissance et plus son poids est fort (grand) tandis qu'à l'inverse plus il est facteur d'une petite puissance et plus son poids est faible (petit).
Dans l'illustration qui figure ci-dessus, le bit qui multiplie la puissance $2^6$ est de poids $6$ par exemple.
Par conséquent, il est très commun d'indexer les bits des champs par leurs poids ; le bit de poids $N$ fait référence au bit qui est facteur de la puissance $2^N$.

Il y a cependant quelques termes qui sont rentrés dans le langage commun de l'informaticien pour des raisons pratiques, ces termes font référence à des bits d'un poids précis ou ayant une particularité au sein du champ qu'il compose.
Pour ne citer que les principaux, le terme __Least Significant Bit__ de l'acronyme __LSB__ fait référence au bit de poids le plus faible d'un champ, c'est-à-dire au bit de poids $0$ d'un nombre codé en __Binary Unsigned__.
A contrario, le terme __Most Significant Bit__ de l'acronyme __MSB__ fait référence au bit de poids le plus fort d'un champ.
Aussi, il existe les acronymes __LSB1__ et __MSB1__ faisant respectivement référence au bit à $1$ de poids le plus faible et de poids le plus fort d'un champ.
