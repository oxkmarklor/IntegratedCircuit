Bonjour.

Ce document est une preuve mathématique d'un circuit électronique du nom de FPU Configuration Unit.
J'ai conceptualisé ce circuit dans le but de résoudre un problème concernant le traitement des nombres flottants dans les unités de calculs flottantes (FPUs) d'un ordinateur.
La micro-architecture du circuit électronique ainsi que les compromis de conceptualisation, le problème que cherche à résoudre le FPU Configuration Unit ou encore la solution qu'il apporte, sont autant de sujets dont je parle dans une documentation dédié au circuit.

Pour les curieux, si vous souhaitez voir à quoi ressemble le circuit, sachez qu'il y a un fichier du nom de _16-bit_FPU_configuration_unit_ qui le schématise.
Le circuit a été directement schématisé en transistor de technologie CMOS, cela permet de mettre en évidence certains compromis sur le choix des portes logiques.
Disons que l'apparence abrupte du schéma rend hommage à la (relative) complexité algorithmique du FPU Configuration Unit, cette complexité est plus tangible qu'avec des lignes de code.

Le document va même au delà de la démonstration mathématique du circuit, il explique les sujets fondamentaux sur lequels la démonstration ainsi que le circuit reposent.
Ce qui explique la préscence de la table des matières suivante:
  - Les encodages
    - L'encodage Binary Unsigned
      - Une propriété du Binary Unsigned qui est fondamentale à la démonstration
    - Les nombres à virgule flottante
      - L'étymologie du terme de nombre à virgule flottante
  - L'écriture scientifique binaire
    - Le significande
      - La transformation d'un nombre à virgule flottante en un significande
      - Détail sur le fonctionnement de l'équation
    - Le multiplicande
      - Lorsque le multiplicande multiplie la valeur du significande
      - Lorsque le multiplicande divise la valeur du significande
      - Un autre regard sur le multiplicande
  - Le standard IEEE-754
    - Le champs de mantisse tronquée
      - L'encodage du champs de mantisse tronquée
      - Le codage du champs de mantisse tronquée
    - Le champs d'exposant
      - L'encodage par biais du champs d'exposant
      - Le codage du champs d'exposant
  - Introduction au FPU Configuration Unit
    - Les points terminaux et non terminaux

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
Malgré tout, dans les prochains chapitres dédiés aux champs d'exposant et de mantisse tronquée, nous verrons que le cas de ces champs est légèrement plus complexe.

Etant donné que le circuit électronique ne prend en charge que des opérandes (nombres) au format Half Precision, le reste du document ne fera plus que référence à ce format.

## Le champs de mantisse tronquée

En introduction se trouve une illustration de la disposition des bits qui composent le format Half Precision, cette illustration montre que le champs de mantisse tronquée $T$ est composé de $10$ bits.
Voici les poids de chacun des bits qui compose le champs de mantisse tronquée :

$$\left[ \quad .. \quad , \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

Les deux sections suivantes concernent l'ensemble de ces bits formant le champs de mantisse tronquée.

### L'encodage du champs de mantisse tronquée

Le nombre $F$ codé au format Half Precision a un champs de mantisse tronquée qui correspond au significande de l'écriture scientifique binaire du même nombre $F$.
Par conséquent, l'encodage du champs de mantisse tronquée est simplement celui d'un nombre à virgule flottante.
Je vous invite donc à relire le chapitre "_Les nombres à virgule flottante_", si nécessaire.

Néanmoins, le dernier paragraphe de ce chapitre donne une information assez importante. 
La valeur de n'importe quel bit à $1$ de poids $i$ dans un nombre à virgule flottante, est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Ce qui veut dire que n'importe quel bit à $1$ de poids $i$ dans un champs de mantisse tronquée, a une valeur qui est elle aussi strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
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
Le résultat de ce calcul forme la valeur que représente réelement le champs.
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

Ladite comparaison est une vérification de supériorité stricte de la valeur absolu de l'un des deux opérandes envers la valeur absolu de l'autre.
Le FPU Configuration Unit n'utilise que la valeur absolu de ses opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$, le bit de signe des opérandes (bit de poids $15$) n'est pas transmis au circuit.
Je vous renvoie vers le chapitre "_Le standard IEEE-754_", si vous souhaitez visualiser l'illustration du codage Half Precision d'un nombre.

En bref, le circuit peut être mis dans deux états, l'état de _point terminal_ et de _point non terminal_.
L'un ou l'autre de ces états est généré par le traitement des champs d'exposant des opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$, c'est pourquoi le circuit traite les champs d'exposant avant les champs de mantisse tronquée des opérandes.
Voyons ci-dessous ce que sont (entre autre) les points terminaux et non terminaux, ainsi que ce qu'ils impliquent.

## Les points terminaux et non terminaux

Cette section aborde les sujets suivants:
  - Les traitements à effectués sur les champs des opérandes.
  - Les états de points terminaux et non terminaux dans lesquels peut se retrouvé le circuit.
  - La raison du traitement prioritaire des champs d'exposant vis à vis des champs de mantisse tronquée.

Nous savons que le FPU Configuration Unit effectue une vérification de supériorité stricte de la valeur absolu de l'un de ses deux opérandes, envers la valeur absolu de l'autre.
Cependant, le tout est de savoir comment est ce que cette comparaison peut réelement avoir lieu, sachant que les opérandes du circuit sont composés de multiples champs.
La réponse est simple, il suffit d'effectuer une vérification de supériorité stricte entre les champs d'exposant $E$ des opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$, et de faire de même entre les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Nous verrons ci-bas comment cela se traduit.

Au travers du chapitre "_le multiplicande_" ainsi que dans les deux sections qui le suivent, nous avons vus que la représentation en écriture scientifique binaire du nombre $F$ en valeur absolu était $\vert \ F \vert = \left(S \times 2^{-c}\right)$.
Suite au chapitre "_Le standard IEEE-754_" et aux sections suivante, nous en déduisons que n'importe quel format défini par la norme représenterait la valeur absolu de ce même nombre de la façon suivante $\vert \ F \vert = \left(\left(1 + T\right) \times \ 2^E\right)$.
N'oubliez pas que le champs de mantisse tronquée $T$ rend implicite le bit à $1$ de la partie entière du significande $S$.
D'où le fait qu'il faille ajouté la valeur de ce bit à celle du champs de mantisse tronquée $\left(1 + T\right)$.
Je vous invite à relire la section "_Le codage du champs de mantisse tronquée_", si nécessaire.

Extrapolons aux opérandes du FPU Configuration Unit ce qu'explique le paragraphe ci-dessus, cela donne $\vert \ \alpha \vert = \left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right)$ ainsi que $\vert \ \beta \vert = \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.
Remarquons qu'à partir du moment où $\left(E_{\alpha} \neq E_{\beta}\right)$, nous pouvons avoir la certitude que $\left(\vert \alpha \vert \neq \vert \beta \vert\right)$.
Dans cette situation, le circuit atteint soudainement l'état de point terminal.
Ce qui traduit l'idée qu'il puisse générer le résultat de la condition qu'il teste, par le seul traitement des champs d'exposant $E$ de ses opérandes.
Le fait que les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$ codent une valeur comprise dans l'intervalle $\left[0;1\right[$, explique que $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \neq \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$ lorsque $\left(E_{\alpha} \neq E_{\beta}\right)$.

L'intérêt des points terminaux est bien évidemment de permettre au circuit de générer son résultat le plus rapidement possible, en court-circuitant les traitements en cours sur les champs de mantisse tronquée.
C'est la raison pour laquelle les champs d'exposant des opérandes du circuit sont traités en priorité, avant les champs de mantisse tronquée de ces même opérandes.

Il reste cependant possible que les champs d'exposant des opérandes soient égaux $\left(E_{\alpha} = E_{\beta}\right)$.
Dans cette situation il n'y a rien dans les champs d'exposant $E$ des opérandes, qui puisse permettre au circuit de déduire de manière anticipée le résultat de la condition qu'il teste.
Au contraire, le circuit n'est pas dans un état de point terminal mais de point non terminal.
Du point de vue des mathématiques, les opérandes du circuit $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right)$ et $\left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$ partagent la même puissance $2$.
Ce qui force le circuit à devoir traité les champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$ pour pouvoir générer un résultat.

Nous l'aurons compris, le circuit atteint un état de point non terminal lorsque le traitement des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ ne l'amène à un état de point terminal.
Dans un état de point non terminal, le circuit doit effectuer le traitement des champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$.
Comme cela a été dit en introduction de cette section, le traitement des champs de mantisse tronquée, à l'instar de celui des champs d'exposant, consiste en une vérification de supériorité stricte entre la valeur des champs.

# Démonstration mathématique

Nous pouvons désormais plongé dans le coeur de ce document, la démonstration mathématique du circuit.
Il y a une documentation dédié au circuit pour comprendre son utilité ainsi que son architecture, si vous souhaitez plus d'information concrète sur ce dernier je vous conseille de la lire.

En bref, le circuit génère deux bits de sortie, l'un pour configuré un circuit soustracteur de nombre flottant et l'autre pour la sortie même de cette unité de calcul.
Le FPU Configuration Unit (le circuit) reçoit en entrée deux opérandes Half Precision en valeur absolu, que nous nommerons $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$.
La génération des deux bits de sortie du circuit passe par une comparaison de supériorité stricte de la valeur absolu de l'un de ses deux opérandes, envers la valeur absolu de l'autre.
Cette démonstration mathématique n'aborde que la partie de la logique de comparaison du circuit, ce qui représente tout de même la majeur partie de la logique du FPU Configuration Unit.
Le reste de la logique manquante ne peut pas être expliqué sans l'apport de beaucoup de nouvelle notion.
Je vous renvoie vers la documentation dédié au circuit si vous voulez en savoir plus sur cette partie de la logique manquante.

__Tout ce qui va suivre de la démonstration, part du principe que le FPU Configuration Unit cherche à verifié la condition__ $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$.

### Définition de quelques opérations fondamentales à la démonstration 

Pour commencer, l'opération __Write__ sert de fonction d'affectation.
L'argument sur le paramètre $y$ est copié dans l'argument sur le paramètre $x$.
Il n'y a pas de taille ni de "type" d'argument, cette fonction est abstraite et vise simplement à faire comprendre qu'il faut retenir $y$ dans $x$.

