Bonjour.

Ce document est une preuve mathématique d'un circuit électronique du nom de FPU Configuration Unit.
J'ai conceptualisé ce circuit dans le but de résoudre un problème concernant le traitement des nombres flottants dans les unités de calculs flottantes (FPUs) d'un ordinateur.
La micro-architecture du circuit électronique ainsi que les compromis de conceptualisation, le problème que cherche à résoudre le FPU Configuration Unit ou encore la solution qu'il apporte, sont autant de sujets dont je parle dans une documentation dédié au circuit.

Pour les curieux, si vous souhaitez voir à quoi ressemble le circuit, sachez qu'il y a un fichier du nom de _16-bit_FPU_configuration_unit_ qui le schématise.
Le circuit a été directement schématisé en transistor de technologie CMOS, cela permet de mettre en évidence certains compromis sur le choix des portes logiques.
Disons que l'apparence abrupte du schéma rend hommage à la (relative) complexité algorithmique du FPU Configuration Unit, cette complexité est plus tangible qu'avec des lignes de code.

Le document va même au delà de la démonstration mathématique du circuit, il explique les sujets fondamentaux sur lequels la démonstration ainsi que le circuit reposent.
Ce qui explique la préscence de la table des matières suivante:

...

# Les encodages

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$.
Pour la culture, je précise que le terme _bit_ vient de la contraction de _Binary Digit_.
Un champs binaire est quant à lui composé d'un ensemble de bit, c'est à dire d'un ensemble de $0$ et de $1$.

Ce sont ces champs binaire qui nous permettent de représenté des nombres dans un ordinateur, faire des calculs, mémorisé des résultats.
Il y a cependant plein de façon de représenté des nombres dans un champs binaire, nous parlons d'_encodage_ des nombres.
Tout les encodages ne se valent pas car il y en a des plus efficasse que d'autre en fonction des besoins, prenons un exemple.
Pour un champs de $16$ bits nous pourrions calculé la somme des bits pour représenter un nombre.
Avec cet encodage nous ne pourrions codé que des valeurs entre $0$ et $16$ inclus, ce qui n'est pas optimal pour bien des situations.
Mais parmis les multiples encodage existant, le plus connu de tous porte le nom de __Binary Unsigned__.

## L'encodage Binary Unsigned

Le _Binary Unsigned_ est un encodage permettant de codé des nombres entier naturel.
Techniquement, cet encodage se base sur les même primitives mathématiques que la base décimale pour représenter des nombres.
En décimale, chaque chiffre qui compose la valeur $103$ est le facteur d'une puissance de $10_{10}$, puis nous faisons la somme des produits pour représenter le nombre sous-jacent.
Veuillez noté que l'indice $X_{10}$ représente la base numérique dans laquelle le nombre $X$ est écrit, $10$ pour la base décimale et $2$ pour la base binaire.

$$103 = \left(1 \times 10^2 + 0 \times 10^1 + 3 \times 10^0\right)$$

Le chiffre des unités $3$ est facteur de $10$ à la puissance $0$, le chiffre des dizaines $0$ est facteur de $10$ à la puissance $1$, tandis que le chiffre des centaines $1$ est le facteur de $10$ à la puissance $2$.
Remarquons que la valeur des puissances commence à partir de $0$ pour le chiffre des unités, et croix en fonction de la position du chiffre.
Mais ceci n'est pas propre à la base décimale.

La base binaire (ou base $2$) permet également de représenté des nombres d'une façon équivalente, mais comme mentionné plus haut, qui dit représentation de nombre dit encodage de nombre.
Vous l'aurez compris, l'encodage qui permet de représenté des nombres de la sorte est le ___Binary Unsigned___.
Chaque bit d'un nombre écrit en _Binary Unsigned_ est le facteur d'une puissance de $2$ positive ou nul.
Le nombre ainsi représenté se calcul par la somme de chacun de ces produits.
De plus, la représentation des nombres n'est pas " _signé_ ", ce que veut dire qu'il n'y a pas de bit permettant le codage explicite d'un signe $\pm$, et ainsi les nombres codés sont naturellement positifs.

$$103 = 1100111_2 = 1 \times 2^6 + 1 \times 2^5 + 0 \times 2^4 + 0 \times 2^3 + 1 \times 2^2 + 1 \times 2^1 + 1 \times 2^0$$

Nous ne parlons pas de bit des unités, des dizaines, centaines ou encore milliers dans un champs binaire, mais il y a ce que l'on appelle le " _poids_ " d'un bit qui est à prendre en compte.
Plus un bit est le facteur d'une grande puissance de $2$, plus le poids de ce bit est important dans le calcul de la valeur d'un nombre.
Par exemple, le bit qui multiplie $2^6$ fait fluctuer la valeur du nombre représenté de $64$, contre une fluctuation de seulement $8$ pour le bit facteur de $2^3$.
Le bit qui multiplie $2^6$ est donc d'un poids (d'une importance) supérieur à celui qui multiplie $2^3$.

Par conséquent, il est très commun d'indicé les bits d'un champs binaire par leurs poids, le bit de poids $3$ fait référence au bit qui multiplie la puissance $2^3$.
Il s'avère que certains termes sont rentrés dans le langage commun et représentent des bits d'un poids bien précis à l'intérieur d'un champs.
Par exemple, le terme de _LSB_ pour _Least Significant Bit_ fait référence au bit de poids le plus faible d'un champs binaire, le bit de poids $0$ pour tout nombre représenté en _Binary Unsigned_.
Il existe aussi le terme de _MSB_ pour _Most Significant Bit_ qui fait référence au bit de poids le plus fort d'un champs binaire.
D'autres termes comme _LSB1_ pour le bit à $1$ de poids le plus faible, ou encore _MSB1_ pour le bit à $1$ de poids le plus fort, existent aussi.

Pour finir, comme cela a été dit plus haut, la valeur d'un nombre _Binary Unsigned_ se calcule par la somme des produits entre chaque bit et sa puissance de $2$ correspondante. Ce qui donne lieu à des calculs inutiles car la valeur d'un bit étant de $0$ ou $1$, chaque produit génère alors un résultat qui est nul ou égale à la puissance de $2$ du bit en question.
C'est pourquoi vous entenderez souvent dire (y compris dans ce document), que la valeur d'un nombre _Binary Unsigned_ se calcul comme la somme des puissances de $2$ dont le bit est à $1$.

### Une propriété du Binary Unsigned qui est fondamentale à la démonstration

Avec un champs binaire d'encodage _Binary Unsigned_, nous pouvons être certains qu'un bit à $1$ de poids $i$ ait une valeur strictement supérieur à la somme des valeurs de chaque bit de poids inférieur à $i$.
Prenons comme exemple le codage _Binary Unsigned_ du nombre $103$, c'est à dire $1100111_2$, que nous nommerons désormais $B$.
Si nous prêtons attention au bit de poids $5$ du champs $B$, nous remarquons que ce dernier a une valeur $\left(1 \times 2^5\right)$ qui est strictement supérieur à la somme des valeurs des bits de poids inférieur à $5$.
De manière plus général, pour tout bit de poids $i$ à $1$:

