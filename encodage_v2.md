# Les encodages

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$, pour la culture, je précise que le terme __bit__ vient de la contraction de __binary digit_.
Un champ binaire est quant à lui composé d'un ensemble de bit, c'est-à-dire d'un ensemble de $0$ et de $1$.
Ce sont ces champs binaires qui nous permettent de représenter des nombres dans un ordinateur, et par extension de faire des calculs ainsi que de mémoriser des résultats.

Il y a cependant de nombreuses façons de représenter des nombres dans un champ binaire, nous parlons de l'__encodage__ des nombres.
Tous les __encodages__ ne se valent pas car il y en a des plus adaptés/efficaces que d'autres en fonction des besoins.
Par exemple, pour représenter des nombres entiers naturels dans un champ de $N$ bits, nous pourrions simplement calculer la somme des bits qui compose le champ.
Cependant, cet __encodage__ ne remplit aucun cahier des charges et risque d'être sous optimal pour bien des situations.

Dans un premier temps, nous allons expliqués comment fonctionnent deux des __encodages__ les plus connus : le __binary unsigned__ ainsi que la __virgule flottante__.
Ces __encodages__ sont néanmoins intrinséquement liés au concept de la __notation positionnelle__, ce qui nous pousse à évoquer ce sujet avant même celui desdits __encodages__.

## La notation positionnelle

La __notation positionnelle__ est une sorte d'__encodage__ qui permet de représenter les nombres de façon générique sous n'importe quel base numérique.
Veuillez noté que la __notation positionnelle__ n'est pas la seule manière qui nous est donné pour représenter des nombres, il existe aussi la __notation scientifique__ qui, comme son nom l'indique, est principalement utilisée dans le monde des sciences ; le document consacre plus loin un chapitre entier à la __notation scientifique__.

Dans notre quotidien, lorsque nous manipulons des nombres (en base décimale) nous les représentons usuellement en __notation positionnelle__, comme ceci avec le nombre $23.75$ par exemple.
Il s'avère qu'en __notation positionnelle__, chaque chiffre constituant le nombre $23.75$ est facteur d'une puissance de $10_{10}$ car ledit nombre est écrit en base décimale, en base $10_{10}$ ; les chiffres constitutifs d'un nombre écrit en base $N$ étant facteurs d'une puissance de $N$.
Plus bas, nous verrons que ces puissances là peuvent être positives, nulles, ou bien négatives, tout ne dépend que du chiffre facteur de ladite puissance.
Enfin, le nombre sous-jacent a pour valeur la somme des produits entre chiffres et puissances.
Voici une illustration qui montre comment est-ce-que la valeur du nombre $23.75$ se déduit en base décimale :

$$23.75 \ = \left(2 \times 10_{10}^{\quad 1} + 3 \times 10_{10}^{\quad 0} + 7 \times 10_{10}^{\quad -1} + 5 \times 10_{10}^{\quad -2}\right)$$

Notez que la syntaxe suivante $X_{10}$ spécifie la base numérique dans laquelle le nombre $X$ est écrit, $10$ pour la base décimale et $2$ pour la base binaire.
Cette syntaxe ne sera utilisée que lorsqu'il y aura une ambiguïté dans l'interprétation d'un nombre composé uniquement de $0$ et de $1$, permettant ainsi de dissocier les nombres écrits en base décimale de ceux écrits en base binaire.

Dans l'illustration qui figure ci-dessus, remarquez que les chiffres de la partie entière du nombre multiplient des puissances de $10_{10}$ positives ou nulles, tandis que les chiffres de la partie décimale multiplient exclusivement des puissances de $10_{10}$ négatives.
Cela s'explique par le fait que les chiffres de la partie entière du nombre $23.75$ requièrent de multiplier des nombres entiers, comme ceux résultants des puissances de $10_{10}$ positives et nulles.
Dans la même veine, les chiffres de la partie décimale du nombre nécessitent quant à eux de multiplier des nombres fractionnaires compris dans l'intervalle $\left]0;1\right[$, des puissances de $10_{10}$ négatives en l'occurrence.
Ce qui précède nous fait comprendre que de manière plus général, tout chiffre constituant la partie entière d'un nombre écrit en base $N$ est facteur d'une puissance de $N$ positive ou nulle, tandis que tout chiffre constituant la partie fractionnaire est facteur d'une puissance de $N$ négative. 

A cela, ajoutons que les valeurs des puissances de $10_{10}$ multipliées par deux chiffres successifs composant le nombre $23.75$, sont séparées d'un facteur $10_{10}$ ; pour généralisé, les valeurs des puissances de $N$ multipliées par deux chiffres successifs constituant un nombre écrit en base $N$, sont séparées d'un facteur $N$.

Par ailleurs, en plus d'expliquer le fonctionnement de la __notation positionnelle__, ce chapitre parle également de l'une des propriétés de cette notation.
Souvenez-vous du fait que la __notation positionnelle__ s'applique à toutes les bases numériques, ce qui insinue que ladite propriété s'applique elle aussi à l'ensemble des bases numériques.
En l'occurrence, cette propriété nous dit que dans tout nombre écrit en base $N$ : la valeur d'un chiffre non nul de poids $i$ est strictement supérieur à la somme des valeurs des chiffres de poids inférieur à $i$.

