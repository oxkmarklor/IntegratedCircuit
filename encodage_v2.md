# Les encodages

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$, pour la culture, je précise que le terme _bit_ vient de la contraction de _binary digit_.
Un champ binaire est quant à lui composé d'un ensemble de bit, c'est-à-dire d'un ensemble de $0$ et de $1$.
Ce sont ces champs binaires qui nous permettent de représenter des nombres dans un ordinateur, et par extension de faire des calculs ainsi que de mémoriser des résultats.

Il y a cependant de nombreuses façons de représenter des nombres dans un champ binaire, nous parlons alors de l'__encodage__ des nombres.
Tous les __encodages__ ne se valent pas car il y en a des plus adaptés/efficaces que d'autres en fonction des besoins.
Par exemple, pour représenter des nombres entiers naturels dans un champ de $N$ bits, nous pourrions simplement calculer la somme des bits qui compose le champ.
Cet __encodage__ ne remplit cependant aucun cahier des charges et risque d'être sous optimal pour bien des situations.

Dans ce premier chapitre, nous allons expliqués comment fonctionnent deux des __encodages__ les plus connus : le __binary unsigned__ ainsi que la __virgule flottante__.
Ces __encodages__ sont néanmoins intrinséquement liés au concept de la __notation positionnelle__, d'où le fait que ce sujet soit abordé dans le chapitre suivant.

## La notation positionnelle

La __notation positionnelle__ est une sorte d'__encodage__ qui permet de représenter les nombres de façon générique sous n'importe quel base numérique.
Veuillez noté que la __notation positionnelle__ n'est pas la seule manière qui nous est donné pour représenter des nombres, il existe aussi la __notation scientifique__ qui, comme son nom l'indique, est principalement utilisée dans le monde des sciences ; le document consacre plus loin un chapitre entier à la __notation scientifique__. 

Dans notre quotidien, lorsque nous manipulons des nombres (en base décimale) nous les représentons communément en __notation positionnelle__, c'est-à-dire de la sorte : $11.75$ .
Il s'avère qu'en __notation positionnelle__ chaque chiffre constituant le nombre $11.75$ est facteur d'une puissance de $10_{10}$ car ledit nombre est écrit en base décimale, en base $10_{10}$ ; les chiffres constitutifs d'un nombre écrit en base $N$ étant facteurs d'une puissance de $N$.
Enfin, le nombre sous-jacent a pour valeur la somme des produits entre chiffres et puissances.
Voici l'illustration de la façon dont la valeur du nombre $11.75$ se calcul en base décimale :

$$11.75 \ = \left(1 \times 10_{10}^{\quad 1} + 1 \times 10_{10}^{\quad 0} + 7 \times 10_{10}^{\quad -1} + 5 \times 10_{10}^{\quad -2}\right)$$

Notez que la syntaxe suivante $X_{10}$ spécifie la base numérique dans laquelle le nombre $X$ est écrit, $10$ pour la base décimale et $2$ pour la base binaire.
Cette syntaxe ne sera utilisée que lorsqu'il y aura une ambiguïté dans l'interprétation d'un nombre composé uniquement de $0$ et de $1$, permettant ainsi de dissocier les nombres écrits en base décimale de ceux écrits en base binaire.

Dans l'illustration qui figure ci-dessus, remarquez que les chiffres de la partie entière du nombre multiplient des puissances de $10_{10}$ positives ou nul, tandis que les chiffres de la partie décimale multiplient des puissances de $10_{10}$ négatives.
A cela, ajoutons que la valeur des puissances de $10_{10}$ multipliées par deux chiffres successifs est séparé d'un facteur $10_{10}$ ; pour généralisé, la valeur des puissances de $N$ multipliées par deux chiffres successifs constituant un nombre écrit en base $N$, est alors séparée d'un facteur $N$.

Par ailleurs, en plus d'expliquer le fonctionnement de la __notation positionnelle__, ce chapitre parle également de l'une de ses propriétés.
Ladite propriété dont nous parlerons dans le reste de ce chapitre est commune à toutes les bases numériques car la __notation positionnelle__ l'est elle même.

// ajout d'un exemple avec le nombre décimale illustré plus haut

Concrètement, cette propriété nous dit que la valeur d'un chiffre non nul de poids $i$ est strictement supérieur à la somme des valeurs des chiffres de poids inférieur à $i$.
En décimale, le poids d'un chiffre fait référence à la puissance de $10_{10}$ dont ce chiffre est facteur, plus cette puissance est grande et plus le poids du chiffre est fort (ou grand), tandis que plus la puissance est petite et plus le poids du chiffre est faible (ou petit).

// faire un exemple avec un nombre en base 10 comme 11.75 pour clotûrer