$$\left(1 \times 2^i\right) \gt \sum_{ii = i - 1}^0 \ \left(B_{ii} \times 2^{ii}\right)$$

Ce qui vient d'être dit ci-dessus est inconditionnel, qu'importe la valeur des bits de poids inférieurs à $i$.
Par ailleurs, ceci n'est pas propre à la base binaire.

## Les nombres à virgule flottante

Il existe en informatique de multiple façon de représenter des nombres à virgule, les nombres à virgule fixe, les nombres à virgule flottante ou encore le standard IEEE-754, sont à ma connaissance les principaux encodages.
L'usage des nombres à virgule fixe se fait très rare de nos jours en 2025.
Malgré tout, les systèmes informatique bancaire utilisent souvent ces derniers pour représenter le solde des déposants et autres.
Les nombres à virgule flottante sont quant à eux une évolution des nombres à virgule fixe, ces derniers permettent principalement aux humains de pouvoir écrire facilement des nombres à virgule en base binaire.
De plus, ils sont utilisés indirectement par la notation scientifique binaire des nombres, dont nous parlerons plus tard.
Pour finir, le standard IEEE-754 permet de normalisé la représentation des nombres à virgule dans un ordinateur, c'est un sujet absolument fondamentale à ce document que nous aborderons plus tard également.

Pour le moment, intéressons nous à l'encodage des nombres à virgule flottante.
Je n'apprend rien à personne en disant qu'un nombre à virgule est scindé en deux parties, la partie entière composée des chiffres derrère la virgule, et la partie fractionnaire composée des chiffres devant la virgule.
La partie entière d'un nombre à virgule flottante est codé en _Binary Unsigned_.
Par conséquent, pour pouvoir représenter des nombres à virgule flottante positif ou négatif, il faut rajouté un signe $\pm$ à la partie entière du nombre.
C'est pourquoi le bit de poids le plus fort de la partie entière d'un nombre flottant, code le signe $-$ lorsqu'il est à $1$, et $+$ quand il vaut $0$.
Cependant, l'encodage de la partie fractionnaire est tout nouveau, malgré une ressemblance avec le _Binary Unsigned_.
Chaque bit de la partie fractionnaire est le facteur d'une puissance de $2$ négative, la partie fractionnaire a comme valeur la somme de ces produits.
Il sera plus simple de comprendre tout cela au travers d'une illustration, regardez ci-bas.

$$+ \ 103.3125 = 0 \ 1100111.0101_2$$

Dans la partie entière du nombre nous retrouvons le bit de poids le plus fort qui est à $0$ (ce dernier représente le signe $+$), ainsi que le codage _Binary Unsigned_ du nombre $103$, qui commence à nous être familier.
Rien d'étonnant pour la partie fractionnaire, nous avons des bits... mais regardons dans le détails ce que valent ces bits.

$$ 0.0101_2 = Integer \ Part\left(0 \times 2^0\right) + Fractional \ Part\left(0 \times 2^{-1} + 1 \times 2^{-2} + 0 \times 2^{-3} + 1 \times 2^{-4}\right)$$

Nous remarquons que le bit de poids le plus fort de la partie fractionnaire est celui de poids $-1$, et que la valeur des puissances de $2$ négatives vont décroissant en fonction de la position du bit correspondant dans le nombre.

Dans la section précédente nous parlions d'une propriété propre à l'encodage _Binary Unsigned_.
Cette propriété peut être mise en oeuvre dans la partie entière d'un nombre à virgule flottante, car rappelons-le cette partie est codé en _Binary Unsigned_.
Malgré cela, l'encodage de la partie fractionnaire permet lui aussi de faire usage de cette propriété.
Pour rappel, cela veut dire que tout bit à $1$ de poids $i$ compris dans la partie fractionnaire, a une valeur strictement supérieur à la somme des valeurs de chaque bit de poids inférieur à $i$.
De fait, il y a une continuité de cette propriété entre la partie entière et fractionnaire d'un nombre à virgule flottante.
Autrement dit, un bit à $1$ de poids $i$ dans la partie entière, a une valeur strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$ de la partie entière et fractionnaire.

### L'étymologie du terme de nombre à virgule flottante

Par ailleurs, si ces nombres sont dits à virgule flottante, c'est parce que la virgule du nombre peut être déplacée sans que cela ne pose de problème.
C'est la solution qui à ce jour, pour l'Homme, est la plus intuitive ainsi que la plus flexible pour représenter des nombres à virgule en base $2$.
Il n'y a pas de contrainte spécifique, le nombre de bits avant et après la virgule peut varié.
Malgré des encodages identiques pour la partie entière et fractionnaire des nombres à virgule flottante et à virgule fixe, ces derniers gèrent la virgule d'une façon bien différente.
Les nombres à virgule fixe ont une certaine quantité de bits devant et derrière la virgule, il ne peut pas y avoir un bit de plus ou de moins, la virgule ne peut pas être déplacée.
Le fait que la partie entière et fractionnaire soient composée d'un nombre fixe de bit, fixe également la précision de codage d'un tel nombre.
Ceci est un frein à beaucoup de cas d'usage mais c'est aussi un avantage dans de rare cas, comme avec les systèmes informatiques des banques.
En effet, il est nécessaire que la précision d'affichage d'un solde soit de l'ordre du centième de centime près admettons, mais il n'y a pas besoin de plus.
La praticité derrière les nombres à virgule fixe c'est que la précision reste constante, elle ne varie pas en fonction du solde comme ce pourrait être le cas avec des nombres à virgule flottante.

# L'écriture scientifique binaire

Pour commencer, qu'est ce que l'_écriture scientifique_?

L'écriture scientifique est une façon comme une autre de représenter des nombres.
Elle existe pour chaque base numérique, comme pour la base binaire par exemple.
Le but de cette notation scientifique des nombres est double.
Permièrement, elle ne permet d'écrire les nombres que d'une seule et unique façon.
En outre, elle simplifie aussi la lecture des très grands ainsi que des très petits nombres.
Par exemple, l'écriture scientifique _en base décimale_ rend facilement lisible la lecture de la vitesse approchée de la lumière dans de le vide spatial (en m/s) $+ \ 3.0 \times 10^8$.

Il faut savoir que d'une base numérique à une autre, le fonctionnement de l'écriture scientifique change peu.
Quant à la composition de la notation scientifique, elle ne change jamais qu'importe la base numérique, il y a toujours:
  - Un ___signe___
  - Un ___significande___
  - Un ___multiplicande___