$$Write \ \left(x, \ y\right) \rightarrow \ x \ := \ y$$

Passons au cas des opérations logique avec l'opération __And__ $\left(x \wedge y\right)$.
C'est une opération de logique bit à bit qui prend deux bits $x$ et $y$ comme argument, et retourne un unique bit de résultat.
L'opération ne retourne un bit à $1$ que lorsque ses deux bits en argument sont eux même à $1$, sinon elle retrourne un bit à $0$.

L'opération logique __Not__ $\left(\overline{x}\right)$, ne prend quant à elle qu'un seul bit comme argument et retourne l'inverse de ce bit $\left(\overline{1} = 0\right)$.

Pour finir, passons à l'opération logique __Nimply__.
Cette opération est défini en tant que fonction car elle n'est pas très connu et ne possède pas son propre symbole calculatoire.
Ladite fonction a deux paramètres $x$ et $y$, et retourne un bit de résultat. 
Selon le corps de la fonction, nous en déduisons qu'elle ne retourne un bit à $1$ que lorsque le bit sur son paramètre $x$ vaut $1$ quand celui sur $y$ est à $0$, sinon le bit retourné vaut $0$.

$$Nimply \ \left(x, \ y\right) \rightarrow \ x \ \wedge \ \overline{y}$$

# Le traitement des champs d'exposant

Comme cela a été dit par le chapitre "_Introduction au FPU Configuration Unit_", puis expliquer par la section "_Les points terminaux et non terminaux_".
Les champs d'exposant des opérandes du circuit sont traités avant les champs de mantisse tronquée de ces même opérandes.
Je vous invite à relire ces chapitres si nécessaire.

Rappelons que le circuit effectue une comparaison de supériorité stricte entre les champs d'exposant de ses opérandes.
Ce qui dans le cas de la condition sur laquelle s'appuie la démonstration mathématique du circuit $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, nécessite de vérifier si $\left(E_{\alpha} \gt E_{\beta}\right)$.
Voici la première phase du traitement des champs d'exposant.

$$\forall \ i \in \left[10;14\right], \quad Write \ \left(\tau_i, \ Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)\right)$$

Nous effectuons l'opération logique $Nimply$ sur chaque bit de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.
La variable $\tau$ (tau) représente un champs binaire de $15$ bits, dont les poids vont de $0$ à $14$.
Chaque bit de résultat d'une opération $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$ pour $i \in \left[10;14\right]$, est inscrit dans $\tau_i$.

Avec ce que nous avons vu de l'opération logique $Nimply$ dans la section précédente, nous comprenons que du moment où $\left(\tau_i = 1\right)$ pour tout poids $i \in \left[10;14\right]$ alors $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$.
Pour exprimer les choses différemment $\left(E_{\beta i} \times 2^i\right) \gt \left(E_{\alpha i} \times 2^i\right)$.
En plus de cela, rappelez vous de la section "_L'encodage par biais du champs d'exposant_".
Cette section explique entre autre que, je cite "_la valeur de n'importe quel bit à_ $1$ _de poids_ $i$ _dans un champs d'exposant, est strictement supérieur à la somme des valeurs des bits de poids inférieur à_ $i$".
Cela traduit l'idée que du moment où $\left(\tau_i = 1\right)$ pour tout $i \in \left[11;14\right]$, nous pouvons être sûrs que $\left(E_{\beta i} \times 2^i\right) \gt \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$.

Pour conclure, nous pouvons être certains que du moment où $\left(\tau_i = 1\right)$ pour tout $i \in \left[10;14\right]$, alors $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$.
Pour le dire autrement, du bit de poids $i$ jusqu'au bit de poids $10$ le champs d'exposant $E_{\beta}$ est supérieur au champs d'exposant $E_{\alpha}$.
Mais cela ne démontre pas pour autant que $\left(E_{\alpha} \lt E_{\beta}\right)$.

## Les zéros anonymes

Nous avons vu dans la section "_Définition de quelques opérations fondamentales à la démonstration_", que l'opération logique $Nimply$ génère un bit de résultat à $1$ sous réserve que le bit sur son paramètre $x$ vaille $1$, et que celui sur $y$ vaille $0$.
Autrement, l'opération renvoie un bit de résultat à $0$.

Sachez qu'il est possible de décliné l'opération logique $Nimply$ en une porte logique, un petit circuit électronique chargé d'effectuer une opération $Nimply$ entre deux bits.
Le FPU Configuration Unit se sert des portes logiques $Nimply$ pour pouvoir déduire la valeur des bits d'opérandes d'une telle opération, en fonction du bit de résultat généré.
C'est exactement ce que nous avons fait dans le chapitre précédent, nous déduisons que du moment où $\left(\tau_i = 1\right)$ alors $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$.
Cependant, les choses se corsent à partir du moment où une opération $Nimply$ génère un bit de résultat à $0$, ce que la documentation dédié au circuit appelle un _zéro anonyme_.

La raison est simple, pour une opération $Nimply$ il n'y a qu'une seule manière de définir un bit de résultat à $1$, mais plusieurs pour un bit de résultat à $0$.
Concrètement, une opération logique $Nimply$ génère un bit de résultat à $0$ lorsque:
  - Ces deux bits d'opérandes $\left(x, \ y\right)$ sont égaux.
  - Le bit sur le paramètre $x$ vaut $0$ pendant que celui sur $y$ vaut $1$.

Par conséquent, un bit de résultat à $0$ ne permet pas de déduire la valeur des bits d'opérandes (qui est comme anonyme), d'où le nom de _zéro anonyme_.
Plus bas, nous allons voir que les _zéros anonymes_ sont la raison pour laquelle le chapitre précédent conclu de la sorte.
Je précise que dans les faits il existe des _zéros anonymes capitaux_ et _non capitaux_, dissociation importante que nous allons expliqués ci-dessous.

Pour finir, la section "_Les points terminaux et non terminaux_" évoque le fait que le traitement des champs d'exposant ainsi que des champs de mantisse tronquée soit identique ou presque.
Nous comprenons d'ores et déjà que le traitement des champs d'exposant passe par l'application d'opération logique $Nimply$ sur les bits des champs $E_{\alpha}$ et $E_{\beta}$.
Ce qui engendre quelques subtilités qu'il faut expliqué, comme celle des _zéros anonymes capitaux_ et _non capitaux_.
Sachez que le traitement des champs de mantisse tronquée utilise aussi des opérations logiques $Nimply$.
Par conséquent, nous recontextualiserons en temps voulu les concepts de _zéro anonyme capital_ et _non capital_ aux cas des champs de mantisse tronquée.
Mais pour l'instant concentrons sur le cas des champs d'exposant.

### Les zéros anonymes non capitaux dus au traitement des champs d'exposant

Les _zéros anonymes capitaux_ et _non capitaux_ sont dus aux même raisons, le chapitre précédent explique très bien cela.
Ce que le chapitre n'explique pas, c'est que les _zéros anonymes non capitaux_ n'ont aucune importance pour le traitement des champs d'exposant $E_{\alpha}$ et $E_{\beta}$, au contraire des _zéros anonymes capitaux_.
Nous verrons plus tard que cela ne se limite pas au traitement des champs d'exposant, mais s'étend aussi au traitement des champs de mantisse tronquée.

Après cette rapide introduction, reprenons là où nous nous étions arrêté dans le chapitre "_Le traitement des champs d'exposant_".
Admettons qu'il y ait un _MSB1_ dans $\tau \in \left[10;14\right]$ et que ce dernier ait un poids $i \in \left[11;14\right]$, alors nous en déduisons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$ pour $\left(\tau_i = 1\right)$.
En outre, comme l'explique ledit chapitre, nous en déduisons que $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$.
Rappelons que cette inéquation fonctionne car la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs d'exposant comme $E_{\beta}$, est _inconditionnellement_ supérieur à la somme des valeurs des bits de poids inférieur à $i$. 
Cela se remarque plus avec la formulation suivante de cette même inéquation $\left(E_{\beta i} \times 2^i\right) \gt \left(\left(E_{\alpha i} \times 2^i\right) + \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right)$, toujours dans le cas où $\left(\tau_i = 1\right)$ rappelons-le.

