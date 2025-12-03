Bonjour,

Ce document est une preuve mathématique du fonctionnement d'un circuit électronique du nom de Floating Point Substractor Configuration Unit ou FPS Configuration Unit.
J'ai conceptualisé ce circuit dans le but de résoudre un problème concernant le traitement des nombres flottants dans les unités de calculs flottantes (FPUs) d'un ordinateur.
La micro-architecture du circuit électronique ainsi que les compromis de conceptualisation, le problème que cherche à résoudre le FPS Configuration Unit ou encore la solution qu'il apporte, sont autant de sujets dont je parle dans une documentation dédiée au circuit.

Pour les curieux, sachez que dans ce repo figure un fichier du nom de _16-bit_floating_point_substractor_configuration_unit_ qui schématise le circuit.
Le circuit a été directement schématisé à base de transisor, principalement en technologie CMOS (Complementary Metal Oxide Semi-conductor), ceci permettant de mettre en évidence certains compromis sur le choix des portes logiques.
Disons que l'apparence abrupte du schéma rend hommage à la (relative) complexité algorithmique du Floating Point Substractor Configuration Unit.

Le document va même au-delà de la démonstration mathématique du circuit ; il parle des sujets fondamentaux sur lesquels le circuit et sa démonstration mathématique reposent.
Ce qui explique la présence de la table des matières suivante :
  - __Les encodages__
    - L'encodage Binary Unsigned
      - Une propriété fondamentale à la démonstration
    - Les nombres à virgule flottante
      - L'étymologie du terme de nombre à virgule flottante
  - __L'écriture scientifique binaire__
    - Le significande
      - La transformation d'un nombre à virgule flottante en un significande
      - Détail sur le fonctionnement de l'équation
    - Le multiplicande
      - Lorsque le multiplicande multiplie la valeur du significande
      - Lorsque le multiplicande divise la valeur du significande
      - Un autre regard sur le multiplicande
  - __Le standard IEEE-754__
    - Le champ de mantisse tronquée
      - L'encodage du champ de mantisse tronquée
      - Le codage du champ de mantisse tronquée
    - Le champ d'exposant
      - L'encodage par biais du champ d'exposant
      - Le codage du champ d'exposant
  - __Introduction au Floating Point Substractor Configuration Unit__
    - Les points terminaux et non terminaux
  - __Démonstration mathématique__
    - Définition de quelques opérations fondamentales à la démonstration
  - __Le traitement des champs d'exposant__
    - Les zéros anonymes
      - Les zéros anonymes non capitaux dus au traitement des champs d'exposant
      - Les zéros anonymes capitaux dus au traitement des champs d'exposant
    - L'obtention d'un point non terminal
  - __Le traitement des champs de mantisse tronquée__
    - Recontextualisation des zéros anonymes capitaux ainsi que non capitaux dus au traitement des champs de mantisse tronquée
      - Les zéros anonymes non capitaux dus au traitement des champs de mantisse tronquée
      - Les zéros anonymes capitaux dus au traitement des champs de mantisse tronquée
  - __Conclusion de la démonstration__

...

# Les encodages

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$, pour la culture, je précise que le terme _bit_ vient de la contraction de _binary digit_.
Un champ binaire est quant à lui composé d'un ensemble de bit, c'est-à-dire d'un ensemble de $0$ et de $1$.
Ce sont ces champs binaires qui nous permettent de représenter des nombres dans un ordinateur, par extension de faire des calculs ainsi que de mémoriser des résultats.

Il y a cependant de nombreuses façons de représenter des nombres dans un champ binaire, nous parlons alors de l'_encodage_ des nombres.
Tous les encodages ne se valent pas car il y en a des plus adaptés/efficaces que d'autres en fonction des besoins ; prenons un exemple : pour un champ de $16$ bits, nous pourrions calculer la somme des bits pour représenter un nombre.
Avec cet encodage nous ne pourrions coder que des valeurs entre $0$ et $16$ inclus, ce qui n'est pas optimal pour bien des situations.
Néanmoins, parmi les multiples encodages existants, le plus connu de tous porte le nom de __Binary Unsigned__.

## L'encodage Binary Unsigned

Le __Binary Unsigned__ est un encodage permettant de coder des nombres entiers naturels par le biais de la notation positionnelle, exactement comme le fait la base décimale pour représenter des nombres.
Par exemple, en décimale chacun des chiffres qui compose la valeur $103$ est le facteur d'une puissance de $10_{10}$, la représentation du nombre sous-jacent se calcule comme la somme de ces produits.
Veuillez noter que la syntaxe suivante $X_{10}$ spécifie la base numérique dans laquelle le nombre $X$ est écrit, $10$ pour la base décimale et $2$ pour la base binaire.
Cette syntaxe ne sera utilisée que lorsqu'il y aura une ambiguïté dans l'interprétation d'un nombre composé uniquement de $0$ et de $1$.

$$103 \ = \left(1 \times 10_{10}^{\quad 2} + 0 \times 10_{10}^{\quad 1} + 3 \times 10_{10}^{\quad 0}\right)$$

Le chiffre des unités $3$ est facteur de $10_{10}$ à la puissance $0$ ; le chiffre des dizaines $0$ est facteur de $10_{10}$ à la puissance $1$, tandis que le chiffre des centaines $1$ est le facteur de $10_{10}$ à la puissance $2$.
Remarquez que le chiffre des unités (qui occupe une certaine position dans le nombre) est facteur d'une puissance de $10_{10}$ qui est dix fois inférieur à celle dont le chiffre des dizaines (le chiffre de la position suivante) est le facteur, ce qui se généralise à l'ensemble des chiffres d'un nombre écrit en base décimale.

La base binaire (ou base $2$) permet également de représenter des nombres d'une façon équivalente, mais comme mentionné plus haut, qui dit représentation de nombre dit encodage de nombre.
Vous l'aurez compris, l'encodage permettant de représenter des nombres de la sorte est le __Binary Unsigned__.
Chaque bit d'un nombre codé en __Binary Unsigned__ est le facteur d'une puissance de $2$ positive ou nul ; le nombre ainsi représenté se calcule par le biais de la somme de ces produits.
En outre, les nombres codés n'ont pas de signe $\pm$ représenté par un bit, d'où le fait que cet encodage ne permette que le codage d'entiers naturels. 

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

### Une propriété fondamentale à la démonstration

Admettons qu'un champ $F$ code un nombre non nul en __Binary Unsigned__, cet encodage nous permet alors de savoir que tout bit à $1$ de poids $i$ dudit champ a une valeur $\left(1 \times 2^i\right)$ qui est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Partons du principe que le champ $F$ code le nombre $103$, c'est-à-dire $1100111_2$, alors nous savons que la valeur du bit de poids $5$, étant de $\left(1 \times 2^5\right)$, est strictement supérieur à la somme des valeurs des bits de poids $0, 1, 2, 3$ et $4$ ; indépendemment de la valeur de ces bits là.

Autrement dit, de manière plus générale, nous trouvons que pour tout bit à $1$ de poids $\left(i \gt 0\right)$ du champ $F$ :

$$\left(1 \times 2^i\right) \gt \sum_{\sigma = i - 1}^0 \ \left(F_{\sigma} \times 2^{\sigma}\right)$$

Premièrement, je précise que la syntaxe $F_{\sigma}$ permet d'indexer le bit de poids $\sigma$ du champ $F$, et deuxièmement le fait que ce dont parle cette section n'est pas propre à la base binaire.

## L'encodage à virgule flottante