Dans l'exemple fournit ci-dessus, le _signe_ est le symbole $\pm$ qui indique si le nombre est positif ou négatif, le _significande_ (aussi appeller _mantisse_) est le terme $3.0$ et enfin le _multiplicande_ est le terme $10^8$.
En notation scientifique décimale, le significande a une valeur comprise dans l'intervalle $\left[1;10\right[$, et le multiplicande est une puissance de $10$.
De manière plus général, pour une notation scientifique en base $N$, la valeur du significande ne peut être comprise qu'entre $\left[1;N\right[$ et le multiplicande est alors une puissance de $N$.
D'où le fait que d'une base numérique à une autre le fonctionnement de l'écriture scientifique change légèrement.

Les chapitres suivants se focalisent sur la notation scientifique en base binaire.

## Le significande

L'écriture scientifique en base binaire peut représenté n'importe quel nombre $F$ codé en virgule flottante, qu'il soit positif ou bien négatif.
Le nombre flottant $F$ à représenté en notation scientifique binaire permet à lui seul de définir deux des trois éléments de sa propre écriture scientifique, le signe et le significande.
Comme son nom l'indique, l'élément qu'est le "signe" ne représente que le signe $\pm$ du nombre flottant $F$.
Tandis que le significande s'obtient depuis une modification (non systèmatique) de la valeur absolu du nombre à virgule flottante $F$.