A ce stade nous savons que tout chiffre constitutif d'un nombre écrit en base $N$ est le facteur d'une puissance de $N$, la valeur d'un chiffre n'est rien de plus que le produit entre ledit chiffre et la puissance de $N$ dont il est le facteur.
Quant au poids d'un chiffre, celui-ci fait référence à l'exposant de la puissance de $N$ dont ledit chiffre est facteur, en __notation positionnelle__ le poids est un moyen universel d'indexer un chiffre dans un nombre, nonobstant la base numérique utilisée.
Prenons pour exemple le chiffre des unités du nombre décimale $23.75$, il se trouve que la valeur de ce chiffre est de $\left(3 \times 10_{10}^{\quad 0}\right)$ tandis que le poids de ce dernier est de $0$.

Avec tout cela en tête, constatons que la valeur du chiffre non nul de poids $0$ du nombre décimale $23.75$ est bel et bien strictement supérieur à la somme des valeurs des chiffres de poids inférieur à $0$ :

$$\left(3 \times 10_{10}^{\quad 0}\right) \gt \left(7 \times 10_{10}^{\quad -1} + 5 \times 10_{10}^{\quad -2}\right)$$

### L'encodage binary unsigned

Grâce au chapitre précédent nous savons désormais ce qu'est la __notation positionnelle__, et qui plus est, qu'elle s'applique à n'importe quel base numérique.
En l'occurrence, il s'avère que le __binary unsigned__ est un encodage qui représente des nombres entiers naturel en base binaire, et ce, par le biais de la __notation positionnelle__.
Comprendre l'encodage __binary unsigned__ demande donc de comprendre comment est-ce-que la __notation positionnelle__ représente des nombres entiers naturel en base binaire.

Le chapitre précédent nous fait comprendre qu'en __notation positionnelle__ : tout chiffre constitutif d'un nombre entier écrit en base $N$ est facteur d'une puissance de $N$ positive ou nulle.
En reportant ceci à tout nombre entier naturel écrit en base $2$, nous en déduisons que l'ensemble des bits qui composent un tel nombre sont facteurs d'une puissance de $2$ positive ou nulle.
La valeur d'un nombre entier naturel écrit en base $2$ n'est alors plus que la somme des produits entre bits et puissance.
Voici une illustration qui montre comment est-ce-que le nombre $23$ s'écrit en base binaire sous sa __notation positionnelle__ :

$$10111_2 \ = \left(1 \times 2^4 + 0 \times 2^3 + 1 \times 2^2 + 1 \times 2^1 + 1 \times 2^0\right)$$

En outre, je vous rappelle qu'il existe un système de poids en __notation positionnelle__, ce dernier permettant l'indexation de tout chiffre dans tout nombre qu'importe la base numérique.
Plus précisément, il s'avère que le poids d'un chiffre constitutif d'un nombre écrit en base $N$ n'est autre que l'exposant de la puissance de $N$ dont ledit chiffre est facteur.
Cependant, pour des raisons pratiques, il existe également des termes qui sont rentrés dans le langage commun de l'informaticien ; ces termes là font référence à des bits d'un poids précis ou ayant une spécificité au sein du champ qu'il compose.

De plus, rappelez-vous du système de poids dont il est le sujet dans le chapitre précédent.
Ce dernier sert à l'indexation de tout chiffre dans tout nombre peu importe la base numérique, je vous rappelle le poids d'un chiffre constitutif d'un nombre écrit en base $N$ n'est rien dautre que l'exposant de la puissance de $N$ que multiplie ledit chiffre.
Seulement, veuillez prendre note du fait que pour des raisons pratiques, quelques termes sont rentrés dans le langage commun de l'informaticien, ces termes font référence à des bits d'un poids précis ou ayant une particularité au sein du champ qu'il compose.
Pour ne citer que les principaux, le terme __Least Significant Bit__ de l'acronyme __LSB__ fait référence au bit de poids le plus faible d'un champ, ou autrement dit au bit de poids $0$ d'un nombre codé en __Binary Unsigned__.
A contrario, le terme __Most Significant Bit__ de l'acronyme __MSB__ fait référence au bit de poids le plus fort d'un champ.
Aussi, il existe les acronymes __LSB1__ et __MSB1__ faisant respectivement référence au bit à $1$ de poids le plus faible et de poids le plus fort d'un champ.




///

Vous l'aurez compris, comprendre l'encodage __binary unsigned__ demande de comprendre le fonctionnement de la __notation positionnelle__ en base binaire, ce qui s'annonce relativement simple car nous nous contenterons de faire référence au chapitre précédent.
En l'occurrence, ledit chapitre mentionne qu'en __notation positionnelle__ : tout chiffre constitutif d'un nombre écrit en base $N$ est le facteur d'une puissance de $N$, ce qui insinue dans le cas d'un nombre binaire, écrit en base $2$, que tout bit constituant le nombre est facteur d'une puissance de $2$.
Enfin, le nombre binaire sous-jacent a pour valeur la somme des produits entre bits et puissances.