Il existe en informatique de multiples encodages pour représenter des nombres entiers naturels (nous venons d'en voir un), et il en va de même pour les nombres à virgule ; la virgule fixe, la virgule flottante ou encore les formats définis par le standard IEEE-754 sont, à ma connaissance, les principaux encodages de nombres à virgule.

A ce jour, en 2025, les usages de l'encodage en virgule fixe se font rares, il s'avère que ce dernier est connu pour être utilisé dans les systèmes informatiques bancaires.
L'encodage en virgule flottante est, quant à lui, une version moins restrictive de l'encodage en virgule fixe, permettant ainsi aux champs d'avoir une plage de codage plus étendue qu'avec un encodage à virgule fixe.
En outre, la virgule flottante est utilisée par la notation scientifique binaire, dont nous parlerons plus tard.
Pour finir, le standard IEEE-754 définit des formats normalisés de nombres à virgule, ceux-ci ayant pour but d'unifier la représentation des nombres à virgule dans les ordinateurs.
C'est un sujet absolument fondamental à ce document que nous aborderons plus tard également, pour le moment, intéressons-nous à l'encodage des nombres à virgule flottante.

Pour commencer, je ne vous apprends rien en disant qu'un nombre à virgule écrit en base décimale est scindé en deux parties : la partie entière composée des chiffres devant la virgule, et la partie décimale composée des chiffres derrière la virgule.
Nous savons d'ores et déjà que la partie entière d'un tel nombre utilise la notation positionnelle, ce qui dans les faits est aussi le cas de la partie décimale ; chaque chiffre de la partie décimale est facteur d'une puissance de $10_{10}$ négative, la somme de ces produits résulte en la valeur de la partie décimale elle-même.
Voici une illustration de la façon dont se calcule la valeur de la partie décimale du nombre $+ 103.375$ :

$$0.375 \ = \left(3 \times 10_{10}^{\quad -1} + 7 \times 10_{10}^{\quad -2} + 5 \times 10_{10}^{\quad -3}\right)$$

Le chiffre des dizièmes $3$ est facteur de $10_{10}$ à la puissance $-1$ ; le chiffre des centièmes $7$ est facteur de $10_{10}$ à la puissance $-2$, tandis que le chiffre des millièmes $5$ est le facteur de $10_{10}$ à la puissance $-3$.
Remarquez que le chiffre des dizièmes (qui occupe une certaine position dans le nombre) est facteur d'une puissance de $10_{10}$ qui est dix fois supérieur à celle dont le chiffre des centièmes (le chiffre de la position précédente) est le facteur.
Cependant, d'une part cela se généralise à l'ensemble des chiffres composant la partie décimale d'un nombre écrit en base $10_{10}$, et d'autre part je dois préciser que tout ceci n'est pas propre à la base décimale elle-même.

En effet, l'encodage en virgule flottante permet de coder des nombres à virgule en base binaire d'une façon équivalente à ce qui se fait en base décimale, modulo quelques changements que l'on doit au passage à la base binaire.
Dans un premier temps, notons que nous ne parlons plus de la partie décimale mais désormais de la partie fractionnaire d'un nombre, car les nombres ne sont plus composés de chiffres décimaux ; qui plus est, il n'existe pas vraiment d'autres termes plus adaptés que _partie fractionnaire_.
Dans un second temps, sachez que l'encodage en virgule flottante utilise l'encodage Binary Unsigned (la notation positionnelle) pour le codage de la partie entière d'un nombre à virgule, je vous invite à relire les chapitres précédent si nécessaire. 
A l'instar de la partie entière d'un nombre à virgule, la notation positionnelle est aussi utilisée pour le codage de la partie fractionnaire, ce qui veut dire que chaque bit de la partie fractionnaire est le facteur d'une puissance de $2$...


En effet, l'encodage en virgule flottante permet de coder des nombres à virgule de façon identique à ce que nous venons de voir, modulo les changements qui sont dus à la base numérique comme la nommenclature ; nous ne parlerons plus de la partie décimale d'un nombre mais de sa partie fractionnaire, base binaire oblige.
L'encodage en virgule flottante utilise le Binary Unsigned (la notation positionnelle) pour le codage de la partie entière d'un nombre à virgule, mais aussi pour le codage de la partie fractionnaire de ce même nombre.

// les bits de poids négatifs sont facteurs des puissances de 2 négatives ...

//

Je n'apprends rien à personne en disant qu'un nombre à virgule est scindé en deux parties, la partie entière composée des chiffres devant la virgule, et la partie fractionnaire composée des chiffres derrière la virgule.
La partie entière d'un nombre à virgule flottante est écrite en Binary Unsigned ; par conséquent, pour pouvoir représenter des nombres à virgule flottante positif ou négatif il faut rajouté un signe $\pm$ à la partie entière du nombre.
Ce qui explique l'ajout d'un bit de poids supérieur au MSB de la partie entière d'un nombre flottant, qui interprète le signe $-$ quand il est à $1$ ou $+$ lorsqu'il vaut $0$.
Cependant, l'encodage de la partie fractionnaire est tout nouveau, malgré une ressemblance avec le Binary Unsigned.
Chaque bit de la partie fractionnaire est le facteur d'une puissance de $2$ négative ; la partie fractionnaire a comme valeur la somme de ces produits.
Il sera plus simple de comprendre tout cela au travers d'une illustration, regardez ci-bas.

//

$$+ \ 103.3125 = 0 \ 1100111.0101_2$$

Dans la partie entière du nombre nous retrouvons le bit de poids le plus fort qui est à $0$ (ce dernier représente le signe $+$), ainsi que le codage Binary Unsigned du nombre $103$, qui commence à nous être familier.
Rien d'étonnant pour la partie fractionnaire, nous avons des bits... mais regardons dans le détails ce que valent ces bits.

$$ 0.0101_2 = Integer \ Part\left(0 \times 2^0\right) + Fractional \ Part\left(0 \times 2^{-1} + 1 \times 2^{-2} + 0 \times 2^{-3} + 1 \times 2^{-4}\right)$$

Nous remarquons que le bit de poids le plus fort de la partie fractionnaire est celui de poids $-1$, et que la valeur des puissances de $2$ négatives vont décroissant en fonction de la position du bit correspondant dans le champs.

Dans la section précédente nous parlions d'une propriété de l'encodage Binary Unsigned.
Cette propriété peut être mise en oeuvre dans la partie entière d'un nombre à virgule flottante, car rappelons-le, cette partie est codée en Binary Unsigned.
En outre, il s'avère que l'encodage de la partie fractionnaire permet lui aussi de faire usage de cette propriété.
Pour rappel, cela veut dire que tout bit à $1$ de poids $i$ compris dans la partie fractionnaire a une valeur $\left(1 \times 2^i\right)$ strictement supérieure à la somme des valeurs des bits de poids inférieur à $i$.
Ce qui insinue qu'il y a une continuité de cette propriété entre les bits de la partie entière et fractionnaire d'un nombre à virgule flottante.
Autrement dit, un bit à $1$ de poids $i$ dans la partie entière a une valeur strictement supérieure à la somme des valeurs des bits de poids inférieur à $i$ compris dans la partie entière et fractionnaire.

### L'étymologie du terme de nombre à virgule flottante

Ces nombres sont dits à virgule flottante parce que la virgule du nombre peut être déplacée sans que cela ne pose de problème.
C'est la solution qui à ce jour, pour l'Homme, est la plus intuitive ainsi que la plus flexible pour l'écriture des nombres à virgule en binaire.
Il n'y a pas de contrainte spécifique sur la virgule d'un tel nombre, au contraire de l'écriture des nombres en virgule fixe.
Les nombres à virgule fixe ont une quantité fixe de bits qui composent la partie entière ainsi que fractionnaire du nombre, ce qui insinue que la virgule ne peut pas être déplacée.
Par conséquent, ces nombres sont assez simple à représenter dans un ordinateur mais sont inutilement restrictifs pour l'Homme.
De surcroît, la précision de codage d'un tel nombre est pré-fixée par la quantité de bits composant la partie fractionnaire, ceci est un frein à beaucoup de cas d'usage mais pas pour les systèmes informatiques des banques.
En effet, il est tout à fait plausible que la précision d'affichage du solde d'un compte courant soit de l'ordre du centième de centime près, mais qu'il n'y ait pas besoin de plus de précision.
L'avantage des nombres à virgule fixe est que la précision de codage reste donc constante, et ce, indépendamment de la valeur du solde ; ce qui ne serait pas le cas si les soldes étaient codés en virgule flottante.
Votre seul ennui est alors d'avoir trop d'argent dans votre compte en banque, de sorte à ce que la précision de codage de votre solde devienne limitante et ne permette plus de quantifier votre fortune.

# L'écriture scientifique binaire

Pour commencer, qu'est-ce-que l'__écriture scientifique__?

L'écriture scientifique est une façon comme une autre de représenter des nombres, elle existe pour chaque base numérique comme pour la base binaire.
Le but de la notation scientifique des nombres est double : elle ne permet d'écrire des nombres que d'une seule et unique façon tout en simplifiant la lecture des très grands ainsi que des très petits nombres.
Par exemple, l'écriture scientifique _en base décimale_ rend facilement lisible la lecture de la vitesse approchée de la lumière dans de le vide spatial (en m/s) $+ \ 3.0 \times 10^8$.

Il faut savoir que d'une base numérique à une autre, le fonctionnement de l'écriture scientifique ne change que peu.
Quant à la composition de la notation scientifique, elle ne change jamais ; qu'importe la base numérique, il y a toujours:
  - Un __signe__
  - Un __significande__
  - Un __multiplicande__

Dans l'exemple fourni ci-dessus, le __signe__ est le symbole $\pm$ qui indique si le nombre est positif ou négatif, le __significande__ (aussi appeller __mantisse__) est le terme $3.0$ et enfin le __multiplicande__ est le terme $10^8$.
En notation scientifique décimale, le __significande__ a une valeur comprise dans l'intervalle $\left[1;10\right[$, et le __multiplicande__ est une puissance de $10$.
De manière plus générale, pour une notation scientifique en base $N$, la valeur du __significande__ ne peut être comprise qu'entre $\left[1;N\right[$ et le __multiplicande__ est alors une puissance de $N$.
D'où le fait que d'une base numérique à une autre, le fonctionnement de l'écriture scientifique change légèrement.

Les chapitres suivants se focalisent sur la notation scientifique _en base binaire_.

## Le significande

L'écriture scientifique en base binaire peut représenter n'importe quel nombre $F$ écrit en virgule flottante, qu'il soit positif ou bien négatif.
Le nombre flottant $F$ à représenter en notation scientifique binaire permet à lui seul de définir deux des trois éléments de sa propre écriture scientifique, le __signe__ et le __significande__.
Comme son nom l'indique, l'élément qu'est le "__signe__" ne représente que le signe $\pm$ du nombre flottant $F$.
Tandis que le __significande__ s'obtient depuis une modification (non systématique) de la valeur absolue du nombre à virgule flottante $F$.

Nous savons qu'en notation scientifique binaire, le significande ne peut interpréter que des valeurs comprises dans l'intervalle $\left[1;2\right[$.
Lorsque la valeur du nombre à virgule flottante $\vert \ F \ \vert$ est comprise dans cette intervalle, alors la valeur de $\vert \ F \ \vert$ forme lui même le significande.
Cependant, si ce n'est pas le cas, il faut modifier la valeur du nombre $\vert \ F \ \vert$ de sorte que le résultat soit compris dans l'intervalle $\left[1;2\right[$.
Néanmoins, les modifications de la valeur de $\vert \ F \ \vert$ ne s'effectuent pas n'importe comment.

Une modification consiste en un déplacement de la virgule du nombre flottant $\vert \ F \ \vert$.
Dans le cas où $\left(\vert F \vert \ge \ 2 \ \right)$, la virgule de $\vert \ F \ \vert$ doit être décalée vers la gauche jusqu'à qu'elle se retrouve devant le MSB1 ; et inversement dans le cas où $\left(\vert F \vert \lt \ 1 \ \right)$, alors la virgule de $\vert \ F \ \vert$ doit être décalée vers la droite jusqu'à qu'elle se retrouve devant le MSB1.
En déplaçant la virgule de $\vert \ F \ \vert$ jusque devant le bit de poids le plus fort à $1$, nous nous assurons du fait que le résultat de la modification soit compris dans l'intervalle de valeur $\left[1;2\right[$.

Il faut cependant faire une exception dans le cas où $\left(F = 0\right)$.
Comme la valeur absolu du nombre n'est pas comprise dans l'intervalle de valeur licite d'un significande $\left[1;2\right[$ alors il faudrait déplacer la virgule du nombre jusque devant son MSB1 ; cependant, le nombre n'a pas de MSB1 car son écriture ne contient aucun bit à $1$.
Par conséquent, nous devons faire une entorce à la règle et considérer que le significande peut être excpetionnellement nulle dans cette situation.
Sachez que nous retrouvons la même problèmatique pour toutes les représentations du chiffre $0$ en notation scientifique, qu'importe la base numérique.

La section suivante cherche à préciser l'impact qu'a un déplacement de la virgule d'un nombre flottant, sur sa valeur.

### La transformation d'un nombre à virgule flottante en un significande

Pour tout nombre à virgule flottante $F$, nous savons que la virgule se trouve entre le LSB de la partie entière (le bit de poids $0$), et le MSB de la partie fractionnaire (bit de poids $-1$).
Par conséquent, un déplacement de la virgule d'un rang vers la gauche force le bit de poids $1$ à devenir le bit de poids $0$, le bit de poids $0$ à devenir celui de poids $-1$, et celui de poids $-1$ devient le bit de poids $-2$, etc.
Autrement dit, chaque bit de la partie entière comme de la partie fractionnaire du nombre $F$, voit son poids être _décrémenté_ de $1$.

L'équation ci-dessous permet de calculer la transformation de la valeur de n'importe quel nombre à virgule flottante $\vert \ F \vert \notin \left[1;2\right[$, en un significande.

$$\sum_{i = msb\left(F\right) - 1}^{lsb\left(F\right)} \left(F_i \ \times \ 2^{\left(i + c\right)}\right) = \vert F \vert \times \ 2^c$$

Les fonctions $msb$ et $lsb$ retournent respectivement le poids le plus fort ainsi que le plus faible du nombre passé en argument.
Quant au terme $c$, il représente le nombre de rang de décalage à induire sur la virgule du nombre $\vert \ F \ \vert$.
Qui plus est, lorsque $\left(c \lt 0\right)$, cela signifie que la virgule doit être décalée vers la gauche et inversement quand $\left(c \gt 0\right)$, cela signifie que la virgule doit être décalée vers la droite.

Essayons de comprendre pourquoi est-ce qu'un décalage d'un rang vers la gauche de la virgule de $\vert \ F \ \vert$ $\left(c = -1\right)$ divise la valeur du nombre par $2$ comme le démontre l'équation ci-dessus.
Comme nous l'avons vu précédemment, nous savons qu'avec un décalage d'un rang vers la gauche de la virgule de $F$, je cite "_chaque bit de la partie entière comme de la partie fractionnaire du nombre_ $F$, _voit son poids être décrémenté de_ $1$".
Avant décalage de la virgule du nombre $\vert \ F \ \vert$ , le bit de poids $i$ est facteur de la puissance $2^i$, mais après décalage, ce dernier n'est plus que facteur de la puissance $2^{\left(i - 1\right)}$, la valeur du bit a été divisée par $2$ et ceci se généralise à l'ensemble des bits du nombre.
Rappelons que nous calculons la valeur de $\vert \ F \ \vert$ comme la somme des puissances dont le bit est à $1$ ; chaque puissance ayant été divisée par $2$ dû au décalage de la virgule, la somme de ces dernières l'est également.
C'est pourquoi la valeur du nombre $\vert \ F \ \vert$ est divisée par $2$ après un décalage d'un rang vers la gauche de la virgule.

Maintenant que nous avons compris pourquoi est-ce qu'un décalage d'un rang vers la gauche de la virgule de $\vert \ F \ \vert$ engendre une division par $2$ du nombre, découvrons désormais les effets d'un décalage d'un rang vers la droite de la virgule de $\vert \ F \ \vert$ $\left(c = 1\right)$, sur la valeur du nombre lui-même.
Dans le cas d'un tel décalage, chaque bit de la partie entière comme de la partie fractionnaire du nombre $F$, voit son poids être _incrémenté_ de $1$.
Après décalage de la virgule du nombre flottant $\vert \ F \ \vert$, le bit de poids $i$ passe de facteur de $2^i$ à $2^{\left(i + 1\right)}$, et cela se généralise à l'ensemble des bits du nombre.
Comme la valeur de $\vert \ F \ \vert$ se calcule selon la somme des puissances dont le bit est à $1$, chaque puissance ayant été multipliée par $2$ dû au décalage de la virgule, la somme de ces dernières l'est également été.
C'est ce que démontre le membre de gauche de l'équation pour $\left(c = 1\right)$, alors que le membre droit se contente de donner le résultat attendu.

Désormais, nous sommes en capacité de comprendre toutes les implications qu'un décalage de la virgule du nombre $\vert \ F \ \vert$  peut avoir sur la valeur du nombre lui-même.
Prenons comme exemple un décalage quelconque de la virgule vers la droite $\left(c \gt 0\right)$.
Nous venons tout juste de voir qu'un décalage d'un rang vers la droite de la virgule de $\vert \ F \ \vert$ $\left(c = 1\right)$  multiplie par $2$ la valeur du nombre.
Par conséquent, pour tout autre décalage de la virgule de $c$ rangs vers la droite, il suffit alors de multiplier $c$ fois par $2$ la valeur de $\vert \ F \ \vert$.
Ce qui est équivalent au membre droit de l'équation $\left(\vert F \vert \times \ 2^c \ \right)$.
Voyons maintenant le cas d'un décalage quelconque de la virgule vers la gauche $\left(c \lt 0\right)$.
Nous savons qu'un décalage d'un rang vers la gauche de la virgule de $\vert \ F \ \vert$ $\left(c = -1\right)$  divise par $2$ la valeur du nombre.
Donc dans le cas de tout autre décalage de la virgule vers la gauche, il suffit alors de diviser $\vert \ c \ \vert$ fois la valeur du nombre $\vert \ F \ \vert$ par $2$.
Cela est une nouvelle fois équivalent au membre droit de l'équation $\left(\vert F \vert \times \ 2^c \ \right)$, nous verrons pourquoi dans le chapitre suivant.

Nous savons désormais que n'importe quel déplacement de la virgule d'un nombre flottant engendre une multiplication ou une division de la valeur du nombre par une puissance de $2$.

### Détail sur le fonctionnement de l'équation

Si vous avez déjà compris le fonctionnement de l'équation, vous pouvez vous rendre au chapitre suivant.

L'équation du chapitre précédent semble être parfaitement appropriée pour les décalages vers la droite de la virgule d'un nombre flottant.
En effet, je vous rappelle qu'un déplacement de $c$ rangs vers la droite de la virgule du nombre $\vert \ F \ \vert$ multiplie $c$ fois la valeur du nombre par $2$, avec $\left(c \gt 0\right)$.
Ce qui colle avec le membre droit de l'équation $\left(\vert F \vert \times \ 2^c \ \right)$.

En parallèle, cette même équation utilise des valeurs négatives de $c$ lors des décalages vers la gauche de la virgule d'un nombre flottant.
Pourtant, le plus simple aurait été de définir les mêmes valeurs de $c$, peu importe la direction des décalages, accompagné de deux équations différentes : une équation pour les décalages de la virgule vers la droite (qui est défini ci-dessus), ainsi qu'une autre équation plus appropriée pour les décalages de la virgule vers la gauche.
Cette seconde équation ressemblerait à ce qui suit.

$$\sum_{i = msb\left(F\right) - 1}^{lsb\left(F\right)} \left(F_i \ \times \ 2^{\left(i - c\right)}\right) = \vert F \vert \div \ 2^c$$

Dans cette seconde équation, le terme $c$ représente donc le nombre de rang de décalage vers la gauche à induire sur la virgule du nombre $\vert \ F \ \vert$, et nous savons que $\left(c \gt 0\right)$.
Remarquez qu'un décalage d'un rang vers la gauche de la virgule de $\vert \ F \ \vert$ $\left(c = 1\right)$, divise la valeur du nombre par $2$.
De façon plus générale, pour tout décalage de la virgule de $c$ rangs vers la gauche, la valeur de $\vert \ F \ \vert$ est divisée par $2^c$.
Ce qui colle parfaitement avec le membre droit de la seconde équation $\left(\vert F \vert \div \ 2^c \ \right)$, libre à vous de vérifier que le membre gauche de cette même équation fonctionne bien.

Ce qui suit explique comment il est possible de passer d'un tel raisonnement à deux équations, à un raisonnement plus singulier d'une seule.

Nous allons essayer de réduire la forme de la seconde équation que nous venons tout juste de définir, en la forme de la première équation ayant été définie dans le chapitre précédent.
Remarquons que le membre droit de cette seconde équation $\left(\vert F \vert \div \ 2^c \ \right)$ pourrait être remplacé par $\left(\vert F \vert \times \left(1 \div 2^c \ \right)\right)$ et que $\left(1 \div 2^c\right) = \ 2^{-c}$.
Par définition, il est donc possible de remplacer $\left(\vert F \vert \times \ 2^{-c} \ \right)$ avec $\left(c \gt 0\right)$, par le membre de droite de la première équation $\left(\vert F \vert \times \ 2^c \ \right)$.
Pour cela, il suffit de faire en sorte que tout déplacement vers la gauche de la virgule du nombre $\vert \ F \ \vert$ soit interprété par une valeur négative de $c$, et non positive.
Quant au membre gauche de la première équation, il y figure un calcul $\left(i + c\right)$ qui lorsque $\left(c \lt 0\right)$ est équivalent au calcul $\left(i - c\right)$ avec $\left(c \gt 0\right)$ du membre gauche de la seconde équation.

En bref, le fait que la variable $c$ puisse être positive ainsi que négative permet de ne faire usage que d'une seule équation, qu'importe la direction du décalage de la virgule qui est à faire.

## Le multiplicande

Lorsque le nombre à virgule flottante $F$ à représenter en notation scientifique binaire a une valeur absolue qui n'est pas comprise dans l'intervalle de valeur licite d'un significande $\left[1;2\right[$, la valeur de son propre significande $S$, qui elle est comprise dans cette intervalle, est différente de $\vert \ F \ \vert$.
C'est pourquoi l'écriture scientifique fait usage d'un multiplicande.
Lorsque $\left(S \ \neq \vert F \vert\right)$, le multiplicande $M$ permet comme son nom l'indique de multiplier le significande $S$, de sorte à ce que $\vert \ F \vert = \left(S \times M\right)$.
Autrement dit, le multiplicande réajuste la valeur du significande à la valeur absolue du nombre $F$ à représenter, quand cela est nécessaire.

Cependant, il est possible que le nombre à virgule flottante $F$ ait une valeur absolue comprise dans l'intervalle de valeur licite d'un significande.
Parce que $\left(S \ = \vert F \vert\right)$, nous en déduisons que dans cette situation le multiplicande $M$ vaut $1$, de sorte qu'une nouvelle fois $\vert \ F \vert = \left(S \times M\right)$.

Je tiens à vous faire remarquer que si $\vert \ F \vert = \left(S \times M\right)$, alors la représentation du nombre flottant $F$ en écriture scientifique binaire est $F \ = \pm \left(S \times M\right)$.
Le signe $\pm$ est celui du nombre flottant $F$, ce dont nous parlons au début du chapitre "_Le significande_".
En bref, nous retrouvons les trois éléments de l'écriture scientifique, le __signe__ $\pm$, le __significande__ $S$ ainsi que le __multiplicande__ $M$.

Les sections suivantes expliquent comment le multiplicande réajuste la valeur du significande à celle du nombre $F$ en valeur absolue.
En outre, sachez que ces dernières font référence à l'équation définie dans le chapitre "_La transformation d'un nombre à virgule flottante en un significande_".

### Lorsque le multiplicande multiplie la valeur du significande

En écriture scientifique binaire, la représentation d'un nombre à virgule flottante $F$ dont la valeur absolue est supérieure à l'intervalle $\left[1;2\right[$ d'un significande, s'accompagne d'une transformation du nombre $\vert \ F \ \vert$ en un significande $S$.
La formation du significande passe par un décalage vers la gauche de la virgule du nombre $\vert \ F \ \vert$, ce qui divise par une puissance de $2$ la valeur du nombre.
L'équation nous dit que la formation du significande $S$ passe par $\left(\vert F \vert \times \ 2^c \ \right)$ avec $\left(c \lt 0\right)$, ce qui est équivalent au fait de dire que $S \ = \left(\vert F \vert \div \ 2^{-c} \ \right)$.
Le significande, par le fait que $\left( \ S \ \lt \vert F \vert\right)$, est un terme de l'écriture scientifique qui à lui seul ne peut pas représenter la valeur du nombre flottant $\vert \ F \ \vert$.
C'est pourquoi un multiplicande $M$ multiplie le significande $S$ de sorte que $\vert \ F \vert = \left(S \times M\right)$.
Parce que $S \ = \left(\vert F \vert \div \ 2^{-c} \ \right)$, le multiplicande $M$ multiplie donc le significande $S$ par la même puissance de $2$ que celle ayant divisé $\vert \ F \ \vert$ pour former le significande.
Ce qui donne $\vert \ F \vert = \left(S \ \times \ 2^{-c}\right)$.
Par conséquent, la représentation du nombre flottant $F$ en écriture scientifique binaire est $F \ = \pm \left(S \ \times \ 2^{-c}\right)$, comme nous l'avons vu plus haut.

Voici comme exemple l'écriture scientifique binaire du nombre $F = 5.5$, qui en virgule flottante vaut $+ \ 101.1_2$ :

$$+ \ 1.011_2 \ \times \ 2^2$$

Le significande $1.011_2$ (ou $1.375$ en décimale) résulte du décalage de $2$ rangs vers la gauche de la virgule du nombre flottant $\vert \ F \ \vert$.
La formation du significande engendre une division de la valeur du nombre $\vert \ F \ \vert$, ce que l'équation traduit par $\left(\vert F \vert \times \ 2^c \ \right)$, avec $\left(c = -2\right)$, ou encore $\left(\vert F \vert \div \ 2^{-c} \ \right)$.
Sachant que $\left( \ 1.375 \ \lt \vert 5.5 \vert\right)$, alors le multiplicande $M$ doit multiplier le significande $1.375$ par $2^{-c}$ de sorte que $\vert \ 5.5 \vert = \left(1.375 \ \times \ M\right)$.
L'écriture scientifique du nombre à virgule flottante $F$ est donc bel et bien $5.5 \ = + \ 1.375 \ \times \ 2^2$.

### Lorsque le multiplicande divise la valeur du significande

Voyons désormais le cas contraire, lorsque le multiplicande réduit la valeur du significande.

En écriture scientifique binaire, la représentation d'un nombre à virgule flottante $F$ dont la valeur absolue est inférieure à l'intervalle $\left[1;2\right[$ d'un significande, s'accompagne d'une transformation du nombre $\vert \ F \ \vert$ en un significande $S$.
La formation du significande passe par un décalage vers la droite de la virgule du nombre $\vert \ F \ \vert$, ce qui multiplie par une puissance de $2$ la valeur du nombre.
L'équation nous dit que la formation du significande $S$ passe par $\left(\vert F \vert \times \ 2^c \ \right)$, avec $\left(c \gt 0\right)$.
Le significande, du fait que $\left( \ S \ \gt \vert F \vert\right)$, est un terme de l'écriture scientifique qui à lui seul ne peut pas représenter la valeur du nombre flottant $\vert \ F \ \vert$.
C'est pourquoi un multiplicande $M$ divise le significande $S$ de sorte que $\vert \ F \vert = \left(S \ \div \ M\right)$.
Parce que $S \ = \left(\vert F \vert \times \ 2^c \ \right)$, le multiplicande $M$ divise donc le significande $S$ par la même puissance de $2$ que celle ayant multiplié $\vert \ F \ \vert$ pour former le significande.
Ce qui donne $\vert \ F \vert = \left(S \ \div \ 2^c\right)$, et nous retombons ainsi sur $\vert \ F \vert = \left(S \ \times \ 2^{-c}\right)$.
Par conséquent, la représentation du nombre flottant $F$ en écriture scientifique binaire tombe une nouvelle fois sur $F \ = \pm \left(S \ \times \ 2^{-c}\right)$.

Voici comme exemple l'écriture scientifique binaire du nombre $F = - 0.140625$, qui en virgule flottante vaut $- 0.001001_2$ :

$$- \ 1.001_2 \ \times \ 2^{-3}$$

Le significande $1.001_2$ (ou $1.125$ en décimale) résulte du décalage de $3$ rangs vers la droite de la virgule du nombre flottant $\vert \ F \ \vert$.
La formation du significande engendre une multiplication de la valeur du nombre $\vert \ F \ \vert$, ce que l'équation traduit par $\left(\vert F \vert \times \ 2^c \ \right)$, avec $\left(c = 3\right)$.
Sachant que $\left( \ 1.125 \ \gt \vert - 0.140625 \vert\right)$, alors le multiplicande $M$ doit divisé le significande $1.125$ par $2^c$ de sorte que $\vert - 0.140625 \vert = \left(1.125 \div M\right)$.
Cela donne $\vert - 0.140625 \vert = \left(1.125 \ \div \ 2^3\right)$, ce qui est équivalent à $\vert - 0.140625 \vert = \left(1.125 \ \times \ 2^{-3}\right)$.
L'écriture scientifique du nombre à virgule flottante $F$ est donc bel et bien $- 0.140625 \ = - 1.125 \ \times \ 2^{-3}$.

### Un autre regard sur le multiplicande

Je vous invite à voir les choses d'une manière différente ; ceci nous sera utile pour plus tard.

Nous savons que le significande $S$ de l'écriture scientifique binaire d'un nombre à virgule flottante $F$, s'obtient par un déplacement de la virgule du nombre lorsque la valeur absolue de celui-ci n'est pas comprise dans l'intervalle $\left[1;2\right[$.
Comme l'explique le chapitre "_La transformation d'un nombre à virgule flottante en un significande_" : un décalage de $c$ rangs de la virgule du nombre $\vert \ F \ \vert$ multiplie ou divise la valeur dudit nombre par une puissance de $2$, ce que l'équation du même chapitre traduit par le calcul $\left(\vert F \vert \times \ 2^c \ \right)$.
Je rappelle que le terme $c$ représente le nombre de rang de décalage à induire sur la virgule du nombre $\vert \ F \ \vert$, ce terme est positif pour les décalages vers la droite et négatif pour ceux vers la gauche.
En résumé, le significande $S$ est obtensible par le calcul $\left(\vert F \vert \times \ 2^c \ \right)$ ce qui est équivalent au fait de déplacer la virgule du nombre $\vert \ F \ \vert$ de $log_2 \left(2^c\right)$ rangs.

Rappelons que le rôle du multiplicande $M$ et de réajuster la valeur du significande $S$ à la valeur absolue du nombre $F$ lorsque $\left( \ S \ \neq \vert F \vert\right)$, de sorte à ce que $\vert \ F \vert = \left(S \times M\right)$.
Le multiplicande doit alors permettre de déplacer la virgule du significande $S$ par le même nombre de rangs que ne l'a été celle de $\vert \ F \ \vert$ pour former le significande (c'est à dire de $c$ rangs), mais dans la direction opposée (donc de $-c$ rangs).
Les sections précédentes démontrent que pour ce faire le multiplicande doit alors multiplié le significande par $2^{-c}$, signifiant ainsi que $\vert \ F \vert = \left(S \times 2^{-c}\right)$ ce qui est équivalent au fait de déplacer la virgule du significande $S$ de $\log_2 \left(2^{-c}\right)$ rangs.

En bref, cette section explique que le multiplicande peut se contenté d'appliquer à la virgule du significande, le décalage inverse de ce qu'a subit la virgule du nombre $\vert \ F \ \vert$.
Plus bas, nous verrons en quoi cela peut bien nous être utile.

# Le standard IEEE-754

La norme __IEEE-754__ standardise la représentation (l'encodage) des nombres à virgule dans un ordinateur.
Ce document est une démonstration mathématique du fonctionnement d'un circuit électronique qui a pour but principale de produire des comparaisons entre deux opérandes définis par le standard __IEEE-754__ ; pour plus d'informations sur le circuit électronique, jetez un oeil à la documentation dédiée.
Il existe plusieurs formats de nombre défini par le standard __IEEE-754__, seul la taille change d'un format à un autre.
Tout format défini par la norme est composé des trois éléments suivant :
  - Le __bit de signe__
  - Le champ d'__exposant__
  - Le champ de __mantisse tronquée__

Les deux formats les plus connus sont le __Simple Precision__ ainsi que le __Double Precision__, qui permettent le codage de nombre à virgule sur des champs binaires de, respectivement, $32$ et $64$ bits.
Cependant, le circuit électronique que j'ai conceptualisé ne prend en charge que les opérandes de format __Half Precision__, ceci est principalement dû aux complexités de schématisation du circuit.
Le format __Half Precision__ permet de coder des nombres à virgule sur des champs binaires de $16$ bits.
Voici la disposition de chacun des bits des trois éléments composant le format __Half Precision__ :

$$\left[S_{15}, \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

$\quad S$: Sign bit, $\quad E$: Exponent field bit, $\quad T$: Truncated mantissa field bit

Chaque indice $i$ de la forme $X_i$ qui est compris dans l'intervalle $\left[0;15\right]$, représente le poids d'un bit $X$ dans le champ binaire d'un nombre codé au format __Half Precision__.

Précédemment, nous avons longuement parlé de l'écriture scientifique binaire ; la raison en est que le standard __IEEE-754__ s'appuie sur l'écriture scientifique binaire pour définir les différents formats de nombres à virgule, tel que le __Half Precision__.
Dans le champs binaire d'un nombre $F$ écrit au format __Half Precision__, le __bit de signe__ (bit de poids $15$) est défini en fonction du signe $\pm$ de l'écriture scientifique binaire du nombre $F$.
Quant aux champs d'__exposant__ et de __mantisse tronquée__, ils correspondent respectivement au __multiplicande__ ainsi qu'au __significande__ de l'écriture scientifique binaire de $F$.
Malgré tout, dans les prochains chapitres dédiés aux champs d'__exposant__ et de __mantisse tronquée__, nous verrons que le cas de ces champs est légèrement plus complexe.

Etant donné que le circuit électronique ne prend en charge que des opérandes (nombres) au format __Half Precision__, le reste du document ne fera plus que référence à ce format.

## Le champ de mantisse tronquée

En introduction se trouve une illustration de la disposition de chacun des bits des trois éléments qui composent le format Half Precision ; cette illustration montre que le champs de __mantisse tronquée__ $T$ est composé de $10$ bits.
Voici les poids de chacun des bits qui compose le champ de __mantisse tronquée__ :

$$\left[ \quad .. \quad , \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

Les deux sections suivantes concernent l'ensemble de ces bits formant le champ de __mantisse tronquée__.

### L'encodage du champ de mantisse tronquée

Nous savons que le champ de __mantisse tronquée__ du nombre $F$ codé au format Half Precision correspond au __significande__ de l'écriture scientifique binaire de ce même nombre.
Rappelons que le __significande__ s'obtient en modifiant (non de manière systématique) la valeur absolue du nombre à virgule flottante $F$.
Par conséquent, nous en déduisons que l'encodage du champ de __mantisse tronquée__ est celui d'un nombre à virgule flottante, et vous invite à relire le chapitre "_Les nombres à virgule flottante_" si nécessaire.

Il s'avère que le dernier paragraphe dudit chapitre nous donne accès à une information assez importante.
La valeur de n'importe quel bit à $1$ de poids $i$ dans un nombre à virgule flottante $\left(1 \times 2^i\right)$, est strictement supérieure à la somme des valeurs des bits de poids inférieur à $i$.
Ce qui insinue que n'importe quel bit à $1$ de poids $i$ dans un champ de mantisse tronquée $\left(T_i = 1\right)$, a une valeur $\left(T_i \times 2^i\right)$ qui est elle aussi strictement supérieure à la somme des valeurs des bits de poids inférieur à $i$.
Autrement dit, le champ de __mantisse tronquée__ partage les propriétés des nombres à virgule flottante... mais il n'est pas le seul.

Cette propriété provient de l'encodage Binary Unsigned, et il se trouve que l'encodage du champ d'__exposant__ en est une déclinaison.
Nous verrons cela plus en détail dans le chapitre dédié au champ d'__exposant__, mais il faut savoir que tout cela a un impact.
De par cette propriété commune entre les encodages des champs de __mantisse tronquée__ et d'__exposant__, la logique du circuit se retrouve à traité ces champs de la même manière (avec le même sous ensemble de circuit électronique), ce qui se reflète dans la démonstration mathématique.
Nous en reparlerons d'ici là.

### Le codage du champ de mantisse tronquée

Précédemment, nous avons vu que le champ de __mantisse tronquée__ du nombre $F$ codé au format Half Precision correspondait au __significande__ de l'écriture scientifique binaire de ce même nombre.
Ce qui est faux, ou plus exactement presque vrai.

En écriture scientifique binaire, le __significande__ ne peut interpréter que des valeurs comprises dans l'intervalle $\left[1;2\right[$ ; cela insinue que la partie entière d'un tel __significande__ est toujours composée d'un seul et unique bit à $1$.
Le standard IEEE-754 a donc fait le choix de ne pas coder la partie entière des __significandes__ dans les champs de __mantisse tronquée__, cela permettant un gain de précision d'un bit sur le codage des __significandes__.
Pour autant, il faut prendre en compte la valeur de ce bit $\left(1 \times 2^0\right)$ lors de l'évaluation de la valeur d'un champ de __mantisse tronquée__ $T$, ce qui mène au calcul $\left(1 + T\right)$.
D'où le fait que ce bit porte le nom de __bit implicite__.
Aussi, sachez que cette optimisation est appliquée à l'ensemble des formats définis par le standrard IEEE-754.

Je tiens à mentionner le fait que le champ de mantisse tronquée $T$ ne vaut exceptionnellement pas $\left(1 + T\right)$ lorsque le significande est nul, nous verrons cela dans un chapitre dédié en fin de document.

Finalement, je vous rappelle qu'en écriture scientifique le __significande__ porte également le nom de __mantisse__.
Ce qui nous permet d'en déduire que le nom de __mantisse tronquée__ est dû au fait que ce champ ne code qu'une partie du __significande__ (de la __mantisse__).

## Le champ d'exposant

En introduction se trouve une illustration de la disposition de chacun des bits des trois éléments qui composent le format Half Precision ; cette illustration montre que le champs d'__exposant__ $E$ est composé de $5$ bits.
Voici les poids de chacun des bits qui compose le champ d'__exposant__ :

$$\left[ \quad .. \quad , \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad .. \quad \right]$$

Les deux sections suivantes concernent l'ensemble de ces bits formant le champ d'__exposant__.

### L'encodage par biais du champ d'exposant

Le champ d'__exposant__ utilise un __encodage par biais__ qui est une déclinaison de l'encodage __Binary Unsigned__.

L'__encodage par biais__ s'appuie sur un champ binaire utilisant l'encodage __Binary Unsigned__, champ qui se charge de coder un nombre entier naturel.
Il faut ensuite ajouter ou déduire une constante appelée biais (qui est aussi un nombre entier naturel), au nombre codé par le champ binaire sous-jacent ; le résultat de ce calcul est alors la valeur qu'interprète réellement le champ.
Désormais, nous comprenons pourquoi est-ce que l'__encodage par biais__ est décrit comme étant une déclinaison de l'encodage __Binary Unsigned__.

Prenons le cas d'un champ d'__exposant__ $E$ d'une taille de $N$ bits, le champ code en __Binary Unsigned__ un nombre entier naturel dont la valeur est comprise dans l'intervalle $\left[0;2^N - 1\right]$.
En parallèle, la valeur du biais $B$ se calcul par la formule $\left(2^{\left(N - 1\right)} - 1\right)$, pour qu'au final la valeur qu'interprète le champ d'__exposant__ soit $\left(E - B\right)$.
Dès lors, nous comprenons que le champ d'__exposant__ peut interpréter des valeurs positives comme négatives.
Sachez que tout ce qui vient d'être dit est valable pour l'ensemble des formats ayant été établis par le standard IEEE-754.

Pour conclure, il s'avère que le circuit est amené à effectuer des comparaisons entre deux champs d'__exposant__ $E_1$ et $E_2$, ayant chacun un même biais $B$.
Ces comparaisons n'ont pas besoin de la valeur réelle des champs d'__exposant__, comme avec $\left(E_1 - B\right) \gt \left(E_2 - B\right)$ par exemple, car le biais $B$ est constant.
Le circuit doit plutôt effectuer le calcul suivant à la place $\left(E_1 \gt E_2\right)$.
Pour ce faire, le circuit utilise l'une des propriétés de l'encodage __Binary Unsigned__ qui, rappelons-le, code la valeur des champs $E_1$ et $E_2$.
Cette propriété est celle dont il est le sujet dans la section "_Une propriété du Binary Unsigned qui est fondamentale à la démonstration_".
En bref, la valeur de n'importe quel bit à $1$ de poids $i$ dans un champ d'__exposant__, est strictement supérieure à la somme des valeurs des bits de poids inférieur à $i$.
Nous verrons plus tard, par le prisme de la démonstration mathématique, que c'est cette propriété qui permet au circuit d'effectuer (entre autre) des comparaisons entre des champs d'__exposant__.

### Le codage du champ d'exposant

Dans l'introduction, il a été dit que le champs d'__exposant__ du nombre $F$ codé au format Half Precision correspondait au __multiplicande__ de l'écriture scientifique binaire de ce même nombre.
Il s'agit cependant d'une approximation.

Rappelons que le rôle d'un __multiplicande__ $M$ est de réajuster la valeur du __significande__ $S$ à la valeur absolue du nombre $F$ à représenter en écriture scientifique, ce qui correspond à $\vert \ F \vert = \left(S \times M\right)$.
Néanmoins, au travers du chapitre "_Le multiplicande_" et des deux sections qui le suivent, nous avons vu que le __multiplicande__ réajustait toujours la valeur du __significande__ en le multipliant par $2^{-c}$, ce qui donne plus précisément $\vert \ F \vert = \left(S \times 2^{-c}\right)$.

Il s'avère que la section "_Un autre regard sur le multiplicande_", explique pourquoi est-ce que la valeur du __significande__ n'a pas besoin d'être multiplié par $2^{-c}$ pour être réajusté à la valeur absolue de $F$.
Dans cette section il est dit je cite : "_Le multiplicande doit alors permettre de déplacer la virgule du significande_ $S$ _par le même nombre de rangs que ne l'a été celle de_ $\vert \ F \ \vert$ _pour former le significande (c'est à dire de_ $c$ _rangs), mais dans la direction opposée (donc de_ $-c$ _rangs)_".
Je vous invite à relire ladite section si vous souhaitez plus d'explications.
En bref, le significande $S$ est obtensible par un décalage de $c$ rangs de la virgule du nombre $\vert \ F \vert \notin \left[1;2\right[$, rappelons que le terme $c$ indique le sens du décalage grâce à son signe. 
Ainsi, le multiplicande a tout intérêt à déplacer la virgule du significande $S$ de $-c$ rangs plutôt que de multiplier ce dernier par $2^{-c}$, ce qui mène au même résultat (la valeur absolue de $F$) mais n'a pas les même conséquences pour le champ d'exposant.

En effet, la valeur de l'exposant $-c$ croît de façon linéaire tandis que la valeur de la puissance $2^{-c}$ croît exponentiellement.
Ce qui insinue que le champ d'exposant devrait être composé de (beaucoup) plus de bits en moyenne pour coder la valeur de la puissance $2^{-c}$, que pour coder l'exposant $-c$ lui-même.
Aussi, situation assez paradoxale, il faudrait que les puissances de $2$ soient codées sous la forme de nombre à virgule dans le champ d'exposant, afin de pouvoir prendre en charge les puissances de $2$ négatives que peut interpréter le multiplicande.
Enfin, pour finir, la gestion des opérandes dans les unités de calcul arithmétique flottantes (FPUs) serait probablement plus complexe avec un champ d'exposant codant une puissance de $2$ : l'alignement des virgules de deux opérandes à soustraire serait plus complexe par exemple.
D'où le fait que le champ d'exposant ne code que l'exposant $-c$ du multiplicande.

# Introduction au Floating Point Substractor Configuration Unit

Comme dit dans le chapitre "_Le standard IEEE-754_", le circuit électronique à l'étude dans ce document compare deux opérandes $\alpha$ et $\beta$ entre elle, deux opérandes de format Half Precision.
Je vous rappel que le circuit, du nom de Floating Point Substractor Configuration Unit (ou FPS Configuartion Unit), ne prend en charge que ce format pour des raisons de complexité de schématisation.

Ladite comparaison est une évaluation de supériorité stricte de la valeur absolu de l'un des deux opérandes envers la valeur absolu de l'autre.
Le FPS Configuration Unit n'utilise que la valeur absolu de ses opérandes $\alpha$ et $\beta$, le bit de signe des opérandes (bit de poids $15$) n'est pas transmis au circuit.
Je vous renvoie vers le chapitre "_Le standard IEEE-754_", si vous souhaitez visualiser l'illustration du codage Half Precision d'un nombre.

En bref, le circuit peut être mis dans deux états, l'état de _point terminal_ et de _point non terminal_.
L'un ou l'autre de ces états est généré par le traitement des champs d'exposant des opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$, c'est pourquoi le circuit traite les champs d'exposant avant les champs de mantisse tronquée des opérandes.
Voyons ci-dessous ce que sont (entre autre) les points terminaux et non terminaux, ainsi que ce qu'ils impliquent.

## Les points terminaux et non terminaux

Cette section aborde les sujets suivants:
  - Les traitements à effectués sur les champs des opérandes.
  - Les états de points terminaux et non terminaux dans lesquels peut se retrouvé le circuit.
  - La raison du traitement prioritaire des champs d'exposant vis à vis des champs de mantisse tronquée.

Nous savons que le FPS Configuration Unit effectue une évaluation de supériorité stricte de la valeur absolu de l'un de ses deux opérandes, envers la valeur absolu de l'autre.
Cependant, le tout est de savoir comment est ce que cette comparaison peut réelement avoir lieu, sachant que les opérandes du circuit sont composés de multiples champs.
La réponse est simple, il suffit d'effectuer une évaluation de supériorité stricte entre les champs d'exposant $E$ des opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$, et de faire de même entre les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Nous verrons ci-bas comment cela se traduit.

Au travers du chapitre "_le multiplicande_" ainsi que dans les deux sections qui le suivent, nous avons vus que la représentation en écriture scientifique binaire du nombre $F$ en valeur absolu était $\vert \ F \vert = \left(S \times 2^{-c}\right)$.
Suite au chapitre "_Le standard IEEE-754_" et aux sections suivante, nous en déduisons que n'importe quel format défini par la norme représenterait la valeur absolu de ce même nombre de la façon suivante $\vert \ F \vert = \left(\left(1 + T\right) \times \ 2^E\right)$.
N'oubliez pas que le champs de mantisse tronquée $T$ rend implicite le bit à $1$ de la partie entière du significande $S$.
D'où le fait qu'il faille ajouté la valeur de ce bit à celle du champs de mantisse tronquée $\left(1 + T\right)$.
Je vous invite à relire la section "_Le codage du champs de mantisse tronquée_", si nécessaire.

Extrapolons aux opérandes du FPS Configuration Unit ce qu'explique le paragraphe ci-dessus, cela donne $\vert \ \alpha \vert = \left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right)$ ainsi que $\vert \ \beta \vert = \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.
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

Nous l'aurons compris, le circuit atteint un état de point non terminal lorsque le traitement des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ ne l'amène pas à un état de point terminal.
Dans un état de point non terminal, le circuit doit effectuer le traitement des champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$.
Comme cela a été dit en introduction de cette section, le traitement des champs de mantisse tronquée, à l'instar de celui des champs d'exposant, consiste en une évaluation de supériorité stricte entre la valeur des champs.

# Démonstration mathématique

Nous pouvons désormais plongé dans le coeur de ce document, la démonstration mathématique du circuit.
Il y a une documentation dédié au circuit pour comprendre son utilité ainsi que son architecture, si vous souhaitez plus d'information concrète sur ce dernier je vous conseille de la lire.

En bref, le circuit génère deux bits de sortie, l'un pour configuré un circuit soustracteur de nombre flottant et l'autre pour la sortie même de cette unité de calcul.
Le Floating Point Configuartion Unit (le circuit) reçoit en entrée deux opérandes Half Precision en valeur absolu, que nous nommerons $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$.
La génération des deux bits de sortie du circuit passe par une comparaison de supériorité stricte de la valeur absolu de l'un de ses deux opérandes, envers la valeur absolu de l'autre.
Cette démonstration mathématique n'aborde que la partie de la logique de comparaison du circuit, ce qui représente tout de même la majeur partie de la logique du FPS Configuration Unit.
Le reste de la logique manquante ne peut pas être expliqué sans l'apport de beaucoup de nouvelle notion.
Je vous renvoie vers la documentation dédié au circuit si vous voulez en savoir plus sur cette partie de la logique manquante.

Tout ce qui va suivre de la démonstration, part du principe que le FPS Configuration Unit cherche à verifié la condition $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$.

## Définition de quelques opérations fondamentales à la démonstration 

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
Selon le corps de la fonction, nous en déduisons qu'elle ne retourne un bit à $1$ que lorsque le bit sur son paramètre $x$ vaut $1$ et celui sur $y$ est à $0$, sinon le bit retourné vaut $0$.

$$Nimply \ \left(x, \ y\right) \rightarrow \ x \ \wedge \ \overline{y}$$

# Le traitement des champs d'exposant

Comme cela a été dit par le chapitre "_Introduction au Floating Point Substractor Configuration Unit_", puis expliquer par la section "_Les points terminaux et non terminaux_".
Les champs d'exposant des opérandes du circuit sont traités avant les champs de mantisse tronquée de ces même opérandes.
Je vous invite à relire ces chapitres si nécessaire.

Rappelons que le circuit effectue une comparaison de supériorité stricte entre les champs d'exposant de ses opérandes.
Ce qui dans le cas de la condition sur laquelle s'appuie la démonstration mathématique du circuit $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, nécessite de vérifier si $\left(E_{\alpha} \gt E_{\beta}\right)$.
Voici la première phase du traitement des champs d'exposant.

$$\forall \ i \in \left[10;14\right], \quad Write \ \left(\tau_i, \ Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)\right)$$

Nous effectuons l'opération logique $Nimply$ sur chaque bit de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.
La variable $\tau$ (tau) représente un champs binaire de $15$ bits, dont les poids vont de $0$ à $14$.
Chaque bit de résultat d'une opération $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$, pour $i \in \left[10;14\right]$, est inscrit dans $\tau_i$.

Avec ce que nous avons vu de l'opération logique $Nimply$ dans la section précédente, nous comprenons que du moment où $\left(\tau_i = 1\right)$ pour tout poids $i \in \left[10;14\right]$ alors $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$.
Pour exprimer les choses différemment $\left(E_{\beta i} \times 2^i\right) \gt \left(E_{\alpha i} \times 2^i\right)$.
En plus de cela, rappelez vous de la section "_L'encodage par biais du champs d'exposant_".
Cette section explique entre autre que, je cite "_la valeur de n'importe quel bit à_ $1$ _de poids_ $i$ _dans un champs d'exposant, est strictement supérieur à la somme des valeurs des bits de poids inférieur à_ $i$".
Cela traduit l'idée que du moment où $\left(\tau_i = 1\right)$, pour tout $i \in \left[11;14\right]$, nous pouvons être sûrs que $\left(E_{\beta i} \times 2^i\right) \gt \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$.

Pour conclure, nous pouvons être certains que du moment où $\left(\tau_i = 1\right)$ pour tout $i \in \left[10;14\right]$, alors $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$.
Pour le dire autrement, du bit de poids $i$ jusqu'au bit de poids $10$ le champs d'exposant $E_{\beta}$ est supérieur au champs d'exposant $E_{\alpha}$.
Mais cela ne démontre pas pour autant que $\left(E_{\alpha} \lt E_{\beta}\right)$.

## Les zéros anonymes

Nous avons vu dans la section "_Définition de quelques opérations fondamentales à la démonstration_", que l'opération logique $Nimply$ génère un bit de résultat à $1$ sous réserve que le bit sur son paramètre $x$ vaille $1$, et que celui sur $y$ vaille $0$.
Autrement, l'opération renvoie un bit de résultat à $0$.

Sachez qu'il est possible de décliné l'opération logique $Nimply$ en une porte logique, un petit circuit électronique chargé d'effectuer une opération $Nimply$ entre deux bits.
Le FPS Configuration Unit se sert des portes logiques $Nimply$ pour pouvoir déduire la valeur des bits d'opérandes d'une porte, en fonction du bit de résultat qu'elle génère.
C'est exactement ce que nous avons fait dans le chapitre précédent, nous déduisons que du moment où $\left(\tau_i = 1\right)$ alors $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$.
Cependant, les choses se corsent à partir du moment où une opération $Nimply$ génère un bit de résultat à $0$, ce que la documentation dédié au circuit appelle un _zéro anonyme_.

La raison est simple, pour une opération $Nimply$ il n'y a qu'une seule manière de définir un bit de résultat à $1$, mais plusieurs pour un bit de résultat à $0$.
Concrètement, une opération logique $Nimply$ génère un bit de résultat à $0$ lorsque:
  - Ces deux bits d'opérandes $\left(x, \ y\right)$ sont égaux.
  - Le bit sur le paramètre $x$ vaut $0$ pendant que celui sur $y$ vaut $1$.

Par conséquent, un bit de résultat à $0$ ne permet pas de déduire la valeur des bits d'opérandes (qui est comme anonyme), d'où le nom de _zéro anonyme_.
Plus bas, nous allons voir que les _zéros anonymes_ sont la raison pour laquelle le chapitre précédent conclu de la sorte.
Je précise que dans les faits il existe des _zéros anonymes capitaux_ et _non capitaux_, dissociation importante que nous allons expliqués ci-dessous.

Pour finir, la section "_Les points terminaux et non terminaux_" évoque le fait que le traitement des champs d'exposant ainsi que des champs de mantisse tronquée soit identique, ou presque.
Nous comprenons d'ores et déjà que le traitement des champs d'exposant passe par l'application d'opération logique $Nimply$ sur les bits des champs $E_{\alpha}$ et $E_{\beta}$.
Ce qui engendre quelques subtilités qu'il faut expliqué, comme celle des _zéros anonymes capitaux_ et _non capitaux_.
Sachez que le traitement des champs de mantisse tronquée utilise aussi des opérations logiques $Nimply$.
Par conséquent, nous recontextualiserons en temps voulu les concepts de _zéros anonymes capitaux_ et _non capitaux_ aux cas des champs de mantisse tronquée.
Mais pour l'instant, concentrons sur le cas des champs d'exposant.

### Les zéros anonymes non capitaux dus au traitement des champs d'exposant

Les _zéros anonymes capitaux_ et _non capitaux_ sont dus aux même raisons, le chapitre précédent explique très bien cela.
Ce que le chapitre n'explique pas, c'est que les _zéros anonymes non capitaux_ n'ont aucune importance pour le traitement des champs d'exposant $E_{\alpha}$ et $E_{\beta}$, au contraire des _zéros anonymes capitaux_.
Nous verrons plus tard que cela ne se limite pas au traitement des champs d'exposant, mais s'étend aussi au traitement des champs de mantisse tronquée.

Après cette rapide introduction, reprenons là où nous nous étions arrêté dans le chapitre "_Le traitement des champs d'exposant_".
Admettons qu'il y ait un _MSB1_ dans $\tau \in \left[10;14\right]$ et que ce dernier ait un poids $i \in \left[11;14\right]$, alors nous en déduisons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$ pour $\left(\tau_i = 1\right)$.
En outre, comme l'explique ledit chapitre, nous en déduisons que $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$.
Rappelons que cette inéquation fonctionne car la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs d'exposant comme $E_{\beta}$, est _inconditionnellement_ supérieur à la somme des valeurs des bits de poids inférieur à $i$. 
Cela se remarque plus avec la formulation suivante de cette même inéquation $\left(E_{\beta i} \times 2^i\right) \gt \left(\left(E_{\alpha i} \times 2^i\right) + \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right)$.

Définissons désormais ce qu'est un _zéro anonyme non capital_.
Par le prisme du traitement des champs d'exposant, un _zéro anonyme non capital_ est un bit à $0$ dans $\tau$ qui est d'un poids $\sigma \in \left[10;i\right[$.
Du moment où le poids $i$ du _MSB1_ est compris dans l'intervalle $\left[11;14\right]$, alors nous pouvons être certain que dans $\tau \in \left[10;14\right]$ il existe au moins un bit $\tau_{\left(i - 1\right)}$, qu'il soit défini à $0$ ou à $1$.
Par conséquent, lorsque $\left(\tau_{\left(i - 1\right)} = 0\right)$ alors nous en déduisons que le bit de résultat est un _zéro anonyme non capital_, et de fait que les valeurs de ses bits d'opérandes $E_{\alpha \left(i - 1\right)}$ ainsi que $E_{\beta \left(i - 1\right)}$ ne sont pas déductible, nous savons seulement que $\left(E_{\alpha \left(i - 1\right)} \ge E_{\beta \left(i - 1\right)}\right)$.
Ce qui se généralise à l'ensemble des bits de résultat $\left(\tau_{\sigma} = 0\right)$, desquels nous déduisons alors que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$.

Finalement, revenons en à l'inéquation $\left(E_{\beta i} \times 2^i\right) \gt \left(\left(E_{\alpha i} \times 2^i\right) + \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right)$. 
Inéquation depuis laquelle nous remarquons que $\left(E_{\beta i} \times 2^i\right)$ est supérieur à la somme des valeurs des bits de poids $\sigma \in \left[10;i\right[$ du champs d'exposant $E_{\alpha}$, nonobstant la valeur de chacun des bits $E_{\alpha\sigma}$.
Dit autrement, rappelez vous du fait que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$ pour tout bit $\left(\tau_{\sigma} = 0\right)$, ce qui sous-entend que $E_{\alpha\sigma}$ peut être défini à $0$ comme à $1$.
Néanmoins, la valeur de ces bits n'a aucune importance car nous trouverons toujours que $\left(E_{\beta i} \times 2^i\right) \gt \sum_{\sigma = i - 1}^{10} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$.
D'où le fait que le circuit n'ait pas à se soucier des _zéros anonymes non capitaux_, et par corollaire des bits d'opérandes de ces derniers.

### Les zéros anonymes capitaux dus au traitement des champs d'exposant

La section précédente explique pourquoi les bits de $\tau \in \left[10;14\right]$, d'un poids inférieur au _MSB1_ de poids $i \in \left[11;14\right]$, ne sont pas pris en compte dans le traitement des champs d'exposant.
Ce qui concerne entre autre les _zéros anonymes non capitaux_.
Cette section aborde quant à elle le sujet des _zéros anonymes capitaux_. 

Reprenons de nouveau là où nous en étions dans le chapitre "_Le traitement des champs d'exposant_".
Admettons que le _MSB1_ de $\tau \in \left[10;14\right]$ ait un poids $i \in \left[10;13\right]$, alors nous savons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$ pour $\left(\tau_i = 1\right)$.
En outre, comme l'explique ledit chapitre, nous en déduisons alors que $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$.
Cependant, comme cela a été mentionné dans ce même chapitre, ceci ne suffit pas à démontrer que $\left(E_{\alpha} \lt E_{\beta}\right)$.

Définissons désormais ce qu'est un _zéro anonyme capital_.
Par le prisme du traitement des champs d'exposant, un _zéro anonyme capital_ est un bit dans $\tau$ qui est d'un poids $\sigma \in \left]i;14\right]$.
Du moment où le poids $i$ du _MSB1_ est compris dans l'intervalle $\left[10;13\right]$, alors nous pouvons être certains que dans $\tau \in \left[10;14\right]$ il existe au moins un bit $\tau_{\left(i + 1\right)}$ qui vaille $0$.
Le bit $\tau_{\left(i + 1\right)}$ ne peut être défini qu'à $0$ étant donné que le bit $\tau_i$ incarne le _MSB1_ de $\tau \in \left[10;14\right]$, ce qui se généralise à l'ensemble des bits $\tau_{\sigma}$.
En conséquence, tout bit de poids $\sigma$ dans $\tau$ est un _zéro anonyme capital_, ce qui insinue que les valeurs des bits d'opérandes $E_{\alpha\sigma}$ ainsi que $E_{\beta\sigma}$ ne sont pas déductible, nous savons seulement que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$.

Rappelons que l'inéquation $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$ démontre que le champs d'exposant $E_{\beta}$ est supérieur au champs d'exposant $E_{\alpha}$, du bit de poids $i$ jusqu'au bit de poids $10$.
Cependant, n'oublions pas qu'il subsiste dans $\tau$ des bits de poids $\sigma$ supérieur à $i$, des _zéros anonymes capitaux_, par lesquels nous déduisons que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$.
Il suffit alors qu'il y ait une occurrence de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$ pour tout bit $\tau_{\sigma}$, afin que nous en déduisions que $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$.
Ce qui traduit l'idée que du bit de poids $\sigma$ jusqu'au bit de poids $10$, le champs d'exposant $E_{\alpha}$ est supérieur au champs d'exposant $E_{\beta}$.
Finalement, pour savoir si le champs d'exposant $E_{\beta}$ est définitivement supérieur au champs d'exposant $E_{\alpha}$, il nous faut prêter attention aux valeurs des bits d'opérandes $E_{\alpha\sigma}$ ainsi que $E_{\beta\sigma}$ de chaque _zéro anonyme capital_.
Nous en déduisons alors qu'il y a deux possibilités, dont la première est la suivante.

Le circuit atteint l'état de point terminal $\left(E_{\alpha} \lt E_{\beta}\right)$ lorsque $\left(E_{\alpha\sigma} = E_{\beta\sigma}\right)$, pour tout poids $\sigma \in \left]i;14\right]$.
Ce qui veut dire que la somme $\lambda_{\alpha}$ de la valeur des bits de poids $\sigma$ du champs d'exposant $E_{\alpha}$, est égale à la somme $\lambda_{\beta}$ de la valeur des bits de même poids du champs d'exposant $E_{\beta}$.
Pour le dire autrement nous savons que $\left(\lambda_{\alpha} = \lambda_{\beta}\right)$, tout en sachant que $\ \lambda_{\alpha} = \sum_{\sigma = i + 1}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$ tandis que $\ \lambda_{\beta} = \sum_{\sigma = i + 1}^{14} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$.
Pour conclure, nous en déduisons que le circuit atteint l'état de point terminal $\left(E_{\alpha} \lt E_{\beta}\right)$ car $\left(\lambda_{\beta} + E_{\beta i} \times 2^i\right) \gt \left(\lambda_{\alpha} + \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)\right)$, et ce dû au fait que $\left(E_{\beta i} \times 2^i\right) \gt \sum_i^{10} \left(E_{\alpha i} \times 2^i\right)$.
Enfin, rappelez vous du fait que la condition sur laquelle s'appuie la démonstration du circuit est $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, condition qui se solde en échec car $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \lt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.
Tel que l'explique le chapitre "_Les points terminaux et non terminaux_", les expressions que sont $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right)$ ainsi que $\left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$ représentent les valeurs respective de $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$.

Cependant, le circuit atteint l'état de point terminal $\left(E_{\alpha} \gt E_{\beta}\right)$ lorsqu'il y a une (ou plusieurs) occurrence de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$, pour tout poids $\sigma \in \left]i;14\right]$.
Pour chacune de ces occurrences où $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(E_{\beta\sigma} \times 2^{\sigma}\right)$, nous en déduisons que $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$.
Ce qui traduit l'idée que du bit de poids $\sigma$ jusqu'au bit de poids $10$, le champs d'exposant $E_{\alpha}$ est supérieur au champs d'exposant $E_{\beta}$.
A ce stade, nous savons que $\left(E_{\alpha} \gt E_{\beta}\right)$ dans le cas où l'une de ces occurrences défini $\left(\sigma = 14\right)$.
Néanmoins, admettons que $\sigma$ soit compris dans l'intervalle $\left]i;14\right[$, alors nous en déduisons qu'il existe des _zéros anonymes capitaux_ de poids $\sigma '$ dans $\tau$, tel que $\sigma ' \in \left]\sigma;14\right]$.
Par conséquent, cela insinue que $\left(E_{\alpha\sigma '} \ge E_{\beta\sigma '}\right)$ pour tout bit $\tau_{\sigma '}$, et donc que la somme $\lambda$ de la valeur des bits de poids $\sigma '$ du champs d'exposant $E_{\alpha}$ est supérieur ou égale à la somme des bits de même poids du champs d'exposant $E_{\beta}$, ou autrement dit $\left(\lambda_{\alpha} \ge \lambda_{\beta}\right)$.
En définitif, nous comprenons que le circuit atteint l'état de point terminal $\left(E_{\alpha} \gt E_{\beta}\right)$ car $\left(\lambda_{\alpha} + E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(\lambda_{\beta} + \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)\right)$.

En conclusion, nous savons désormais pourquoi est-ce qu'une unique occurrence de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$, pour tout poids $\sigma \in \left]i;14\right]$, suffit à en déduire que $\left(E_{\alpha} \gt E_{\beta}\right)$. 
De surcroît, ceci nous permet de savoir que $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$, ou autrement dit que l'évaluation de la condition sur laquelle s'appuie la démonstration du circuit $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$ se solde par un succès.

## L'obtention d'un point non terminal

Je précise que ce chapitre parle des états de points non terminaux du circuit.
En cas de besoin, je vous invite à lire la section "_Les points terminaux et non terminaux_" pour en savoir plus sur cet état.

Dans les sections précédente nous avions toujours un _MSB1_ dans $\tau \in \left[10;14\right]$, mais il pourrait ne pas y en avoir.
Par essence, une telle situation veut que chaque bit de $\tau \in \left[10;14\right]$ soit un _zéro anonyme capital_, car ces bits sont inévitablement à $0$ et qu'ils ne peuvent pas être d'un poids inférieur à un _MSB1_, étant donné qu'il n'en n'existe pas.
Plus précisément, chacun de ces bits a le pouvoir de faire basculé le circuit d'un état de point non terminal vers un état de point terminal, comme nous le verrons ci-bas.
Tout ceci nous permet d'en déduire que $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$ pour tout les bits de poids $\sigma \in \left[10;14\right]$, ce qui veut intrinséquement dire que $\left(E_{\alpha} \ge E_{\beta}\right)$.

Le circuit passe dans un état de point terminal $\left(E_{\alpha} \gt E_{\beta}\right)$ du moment où il y a une (ou plusieurs) occurrence de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$, pour tout poids $\sigma \in \left[10;14\right]$.
Pour chacune de ces occurrences où $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(E_{\beta\sigma} \times 2^{\sigma}\right)$, nous en déduisons que $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)$. 
Cependant, admettons que $\sigma$ soit compris dans l'intervalle $\left[10;14\right[$, alors nous savons qu'il existe des _zéros anonymes capitaux_ de poids $\sigma '$ dans $\tau$, tel que $\sigma ' \in \left]\sigma;14\right]$.
En bref, cela veut dire que $\left(E_{\alpha\sigma '} \ge E_{\beta\sigma '}\right)$ pour tout bit $\tau_{\sigma '}$.
Ce qui insinue que la somme $\lambda$ de la valeur des bits de poids $\sigma '$ du champs d'exposant $E_{\alpha}$, est supérieur ou égale à la somme des bits de même poids du champs d'exposant $E_{\beta}$, ou autrement dit $\left(\lambda_{\alpha} \ge \lambda_{\beta}\right)$.
Par conséquent, le circuit détermine que $\left(E_{\alpha} \gt E_{\beta}\right)$ car $\left(\lambda_{\alpha} + E_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(\lambda_{\beta} + \sum_{\sigma}^{10} \left(E_{\beta\sigma} \times 2^{\sigma}\right)\right)$.
Au final, l'évaluation de la condition sur laquelle s'appuie la démonstration du circuit $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$ réussie, parce que $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$.

Cependant, le circuit passe dans un état de point non terminal du moment où $\left(E_{\alpha} = E_{\beta}\right)$.
Cet état de point non terminal ne peut s'obtenir que dans le cas où l'ensemble des bits de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ sont identiques, ou autrement dit lorsque $\left(E_{\alpha\sigma} = E_{\beta\sigma}\right)$ pour tout les bits de poids $\sigma \in \left[10;14\right]$.
Concrètement, cela veut dire que par le seul traitement des champs d'exposant de ses opérandes, le circuit FPS Configuration Unit n'est pas en capacité d'anticipé le résultat de la condition sur laquelle s'appuie la démonstration mathématique $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$.
Rappelons que cette condition ressemble plus formellement à $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$, et remarquons que l'évaluation de la condition ne dépend pas des champs d'exposant lorsque $\left(E_{\alpha} = E_{\beta}\right)$, mais uniquement des champs de mantisse tronquée $T$.
C'est pourquoi le circuit traite les champs de mantisse tronquée lorsqu'il atteint un état de point non terminal.

# Le traitement des champs de mantisse tronquée

La démonstration mathématique du circuit s'appuie sur le test de la condition suivante $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$, condition qui ressemble plus formellement à $\left(\left(1 + T_{\alpha}\right) \times \ 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times \ 2^{E_{\beta}}\right)$. 
Nous remarquons alors qu'en état de point non terminal $\left(E_{\alpha} = E_{\beta}\right)$, le circuit ne peut départagé la valeur de ses deux opérandes que par l'évaluation de $\left(1 + T_{\alpha}\right) \gt \left(1 + T_{\beta}\right)$, car il ne reste plus que cela de pertinent dans la condition d'origine. 
En bref, le traitement des champs de mantisse tronquée passe par une comparaison de supériorité stricte entre les champs $T_{\alpha}$ et $T_{\beta}$, à l'instar des champs d'exposant.
Mais les ressemblances ne s'arrêtent pas là.

Rappelez vous du fait que les encodages des champs d'exposant ainsi que de mantisse tronquée ont des points en commun, ils partagent les propriétés de l'encodage Binary Unsigned.
Je vous invite à relire la section "_L'encodage du champs de mantisse tronquée_" si nécessaire.
En bref, ce qu'il faut retenir de cette section c'est que la valeur de n'importe quel bit à $1$ de poids $i$ dans un champs de mantisse tronquée, est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Ce qui nous rappelle que c'est également la propriété qui sous-tend toute la logique de traitement des champs d'exposant.
De surcroît, le traitement des champs d'exposant et de mantisse tronquée étant identique ou presque, nous utiliserons les même leviers logique pour le traitement des champs de mantisse tronquée que pour les champs d'exposant. 
Voici à quoi ressemble la première phase du traitement des champs de mantisse tronquée.

$$\forall \ i \in \left[0;9\right], \quad Write \ \left(\tau_i, \ Nimply \ \left(T_{\beta i}, \ T_{\alpha i}\right)\right)$$

Le champs $\tau$ est techniquement le même que celui utilisé dans la première partie de la démonstration "_Le traitement des champs d'exposant_".
La seule différence est que le bit de résultat de chaque opération $Nimply$ sur $T_{\beta i}$ et $T_{\alpha i}$ est inscrit dans $\tau_i$, pour $i \in \left[0;9\right]$.

En outre, grâce au chapitre "_Définition de quelques opérations fondamentales à la démonstration_" nous savons que dans le cas d'une occurrence où $\left(\tau_i = 1\right)$, pour tout bit de poids $i \in \left[0;9\right]$, alors $\left(T_{\beta i} = 1\right)$ tandis que $\left(T_{\alpha i} = 0\right)$.
Avec ce que nous avons dit précédemment, nous en déduisons que pour toute occurrence où $\left(\tau_i = 1\right)$ alors $\left(T_{\beta i} \times 2^i\right) \gt \sum_i^0 \left(T_{\alpha i} \times 2^i\right)$.

## Recontextualisation des zéros anonymes capitaux ainsi que non capitaux dus au traitement des champs de mantisse tronquée

Comme cela a été mentionné dans le chapitre "_Les zéros anonymes_", je cite "_nous recontextualiserons en temps voulu les concepts de zéros anonymes capitaux et non capitaux aux cas des champs de mantisse tronquée_".
Eh bien... le moment est venu de le faire.
Par la même occasion, cela va nous permettre de parler du traitement des champs de mantisse tronquée, comme nous l'avons finalement fait avec les champs d'exposant.

Sachez qu'il n'y a pas grand chose qui change entre les _zéros anonymes capitaux_ et _non capitaux_ qui sont issu du traitement des champs d'exposant, vis à vis de ceux qui sont issus du traitement des champs de mantisse tronquée.
C'est pourquoi nous survolerons ce qui a déjà été expliqué plus en amont, en rentrant moins dans le détails.

### Les zéros anonymes non capitaux dus au traitement des champs de mantisse tronquée

Comme cela a été dit et démontrer dans la section "_Les zéros anonymes non capitaux dus au traitement des champs d'exposant_", les _zéros anonymes non capitaux_ n'ont aucune importance pour le traitement des champs d'exposant.
En somme, il s'avère que c'est aussi le cas pour le traitement des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.

Revenons en où nous en étions dans le chapitre "_Le traitement des champs de mantisse tronquée_".
Admettons qu'il y ait un _MSB1_ dans $\tau \in \left[0;9\right]$ et que ce dernier ait un poids $i \in \left[1;9\right]$, alors pour $\left(\tau_i = 1\right)$ ledit chapitre nous permet d'en déduire que $\left(T_{\beta i} \times 2^i\right) \gt \sum_i^0 \left(T_{\alpha i} \times 2^i\right)$.
En outre, par le prisme du traitement des champs de mantisse tronquée, un _zéro anonyme non capital_ est un bit à $0$ dans $\tau$ qui est d'un poids $\sigma \in \left[0;i\right[$.
Il s'avère qu'il existe au moins un bit $\tau_{\left(i - 1\right)}$ qui vaille $0$ ou $1$ dans $\tau \in \left[0;9\right]$.
Nous en concluons alors que le bit $\tau_{\left(i - 1\right)}$ incarne un _zéro anonyme non capital_ dans le cas où ce dernier vaut $0$.
Ce qui se généralise à l'ensemble des bits $\tau_{\sigma}$, desquels nous déduisons alors que $\left(T_{\alpha\sigma} \ge T_{\beta\sigma}\right)$.

En fin de compte, nous savons que $\left(T_{\beta i} \times 2^i\right) \gt \sum_i^0 \left(T_{\alpha i} \times 2^i\right)$ lorsque $\left(\tau_i = 1\right)$, ce que nous allons exprimé différemment pour des raisons pratique $\left(T_{\beta i} \times 2^i\right) \gt \left(\left(T_{\alpha i} \times 2^i\right) + \sum_{\sigma = i - 1}^0 \left(T_{\alpha\sigma} \times 2^{\sigma}\right)\right)$.
Remarquons que $\left(T_{\beta i} \times 2^i\right)$ est supérieur à la somme des valeurs des bits de poids $\sigma$ du champs de mantisse tronquée $T_{\alpha}$, nonobstant la valeur de ces bits.
Pour le dire autrement, rappelez vous du fait que $\left(T_{\alpha\sigma} \ge T_{\beta\sigma}\right)$ pour tout bit $\left(\tau_{\sigma} = 0\right)$, ce qui sous-entend que $T_{\alpha\sigma}$ peut être défini à $0$ comme à $1$.
Mais cela n'a pas d'importance car nous trouverons toujours que $\left(T_{\beta i} \times 2^i\right) \gt \sum_{\sigma = i - 1}^0 \left(T_{\alpha\sigma} \times 2^{\sigma}\right)$.

Pour resumer, un _zéro anonyme non capital_ de poids $\sigma \in \left[0;i\right[$ ne permet pas de déduire la valeur de ses bits d'opérandes $T_{\alpha\sigma}$ ainsi que $T_{\beta\sigma}$.
Néanmoins, le circuit n'a pas besoin de prendre en charge les _zéros anonymes non capitaux_, car il est déductible que le champs de mantisse tronquée $T_{\beta}$ est supérieur au champs de mantisse tronquée $T_{\alpha}$, du bit de poids $i$ jusqu'au bit de poids $0$.
Qu'importe ce que l'ont peut déduire des bits d'opérande des _zéros anonymes non capitaux_, cela ne changerait rien à ce qui vient d'être dit.

### Les zéros anonymes capitaux dus au traitement des champs de mantisse tronquée

La section précédente explique pourquoi les _zéros anonymes non capitaux_ ne sont pas pris en compte dans le traitement des champs de mantisse tronquée.
Voyons désormais ce qu'il en est des _zéros anonymes capitaux_. 

Reprenons là où nous en étions dans le chapitre "_Le traitement des champs de mantisse tronquée_".
Admettons qu'il y ait un _MSB1_ dans $\tau \in \left[0;9\right]$ et qu'il soit d'un poids $i \in \left[0;8\right]$, alors pour $\left(\tau_i = 1\right)$ ledit chapitre nous permet d'en déduire que $\left(T_{\beta i} \times 2^i\right) \gt \sum_i^0 \left(T_{\alpha i} \times 2^i\right)$.
En outre, par le prisme du traitement des champs de mantisse tronquée, un _zéro anonyme capital_ est un bit dans $\tau$ qui est d'un poids $\sigma \in \left]i;9\right]$.
Nous en déduisons donc que dans $\tau \in \left[0;9\right]$, il existe au moins un bit $\tau_{\left(i + 1\right)}$ qui vaille $0$.
Le bit $\tau_{\left(i + 1\right)}$ ne peut être défini qu'à $0$ étant donné que $\tau_i$ incarne le _MSB1_ de $\tau \in \left[0;9\right]$.
Ceci se généralise à l'ensemble des bits $\tau_{\sigma}$, desquels nous déduisons alors que $\left(T_{\alpha\sigma} \ge T_{\beta\sigma}\right)$.

Il s'avère que l'inéquation $\left(T_{\beta i} \times 2^i\right) \gt \sum_i^0 \left(T_{\alpha i} \times 2^i\right)$ démontre que le champs de mantisse tronquée $T_{\beta}$ est supérieur au champs de mantisse tronquée $T_{\alpha}$, du bit de poids $i$ jusqu'au bit de poids $0$.
Cependant, il suffit qu'il y ait une occurrence de $\tau_{\sigma}$ due au fait que $\left(T_{\alpha\sigma} \gt T_{\beta\sigma}\right)$, pour qu'il en soit déduit que $\left(T_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^0 \left(T_{\beta\sigma} \times 2^{\sigma}\right)$.
Par conséquent, pour savoir si le champs de mantisse tronquée $T_{\beta}$ est supérieur au champs de mantisse tronquée $T_{\alpha}$, il nous faut prêter attention à la valeur des bits d'opérandes $T_{\alpha\sigma}$ et $T_{\beta\sigma}$ de l'ensemble des _zéros anonymes capitaux_.

Rappelons que la condition sur laquelle s'appuie la démonstration mathématique du circuit est $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$.

L'évaluation de cette condition se solde par un échec lorsque $\left(T_{\alpha\sigma} = T_{\beta\sigma}\right)$, pour tout bit de poids $\sigma \in \left]i;9\right]$.
Cela est dû au fait que la somme $\lambda$ de la valeur des bits de poids $\sigma$ du champs de mantisse tronquée $T_{\alpha}$, est égale à la somme de la valeur des bits de même poids du champs de mantisse tronquée $T_{\beta}$.
Pour le dire autrement $\left(\lambda_{\alpha} = \lambda_{\beta}\right)$, en sachant que $\lambda_{\alpha} = \sum_{\sigma = i + 1}^9 \left(T_{\alpha\sigma} \times 2^{\sigma}\right)$ tandis que $\lambda_{\beta} = \sum_{\sigma = i + 1}^9 \left(T_{\beta\sigma} \times 2^{\sigma}\right)$.
Par conséquent, le circuit détermine que $\left(T_{\alpha} \lt T_{\beta}\right)$ car $\left(\lambda_{\beta} + T_{\beta i} \times 2^i\right) \gt \left(\lambda_{\alpha} + \sum_i^0 \left(T_{\alpha i} \times 2^i\right)\right)$.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$, nous en déduisons finalement que $\left(\left(1 + T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \lt \left(\left(1 + T_{\beta}\right) \times 2^{E_{\beta}}\right)$. 
D'où le fait que l'évaluation de la condition se solde par un échec.

En revanche, l'évaluation de cette même condition se solde par un succès lorsqu'il y a une (ou plusieurs) occurence de $\left(T_{\alpha\sigma} \gt T_{\beta\sigma}\right)$, pour tout poids $\sigma \in \left]i;9\right]$.
Il s'avère que pour chaque occurence de $\left(T_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(T_{\beta\sigma} \times 2^{\sigma}\right)$ nous en déduisons que $\left(T_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^0 \left(T_{\beta\sigma} \times 2^{\sigma}\right)$. 
Cependant, admettons que $\sigma$ soit compris dans l'intervalle $\left]i;9\right[$, alors nous savons qu'il existe des _zéros anonymes capitaux_ de poids $\sigma '$ dans $\tau$, tel que $\sigma ' \in \left]\sigma;9\right]$.
En bref, cela veut dire que $\left(T_{\alpha\sigma '} \ge T_{\beta\sigma '}\right)$ pour tout bit $\tau_{\sigma '}$.
Ce qui insinue que la somme $\lambda$ de la valeur des bits de poids $\sigma '$ du champs de mantisse tronquée $T_{\alpha}$, est supérieur ou égale à la somme des bits de même poids du champs de mantisse tronquée $T_{\beta}$, ou pour le dire autrement $\left(\lambda_{\alpha} \ge \lambda_{\beta}\right)$.
Par conséquent, le circuit détermine que $\left(T_{\alpha} \gt T_{\beta}\right)$ car $\left(\lambda_{\alpha} + T_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(\lambda_{\beta} + \sum_{\sigma}^0 \left(T_{\beta\sigma} \times 2^{\sigma}\right)\right)$.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$, nous en déduisons finalement que $\left(\left(1 + T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times 2^{E_{\beta}}\right)$.
Voici pourquoi l'évaluation de la condition se solde par un succès.

Précédemment, nous partions du principe qu'il y avait un _MSB1_ dans $\tau \in \left[0;9\right]$, mais il est possible qu'il n'y en ait pas.
Par essence, une telle situation veut que chaque bit de $\tau \in \left[0;9\right]$ soit un _zéro anonyme capital_, car ces bits sont inévitablement à $0$ et qu'ils ne peuvent pas être d'un poids inférieur à un _MSB1_, étant donné qu'il n'en n'existe pas.
En bref, nous en déduisons alors que $\left(T_{\alpha\sigma} \ge T_{\beta\sigma}\right)$ pour tout poids $\sigma \in \left[0;9\right]$, ce qui veut intrinséquement dire que $\left(T_{\alpha} \ge T_{\beta}\right)$.

Par conséquent, nous savons que l'évaluation de la condition sur laquelle s'appuie la démonstration du circuit échoue lorsque $\left(T_{\alpha\sigma} = T_{\beta\sigma}\right)$, pour tout les bits de poids $\sigma \in \left[0;9\right]$.
Dès lors, nous en déduisons que les champs de mantisse tronquée sont égaux $\left(T_{\alpha} = T_{\beta}\right)$.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$, nous en tirons finalement comme conséquence que $\left(\left(1 + T_{\alpha}\right) \times 2^{E_{\alpha}}\right) = \left(\left(1 + T_{\beta}\right) \times 2^{E_{\beta}}\right)$.
D'où le fait que l'évaluation de la condition échoue.

Finalement, l'évaluation de cette même condition se solde par un succès lorsqu'il y a une (ou plusieurs) occurence de $\left(T_{\alpha\sigma} \gt T_{\beta\sigma}\right)$, pour tout poids $\sigma \in \left[0;9\right]$.
Il s'avère que pour chaque occurence de $\left(T_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(T_{\beta\sigma} \times 2^{\sigma}\right)$ nous en déduisons que $\left(T_{\alpha\sigma} \times 2^{\sigma}\right) \gt \sum_{\sigma}^0 \left(T_{\beta\sigma} \times 2^{\sigma}\right)$. 
Cependant, admettons que $\sigma$ soit compris dans l'intervalle $\left[0;9\right[$, alors nous savons qu'il existe des _zéros anonymes capitaux_ de poids $\sigma '$ dans $\tau$, tel que $\sigma ' \in \left]\sigma;9\right]$.
En bref, cela veut dire que $\left(T_{\alpha\sigma '} \ge T_{\beta\sigma '}\right)$ pour tout bit $\tau_{\sigma '}$.
Ce qui insinue que la somme $\lambda$ de la valeur des bits de poids $\sigma '$ du champs de mantisse tronquée $T_{\alpha}$, est supérieur ou égale à la somme des bits de même poids du champs de mantisse tronquée $T_{\beta}$, ou pour le dire autrement $\left(\lambda_{\alpha} \ge \lambda_{\beta}\right)$.
Par conséquent, le circuit détermine que $\left(T_{\alpha} \gt T_{\beta}\right)$ car $\left(\lambda_{\alpha} + T_{\alpha\sigma} \times 2^{\sigma}\right) \gt \left(\lambda_{\beta} + \sum_{\sigma}^0 \left(T_{\beta\sigma} \times 2^{\sigma}\right)\right)$.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$, nous en déduisons finalement que $\left(\left(1 + T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right) \times 2^{E_{\beta}}\right)$.
En somme, c'est la raison qui fait que l'évaluation de ladite condition se solde par un succès.

# Conclusion de la démonstration

Comme nous avons pu le voir dans la première partie de la démonstration "_Le traitement des champs d'exposant_", ainsi que dans la seconde partie "_Le traitement des champs de mantisse tronquée_", les procédés de traitement des champs d'exposant et de mantisse tronquée sont identiques ou presque.
Dans les faits, le circuit électronique n'est composé que d'un seul ensemble de circuit logique, prenant ainsi en charge le traitement des champs d'exposant ainsi que des champs de mantisse tronquée.
La documentation du Floating Point Substractor Configuration Unit aborde le sujet de l'architecture du circuit lui même.

Je tient aussi à rappellé que tout ce que nous avons vu jusqu'ici n'est qu'une preuve mathématique du bon fonctionnement de la logique de comparaison du circuit.
Cependant, les circuits logiques du FPS Configuration Unit n'effectuent pas les même calculs que ceux utilisés dans la démonstration, par exemple, le circuit ne comporte pas d'additionneur pour effectuer des calculs de somme.
Les raisons à cela sont multiples.
En effet, il est globalement plus optimal que le circuit fasse usage de circuit simple, plus rapide, moins énergivore et nécessitant moins de câblage ainsi que de transitor qu'un additionneur à propagation de retenue, qui est un circuit d'une certaine complexité.
In fine, ceci me permet de mentionné le fait qu'il y ait un réel décalage entre la démonstration mathématique du circuit, et son architecture.

# Annexe: Les spécificités des formats du standard IEEE-754

Du début jusqu'à la fin, la démonstration mathématique part du principe que le FPU Configuration Unit ne prend en charge que deux opérandes de format __Half Precision__.
Il n'y a rien de faux là dedans, mais cela ne laisse pas transparaitre le fait que les formats définis par le standard IEEE-754 puissent coder différents types de nombre.
Enfaite, chaque format comme le __Half Precision__ peut coder :

  - des nombres __normaux__,
  - des nombres __dénormaux__,
  - des __NaN__,
  - ou encore l'__infini positif/négatif__

Précédemment dans la démonstration mathématique, nous sommes implicitement partis du principe que les opérandes du circuit codaient des nombres __normaux__, ce qui n'est pas forcé d'être le cas dans les faits.
La suite du document explique pourquoi est-ce que le FPU Configuration Unit prend (ou ne prend pas) en charge ces différents types de nombres, allant des nombres __normaux__ jusqu'aux nombres __dénormaux__.

Pour commencer, comprenons que chacun de ces types de nombres se différencie des autres par le biais d'une plage de codage qui lui est propre dans les champs d'exposant et de mantisse tronquée.
Il y a bien évidemment d'autres différences fondamentale qui existent entre ces types de nombres, nous parlerons de tout cela dans des chapitres dédiés.

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