Définissons désormais ce qu'est un _zéro anonyme non capital_.
Par le prisme du traitement des champs d'exposant, un _zéro anonyme non capital_ est un bit à $0$ dans $\tau$ qui est d'un poids $\sigma \in \left[10;i\right[$.
Du moment où le poids $i$ du _MSB1_ est compris dans l'intervalle $\left[11;14\right]$, alors nous pouvons être certain que dans $\tau \in \left[10;14\right]$ il existe au moins un bit $\tau_{\left(i - 1\right)}$, qu'il soit à $0$ ou à $1$.
En conséquence, lorsque $\left(\tau_{\left(i - 1\right)} = 0\right)$ il est possible d'en déduire que le bit est un _zéro anonyme non capital_, et de fait que les valeurs des bits d'opérandes $E_{\alpha \left(i - 1\right)}$ ainsi que $E_{\beta \left(i - 1\right)}$ ne sont pas déductible, nous savons seulement que $\left(E_{\alpha \left(i - 1\right)} \ge E_{\beta \left(i - 1\right)}\right)$.

En reprenant l'inéquation $\left(E_{\beta i} \times 2^i\right) \gt \left(\left(E_{\alpha i} \times 2^i\right) + \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right)$, nous remarquons que $\left(E_{\beta i} \times 2^i\right)$ est supérieur à la somme des valeurs des bits de poids $\sigma$ du champs d'exposant $E_{\alpha}$, nonobstant la valeur de ces bits dont celle de $\left(E_{\alpha \left(i - 1\right)} \times 2^{\left(i - 1\right)}\right)$.
Autrement dit, peu importe que les bits de poids $\sigma$ du champs d'exposant $E_{\alpha}$ soient définis à $0$ ou à $1$ (ce qui concerne $E_{\alpha \left(i - 1\right)}$), car nous trouverons toujours que $\left(E_{\beta i} \times 2^i\right) \gt \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$.

Pour resumer, un _zéro anonyme non capital_ dans $\tau_{\sigma}$ pour tout poids $\sigma \in \left[10;i\right[$, ne permet pas de déduire la valeur des bits d'opérandes $E_{\alpha\sigma}$ ainsi que $E_{\beta\sigma}$.
Cependant, pour pouvoir générer un état de point terminal, le circuit n'a pas besoin de prendre en compte les bits de poids $\tau_{\sigma}$ car il est déductible que le champs d'exposant $E_{\beta}$ est supérieur à $E_{\alpha}$ du bit de poids $i$, le poids du _MSB1_ de $\tau \in \left[10;14\right]$, jusqu'au bit de poids $10$.
Voici en quelque sorte l'étymologie du terme "_zéro anonyme non capital_".

### Les zéros anonymes capitaux dus au traitement des champs d'exposant

La section précédente explique pourquoi les bits de $\tau$ d'un poids inférieur au _MSB1_ de $\tau \in \left[10;14\right]$ ne sont pas pris en compte dans le traitement des champs d'exposant, ce qui concerne entre autre les _zéros anonymes non capitaux_.
Voyons désormais ce qu'il en est des bits de $\tau$ qui sont d'un poids supérieur au _MSB1_. 

Reprenons là où nous en étions dans le chapitre "_Le traitement des champs d'exposant_".
Admettons que le _MSB1_ de $\tau \in \left[10;14\right]$ ait un poids $i \in \left[10;13\right]$, alors nous savons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$ pour $\left(\tau_i = 1\right)$.
En outre, comme l'explique ledit chapitre, nous en déduisons alors que $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$.
Cependant, comme cela a été mentionné dans ce même chapitre, ceci ne suffit pas à démontrer que $\left(E_{\alpha} \lt E_{\beta}\right)$.

Définissons désormais ce qu'est un _zéro anonyme capital_.
Par le prisme du traitement des champs d'exposant, un _zéro anonyme capital_ est un bit à $0$ dans $\tau$ qui est d'un poids $\sigma \in \left]i;14\right]$.
Du moment où le poids $i$ du _MSB1_ est compris dans l'intervalle $\left[10;13\right]$, alors nous pouvons être certain que dans $\tau \in \left[10;14\right]$ il existe au moins un bit $\tau_{\left(i + 1\right)}$ qui vaille $0$.
Le bit $\tau_{\left(i + 1\right)}$ ne peut être défini qu'à $0$ étant donné que $\tau_i$ incarne le _MSB1_ de $\tau \in \left[10;14\right]$, ceci se généralise à l'ensemble des bits $\tau_{\sigma}$.
En conséquence, n'importe quel bit de poids $\tau_{\sigma}$ est un _zéro anonyme capital_, ce qui insinue que les valeurs des bits d'opérandes $E_{\alpha\sigma}$ ainsi que $E_{\beta\sigma}$ ne sont pas déductible, nous savons seulement que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$.

Rappelons que l'inéquation $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$ démontre que le champs d'exposant $E_{\beta}$ est supérieur au champs d'exposant $E_{\alpha}$, du bit de poids $i$ jusqu'au bit de poids $10$.
N'oublions pas que cette inéquation fonctionne car la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs d'exposant comme $E_{\beta}$, est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Mais n'oublions pas non plus qu'il subsite dans $\tau$ des bits de poids $\sigma$ supérieur à $i$, des _zéros anonymes capitaux_, par lesquels nous déduisons que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$.
Il suffit alors qu'il y ait une occurrence de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$ pour que nous en déduisions que $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$, ce qui traduit l'idée que du bit de poids $\sigma$ jusqu'au bit de poids $10$, le champs d'exposant $E_{\alpha}$ soit supérieur au champs d'exposant $E_{\beta}$.
Finalement, pour savoir si le champs d'exposant $E_{\beta}$ est définitivement supérieur au champs d'exposant $E_{\alpha}$ ou non, il nous faut prêter attention à la valeur des bits d'opérandes $E_{\alpha\sigma}$ et $E_{\beta\sigma}$ des _zéros anonymes capitaux_.

Il y a grosso modo deux possibilités dont la première est la suivante.
Le circuit atteint l'état de point terminal $\left(E_{\alpha} \lt E_{\beta}\right)$ lorsque $\left(E_{\alpha\sigma} = E_{\beta\sigma}\right)$ pour tout les bits de poids $\sigma \in \left]i;14\right]$.
Cela insinue que les champs d'exposant $E_{\alpha}$ ainsi que $E_{\beta}$ sont égaux du bit de poids $\left(i + 1\right)$ jusqu'au bit de poids $14$.
Pour verifier cela, il suffit de calculer la somme $\lambda$ (lambda) de la valeur des bits de poids $\sigma$ des champs d'exposant $E_{\alpha}$ ainsi que $E_{\beta}$, puis de comparer.
Dans les faits nous trouvons $\left(\lambda_{\alpha} = \lambda_{\beta}\right)$, en sachant que $\lambda_{\alpha} = \sum_{\sigma = i + 1}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$ tandis que $\lambda_{\beta} = \sum_{\sigma = i + 1}^{14} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$.
En outre, n'oublions pas que l'inéquation $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$ démontre que du bit de poids $i$ jusqu'au bit de poids $10$, le champs d'exposant $E_{\beta}$ est supérieur au champs d'exposant $E_{\alpha}$.
Nous en déduisons donc que le circuit atteint l'état de point terminal $\left(E_{\alpha} \lt E_{\beta}\right)$, car $\left(\lambda_{\beta} + E_{\beta i} \times 2^i\right) \gt \left(\lambda_{\alpha} + \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)\right)$.
Enfin, rappelez vous du fait que la condition sur laquelle s'appuie la démonstration mathématique du circuit est $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, condition qui se solde en échec car $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \lt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.
Tel que l'explique le chapitre "_Les points terminaux et non terminaux_", $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right)$ ainsi que $\left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$ représentent les valeurs respective de $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$.

// relire

Néanmoins, parmis les bits d'opérande $E_{\alpha\sigma}$ et $E_{\beta\sigma}$ des _zéros anonymes capitaux_, pour tout poids $\sigma \in \left]i;14\right]$, il est possible qu'il y ait une (ou plusieurs) occurrence de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$.
Auquel cas, le circuit atteint alors l'état de point terminal $\left(E_{\alpha} \gt E_{\beta}\right)$.
Pour chaque occurrence de $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(E_{\beta\sigma} \times 2^{\sigma}\right)$ nous en déduisons que $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$, ce qui traduit l'idée que le champs d'exposant $E_{\alpha}$ soit supérieur au champs d'exposant $E_{\beta}$, du bit de poids $\sigma$ jusqu'au bit de poids $10$.
Cependant, admettons que $\sigma$ soit compris dans l'intervalle $\left]i;14\right[$, alors nous savons qu'il existe des _zéros anonymes capitaux_ de poids $\sigma '$ dans $\tau$, tel que $\sigma ' \in \left]\sigma;14\right]$.
En bref, cela veut dire que tout bit $\tau_{\sigma '}$ a pour bit d'opérande $E_{\alpha\sigma '}$ ainsi que $E_{\beta\sigma '}$ dont il est déductible que $\left(E_{\alpha\sigma '} \ge E_{\beta\sigma '}\right)$. 
Ce qui insinue que la somme $\lambda$ de la valeur des bits de poids $\sigma '$ du champs d'exposant $E_{\alpha}$, est supérieur ou égale à la somme des bits de même poids du champs d'exposant $E_{\beta}$, ou pour le dire autrement $\left(\lambda_{\alpha} \ge \lambda_{\beta}\right)$.
Par conséquent, le circuit atteint l'état de point terminal $\left(E_{\alpha} \gt E_{\beta}\right)$ car $\left(\lambda_{\alpha} + E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(\lambda_{\beta} + \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)\right)$.
Cette fois-ci, la condition sur laquelle s'appuie la démonstration mathématique du circuit $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$ réussie, car nous trouvons que $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.

// save

Cependant, il suffit qu'une ou plusieurs occurences de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$ avec $\sigma \in \left]i;14\right]$, pour que le circuit atteigne l'état de point terminal $\left(E_{\alpha} \gt E_{\beta}\right)$.
Dans le chapitre "_L'encodage par biais du champs d'exposant_", nous expliquons que la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs d'exposant, est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Par conséquent, pour chacune des occurences $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(E_{\beta\sigma} \times 2^{\sigma}\right)$ nous en déduisons que $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$.
Dans le cas où $\sigma \in \left]i;14\right[$, alors nous savons qu'il existe $\left(\tau_{\sigma '} = 0\right)$ pour tout poids $\sigma ' \in \left]\sigma;14\right]$, ce qui permet d'en déduire que $\left(E_{\alpha\sigma '} \ge E_{\beta\sigma '}\right)$.
Cela veut dire que la somme $\lambda$ de la valeur des bits de poids $\sigma '$ du champs d'exposant $E_{\alpha}$, est supérieur ou égale à celle du champs d'exposant $E_{\beta}$, ou $\left(\lambda_{\alpha} \ge \lambda_{\beta}\right)$ pour le dire autrement.
Gardons à l'esprit que l'inéquation $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$, démontre que du bit de poids $\sigma$ jusqu'au bit de poids $10$ le champs d'exposant $E_{\alpha}$ est supérieur au champs d'exposant $E_{\beta}$.
Ce qui insinue que $\left(E_{\alpha} \gt E_{\beta}\right)$ car $\left(\lambda_{\alpha} + E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(\lambda_{\beta} + \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)\right)$.
En outre, il me faut conclure en évoquant la réussite du test de la condition sur laquelle s'appuie la démonstration mathématique du circuit $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, cette réussite est due au fait que $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.

// test 1