//

Pour y voir plus clair, prenons le nombre entier naturel $23$ comme exemple, nombre qui sous l'impulsion de la __notation positionnelle__ s'écrit $10111_2$ en base binaire.
Voici une illustration qui montre comment est-ce-que la valeur de ce nombre est réelement calculée :

$$10111_2 \ = \left(1 \times 2^4 + 0 \times 2^3 + 1 \times 2^2 + 1 \times 2^1 + 1 \times 2^0\right)$$

Pour commencer, remarquons que l'ensemble des bits du nombre multiplient une puissance de $2$, qui plus est, une puissance positive ou nul à l'image des puissances que multiplie les chiffres de la partie entière du nombre décimale $11.75$ pris pour exemple dans le chapitre précédent.
Autrement dit de façon plus général, tout chiffre constitutif d'un nombre entier relatif écrit en base $N$ est le facteur d'une puissance positive ou nul de $N$.
Il est simple de comprendre pourquoi, les puissances de $N$ négatives interprètent des nombres fractionnaire compris dans l'intervalle $\left]0;1\right[$, ce qui explique le fait que les chiffres constituant tout nombre entier ne multiplient que des puissances positives ou nul.


//

Pour commencer, remarquons que l'ensemble des bits du nombre multiplient une puissance de $2$, qui plus est, une puissance positive ou nul.
Gardons à l'esprit que le nombre sous-jacent est un entier, les bits qui le compose ne peuvent pas être facteur d'une puissance de $2$ négative car ces dernières interprètent des nombres fractionnaire compris dans l'intervalle $\left]0;1\right[$.
Cependant, ceci n'est pas propre à la base binaire, faisons un rapide aparté pour parler du cas du chapitre précédent "_La notation positionnelle_".
Il s'avère que ledit chapitre utilise le nombre décimale $11.75$ pour expliquer le fonctionnement de la __notation positionnelle__ en base décimale, ce dernier mentionne entre autre le fait que les chiffres de la partie entière dudit nombre multiplient des puissances de $10_{10}$ positives ou nul.





// autre exemple avec ref

Faisons un rapide aparté pour revenir au chapitre "_La notation positionnelle_", ce qui nous intéresse c'est plus particulièrement le nombre décimale $11.75$ ayant été choisit pour expliquer le fonctionnement de la __notation positionnelle__ en base $10_{10}$.
Dans ce chapitre il est expliqué que les chiffres de la partie entière dudit nombre multiplient des puissances de $10_{10}$ qui sont exclusivement positives ou nul.



// après

A cela, ajoutons que la valeur des puissances de $10_{10}$ multipliées par deux chiffres successifs est séparé d'un facteur $10_{10}$ ; pour généralisé, la valeur des puissances de $N$ multipliées par deux chiffres successifs constituant un nombre écrit en base $N$, est alors séparée d'un facteur $N$.


//

Ce qui suit va expliquer le fonctionnement de la __notation positionnelle__ en base binaire, pour ce faire nous allons entre autre prendre le nombre entier naturel $11$ pour exemple ; ce nombre faisant référence à la partie entière du nombre $11.75$ utilisé comme exemple dans le chapitre précédent.

Il va être fait plusieurs fois référence au chapitre précédent dans ce qui suit parce que nous allons expliquer le fonctionnement de la __notation positionnelle__ en base binaire.


// comment fonctionne la notation positionnelle en base binaire?


//

A l'instar de tout encodage, le __binary unsigned__ permet de représenter des nombres, qui plus est, des nombres entiers naturels.
L'encodage desdits nombres s'effectue en __notation positionnelle__, d'où le fait qu'il ait été fait antérieurement mention d'un lien entre cet encodage et la __notation positionnelle__, nous y reviendrons.
Par ailleurs, nous devons le nom __binary unsigned__ de cet encodage, traduit en français par __binaire non signé__, au fait que les nombres pouvant être codés le sont sans leur signe.

Comme nous l'avons vu avec le chapitre précédent, la __notation positionnelle__ s'applique à toutes les bases numériques dont la base binaire.
Il est dit dans ce chapitre que les chiffres constitutifs d'un nombre écrit sous une base $N$ en __notation positionnelle__ sont facteurs d'une puissance de $N$, ce qui veut dire que tout bit d'un nombre codé en __binary unsigned__ est facteur d'une puissance de $2$.
Observons comment est-ce-qu'il est possible de coder la partie entière $11$ en __binary unsigned__ du nombre $11.75$ pris pour exemple dans le chapitre qui précède.

// expliquer d'abord le fonctionnement de la notation positionnelle en binaire pour la représentation du nombre $11$, puis faire le "lien" avec le binary unsigned?


//

## L'encodage Binary Unsigned (à faire)

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