En outre, chaque bit au sein d'un champ binaire a un positionnement qui lui est propre mais il n'existe pas de nom pour ces positions, dans les faits nous ne parlons pas de bit des unités, des dizaines, et ainsi de suite mais plutôt du _poids_ d'un bit.
Le poids d'un bit fait référence à la puissance de $2$ dont ce bit est facteur, plus un bit est facteur d'une grande puissance et plus son poids est fort (grand) tandis qu'à l'inverse plus il est facteur d'une petite puissance et plus son poids est faible (petit).
Dans l'illustration qui figure ci-dessus, le bit qui multiplie la puissance $2^6$ est de poids $6$ par exemple.
Par conséquent, il est très commun d'indexer les bits des champs par leurs poids ; le bit de poids $N$ fait référence au bit qui est facteur de la puissance $2^N$.

// réfléchir à la façon dont je peux expliquer la propriété (par le biais d'un exemple sur la base décimale exclusivement?) 


/*
 6- la propriété élémentaire que confère la notation positionnelle à l'ensemble des bases numériques
 
 7- exemple de cette propriété rattacher au cas des nombres écrits en base 10 (103, 103.375, ou les deux?)
*/


La __notation positionnelle__ confère une propriété intéressante à toute les bases numérique : dans tout nombre, la valeur d'un chiffre non nul de poids $i$ est strictement supérieur à la somme des valeurs des chiffres de poids inférieur à $i$.
Prenons comme exemple le nombre $11.75$ écrit en base décimale :

$$11.75 \ = \left(1 \times 10_{10}^{\quad 1} + 1 \times 10_{10}^{\quad 0} + 7 \times 10_{10}^{\quad -1} + 5 \times 10_{10}^{\quad -2}\right)$$

Remarquons que le chiffre $1$ des dizaines est non nul, alors nous savons que la valeur de ce chiffre $\left(1 \times 10_{10}^{\quad 1}\right)$ est strictement supérieur à la somme des valeurs des chiffres inférieurs au rang des dizaines ; et ce, indépendemment de la valeur des chiffres sous-jacent.

$$\left(1 \times 10_{10}^{\quad 1}\right) \gt \left(1 \times 10_{10}^{\quad 0} + 7 \times 10_{10}^{\quad -1} + 5 \times 10_{10}^{\quad -2}\right)$$

Bien évidemment, cela fonctionne pour la base décimale (entre autre) mais aussi pour la base binaire, à la seule différence que nous ne parlons dès lors plus de chiffres décimaux mais de bits.
De facto, vu qu'un bit non nul est à $1$, il est préférable de dire que : dans tout nombre binaire, la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$. 
Voici une illustration du codage en __virgule flottante__ du nombre $+ 11.75$ :

$$+ \ 11.75 \ = \ + 1011.11_2 \ = \left(1 \times 2^3 + 0 \times 2^2 + 1 \times 2^1 + 1 \times 2^0 + 1 \times 2^{-1} + 1 \times 2^{-2}\right)$$

Avec ce qui vient d'être dit, nous en déduisons par exemple que la valeur $\left(1 \times 2^1\right)$ du bit à $1$ de poids $1$, est strictement supérieur à la somme des valeurs des bits de poids $0$, $-1$ et $-2$ ; nonobstant la valeur sous-jacente de ces bits.

$$\left(1 \times 2^1\right) \gt \left(1 \times 2^0 + 1 \times 2^{-1} + 1 \times 2^{-2}\right)$$

En outre, rappelez-vous des chapitres précédent sur l'encodage __Binary Unsigned__ ainsi que sur l'encodage des nombres à __virgule flottante__, ces chapitres expliquent (entre autre) que ces encodages s'appuient pleinement sur la __notation positionnelle__.
Naturellement, cela confère à ces encodages toutes les propriétés de la __notation positionnelle__, dont celle évoquée ci-dessus.
Cela insinue que les encodages à __virgule flottante__ ainsi que __Binary Unsigned__ ont cette même propriété en commun, et pas des moindres... car cette dernière va jusqu'à affecter le traitement des données par le FPS Configuration Unit. 

En bref, pour un champ $F$ qui code un nombre en __Binary Unsigned__ ou en __virgule flottante__, la valeur de tout bit à $1$ de poids $\left(i \gt lsb\left(F\right)\right)$ est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$ :

$$\left(1 \times 2^i\right) \gt \sum_{\sigma = i - 1}^{lsb\left(F\right)} \left(F_{\sigma} \times 2^{\sigma}\right)$$

Je précise que la syntaxe $F_{\sigma}$ permet d'indexer le bit de poids $\sigma$ du champ $F$, de plus, ajoutez à cela le fait que la fonction $lsb\left(\right)$ retourne simplement le poids le plus faible du nombre $F$ passé en argument.


/// 

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