Néanmoins, parmis l'ensemble des bits d'opérandes de poids $\sigma \in \left]i;14\right]$ où $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$, il suffit qu'il y ait une occurrence de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$ pour que le circuit passe dans l'état de point terminal $\left(E_{\alpha} \gt E_{\beta}\right)$.
Par conséquent, nous savons que $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(E_{\beta\sigma} \times 2^{\sigma}\right)$ et nous en déduisons que $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$.
Dans le cas où le poids $\sigma \in \left]i;14\right[$, alors nous devinons l'existence d'un ou de plusieurs _zéro anonyme capital_ $\tau_{\sigma '}$, tel que $\sigma ' \in \left]\sigma;14\right]$.
De ce fait, nous en déduisons que pour chaque bit $\tau_{\sigma '}$ correspond $\left(E_{\alpha\sigma '} \ge E_{\beta\sigma '}\right)$.
Concrètement, cela insinue que la somme $\lambda$ de la valeur des bits de poids $\sigma '$ du champs d'exposant $E_{\alpha}$, est supérieur ou égale à la somme des bits de même poids du champs d'exposant $E_{\beta}$, ou pour le dire autrement $\left(\lambda_{\alpha} \ge \lambda_{\beta}\right)$.
En outre, n'oublions pas que l'inéquation $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$ démontre que du bit de poids $\sigma$ jusqu'au bit de poids $10$, le champs d'exposant $E_{\alpha}$ est supérieur au champs d'exposant $E_{\beta}$.
Nous en déduisons que $\left(E_{\alpha} \gt E_{\beta}\right)$ de par le fait que $\left(\lambda_{\alpha} + E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(\lambda_{\beta} + \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)\right)$.
Pour conclure, la condition sur laquelle s'appuie la démonstration mathématique du circuit $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$ réussie, car $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.

### Les zéros anonymes non capitaux dus au traitement des champs d'exposant

// à relire pour vérif

Nous comprenons désormais l'importance des _zéros anonymes_ de poids supérieur au _MSB1_ de $\tau \in \left[10;14\right]$, voyons ce qu'il en est de ceux d'un poids inférieur au _MSB1_.
Par le prisme du traitement des champs d'exposant, un _zéro anonyme non capital_ est un bit à $0$ dans le champs $\tau \in \left[10;14\right]$, dont le poids est inférieur au _MSB1_ de $\tau \in \left[10;14\right]$.
Par conséquent, lorsque le _MSB1_ a un poids $i \in \left[11;14\right]$, alors les bits à $0$ de $\tau$ qui sont d'un poids $\sigma \in \left[10;i\right[$ sont par définition des _zéros anonymes non capitaux_.

Reprenons d'où nous en étions dans le chapitre "_Le traitement des champs d'exposant_".
Admettons que le _MSB1_ de $\tau \in \left[10;14\right]$ ait un poids $i \in \left[11;14\right]$, alors nous en déduisons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$ pour $\left(\tau_i = 1\right)$.
Je vous invite à vous rappelez du fait que la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs d'exposant comme $E_{\beta}$, est _inconditionnellement_ supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Le fait que ceci soit _inconditionnel_ nous fait comprendre que la valeur $\left(E_{\beta i} \times 2^i\right)$ du bit $E_{\beta i}$, est supérieur à la somme des valeurs des bits de poids inférieur à $i$, nonobstant la valeur de ces derniers.
Ledit chapitre se sert de cela pour en déduire que lorsque $\left(\tau_i = 1\right)$, alors $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$.

Abordons les choses quelques peu différemment, disons que parmis les bits de poids inférieur à $\tau_i$ se trouve $\left(\tau_{\sigma} = 0\right)$ pour $\sigma \in \left[10;i\right[$.
Nous en déduisons donc que le bit $\tau_{\sigma}$ est un _zéro anonyme non capital_ et que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$, ce qui veut dire que le bit $E_{\alpha\sigma}$ peut être défini à $0$ comme à $1$.
En outre, n'oublions pas que le poids $i$ du bit $E_{\beta i}$ est supérieur au poids $\sigma$ du bit $E_{\alpha\sigma}$.
Par conséquent, peu importe que les bits de poids $\sigma$ du champs d'exposant $E_{\alpha}$ soient défini à $0$ ou à $1$, nous trouverons quoi qu'il arrive que $\left(E_{\beta i} \times 2^i\right) \gt \sum_{\sigma}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$.

//

Reprenons d'où nous en étions dans le chapitre "_Le traitement des champs d'exposant_".
Admettons que le _MSB1_ de $\tau \in \left[10;14\right]$ ait un poids $i \in \left[11;14\right]$, alors nous en déduisons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$ pour $\left(\tau_i = 1\right)$.
Je vous invite à vous rappelez du fait que la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs d'exposant comme $E_{\beta}$, est _inconditionellement_ supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Concrètement, cela traduit l'idée qu'avec $\left(\tau_i = 1\right)$ alors $\left(E_{\beta i} \times 2^i\right) \gt \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$.

Imaginons que parmis les bits de poids inférieur à $\tau_i$ se trouve $\left(\tau_{\sigma} = 0\right)$, pour tout poids $\sigma \in \left[10;i\right[$.
Par conséquent, $\tau_{\sigma}$ est un _zéro anonyme non capital_ et nous en déduisons que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$.
Remarquons que le bit $E_{\alpha\sigma}$ peut être défini à $0$ comme à $1$, mais remarquons aussi que le poids $i$ du bit $E_{\beta i}$ est supérieur au poids $\sigma$ du bit $E_{\alpha\sigma}$.
Etant donné que $\left(E_{\beta i} = 1\right)$, alors la valeur du bit $\left(E_{\beta i} \times 2^i\right)$ est _inconditionellement_ supérieur à la somme des valeurs des bits de poids inférieur à $i$ du champs d'exposant $E_{\alpha}$, somme à laquelle contribue le bit $E_{\alpha\sigma}$.
Pour le dire autrement, peu importe que les bits de poids $\sigma$ du champs d'exposant $E_{\alpha}$ soient défini à $0$ ou à $1$, nous trouverons quoi qu'il arrive que $\left(E_{\beta i} \times 2^i\right) \gt \sum_{\sigma}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$.

//

Pour résumer, un _zéro anonyme non capital_ dans $\tau_{\sigma}$ pour tout poids $\sigma \in \left[10;i\right[$, ne permet pas de déduire la valeur des bits $E_{\alpha\sigma}$ ainsi que $E_{\beta\sigma}$.
Cependant, pour pouvoir générer un état de point terminal, le circuit n'a pas besoin de prendre en compte les bits de $\tau$ qui sont d'un poids inférieur au poids $i$ du _MSB1_ de $\tau \in \left[10;14\right]$.
D'où le fait que ces _zéros anonymes_ soient _non capitaux_.

## L'obtention d'un point non terminal

Je précise que ce chapitre parle des états de points terminaux ainsi que non terminaux du circuit.
Si nécessire, vous pouvez lire de nouveau la section "_Les points terminaux et non terminaux_" qui explique ce que sont ces états.

Dans les sections précédente nous avions toujours un _MSB1_ dans $\tau \in \left[10;14\right]$, mais il pourrait ne pas y en avoir.
Par essence, une telle situation veut que chaque bit de $\tau \in \left[10;14\right]$ soit un _zéro anonyme_, ce qui nous permet de déduire que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$ pour tout poids $\sigma \in \left[10;14\right]$.
Plus précisément, ces bits sont des _zéros anonymes capitaux_, car chacun d'eux a le pouvoir de faire basculé le circuit d'un état de point non terminal à celui de point terminal, comme nous allons le voir.
Pour le dire autrement, nous savons que $\left(E_{\alpha} \ge E_{\beta}\right)$.

Il s'avère que le circuit rentre dans un état de point terminal du moment où $\left(E_{\alpha} \gt E_{\beta}\right)$.
Cet état de point terminal ne peut s'obtenir qu'à la condition qu'il y ait une ou plusieurs occurences de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$ pour tout poids $\sigma \in \left[10;14\right]$.
Par conséquent, nous en déduisons que pour chacune des ces occurences $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$, et le circuit atteint l'état de point terminal $\left(E_{\alpha} \gt E_{\beta}\right)$.
L'une des conséquences à cela est que la condition sur laquelle s'appuie la démonstration mathématique du circuit $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$ réussie, car $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.

Cependant, le circuit rentre dans un état de point non terminal du moment où $\left(E_{\alpha} = E_{\beta}\right)$.
Cet état de point non terminal ne peut s'obtenir que dans le cas où l'ensemble des bits de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ sont identiques, ou autrement dit lorsque $\left(E_{\alpha\sigma} = E_{\beta\sigma}\right)$ pour tout poids $\sigma \in \left[10;14\right]$.
Une conséquence à cela est que le FPU Configuration Unit n'est pas en capacité d'anticipé le résultat de la condition $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, par le seul traitement des champs d'exposant de ses opérandes.
Rappelons que cette condition ressemble plus formellement à $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$, et remarquons que les opérandes du circuit partagent la même puissance $2$.
C'est ce qui force le circuit à devoir traité les champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$ pour pouvoir générer un résultat vérifiant cette condition ou non, chose que nous verrons dans les prochains chapitres.

# Le traitement des champs de mantisse tronquée

Pour ceux qui ont besoin d'un rafraichissement de mémoire, je vous rappelle que les encodages des champs d'exposant et de mantisse tronquée ont des points en commun, ils partagent les même propriétés que l'encodage _Binary Unsigned_.
C'est la raison derrière le fait que le circuit traite les champs de mantisse tronquée d'une façon similaire aux champs d'exposant.
Je vous renvoie vers les sections "_L'encodage du champs de mantisse tronquée_" ainsi que "_L'encodage par biais du champs d'exposant_" si nécessaire.

Rappelons que la démonstration mathématique du circuit s'appuie sur le test de la condition suivante $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, c'est à dire $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.
Ce chapitre parle du traitement des champs de mantisse tronquée, et nous savons que le circuit ne traite les champs de mantisse tronquée de ses opérandes que lorsqu'il est dans un état de point non terminal $\left(E_{\alpha} = E_{\beta}\right)$.
Par conséquent, de la condition que test le circuit il ne reste de pertinent plus que $\left(1 + T_{\alpha}\right) \gt \left(1 + T_{\beta}\right)$, ou autrement dit $\left(T_{\alpha} \gt T_{\beta}\right)$.

En définitif, le circuit est dans un état de point non terminal et génère un résultat vérifiant ou non la condition $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, par une comparaison de supérioté stricte entre les champs de mantisse tronquée de ses opérandes.
D'où le fait qu'il soit dit plus haut que le traitement des champs de mantisse tronquée soit le même que celui des champs d'exposant.
Voici la première phase du traitement des champs de mantisse tronquée.

$$\forall \ i \in \left[0;9\right], \quad Write \ \left(\tau_i, \ Nimply \ \left(T_{\beta i}, \ T_{\alpha i}\right)\right)$$

La variable $\tau$ est techniquement la même que celle utilisée dans la première partie de la démonstration "_Le traitement des champs d'exposant_".
Le seul changement est que le bit de résultat de chaque opération logique $Nimply$ sur $T_{\beta i}$ et $T_{\alpha i}$ est inscrit dans $\tau_i$, pour $i \in \left[0;9\right]$.

En outre, nous comprenons qu'avec $\left(\tau_i = 1\right)$ pour tout $i \in \left[0;9\right]$, il est certain que $\left(T_{\beta i} \times 2^i\right) \gt \left(T_{\alpha i} \times 2^i\right)$.
Par ailleurs, cela a été dit en introduction, l'encodage du champs de mantisse tronquée partage les propriétés du _Binary Unsigned_.
La section du nom de "_L'encodage du champs de mantisse tronquée_", explique entre autre que la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs de mantisse tronquée, est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Ce qui nous permet de conclure en disant que du moment où $\left(\tau_i = 1\right)$ pour tout $i \in \left[0;9\right]$, alors $\left(T_{\beta i} \times 2^i\right) \gt \sum_i^0 \left(T_{\alpha i} \times 2^i\right)$.
Cependant, comme dans le cas des champs d'exposant, cela ne démontre pas pour autant que $\left(T_{\alpha} \lt T_{\beta}\right)$.

## La considération des zéros anonymes capitaux



### Un échec certain de la condition 

// à vérif (il faut delete je crois)

Dans le chapitre "_La mantisse tronquée, une histoire de puissance de 2_", qui introduit à l'encodage du champs de mantisse tronquée.
Il est dit que la valeur d'un bit à $1$ de poids $i$ d'un champs de mantisse tronquée $T$, est _inconditionnellement_ supérieur à la somme de la valeur de chacun des bits de poids inférieur à $i$.
Rappellons que c'est aussi le cas des champs d'exposant, comme cela a été mentionné dans le chapitre "_L'encodage par biais du champs d'exposant_".
Comme nous avons pu le voir dans la première partie de la démonstration, la comparaison entre les champs d'exposant utilise ces primitives mathématiques.
Ci-dessous, nous allons voir qu'il en va de même pour la comparaison entre les champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$. 

// à traité

Prenons le cas de $\left(\tau_i = \tau_9 = 1\right)$.
Vu que $\left(\tau_9 = 1\right)$, nous savons que $\left(T_{\beta 9} = 1\right)$ tandis que $\left(T_{\alpha 9} = 0\right)$ et par conséquent $\left(T_{\beta 9} \times 2^9\right) \gt \sum_{i=9}^0 \ \left(T_{\alpha i} \times 2^i\right)$.
Etant donné que $T_{\alpha 9}$ ainsi que $T_{\beta 9}$ sont les _MSB_ respectif des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$, nous pouvons alors conclure que $\left(T_{\alpha} \lt T_{\beta}\right)$. 
Sachant que $\vert \ \alpha \ \vert$ est égale à $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right)$ et que $\vert \ \beta \ \vert$ vaut $\left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$, avec $\left(E_{\alpha} = E_{\beta}\right)$ nous obtenons alors le résultat $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$.
Par conséquent, la vérification de la condition sur laquelle repose la démonstration mathématique du circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ se solde par un échec.

N'oublions pas qu'il peut y avoir des _zéros anonymes_ dans $\tau \in \left[0;8\right]$, mais ces derniers ne peuvent être que de poids inférieur au _MSB1_ de $\tau$, qui est $\tau_9$.
Ces potentiels bit à $0$ sont alors des _zéros anonymes non capitaux_, sans aucune importance pour le résultat.

### Lorsque les zéros anonymes rendent la génération du résultat incertain

Prenons désormais le cas dans lequel le poids du _MSB1_ de $\tau$ est $\tau_i$ pour $i \in \left[0;8\right]$.
Dans cette situation, nous sommes certains qu'il y a _au moins_ le bit $\tau_{\left(i+1\right)}$ qui est d'un poids supérieur à $\tau_i$.
Les bits de poids supérieur à $\tau_i$ sont tous des _zéros anonymes capitaux_, nous devons donc y prêté attention.

Calculons la somme $\varphi$ (phi) de la valeur des bits de poids supérieur à $i$ du champs de mantisse tronquée $T_{\alpha}$ avec $\left(\varphi_{\alpha} = \sum_{\sigma=i+1}^9 \ \left(T_{\alpha\sigma} \times 2^{\sigma}\right)\right)$, puis de $T_{\beta}$ avec $\left(\varphi_{\beta} = \sum_{\sigma=i+1}^9 \ \left(T_{\beta\sigma} \times 2^{\sigma}\right)\right)$.
Nous savons que $\left(\varphi_{\alpha} \ge \varphi_{\beta}\right)$ car les bits dans $\tau$ de poids $\sigma \in \left]i;9\right]$ sont tous des _zéros anonymes capitaux_.
Ce qui veut dire que toute occurence de $T_{\alpha\sigma}$ est supérieur ou égale à $T_{\beta\sigma}$.

Dans le cas où $\left(\varphi_{\alpha} = \varphi_{\beta}\right)$ alors $\left(T_{\alpha} \lt T_{\beta}\right)$ car $\left(\varphi_{\beta} + T_{\beta i} \times 2^i\right) \gt \left(\varphi_{\alpha} + \sum_i^0 \ \left(T_{\alpha i} \times 2^i\right)\right)$.
Etant donné que $\left(E_{\alpha} = E_{\beta}\right)$ alors $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \lt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$, ou autrement dit $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$.
La vérification de la condition sur laquelle se base la démonstration mathématique du circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$, se solde par un échec.

Il est cependant possible que $\left(\varphi_{\alpha} \gt \varphi_{\beta}\right)$, à l'unique condition qu'il existe au moins une occurence de $\left(T_{\alpha\sigma} \gt T_{\beta\sigma}\right)$ pour $\sigma \in \left]i;9\right]$.
Pour chacune de ces occurences $\left(T_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^0 \ \left(T_{\beta\sigma} \times 2^{\sigma}\right)$ et donc $\left(T_{\alpha} \gt T_{\beta}\right)$.
Nous savons que $\left(E_{\alpha} = E_{\beta}\right)$, mais cette fois-ci $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$ et la condition sur laquelle repose la démonstration mathématique du circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ réussie.

Pour finir, il est possible qu'il n'y ait pas de _MSB1_ dans $\tau \in \left[0;9\right]$, tout ces bits sont alors des _zéros anonymes capitaux_.
Nous pouvons donc affirmé que $\left(T_{\alpha} \ge T_{\beta}\right)$.

Si jamais il existe au moins une occurence de $\left(T_{\alpha i} \gt T_{\beta i}\right)$ pour $i \in \left[0;9\right]$, alors $\left(T_{\alpha} \gt T_{\beta}\right)$.
Les champs d'exposant $E$ des opérandes $\alpha$ et $\beta$ sont égaux, par conséquent $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$.
La vérification de la condition sur laquelle se base la démonstration mathématique du circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ réussie.

En outre, $\left(T_{\alpha} = T_{\beta}\right)$ si jamais $\forall \ i \in \left[0;9\right]$ $\left(T_{\alpha i} = T_{\beta i}\right)$.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$.
La valeur de $\vert \ \alpha \ \vert$ vaut $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right)$ et est égale à $\vert \ \beta \ \vert$ qui vaut $\left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$.
La vérification de la condition sur laquelle se repose la démonstration mathématique du circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$, se solde par un échec.

# Conclusion de la démonstration

Comme nous avons pu le voir dans la première partie de la démonstration "_Le traitement des champs d'exposant_", ainsi que dans la seconde partie "_Le traitement des champs de mantisse tronquée_".
Les procédures de traitement des champs d'exposant $E$ et des champs de mantisse tronquée $T$ des opérandes $\alpha$ et $\beta$ sont identiques.
Dans les faits, le circuit électronique n'est composé que d'un seul ensemble de circuit logique, prenant ainsi en charge le traitement de chacun des champs de ses opérandes.
La documentation du circuit électronique aborde le sujet de l'architecture de cet ensemble de circuit.

Je tient aussi à rappellé que tout ce que nous avons vu jusqu'ici, n'est qu'une preuve mathématique du bon fonctionnement de la logique de comparaison du FPU Configuration Unit.
Cependant, les calculs que produit réelement le circuit ne sont pas les même que ceux qui ont été utilisés dans la démonstration ci-dessus.
Par exemple, le circuit électronique ne contient pas d'unité additionneur flottante pour faire des calculs de somme.
La raison à cela est qu'il est plus optimal de faire usage de circuit plus simple, plus rapide, moins énergivore, et nécessitant moins de câblage ainsi que de transistor pour leur fabrication.
Par ailleurs, le FPU Configuartion Unit génère deux bits de sortie dont l'un directement par le biais de son comparateur.
Dû à cela, le circuit comparateur doit suivre une architecture quelque peu spécifique et non n'importe laquelle.
Quant au second bit de sortie, il est pris en charge par une porte logique _Xor_ qui l'aggrège avec un autre bit dont il est trop difficile d'expliqué l'origine ici.
Tout ces sujets sont abordés dans la documentation du circuit électronique.

Tout ça pour dire qu'il y a un réel décalage entre la démonstration mathématique et l'architecture du circuit lui même.

# Annexe: Les différents types nombres du standard IEEE-754

Il me faut précisé que le standard IEEE-754 peut représenté plusieurs type de nombre:
  - Les nombres _normaux_ (ceux dont nous parlons implicitement depuis le début du document)
  - Les nombres ___dénormaux___
  - Les _NaN_ (Not a Number)
  - L'_infini_ positif ou négatif

Chacun de ces types de nombre peut être codé dans n'importe quel format de nombre flottant IEEE-754.
Chaque type de nombre se différencie des autres en utilisant une certaine plage de codage dans le champs d'exposant ainsi que le champs de mantisse tronquée.
Plage de codage qui a été fixé dans le standard IEEE-754.
Le but de cet annexe est d'évoqué le sujet des nombres _dénormaux_, ce que nous ferons après avoir brièvement défini les autres types de nombres.

### Les nombres normaux

Les nombres _normaux_ (ou nombres _normalisés_) utilisent une certaine partie de la plage de codage du champs d'exposant.

N'oublions pas que le champs d'exposant a un encodage quelques peu spéciale, ce qui est le sujet du chapitre "_L'encodage par biais du champs d'exposant_".
En bref, avec l'encodage par biais du champs d'exposant il faut faire la distinction entre le nombre codé dans le champs binaire, et la valeur que représente réelement le champs d'exposant lui même.
Le champs d'exposant $E$ code un nombre en _Binary Unsigned_ et possède un biais dont la valeur se calcul comme $\left(2^{\left(N - 1\right)} - 1\right)$, avec $N$ le nombre de bits qui compose le champs.
Pour obtenir la valeur que représente le champs d'exposant, il faut soustraire le biais du champs au nombre codé dans le champs lui même $\left(E - biais\right)$.

Tout nombre _normalisé_ a un champs d'exposant $E$ dans lequel il peut codé un nombre dont la valeur se situe entre $\left[1;\left(2^N - 1\right)\right[$.
C'est ce que nous appelons _une plage de codage_.
Maintenant que nous connaissons la borne minimal du codage de ce nombre, nous pouvons calculé quel est la plus petite valeur pouvant être représentée par le champs d'exposant avec $\left(1 - \left(2^{\left(N - 1\right)} - 1\right)\right)$.
Nous pouvons aussi faire ceci avec la borne maximal, ce qui donne $\left(\left(2^N - 2\right) - \left(2^{\left(N - 1\right)} - 1\right)\right)$.
Il n'y a pas de plage de codage restreinte pour le champs de mantisse tronquée, les nombres _normaux_ sont identifiés par la seule valeur du champs d'exposant.
Quant au bit de signe, il remplit simplement son rôle en indiquant si le nombre est positif ou négatif (pas de contrainte non plus).

Un nombre est dit "_normalisé_" lorsque dans son codage, le champs de mantisse tronquée est contraint de la même manière que peut l'être un significande de la notation scientifique binaire.
Autrement dit, un nombre est _normalisé_ quand la valeur de son champs de mantisse tronquée oscille entre $\left[1;2\right[$, même si dans les faits ce n'est pas tout à fait le cas.
Le codage d'un nombre _normalisé_ sollicte la notation scientifique binaire de ce nombre.
Le signe $\pm$ de la notation scientifique est convertit en le bit de signe, tandis que le champs d'exposant code la puissance auquel la base $2$ du multiplicande est élevée (dans la mesure du possible car le champs d'exposant doit rappelons-le respecté une plage de codage).
Enfin, le significande binaire est recopié dans le champs de mantisse tronquée.
Un significande est un nombre réel compris dans l'intervalle $\left[1;2\right[$, _sa partie entière est donc toujours composée d'un seul et unique bit à_ $1$.
Pour gagner en précision sur le codage d'un significande, le champs de mantisse tronquée d'un nombre _normalisé_ ne code pas le bit de la partie entière (qui est toujours à $1$), ce bit devient dès lors implicite.
En pratique, le champs de mantisse tronquée $T$ d'un nombre _normalisé_ a une valeur comprise dans l'intervalle $\left[0;1\right[$.
Dans les faits, sa valeur doit être interprétée comme $\left(1 + T\right)$, où le terme $1$ est la valeur du bit rendu implicite.

Comme nous pouvons le remarqué, tout ceci fait échos aux notions des chapitres précédents.

### Les nombres NaN

Pour le standard IEEE-754, il existe des nombres invalides du nom de ___NaN___ (_Not a Number_).
Un nombre _NaN_ est généré par un calcul considéré comme invalide par le standard lui même et/ou par les mathématiques, c'est parfois une sorte d'alternative à la génération d'exception matériel.
Par exemple, le calcul $\left(\left(+ \ \infty\right) \div 0\right)$ génère un _NaN_ pour deux raisons.
Premièrement, divisé par $X$ l'infini positif ne donne pas de résultat concret, de plus si $\left(X = 0\right)$ alors le calcul se retrouve soudainement à être invalide mathématiquement parlant. 
Comme autre exemple, si un calcul se base sur un opérande qui est un _NaN_, alors il génère également _NaN_ en résultat $\left(NaN - 3.5\right) = NaN$.

En bref, le codage d'un _NaN_ recquière que le champs d'exposant code la valeur $\left(2^N - 1\right)$, pour $N$ le nombre de bits qui compose le champs.
Parallèlement, le champs de mantisse tronquée doit être non nul.
Aussi, comme cela a été mentionné ci-dessus, un nombre _NaN_ utilisé en tant qu'opérande d'un calcul arithmétique ne génère jamais de résultat valide.
Donc la FPU Configuration Unit ne prend pas en charge les nombres _NaN_.

### L'infini positif et négatif

Aussi, il se trouve que le standard IEEE-754 définit un moyen de codé un nombre infini positif comme négatif $\left(\pm \ \infty\right)$.
Premièrement, pour le codage de l'infini $\left(\infty\right)$, le champs d'exposant doit codé la valeur est $\left(2^N - 1\right)$, avec $N$ le nombre de bits qui compose le champs.
De plus, il faut que le champs de mantisse tronquée soit nul (composé uniquement de bits à $0$), au contraire d'un nombre _NaN_.
Ce qui permet de différencié efficassement un nombre $\left(\pm \ \infty\right)$, d'un nombre _NaN_.
Ajoutons à cela le bit de signe qui code le signe $\left(\pm\right)$.

Comme nous avons pu le voir avec l'un des exemples de la section précédente "_les nombres NaN_", un calcul arithmétique ayant pour opérande l'infini positif ou négatif génère systèmatiquement un _NaN_ en résultat.
La FPU Configuration Unit ne prend pas non plus en charge les opérandes de valeur infini positif comme négatif.

## La plage de codage des nombres dénormaux

Les nombres _dénormaux_ représentent des nombres concrets, au contraire de l'_infini_ positif/négatif ou encore des nombres _NaN_ que nous avons vu ci-dessus.

Les nombres _dénormaux_ permettent de représenté des nombres très proche de $0$, qui plus est, plus proche de $0$ que ne serait capable de le faire une représentation _normalisé_.
Par conséquent, il existe deux plages de codage des nombres _dénormaux_.
La plage positive qui code des nombres supérieurs à zéro, ainsi que la plage négative des nombres inférieurs à zéro.
La plage de codage des _dénormaux_ positifs se situe entre $0$, et le __plus petit__ nombre positif _normalisé_ pouvant être codé sur n'importe quel format de flottant IEEE-754.
Quant à la plage de codage des _dénormaux_ négatifs, elle va de $0$ au __plus grand__ nombre négatif _normalisé_ pouvant être codé sur ce même format de flottant IEEE-754.

La seule différence dans le codage d'un nombre _dénormalisé_ positif, vis à vis de son opposé (négatif), est la valeur du bit de signe.
Au delà du bit de signe, les plages de codage des nombres _dénormaux_ positif et négatif sont les même.
C'est pourquoi nous ne parlerons ci-bas que des nombres _dénormaux_ positif, cela rendra moins verbeux et complexe les explications suivantes.

Pour commencer, nous allons devoir définir le codage de la borne basse (c'est à dire $0$), ainsi que la borne haute (le plus petit nombre positif _normalisé_) de la plage de codage des _dénormaux_ positifs.
Evidemment, les plages de codage des nombres _dénormaux_ change en fonction du format de flottant IEEE-754.
Donc, les explications ci-dessous ne ciblerons que le format _Half Precision_.

### Le codage du plus petit nombre positif normalisé au format Half Precision (borne haute)

Rappelons que le champs d'exposant $E$ représente une valeur de $\left(E - biais\right)$.
La plage de codage du champs d'exposant d'un nombre _normalisé_ est de $\left[1;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits qui compose le champs.
Quant au biais du champs d'exposant, il se calcul de la manière suivante $\left(2^{\left(N - 1\right)} - 1\right)$.
Le champs d'exposant d'un nombre flottant au format _Half Precision_ est composé de cinq bits $\left(N = 5\right)$.
Par conséquent, la plus petite puissance pouvant être représentée par un tel champs d'exposant est $-14$, ou autrement dit $\left(1 - 15\right)$.
Le plus petit nombre positif _normalisé_ code alors la valeur $1$ sur son champs d'exposant.

Rappelons également que le champs de mantisse tronquée $T$ d'un nombre _normalisé_ a une valeur effective de $\left(1 + T\right)$.
Etant donné que $T \in \left[0;1\right[$, le plus petit nombre positif _normalisé_ a donc un champs de mantisse tronquée nul (composé de dix bits à $0$).

Voici à quoi ressemble le codage du plus petit nombre positif et _normalisé_ pouvant être codé sur un flottant au format _Half Precision_:

$$\left(1\right) \quad \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

### Le codage du zéro positif ainsi que négatif au format Half Precision (borne basse)

Il s'avère que la norme IEEE-754 supporte un zéro positif $\left(+0\right)$ ainsi que négatif $\left(-0\right)$, en fonction de la valeur du bit de signe.
Ceci engendre quelques diffculté pour les comparaisons.
Par exemple, quel résultat générer pour la comparaison suivante $\left(\left(+0\right) = \left(-0\right)\right)$?

En revanche, le circuit n'est pas affecté par cela car rappellons-le, il n'utilise que la valeur absolu de ses opérandes flottants.
Par conséquent, la FPU Configuration Unit est donc capable de prendre en charge des opérandes nuls.
Quant au codage d'un zéro positif comme négatif $\left(\pm 0\right)$, il est nécessaire que le champs d'exposant ainsi que le champs de mantisse tronquée soient nuls.
Le bit de signe est pour sa part fautif du fait de l'existence d'un zéro positif comme négatif, même si ce dernier n'est pas pris en compte par le circuit.

Voici l'illustration du codage d'un zéro positif ou négatif au format _Half Precision_:

$$\left(2\right) \quad \left[S_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

Avec $\left(S = 0\right)$ pour un zéro positif, et $\left(S = 1\right)$ pour un zéro négatif.

## Le codage des nombres dénormaux

Comme dit dans plus haut "_Les nombres dénormaux permettent de représenté des nombres très proche de_ $0$ _, qui plus est, plus proche de_ $0$ _que ne serait capable de le faire une représentation normalisé_".
Ceci est dû a deux différences majeur entre les nombres _normaux_ et _dénormaux_:
  - Le bit implicite du champs de mantisse tronquée
  - L'interprétation que l'on fait de la valeur codé par le champs d'exposant

Nous allons voir que d'une part, le codage du champs d'exposant d'un nombre _dénormalisé_ est à la source d'une interprétation assez spécifique de la valeur qu'il code.
Par ailleurs, nous verrons aussi que la valeur effective du champs de mantisse tronquée des nombres _dénormaux_, est différente de celle des nombres _normaux, sans pour autant que ce soit dû à l'encodage de ce champs.

Avec la section suivante, commençons par parler du codage du champs d'exposant d'un nombre _dénormalisé_, ainsi que de son interprétation.

### Codage et interprétation du champs d'exposant des nombres dénormaux

Rappelons que la plage de codage du champs d'exposant des nombres _normaux_ est de $\left[1;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits du champs.
Le codage d'un nombre _dénormalisé_ se démarque de celui d'un nombre _normalisé_, par le fait que le champs d'exposant doit être obligatoirement nul.
C'est cela qui entraine une interprétation inédite de la valeur que représente le champs d'exposant.

La valeur que représente le champs d'exposant $E$ d'un nombre _normalisé_ se calcul comme suit $\left(E - biais\right)$.
En outre, la valeur qu'interprète le champs d'exposant d'un nombre _dénormalisé_ est la plus petite puissance dont peut faire usage un nombre _normalisé_.
Autrement dit, le champs d'exposant des nombres _dénormaux_ représente la valeur $\left(1 - biais\right)$.
Nous verrons plus bas dans le chapitre du nom de "_La continuité de représentation entre les nombres normaux et dénormaux_", qu'il existe une continuité de codage assez fondamentale entre les nombres _normaux_ et _dénormaux_.
Nous devons cette continuité de codage aux interprétations distinctes que l'ont fait des champs d'exposant des nombres _normaux_ et _dénormaux_.

### Le bit implicite du champs de mantisse tronquée des nombres dénormaux

Comme nous avons pu le voir dans les chapitres précédents, plus particulièrement dans la section "_Les nombres normaux_", le champs de mantisse tronquée $T$ d'un nombre _normalisé_ a une valeur effective de $\left(1 + T\right)$.
Le terme $1$ représente la valeur du bit implicite du champs.
Dans cette section, il est dit je cite "_Le codage d'un nombre normalisé sollicte la notation scientifique binaire de ce nombre._", ce qui n'est absolument pas le cas des nombres _dénormaux_.
Le bit implicite à $1$ du champs de mantisse tronquée des nombres _normaux_, est le bit de la partie entière du significande de la notation scientifique binaire sur laquelle repose le codage des nombres _normaux_.

Rappelons que l'objectif des nombres _dénormaux_ est de pouvoir codé des valeurs plus proche de $0$ que ne sont capable de le faire les nombres _normaux_.
Pour ce faire, le codage d'un nombre _dénormalisé_ ne se repose pas sur la notation scientifique binaire.
Le champs de mantisse tronquée a toujours un bit implicite qui représente la partie entière du nombre interprété par le champs, mais ce dernier est toujours à $0$.
Par conséquent, la valeur effective du champs de mantisse tronquée $T$ d'un nombre _dénormalisé_ est de $\left(0 + T\right)$, et ce champs code une valeur systèmatiquement comprise dans l'intervalle suivante $\left]0;1\right[$.

Plus haut nous avons vu que les nombres _dénormaux_ ont un champs d'exposant nul, permettant ainsi de dissocier le codage de ces derniers vis à vis de celui des nombres _normaux_.
Mais rappelons que dans la section "_Le codage du zéro positif ainsi que négatif au format Half Precision_", nous formalisons le codage des valeurs $\pm \ 0$ qui nécessitent un champs d'exposant et de mantisse tronquée nul.
D'où la nécessité que le champs de mantisse tronquée $T$ des nombres _dénormaux_ soit non nul.
Ceci afin de ne pas confondre le codage d'un nombre _dénormalisé_ avec celui d'un zéro positif ou négatif.

Au terme de ces explications, il est probable que vous ayez une question en tête.
Pourquoi les nombres _dénormaux_ sont capables de codé des nombres plus proche de $0$ que les nombres _normaux_?
En quelque mots, c'est grâce au bit implicite à $0$ du champs de mantisse tronquée des _dénormaux_.
La section suivante démontre formellement ceci. 

### La précision de codage des nombres dénormaux 

Dans une représentation "classique" des nombres _normaux_, le plus petit nombre positif et _normalisé_ qui peut être codé au format _Half Precision_ est $\left(\left(1 + 0.0\right) \times 2^{\left(1 - 15\right)}\right)$ soit $2^{-14}$.
Tout droit tiré de la section "_Le codage du plus petit nombre positif normalisé au format Half Precision_", voici l'illustration du codage du nombre sous-jacent:

$$\left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

Je vais vous demandez un court instant d'imaginer que les nombres _dénormaux_ n'existent pas.
Dans cette hypothèse, la plage de codage du champs d'exposant d'un nombre _normalisé_ pourrait être étendu à $\left[0;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits du champs.
Si nous omettons le conflit que cela poserait avec le codage d'un zéro positif, alors le plus petit nombre positif _normalisé_ serait $\left(\left(1 + 0.0\right) \times 2^{\left(0 - 15\right)}\right)$ soit $2^{-15}$.

Voici l'illustration du codage de cet hypothétique nombre au format _Half Precision_:

$$\left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

Nous voyons clairement que le codage de ce nombre pose un problème car les champs d'exposant et de mantisse tronquée sont tout les deux nuls, à l'instar du codage d'un zéro positif.
Mais tout ceci ne reste qu'une expérience de pensée, alors passons.

Pour conclure, nous pouvons remarqué que $\left(2^{-14} \div 2^{-15}\right) = 2$.
Ce qui veut dire que si les nombres _dénormaux_ n'existaient pas, nous pourrions codé des nombres _normaux_ deux fois plus petit.

Désormais, avec un nombre _dénormalisé_ essayons de codé la valeur $2^{-15}$.
Ce nombre _dénormalisé_ a un champs d'exposant nul, mais conformément à ce qui est dit dans la section "_Codage et interprétation du champs d'exposant des nombres dénormaux_", il faudra interprété la valeur du champs comme étant la plus petite puissance dont peut faire usage un nombre _normalisé_.
C'est à dire $\left(1 - 15\right) = -14$.
Quant au champs de mantisse tronquée, il code la valeur $1000000000_2$. 
Par ailleurs, n'oublions pas que le bit implicite du champs de mantisse tronquée est à $0$, comme l'explique la section "_Le bit implicite du champs de mantisse tronquée des nombres dénormaux_".
Pour finir, la valeur effective du champs de mantisse tronquée est de $\left(0 + 0.1_2\right)$ $=$ $0.5$.

Voici l'illustration du codage du nombre _dénormalisé_ sous-jacent au format _Half Precision_:

$$\left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 1_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

La valeur du nombre est de $\left(\left(0 + 0.1_2\right) \times 2^{\left(1 - 15\right)}\right)$, ou autrement dit $\left(0.5 \times 2^{-14}\right)$ ce qui est égale à $2^{-15}$.
Il est donc possible d'obtenir le même gain de précision avec les nombres _dénormaux_, qu'avec une extension de la plage de codage du champs d'exposant des nombres _normaux_, et même plus!
Dans les faits, pour le plus petit nombre positif _dénormalisé_ au format _Half Precision_, le gain de précision est de $1024$.
Autrement dit, le plus petit nombre positif et _dénormalisé_ au format _Half Precision_ $\left(\left(0 + 0.0000000001\right) \times 2^{-14}\right)$, est $1024$ fois inférieur au plus petit nombre positif _normalisé_ du même format IEEE-754.

Les nombres _dénormaux_ permettent ainsi d'avoir une bien meilleur précision sur le codage des nombres proches de $0$.

## La continuité de représentation entre les nombres normaux et dénormaux

Faisons un très bref rappel de ce que sont les nombres _dénormaux_.
Concrétement, les _dénormaux_ codent des nombres non nul très proche de $0$.
Au format _Half Precision_, les nombres _dénormaux_ positifs sont censés couvrir tout les nombres dont la valeur se situe entre $0$ et le plus petit nombre positif _normalisé_.
Dans les limites de la base binaire, bien évidemment.
Je dit bien censé, car cela n'est vrai que si il y a continuité de codage entre les nombres _normaux_ et _dénormaux_.

Les nombres _normaux_ sont tous consécutifs les uns aux autres (directement, ou indirectement), nous pouvons donc dire qu'il y a continuité dans la représentation des nombres _normaux_.
Il en va de même pour les nombres _dénormaux_ entre eux.
Cependant, ce que nous cherchons à savoir c'est si il y a continuité entre les nombres _normaux_ et _dénormaux_.
Rappelez-vous de ce que dit la section "_Codage et interprétation du champs d'exposant des nombres dénormaux_", au propos de la continuité entre les nombres _normaux_ ainsi que _dénormaux_.
Je cite "_Nous devons cette continuité de codage aux interprétations distinctes que l'ont fait des champs d'exposant des nombres normaux et dénormaux._".

La section suivante va dans un premier temps nous démontrer les problèmes que pose une représentation non continu entre les nombres _normaux_ et _dénormaux_.
Enfin, la dernière section de cet annexe formalisera ce qu'est la continuité de représentation entre les nombres _normaux_ et _dénormaux_.

### Représentation non continu des nombres dénormaux

Commençons par voir les problèmes auquels nous ferions face si le standard IEEE-754 interprétait la valeur du champs d'exposant $E$ d'un nombre _dénormalisé_, comme celle d'un nombre _normalisé_.
C'est à dire, par la soustraction du biais du champs d'exposant à la valeur qu'il code $\left(E - biais\right)$.
Pour cela, nous allons avoir besoin de définir les trois nombres suivants $\alpha$, $\beta$ ainsi que $\tau$.

Nous avons déjà vu l'illustration du plus petit nombre positif _normalisé_ au format _Half Precision_, ce nombre sera connu sous le nom de $\alpha$:

$$\alpha: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

La valeur du nombre _normalisé_ $\alpha$ est de $\left(\left(1 + 0.0\right) \times 2^{\left(1 - 15\right)}\right)$ ce qui donne $0.00006103515625$.

En prime, voici l'illustration du plus grand nombre positif _dénormalisé_ pouvant être codé au format _Half Precision_, ce nombre sera connu sous le nom de $\beta$:

$$\beta: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 1_9, \ 1_8, \ 1_7, \ 1_6, \ 1_5, \ 1_4, \ 1_3, \ 1_2, \ 1_1, \ 1_0\right]$$

Rappelons que nous allons calculons la puissance que représente le champs d'exposant $E$ des nombres _dénormaux_ avec $\left(E - biais\right)$, comme nous le ferions pour n'importe quel nombre _normalisé_.
Par conséquent, la valeur du nombre _dénormalisé_ $\beta$ est de $\left(\left(0 + 0.9990234375\right) \times 2^{\left(0 - 15\right)}\right)$ ce qui donne $0.0000304877758026123046875$.

Enfin, voici l'illustration du plus petit nombre positif _dénormalisé_ pouvant être codé au format _Half Precision_, ce nombre sera connu sous le nom de $\tau$:

$$\tau: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 1_0\right]$$

Par définition, $\tau$ est le plus petit des nombres positif non nul pouvant être codé au format _Half Precision_, c'est le nombre le plus proche de $0$ toute représentation confondu.
Ce nombre, au même titre que $\beta$, est _dénormalisé_.
D'où le fait que nous calculons la puissance que représente le champs d'exposant $E$ de $\tau$ avec $\left(E - biais\right)$, comme nous avons pu le faire pour $\beta$.
La valeur du nombre $\tau$ est de $\left(\left(0 + 0.0009765625\right) \times 2^{\left(0 - 15\right)}\right)$ ce qui donne $0.0000000298023223876953125$.

Pour commencer, remarquons que le ratio suivant $\left(\alpha \div \beta\right)$ a une valeur qui vaut environ $2.0019$.
Voici l'une des phrases d'introduction du chapitre "_La continuité de représentation entre les nombres normaux et dénormaux_", je cite " $\left[..\right]$ _les nombres dénormaux positifs sont censés couvrir tout les nombres dont la valeur se situe entre_ $0$ _et le plus petit nombre positif normalisé._".
Dans les faits, étant donné que le plus grand nombre positif _dénormalisé_ $\beta$ est deux fois inférieur au plus petit nombre positif _normalisé_ $\alpha$, il y a alors une partie des nombres compris dans l'intervalle $\left]0;\alpha\right[$ qui ne sont pas représentable.
Pour cause, tout les nombres qui se situent dans cet intervalle $\left]\beta;\alpha\right[$, sont soit trop grands pour être représenté par des nombres _dénormaux_, soit trop petit pour l'être par des nombres _normaux_.
Parmis tout les nombres de l'intervalle $\left]0;\alpha\right[$, nous pouvons savoir qu'il y a $\left(\beta \div \alpha\right)$ c'est à dire $\approx 49.9$% d'entre eux qui ne peuvent pas être représentés.

Prenons pour exemple le résultat du calcul $\left(\alpha - \tau\right)$, aussi proche que le résultat puisse être de $\alpha$ il n'est pour autant pas représentable.
Nous venons tout juste de voir pourquoi.
Le résultat de ce calcul est $0,0000610053539276123046875$, ce qui est très légèrement inférieur au nombre $\alpha$.
Par conséquent, ce résultat se situe dans l'intervalle $\left]\beta;\alpha\right[$, d'où le fait qu'il ne puisse être ni représenté par un nombre _normalisé_, ni par un nombre _dénormalisé_.
Le résultat de ce calcul fait donc partie d'une sorte de "trou" de valeur non représentable.

Dans les faits, les calculs sur les flottants occasionnent souvent des arrondis, du fait de l'incapacité de toute base numérique à pouvoir représenté certain nombre de manière fini.
Prenez l'exemple de $\left(1 \div 3\right)$ en base décimale, ou encore $\left(1 \div 10\right)$ en binaire pour vous en convaincre.
Le standard IEEE-754 défini des méthodes d'arrondissement que nous ne détaillerons pas dans ce document.
Cependant, le résultat du calcul $\left(\alpha - \tau\right)$ est si proche de la valeur de $\alpha$, qu'il pourrait être possible d'utiliser une méthode d'arrondi par le haut pour obtenir un résultat dont la précision conviendrai à certains cas d'usage.
Malgré tout cela n'est pas toujours possible, ça ne l'est pas pour un nombre comme $0.000045$ par exemple.
Ce nombre est pour ainsi dire aussi lointain du plus grand nombre positif _dénormalisé_ $\beta$, que du plus petit nombre positif _normalisé_ $\alpha$.
N'importe quel méthode d'arrondi transformerait le nombre jusqu'à le rendre trop approximatif pour presque tout les cas d'usage.

Voilà en quoi l'absence de continuité entre la représentation des nombres _normaux_ et _dénormaux_ est un problème sérieux.

### Représentation continu des nombres dénormaux

Nous avons vu dans le chapitre "_Codage et interprétation du champs d'exposant des nombres dénormaux_", comment est ce que le champs d'exposant d'un nombre _dénormalisé_ devait être interprété.
En bref, le champs d'exposant d'un nombre _dénormalisé_ est nul, mais la valeur que représente ce champs est celle de la plus petite puissance dont peut faire usage un nombre _normalisé_.
Le champs d'exposant interprète alors la valeur $\left(1 - biais\right)$.

Dans ce qui suit, nous allons réutiliser les trois nombres $\alpha$, $\beta$ et $\tau$ de la section précédente.
Dans un premier temps, nous allons redéfinir la valeur des nombres _dénormaux_ $\beta$ et $\tau$, avec cette fois-ci une interprétation correcte de leurs champs d'exposant:

  - Le codage de $\beta$ reste le même et une illustration de ce dernier est présent dans la section précédente.
    Quant à sa valeur, le nombre _dénormalisé_ $\beta$ est de $\left(\left(0 + 0.9990234375\right) \times 2^{\left(1 - 15\right)}\right)$ ce qui donne $0.000060975551605224609375$.

  - Il en va de même du codage de $\tau$ qui reste le même, une illustration de ce dernier est aussi présente dans la section précédente.
    Quant à sa valeur, le nombre _dénormalisé_ $\tau$ est de $\left(\left(0 + 0.0009765625\right) \times 2^{\left(1 - 15\right)}\right)$ ce qui donne $0.000000059604644775390625$.

  - Je rappelle simple que le nombre _normalisé_ $\alpha$ ne voit ni son codage ni sa valeur changer, sa valeur est de $0.00006103515625$.

Reprenons l'exemple du calcul $\left(\alpha - \tau\right)$ de la section précédente, seulement, cette fois-ci le résultat du calcul est $\beta$. 
Rappelons-nous du fait que $\tau$ est le plus petit nombre positif _dénormalisé_ pouvant être codé au format _Half Precision_.
Obtenir le plus grand nombre positif _dénormalisé_ $\beta$ en guise de résultat de la soustraction de $\tau$ au plus petit nombre positif _normalisé_ $\alpha$, démontre qu'il y a continuité dans le codage des nombres _normaux_ ainsi que _dénormaux_.

Pour dire les choses autrement.
La valeur effective du champs de mantisse tronquée $T$ des nombres $\alpha$, $\beta$ et $\tau$, est multiplié par $2^{\left(1 - 15\right)}$.
Par conséquent si $\left(\alpha - \tau\right) = \beta$, il faut en conclure que $\left(T_{\alpha} - T_{\tau}\right) = T_{\beta}$ ce qui donne $\left(1.0000000000 - 0.0000000001\right) = 0.1111111111$.
Il devient soudainement plus simple d'observer la continuité de codage entre le plus petit nombre positif _normalisé_ $\alpha$, et le plus grand nombre positif _dénormalisé_ $\beta$.
Au format _Half Precision_, les nombres $\alpha$ et $\beta$ sont donc consécutifs.
Il y a alors continuité de représentation entre les nombres _normaux_ et _dénormaux_, ce qui évite une partie des soucis dont il a été le sujet en fin de section précédente.

Je dit bien une partie seulement, car des méthodes d'arrondis sont toujours nécessaire dans des cas comme la convertion de certains nombres de la base décimale vers la base binaire, ou autre.
Ce qui peut nuire à la précision de représentation des nombres.
Cependant, il n'est pas question de faire quoi que ce soit contre ce manque de précision, car ce dernier est inhérent à la taille du format de flottant IEEE-754 ayant été choisit.
Pour plus de précision, il faut passer a un format de flottant IEEE-754 plus grand.

Enfin, comme nous avons pu le voir durant tout ce document, la FPU Configuration Unit prend en charge les nombres _normaux_.
Mais elle gère aussi les nombres _dénormaux_, car ces derniers peuvent être utilisés dans des soustracteurs flottants que le circuit prend en charge.
Lorsque le comparateur traite deux nombres _dénormaux_, le circuit n'atteint jamais de point terminal car les champs d'exposant des deux opérandes sont nuls. 
D'autre part, le codage des champs de mantisse tronquée des nombres _dénormaux_ suit celui des nombres _normaux_.
Ces derniers subissent alors le même traitement, qui plus est, par l'intermédiaire des même circuits électroniques.

Mais comme nous l'avons vu avec l'exemple du calcul $\left(\alpha - \tau\right)$ de cette section, et de la précédente.
Un soustracteur flottant est tout à fait capable de traité un opérande _normalisé_ ainsi que _dénormalisé_, provenant d'un seul et unique calcul arithmétique.
Cela tombe bien car la FPU Configuration Unit gère la comparaison entre deux nombres d'une part _normalisé_, et de l'autre _dénormalisé_.
Dans une telle situation, le circuit atteint systèmatiquement un état de point terminal.
La raison en est que le champs d'exposant de l'opérande _normalisé_ a une plage de codage qui est rappelons-le de $\left[1;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits du champs.
Tandis que l'opérande _dénormalisé_ a un champs d'exposant nul.