Nous savons qu'en notation scientifique binaire, le significande ne peut interprété que des valeurs comprises dans l'intervalle $\left[1;2\right[$.
Lorsque la valeur du nombre à virgule flottante $\vert \ F \ \vert$ est comprise dans cette intervalle, alors la valeur de $\vert \ F \ \vert$ forme le significande du nombre $F$.
Cependant, si ce n'est pas le cas, il faut modifier la valeur du nombre $\vert \ F \ \vert$ de sorte à ce que le résultat soit compris dans l'intervalle $\left[1;2\right[$.
Néanmoins, ces modifications ne s'effectuent pas n'importe comment.

Une modification consiste en un déplacement de la virgule du nombre flottant $\vert \ F \ \vert$.
Dans le cas où $\left(\vert F \vert \ge \ 2\right)$, alors la virgule de $\vert \ F \ \vert$ subit un décalage vers la gauche jusqu'à que cette dernière se retrouve devant le _MSB1_.
Mais ce n'est tout car dans le cas où $\left(\vert F \vert \lt \ 1\right)$, alors la virgule de $\vert \ F \ \vert$ subit aussi un décalage vers la droite jusqu'à qu'elle se retrouve devant le _MSB1_.
En déplaçant la virgule de $\vert \ F \ \vert$ jusque devant le bit de poids le plus fort à $1$, nous nous assurons du fait que le résultat de la modification du nombre $\vert \ F \ \vert$ soit compris dans l'intervalle de valeur $\left[1;2\right[$.

Je me permet de faire une apparté.
Rappelons que le nombre à virgule flottante $F$ peut être positif comme négatif, tandis que le significande ne peut interprété que des valeurs positives de l'intervalle $\left[1;2\right[$.
Pour tout $\left(F \lt 0\right)$, le nombre ne pourrait quoi qu'il arrive jamais être dans l'intervalle de valeur licite d'un significande, car un déplacement de sa virgule ne changerait rien au fait que le nombre soit négatif. 
C'est pourquoi la formation d'un significande s'appuie sur $\vert \ F \ \vert$, et non directement sur le nombre signé $F$.
Malgré cela, n'oubliez pas que le signe $\pm$ du nombre flottant $F$ est pris en compte par l'écriture scientifique binaire du nombre.

Il subsiste cependant une exception, la représentation du nombre $0$ en notation scientifique binaire.
La valeur de ce nombre n'étant pas comprise dans l'intervalle de valeur licite d'un significande $\left[1;2\right[$, alors il faudrait apporté des modifications au nombre.
Cela demanderait de déplacer la virgule du nombre jusque devant son _MSB1_, mais il n'y a pas de _MSB1_ dans le codage de $0$ en virgule flottante.
__Par conséquent, le significande est alors excpetionnellement nul pour pouvoir représenté la valeur__ $0$ __en notation scientifique binaire.__
Par ailleurs, ceci est commun à chaque écriture scientifique, pour n'importe quel base numérique.

La section suivante cherche à précisé l'impacte qu'a un déplacement de la virgule d'un nombre flottant, sur sa valeur. 

### La transformation d'un nombre à virgule flottante en un significande

Pour tout nombre à virgule flottante $F$, nous savons que la virgule se trouve entre le _LSB_ de la partie entière (le bit de poids $0$), et le _MSB_ de la partie fractionnaire (bit de poids $-1$).
Par conséquent, un déplacement de la virgule d'un rang vers la gauche force le bit de poids $1$ à devenir le bit de poids $0$, le bit de poids $0$ à devenir celui de poids $-1$, et celui de poids $-1$ devient le bit de poids $-2$, etc.
Pour le dire autrement, chaque bit de la partie entière comme de la partie fractionnaire du nombre $F$, voit son poids être _décrémenté_ de $1$. 

L'équation ci-dessous permet de calculer la transformation de n'importe quel nombre à virgule flottante $\vert \ F \vert \notin \left[1;2\right[$, en un significande.

$$\sum_{i = msb\left(F\right) - 1}^{lsb\left(F\right)} \left(F_i \ \times \ 2^{\left(i + c\right)}\right) = \vert F \vert \times \ 2^c$$

Les fonctions $msb$ et $lsb$ retournent respectivement le poids de numéro le plus fort ainsi que le plus faible du nombre passé en argument.
Quant au terme $c$, il représente le nombre de rang de décalage à induire sur la virgule du nombre $\vert \ F \ \vert$.
Qui plus est, lorsque $\left(c \lt 0\right)$ c'est que la virgule doit être décalée vers la gauche.
Inversement quand $\left(c \gt 0\right)$, c'est que la virgule subit un décalage vers la droite.

Essayons de comprendre pourquoi est ce qu'un décalage d'un rang vers la gauche de la virgule de $\vert \ F \ \vert$ $\left(c = -1\right)$, divise la valeur du nombre par $2$ comme le démontre l'équation ci-dessus.
N'oublions pas qu'avec un décalage d'un rang vers la gauche de la virgule de $F$, je cite "_chaque bit de la partie entière comme de la partie fractionnaire du nombre_ $F$, _voit son poids être décrémenté de_ $1$".
Avant décalage de la virgule du nombre $\vert \ F \ \vert$ le bit de poids $i$ est facteur de la puissance $2^i$, mais après décalage, ce dernier n'est plus que facteur de la puissance $2^{\left(i - 1\right)}$, la valeur du bit a été divisé par $2$.
Ceci se généralise à l'ensemble des bits du nombre.
Rappelons que nous calculons la valeur de $\vert \ F \ \vert$ comme la somme des puissances dont le bit est à $1$, chaque puissance ayant été divisé par $2$ dû au décalage de la virgule, la somme de ces dernières l'est également.
C'est pourquoi la valeur du nombre $\vert \ F \ \vert$ est divisé par $2$ après un décalage d'un rang vers la gauche de la virgule.

Maintenant que nous avons compris pourquoi un décalage d'un rang vers la gauche de la virgule de $\vert \ F \ \vert$, engendre une division par $2$ du nombre.
Découvrons désormais les effets d'un décalage d'un rang vers la droite de la virgule de $\vert \ F \ \vert$, sur la valeur du nombre lui même.
Dans le cas d'un tel décalage, chaque bit de la partie entière comme de la partie fractionnaire du nombre $F$, voit son poids être _incrémenté_ de $1$.
Après décalage de la virgule du nombre flottant $\vert \ F \ \vert$, le bit de poids $i$ passe de facteur de $2^i$ à $2^{\left(i + 1\right)}$, et cela se généralise à l'ensemble des bits du nombre.
Comme la valeur de $\vert \ F \ \vert$ se calcul selon la somme des puissances dont le bit est à $1$, chaque puissance ayant été multiplié par $2$ dû au décalage de la virgule, la somme de ces dernières l'a également été.
C'est ce que démontre le membre de gauche de l'équation pour $\left(c = 1\right)$, alors que le membre droit se contente de donné le résultat attendu.

Désormais, nous sommes en capacité de comprendre toutes les implications qu'un décalage de la virgule du nombre $\vert \ F \ \vert$, peut avoir sur la valeur du nombre lui même. 
Prenons comme exemple un décalage quelconque de la virgule vers la droite $\left(c \gt 0\right)$.
Nous venons tout juste de voir qu'un décalage d'un rang vers la droite de la virgule de $\vert \ F \ \vert$ $\left(c = 1\right)$, multiplie par $2$ la valeur du nombre.
Par conséquent, pour tout autre décalage de la virgule de $c$ rangs vers la droite, il suffit alors de multiplié $c$ fois par $2$ la valeur de $\vert \ F \ \vert$.
Ce qui est équivalent au membre droit de l'équation $\left(\vert F \vert \times \ 2^c\right)$.
Voyons maintenant le cas d'un décalage quelconque de la virgule vers la gauche $\left(c \lt 0\right)$.
Nous savons qu'un décalage d'un rang vers la gauche de la virgule de $\vert \ F \ \vert$ $\left(c = -1\right)$, divise par $2$ la valeur du nombre.
Donc dans le cas de tout autre décalage de la virgule vers la gauche, il suffit alors de divisé $\vert \ c \ \vert$ fois la valeur du nombre $\vert \ F \ \vert$ par $2$.
Cela est une nouvelle fois équivalent au membre droit de l'équation $\left(\vert F \vert \times \ 2^c\right)$, nous verrons pourquoi dans le chapitre suivant.

__Nous savons désormais que n'importe quel déplacement de la virgule d'un nombre flottant, engendre une multiplication ou une division de la valeur du nombre par une puissance de__ $2$.

### Détail sur le fonctionnement de l'équation

Si vous avez déjà compris le fonctionnement de l'équation, vous pouvez vous rendre au chapitre suivant.

Dans la section précédente, je dit que la valeur du nombre $\vert \ F \ \vert$ est divisé par une puissance de $2$ lorsque sa virgule subit un décalage vers la gauche.
L'équation sur laquelle ces calculs s'effectuent ne fait pourtant pas usage de division, essayons de comprendre pourquoi.

L'équation du chapitre précédent semble être parfaitement appropriée pour les décalages vers la droite de la virgule d'un nombre flottant.
En effet, je vous rappelle qu'un déplacement de $c$ rangs vers la droite de la virgule du nombre $\vert \ F \ \vert$, multiplie la valeur du nombre par $2^c$, avec $\left(c \gt 0\right)$. 
Ce qui colle avec le membre droit de l'équation $\left(\vert F \vert \times \ 2^c\right)$.

En parallèle, cette même équation utilise des valeurs négatives de $c$ lors des décalages vers la gauche de la virgule d'un nombre flottant.
Pourtant, le plus simple aurait été de définir les même valeurs de $c$ peu importe la direction des décalages, accompagné de deux équations différentes.
Une équation pour les décalages de la virgule vers la droite (celle déjà défini), ainsi qu'une autre équation plus approprié pour les décalages de la virgule vers la gauche.
Cette seconde équation ressemblerait à ce qui suit. 

$$\sum_{i = msb\left(F\right) - 1}^{lsb\left(F\right)} \left(F_i \ \times \ 2^{\left(i - c\right)}\right) = \vert F \vert \div \ 2^c$$

Avec cette équation $\left(c \gt 0\right)$, le terme représente toujours le nombre de rang de décalage à induire sur la virgule du nombre $\vert \ F \ \vert$.
Remarquez qu'un décalage d'un rang vers la gauche de la virgule de $\vert \ F \ \vert$ $\left(c = 1\right)$, divise la valeur du nombre par $2$.
De façon plus général, pour tout décalage de la virgule de $c$ rangs vers la gauche, la valeur de $\vert \ F \ \vert$ est divisé par $2^c$.
Ce qui colle parfaitement avec le membre droit de la seconde équation $\left(\vert F \vert \div \ 2^c\right)$, libre à vous de vérifier que le membre de gauche de cette même équation fonctionne bien.

Ce qui suit explique comment il est possible de passé d'un tel raisonnement à deux équations, à un raisonnement plus singulier d'une seule.

Nous allons essayés de réduire la forme de la seconde équation que nous venons tout juste de définir, en la forme de la première équation ayant été défini dans le chapitre précédent.
Remarquons que le membre droit de cette seconde équation $\left(\vert F \vert \div \ 2^c\right)$ pourrait être remplacé par $\left(\vert F \vert \times \left(1 \div 2^c\right)\right)$ et que $\left(1 \div 2^c\right) = 2^{-c}$.
Par définition, il est donc possible de remplacé $\left(\vert F \vert \times \ 2^{-c}\right)$ avec $\left(c \gt 0\right)$, par le membre de droite de la première équation $\left(\vert F \vert \times \ 2^c\right)$.
Pour cela, il suffit de faire en sorte que tout déplacement vers la gauche de la virgule du nombre $\vert \ F \ \vert$, soit interprété par une valeur négative de $c$, et non positive.
Par ailleurs, le membre de gauche de la première équation procède au calcul d'exposant $\left(i + c\right)$, avec $\left(c \lt 0\right)$.
Ce qui revient au même que le calcul d'exposant du membre de gauche de la seconde équation $\left(i - c\right)$, lorsque $\left(c \gt 0\right)$.

En bref, le fait que la variable $c$ puisse être positive ainsi que négative permet de ne faire usage que d'une seule équation, qu'importe la direction du décalage de la virgule qui est à faire.

## Le multiplicande

Lorsque le nombre à virgule flottante $F$ à représenter en notation scientifique binaire, a une valeur absolu qui n'est pas comprise dans l'intervalle de valeur licite d'un significande $\left[1;2\right[$.
La valeur de son propre significande $S$, qui elle est comprise dans cette intervalle, est différente de $\vert \ F \ \vert$.
C'est pourquoi l'écriture scientifique fait usage d'un multiplicande.
Lorsque $\left(S \ \neq \vert F \vert\right)$, le multiplicande $M$ permet comme son nom l'indique de multiplié le significande $S$, de sorte à ce que $\vert \ F \vert = \left(S \times M\right)$.
Autrement dit, le multiplicande réajuste la valeur du significande à la valeur absolu du nombre $F$ à représenter, quand c'est nécessaire.

Cependant, il est possible que le nombre à virgule flottante $F$ ait une valeur absolu comprise dans l'intervalle de valeur licite d'un significande.
Parce que $\left(S \ = \vert F \vert\right)$, nous en déduisons que dans cette situation le multiplicande $M$ vaut $1$, de sorte à ce qu'une nouvelle fois $\vert \ F \vert = \left(S \times M\right)$.

Je tient à vous faire remarqué que si $\vert \ F \vert = \left(S \times M\right)$, alors la représentation du nombre flottant $F$ en écriture scientifique binaire est $F \ = \pm \left(S \times M\right)$.
Le signe $\pm$ est celui du nombre flottant $F$, ce dont nous parlons au début du chapitre "_Le significande_".
En bref, nous retrouvons les trois éléments de l'écriture scientifique, le _signe_ $\pm$, le _significande_ $S$ ainsi que le _multiplicande_ $M$.

Les sections suivantes expliquent comment le multiplicande réajuste la valeur du significande à celle du nombre $F$ en valeur absolu.
En outre, sachez que ces dernières font références à l'équation défini dans le chapitre "_La transformation d'un nombre à virgule flottante en un significande_".

### Lorsque le multiplicande multiplie la valeur du significande

En écriture scientifique binaire, la représentation d'un nombre à virgule flottante $F$ dont la valeur absolu est supérieur à l'intervalle $\left[1;2\right[$ d'un significande, s'accompagne d'une transformation du nombre $\vert \ F \ \vert$ en un significande $S$.
La formation du significande passe par un décalage vers la gauche de la virgule du nombre $\vert \ F \ \vert$, ce qui divise par une puissance de $2$ la valeur du nombre.
L'équation nous dit que la formation du significande $S$ passe par $\left(\vert F \vert \times \ 2^c\right)$ avec $\left(c \lt 0\right)$, ce qui est équivalent au fait de dire que $S \ = \left(\vert F \vert \div \ 2^{-c}\right)$.
Le significande, par le fait que $\left(S \ \lt \vert F \vert\right)$, est un terme de l'écriture scientifique qui à lui seul ne peut pas représenté la valeur du nombre flottant $\vert \ F \ \vert$.
C'est pourquoi un multiplicande $M$ multiplie le significande $S$ de sorte à ce que $\vert \ F \vert = \left(S \times M\right)$.
Parce que $S \ = \left(\vert F \vert \div \ 2^{-c}\right)$, le multiplicande $M$ multiplie donc le significande $S$ par la même puissance de $2$ que celle ayant divisé $\vert \ F \ \vert$ pour formé le significande.
Ce qui donne $\vert \ F \vert = \left(S \ \times \ 2^{-c}\right)$.
Par conséquent, la représentation du nombre flottant $F$ en écriture scientifique binaire est $F \ = \pm \left(S \ \times \ 2^{-c}\right)$, comme nous l'avons vu plus haut.

Voici comme exemple l'écriture scientifique binaire du nombre $F = 5.5$, qui en virgule flottante vaut $+ \ 101.1_2$ :

$$+ \ 1.011_2 \ \times \ 2^2$$

Le significande $1.011_2$ (ou $1.375$ en décimale) résulte du décalage de $2$ rangs vers la gauche de la virgule du nombre flottant $\vert \ F \ \vert$.
La formation du significande engendre une division de la valeur du nombre $\vert \ F \ \vert$, ce que l'équation traduit par $\left(\vert F \vert \times \ 2^c\right)$ avec $\left(c = -2\right)$, ou encore $\left(\vert F \vert \div \ 2^{-c}\right)$.
Sachant que $\left(1.375 \ \lt \vert 5.5 \vert\right)$, le multiplicande $M$ doit multiplié le significande $1.375$ par $2^{-c}$ de sorte à ce que $\vert \ 5.5 \vert = \left(1.375 \ \times \ M\right)$.
L'écriture scientifique du nombre à virgule flottante $F$ est donc bel et bien $5.5 \ = + \ 1.375 \ \times \ 2^2$.

### Lorsque le multiplicande divise la valeur du significande

Voyons désormais le cas contraire, celui dans lequel le multiplicande réduit la valeur du significande.

En écriture scientifique binaire, la représentation d'un nombre à virgule flottante $F$ dont la valeur absolu est inférieur à l'intervalle $\left[1;2\right[$ d'un significande, s'accompagne d'une transformation du nombre $\vert \ F \ \vert$ en un significande $S$.
La formation du significande passe par un décalage vers la droite de la virgule du nombre $\vert \ F \ \vert$, ce qui multiplie par une puissance de $2$ la valeur du nombre.
L'équation nous dit que la formation du significande $S$ passe par $\left(\vert F \vert \times \ 2^c\right)$, avec $\left(c \gt 0\right)$.
Le significande, par le fait que $\left(S \ \gt \vert F \vert\right)$, est un terme de l'écriture scientifique qui à lui seul ne peut pas représenté la valeur du nombre flottant $\vert \ F \ \vert$.
C'est pourquoi un multiplicande $M$ divise le significande $S$ de sorte à ce que $\vert \ F \vert = \left(S \ \div \ M\right)$.
Parce que $S \ = \left(\vert F \vert \times \ 2^c\right)$, le multiplicande $M$ divise donc le significande $S$ par la même puissance de $2$ que celle ayant multiplié $\vert \ F \ \vert$ pour formé le significande. 
Ce qui donne $\vert \ F \vert = \left(S \ \div \ 2^c\right)$, et nous retombons ainsi sur $\vert \ F \vert = \left(S \ \times \ 2^{-c}\right)$.
Par conséquent, la représentation du nombre flottant $F$ en écriture scientifique binaire tombe une nouvelle fois sur $F \ = \pm \left(S \ \times \ 2^{-c}\right)$.

Voici comme exemple l'écriture scientifique binaire du nombre $F = -0.140625$, qui en virgule flottante vaut $- \ 0.001001_2$ :

$$- \ 1.001_2 \ \times \ 2^{-3}$$

Le significande $1.001_2$ (ou $1.125$ en décimale) résulte du décalage de $3$ rangs vers la droite de la virgule du nombre flottant $\vert \ F \ \vert$.
La formation du significande engendre une multiplication de la valeur du nombre $\vert \ F \ \vert$, ce que l'équation traduit par $\left(\vert F \vert \times \ 2^c\right)$ avec $\left(c = 3\right)$.
Sachant que $\left(1.125 \ \gt \vert - \ 0.140625 \vert\right)$, le multiplicande $M$ doit divisé le significande $1.125$ par $2^c$ de sorte à ce que $\vert - \ 0.140625 \vert = \left(1.125 \div M\right)$.
Cela donne $\vert - 0.140625 \vert = \left(1.125 \ \div \ 2^3\right)$, ce qui est équivalent à $\vert - \ 0.140625 \vert = \left(1.125 \ \times \ 2^{-3}\right)$.
L'écriture scientifique du nombre à virgule flottante $F$ est donc bel et bien $- \ 0.140625 \ = - \ 1.125 \ \times \ 2^{-3}$.

### Un autre regard sur le multiplicande

Je vous invite à voir les choses d'une manière différente, ceci nous sera utile pour plus tard.

Nous savons que la formation du significande d'un nombre flottant $F$ dont $\vert \ F \vert \notin \left[1;2\right[$, requiert de décalé de $c$ rangs la virgule du nombre $\vert \ F \ \vert$.
Selon l'équation, le significande $S$ ainsi obtenu vaut $\left(\vert F \vert \times \ 2^c\right)$.
Pour le dire autrement, obtenir le significande $S$ par l'intermédiaire du calcul $\left(\vert F \vert \times \ 2^c\right)$, revient au même que de décalé la virgule du nombre $\vert \ F \ \vert$ de $log_2\left(2^c\right)$ rangs.

Rappelons que le rôle du multiplicande est de réajusté la valeur du significande à la valeur absolu du nombre $F$.
Nous avons vu dans les deux précédentes section que le multiplicande réajuste toujours la valeur du significande en le multipliant par $2^{-c}$, ce qui donne $\vert \ F \vert = \left(S \times 2^{-c}\right)$.
Une nouvelle fois, retrouver la valeur de $\vert \ F \ \vert$ par l'intermédiaire du calcul $\left(S \times 2^{-c}\right)$, revient au même que de décalé la virgule du significande $S$ de $log_2\left(2^{-c}\right)$ rangs.

Cette section explique que pour remplir le rôle d'un multiplicande.
Il suffit de décalé la virgule du significande $S$ par le même nombre de rang que ne l'a été celle de $\vert \ F \ \vert$ pour former le significande, mais dans la direction opposé.
Rendons cela le plus clair possible avec un exemple.
Imaginons que la formation d'un significande $S$ soit obtenu par un décalage de ___trois rangs vers la gauche___ de la virgule du nombre $\vert \ F \ \vert$, donc $\left(c = -3\right)$.
Retrouver la valeur du nombre $\vert \ F \ \vert$ depuis le significande $S$, requiert un décalage de ___trois rangs vers la droite___ de la virgule du significande, ou autrement dit de décalé la virgule du significande $S$ de $-c$ rangs.
Bien entendu, cela fonctionne aussi quand $c$ est positif.

# Le standard IEEE-754

La norme IEEE-754 standardise la représentation (l'encodage) des nombres à virgule dans un ordinateur.
Ce document est une démonstration mathématique d'un circuit électronique qui a pour fonction principal de produire des comparaisons entre deux opérandes de standard IEEE-754.
Pour plus d'information sur le circuit électronique, jetez un oeil à la documentation dédié au circuit.
Il existe plusieurs format de nombre défini par le standard IEEE-754, chaque format ne change des autres que par sa taille.
Un format de nombre à virgule défini par la norme est composé des trois éléments suivant :
  - Le ___bit de signe___
  - Le champs d'___exposant___
  - Le champs de ___mantisse tronquée___

Les deux formats les plus connus sont le _Simple Precision_ ainsi que le _Double Precision_, qui permettent le codage de nombre à virgule sur des champs binaire de respectivement $32$ et $64$ bits.
Cependant, le circuit électronique que j'ai conceptualisé ne prend en charge que les opérandes de format _Half Precision_, ceci est principalement dû aux complexités de schématisation du circuit.
Le format Half Precision permet de codé des nombres à virgule sur des champs binaire de $16$ bits.
Voici la disposition de chacun des bits des trois éléments composant le format Half Precision :

$$\left[S_{15}, \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

$\quad S$: Sign bit, $\quad E$: Exponent, $\quad T$: Truncated mantissa

Chaque indice $i$ de la forme $X_i$ qui est compris dans l'intervalle $\left[0;15\right]$, représente le poids d'un bit précis dans le champs binaire d'un nombre codé au format Half Precision.
Il est en effet très commun d'indicé les bits d'un champs binaire par leurs poids.

Précédemment, nous avons longuement parlé de l'écriture scientifique binaire des nombres à virgule flottante.
La raison en est que le standard IEEE-754 s'appuie sur l'écriture scientifique binaire pour définir les différents formats de nombre à virgule, tel que le Half Precision.
Dans le champs binaire d'un nombre $F$ écrit au format Half Precision, le bit de signe (bit de poids $15$) est défini en fonction du signe $\pm$ de l'écriture scientifique binaire du nombre $F$.
Quant aux champs d'exposant et de mantisse tronquée, ils correspondent respectivement au multiplicande ainsi qu'au significande de l'écriture scientifique de $F$.
Malgré tout, dans les prochains chapitres dédiés aux champs d'exposant et de mantisse tronquée nous verrons que le cas de ces champs est légèrement plus complexe.

Etant donné que le circuit électronique ne prend en charge que des opérandes (nombres) au format Half Precision, le reste du document ne fera plus que référence à ce format.

## Le champs de mantisse tronquée

En introduction se trouve une illustration de la disposition des bits qui composent le format Half Precision, cette illustration montre que le champs de mantisse tronquée $T$ est composé de $10$ bits.
Voici les poids de chacun des bits qui compose le champs de mantisse tronquée :

$$\left[ \quad .. \quad , \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

Les deux sections suivantes concernent l'ensemble de ces bits formant le champs de mantisse tronquée.

### L'encodage du champs de mantisse tronquée

Le nombre $F$ codé au format Half Precision a un champs de mantisse tronquée qui correspond au significande de l'écriture scientifique binaire du même nombre $F$.
Par conséquent, l'encodage du champs de mantisse tronquée est simplement celui d'un nombre à virgule flottante.
Je vous invite donc à relire le chapitre "_les nombres à virgule flottante_", si nécessaire.

Néanmoins, le dernier paragraphe de ce chapitre donne une information assez importante. 
La valeur de n'importe quel bit à $1$ de poids $i$ dans un nombre à virgule flottante, est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Ce qui veut dire que n'importe quel bit à $1$ de poids $i$ dans le champs de mantisse tronquée, est lui aussi strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Autrement dit, le champs de mantisse tronquée partage les propriétés des nombres à virgule flottante, mais il n'est pas le seul.

Cette propriété provient de l'encodage _Binary Unsigned_, et il se trouve que l'encodage du champs d'exposant en est une déclinaison.
Nous verrons cela plus dans le détails à l'intérieur du chapitre dédié au champs d'exposant, mais il faut savoir que tout cela a un impacte.
La logique du circuit traite les champs de mantisse tronquée comme les champs d'exposant, avec le même sous ensemble de circuit, ce qui se reflète aussi dans la démonstration mathématique.
Nous en reparlerons d'ici là.

### Le codage du champs de mantisse tronquée

Précédemment, nous disions que le nombre $F$ codé au format Half Precision, avait un champs de mantisse tronquée qui correspondait au significande de l'écriture scientifique binaire de ce même nombre $F$.
Ce qui est faux, ou plutôt presque vrai.

En écriture scientifique binaire, le significande ne peut interprété que des valeurs comprises dans l'intervalle $\left[1;2\right[$.
Cela insinue que la partie entière d'un tel singificande est toujours composée d'un seul et unique bit à $1$.
Le standard IEEE-754 a donc fait le choix de ne pas codé la partie entière des significandes dans les champs de mantisse tronquée, cela permet un gain de précision de $1$ bit sur le codage des significandes.
Pour autant, la valeur du bit de la partie entière d'un significande est toujours prise en compte, ce dernier ne porte pas le nom de _bit implicite_ pour rien.
Cette optimisation est appliquée peu importe le format.

Je vous rappelle qu'en écriture scientifique, le significande s'appelle aussi une _mantisse_.
C'est dû au fait que le champs de mantisse tronquée ne code qu'une partie du significande (de la mantisse), qui a value un tel nom à ce champs.

## Le champs d'exposant

En introduction se trouve une illustration de la disposition des bits qui composent le format Half Precision, cette illustration montre que le champs d'exposant $E$ est composé de $5$ bits.
Voici les poids de chacun des bits qui compose le champs d'exposant :

$$\left[ \quad .. \quad , \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad .. \quad \right]$$

Les deux sections suivantes concernent l'ensemble de ces bits formant le champs d'exposant.

### L'encodage par biais du champs d'exposant

Le champs d'exposant utilise un _encodage par biais_ qui est une déclinaison de l'encodage _Binary Unsigned_.

L'encodage par biais s'appuie sur un champs binaire utilisant l'encodage Binary Unsigned, le champs en question se charge de codé un nombre entier naturel.
Il faut ensuite ajouter ou déduire une constante appellée biais (qui est aussi un nombre entier naturel), au nombre codé par le champs binaire sous-jacent.
Le résultat de ce calcul forme la valeur réel que représente le champs.
Nous comprenons donc qu'un encodage par biais est en quelque sorte un encodage Binary Unsigned déguisé.

Dans le cas d'un champs d'exposant $E$ d'une taille de $N$ bits, le champs code en Binary Unsigned un nombre entier naturel dont la valeur est comprise dans l'intervalle $\left[0; \ 2^N - 1\right]$.
La valeur que représente un champs d'exposant $E$ est issu du calcul $\left(E - B\right)$, pour un biais $B$ d'une valeur de $\left(2^{\left(N - 1\right)} - 1\right)$.
Un champs d'exposant peut donc représenté des valeurs positives comme négatives.
Par ailleurs, tout ce qui vient d'être dit est valable pour tout les formats de nombre à virgule ayant été établit par le standard IEEE-754.

Il s'avère que le circuit et sa démonstration mathématique sont menés à effectuer des comparaisons entre deux champs d'exposant $E_1$ et $E_2$, ayant un même biais $B$.
Ces comparaisons n'ont pas besoin de la valeur réel des champs d'exposant comme avec $\left(E_1 - B\right) \gt \left(E_2 - B\right)$, car le biais $B$ est constant.
Cette compraison se transforme alors en $\left(E_1 \gt E_2\right)$, seul la valeur des nombres codés dans les champs d'exposant compte.
En outre, n'oublions pas que l'_encodage par biais_ est une déclinaison du _Binary Unsigned_.
Cela veut dire que les deux encodages partagent les même propriétés, notamment celle dont il a été le sujet dans la section "_Une propriété du Binary Unsigned qui est fondamentale à la démonstration_".
En bref, cela veut dire que la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs d'exposant, est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Nous le verrons plus tard, mais c'est cette propriété qui permet à la démonstration mathématique d'effectuer entre autre des comparaisons entre des champs d'exposant.

### Le codage du champs d'exposant

Dans l'introduction, il est dit qu'un nombre $F$ codé au format Half Precision a un champs d'exposant qui correspond au multiplicande de l'écriture scientifique binaire de ce même nombre.
Il s'agit cependant d'une approximation.

Rappelons que le rôle d'un multiplicande $M$ est de réajusté la valeur du significande $S$ à la valeur absolu du nombre $F$ à représenté en écriture scientifique, ce qui correspond à $\vert \ F \vert = \left(S \times M\right)$.
Au travers du chapitre "_Le multiplicande_" et des deux sections qui le suivent, nous avons vus que le multiplicande réajustait toujours la valeur du significande en le multipliant par $2^{-c}$, ce qui donne $\vert \ F \vert = \left(S \times 2^{-c}\right)$.

Cependant, la section du nom de "_Un autre regard sur le multiplicande_", explique pourquoi la valeur du significande $S$ n'a pas besoin d'être multiplié par $2^{-c}$ pour être réajusté à la valeur de $\vert \ F \ \vert$.
Je cite "_Il suffit de décalé la virgule du significande_ $S$ _par le même nombre de rang que ne l'a été celle de_ $\vert \ F \ \vert$ _pour former le significande, mais dans la direction opposé_".
Le multiplicande interprète la puissance $2^{-c}$, où le terme $c$ quantifie le nombre de rang de décalage à induire sur la virgule du nombre $\vert \ F \ \vert$, ainsi que la direction du décalage.
C'est pourquoi le champs d'exposant ne code que l'exposant $-c$ du multiplicande, la négation de $c$ inverse simplement le sens du décalage.
D'où le fait que les propos tenus précédemment soient approximatifs, le champs d'exposant ne correspond qu'à une partie du multiplicande et non à l'entièreté de celui-ci.
Je vous redirige vers ladite section si vous souhaitez plus d'explication.

Il y a de nombreux avantages à ce que le champs d'exposant ne code que l'exposant et non la puissance de $2$ elle même.
Dans un premier temps, la valeur de l'exposant $-c$ croix de façon linéaire, tandis que la valeur de la puissance qu'interprète le multiplicande $\left(2^{-c}\right)$ croix exponentiellement.
Cela veut dire que le champs d'exposant devrait être composé de (beaucoup) plus de bits pour codé la puissance $2^{-c}$, que pour codé l'exposant $-c$ lui même.
En outre, il faudrait que les puissances de $2$ soient codés sous la forme de nombre à virgule dans le champs d'exposant, afin de pouvoir prendre en charge les puissances de $2$ négatives que peut interprété le multiplicande.
Enfin pour finir, le traitement des nombres par les unités de calcul arithmétique (FPUs) serait probablement plus complexe avec un champs d'exposant codant une puissance de $2$.
L'alignement des virgules de deux opérandes à soustraire serait plus complexe par exemple.

# Introduction au FPU Configuration Unit

Comme dit dans le chapitre "_Le standard IEEE-754_", le circuit électronique à l'étude dans ce document compare deux opérandes $\alpha$ et $\beta$ entre elle, deux opérandes de format Half Precision.
Je vous rappel que le circuit, du nom de FPU Configuration Unit, ne prend en charge que ce format pour des raisons de complexité de schématisation.

Ladite comparaison est une vérification de la supériorité stricte de la valeur absolu de l'un des deux opérandes envers la valeur absolu de l'autre.
Le FPU Configuration Unit n'utilise que la valeur absolu de ses opérandes $\alpha$ et $\beta$, le bit de signe des opérandes (bit de poids $15$) n'est pas transmis au circuit.
Je vous renvoie vers le chapitre "_Le standard IEEE-754_", si vous souhaitez revoir l'illustration du codage Half Precision d'un nombre.

En bref, le circuit peut être mis dans deux états, l'état de _point terminal_ et de _point non terminal_.
L'un ou l'autre de ces états est généré par le traitement des champs d'exposant des opérandes $\alpha$ et $\beta$, c'est pourquoi le circuit traite les champs d'exposant avant les champs de mantisse tronquée des opérandes.
Voyons ci-dessous ce que sont les points terminaux et non terminaux, ainsi que ce qu'ils impliquent.

### Les points terminaux et non terminaux

Cette section explique ce que sont les points terminaux et non terminaux.
Mais aussi la raison du traitement prioritaire des champs d'exposant $E$ des opérandes $\alpha$ et $\beta$, vis à vis des champs de mantisse tronquée $T$ de ces même opérandes.

Nous avons vu au travers du chapitre "_Le multiplicande_" et des deux sections qui le suivent, que l'écriture du nombre $F$ en notation scientifique binaire ressemblait à $F \ = \pm \left(S \times 2^{-c}\right)$.
Par conséquent, nous savons désormais que pour n'importe quel format IEEE-754, cela est équivalent à $F \ = \pm$ $\left(\left(1 + T\right) \times \ 2^E\right)$.
N'oubliez pas que le champs de mantisse tronquée rend implicite le bit à $1$ de la partie entière du significande $S \in \left[1;2\right[$.
D'où le fait qu'il faille ajouté la valeur de ce bit à celle du champs de mantisse tronquée $T$ pour obtenir la valeur réel qu'interprète le champs.

//

Nous avons vu dans le chapitre "_Le multiplicande_", comment est ce qu'en écriture scientifique binaire nous pouvions obtenir le nombre d'origine $F$ depuis le significande $S$. Précisément au travers du calcul suivant $F \ = \left(S\times 2^{-c}\right)$.
En tenant compte des correspondances entre les éléments de l'encodage IEEE-754 et de l'écriture scientifique binaire du nombre $F$, nous en déduisons qu'en IEEE-754 $F \ =$ $\left(\left(1 + T\right) \times 2^E\right)$.

N'oublions pas que si le significande $S$ de la notation scientifique binaire du nombre $F$, est compris dans l'intervalle de valeur suivante $\left[1;2\right[$.
Ce n'est pas le cas du champs de mantisse tronquée $T$, __car le bit à 1 de sa partie entière est rendu implicite__.
D'où le fait que pour représenté la valeur réel du champs, il nécessaire d'ajouté ce que vaut la partie entière (c'est à dire $1$) au champs de mantisse tronquée lui même. 
Ce qui donne $\left(1 + T\right)$.

Au travers du calcul suivant $\left(\left(1 + T\right) \times 2^E\right)$, nous observons quelques chose d'intéressant.
Par essence $\left(2^i = 2 \times 2^{\left(i-1\right)}\right)$.
Etant donné que la valeur codé dans le champs de la mantisse tronquée $T$ est strictement inférieur à $1$, alors même pour $T = 0.99..9$ nous obtenons avec $\left(\left(1 + T\right) \times 2^E\right)$ une valeur __strictement inférieur__ à $\left(1 \times 2^{\left(E+1\right)}\right)$.
Nous voyons ici pourquoi est ce que les champs d'exposant $E$ des opérandes sont traités avant les champs de mantisse tronquée $T$.

Imaginons que pour deux opérandes $\alpha$ et $\beta$, le circuit teste la condition $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$:
  - Si $\left(E_{\alpha} \lt E_{\beta}\right)$ alors nous savons que la comparaison __échoue__ car $\left(\left(1 + T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \lt \left(\left(1 + T_{\beta}\right)\times 2^{E_{\beta}}\right)$ et ce même si $T_{\alpha} = 0.99..9$ lorsque $T_{\beta} = 0$.

  - Si $\left(E_{\alpha} \gt E_{\beta}\right)$ alors la comparaison __réussie__ cette fois-ci, car $\left(\left(1 + T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times 2^{E_{\beta}}\right)$ et ce même si $T_{\alpha} = 0$ lorsque $T_{\beta} = 0.99..9$.

Dans les deux cas qui figurent ci-dessus, __le circuit atteint un point terminal__.
C'est à dire qu'il est capable de générer le résultat d'une comparaison par le seul traitement des champs d'exposant de ses opérandes, il court-circuite le reste des calculs sur les champs de mantisse tronquée afin de rendre disponible le résultat le plus rapidement possible.
En bref, un point terminal sera atteint lorsque $\left(E_{\alpha} \neq E_{\beta}\right)$.

Mais il existe une autre possibilité à laquelle le circuit peut être confronté, celle dans laquelle les champs d'exposant de ses opérandes $\alpha$ et $\beta$ sont égaux $\left(E_{\alpha} = E_{\beta}\right)$.
A ce moment là, le circuit atteint alors un __point non terminal__.
Dans ce cas spécifique, les produits $\left(\left(1 + T_{\alpha}\right) \times 2^{E_{\alpha}}\right)$ ainsi que $\left(\left(1 + T_{\beta}\right) \times 2^{E_{\beta}}\right)$ ont la même puissance de $2$ en commun.
Par conséquent, le circuit électronique se repose sur les champs $T_{\alpha}$ et $T_{\beta}$ pour pouvoir départagé si $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ ou non.

Nous comprenons désormais pourquoi est ce que le circuit traite les champs d'exposant $E$ de ses opérandes $\alpha$ et $\beta$, avant les champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$.
Le résultat d'une comparaison peut être généré rapidement en cas de _point terminal_ occasionner par une non égalité entre les champs d'exposant $E$ des opérandes du circuit.
En cas de dernier recours si $E_{\alpha}$ et $E_{\beta}$ sont égaux, le circuit ira jusqu'à comparer les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$ pour générer un résultat.
Ce qui est plus long.
