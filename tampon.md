// Maj post démonstration: Le Binary Unsigned -> L'écriture scientifique binaire -> Le standard IEEE-754 -> L'encodage des champs d'un nombre IEEE-754 -> Le codage des champs d'un nombre IEEE-754 -> Ordre de traitement des champs + Les points terminaux et non terminaux








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

$${\color{Cyan}1}0{\color{Red}3} = \left({\color{Cyan}1} \times 10^2 + 0 \times 10^1 + {\color{Red}3} \times 10^0\right)$$

Le chiffre des unités ${\color{Red}3}$ est facteur de $10$ à la puissance $0$, le chiffre des dizaines $0$ est facteur de $10$ à la puissance $1$, tandis que le chiffre des centaines ${\color{Cyan}1}$ est le facteur de $10$ à la puissance $2$.
Remarquons que la valeur des puissances commence à partir de $0$ pour le chiffre des unités, et augmente plus ou moins en fonction de la position de son chiffre.
Mais ceci n'est pas propre à la base décimale.

La base binaire (ou base $2$) permet également de représenté des nombres d'une façon équivalente, mais comme mentionné plus haut, qui dit nombre dit encodage de nombre.
Vous l'aurez compris, l'encodage qui permet de représenté des nombres de la sorte est le ___Binary Unsigned___.
Un champs binaire en _Binary Unsigned_ utilise chaque bit le composant comme étant facteur d'une puissance de $2$ positive ou nul.
Le nombre ainsi représenté se calcul par la somme de chacun de ces produits.
De plus, la représentation des nombres n'est pas " _signé_ ", ce que veut dire qu'il n'y a pas de bit permettant le codage explicite d'un signe $\pm$, et ainsi les nombres codés sont naturellement positifs.
Regardez ci-dessous:

$$103 = 1100111_2 = 1 \times 2^6 + 1 \times 2^5 + 0 \times 2^4 + 0 \times 2^3 + 1 \times 2^2 + 1 \times 2^1 + 1 \times 2^0$$

Nous ne parlons pas de bit des unités, des dizaines, centaines ou encore milliers dans un champs binaire, mais il y a ce que l'on appelle le " _poids_ " d'un bit qui est à prendre en compte.
Plus un bit est facteur d'une grande puissance de $2$, plus le poids de ce bit est important dans la représentation d'un nombre.
Par exemple, le bit qui multiplie $2^6$ fait fluctuer la valeur du nombre représenté de $64$, contre une fluctuation de seulement $8$ pour le bit facteur de $2^3$.
Le bit qui multiplie $2^6$ est donc d'un poids (d'une importance) supérieur à celui qui multiplie $2^3$.

Par conséquent, il est très commun d'indicé les bits d'un champs binaire par leurs poids, le bit de poids $3$ fait référence au bit qui multiplie la puissance $2^3$.
Il s'évère que certains termes sont rentrés dans le langage commun et représentent des bits d'un poids bien précis à l'intérieur d'un champs.
Par exemple, le terme de _LSB_ pour _Least Significant Bit_ fait référence au bit de poids le plus faible d'un champs binaire, le bit de poids $0$ pour tout nombre représenté en _Binary Unsigned_.
Il existe aussi le terme de _MSB_ pour _Most Significant Bit_ qui fait référence au bit de poids le plus fort d'un champs binaire.
D'autres termes comme _LSB1_ pour le bit à $1$ de poids le plus faible, ou encore _MSB1_ pour le bit à $1$ de poids le plus fort, existent aussi.

Pour finir, la représentation d'un nombre _Binary unsigned_ peut se calculé par la somme de chacun de ces produits $\left(bit \times power \ of \ 2\right)$ ce qui donne lieu à des calculs inutiles.
La valeur d'un bit vaut soit $0$, soit $1$.
Lorsqu'un bit est à $0$ le résultat du produit l'est aussi et il est donc souvent dit qu'un nombre codé en _Binary Unsigned_ à une valeur équivalente à la somme des poids des bits à $1$.

### Une propriété importante de l'encodage Binary Unsigned

Avec un champs dont l'encodage est en __Binary Unsigned__, nous avons la certitude que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des valeurs de chaque bit de poids inférieur à $i$.
Prenons comme exemple un champs binaire de $8$ bits, pour lequel nous n'allons faire attention qu'à un seul bit, celui de poids $6$ admettons.
Disons que ce champs ressemble à ceci $01111111_{2}$, la valeur du nombre représenté est $127$ et le bit qui nous intéresse se trouve être le __MSB1__ du champs.
La somme des valeurs des bits de poids inférieur au bit de poids $6$ donne un résultat strictement inférieur à $1 \times 2^6 = 64$.
Cela est systèmatiquement vrai, même lorsque tout les bits de poids inférieur sont à $1$, comme dans cet exemple.

$$\left(1 \times 2^6 = 64\right) \gt \left(63 = 1 \times 2^5 + 1 \times 2^4 + 1 \times 2^3 + 1 \times 2^2 + 1 \times 2^1 + 1 \times 2^0\right)$$

Encore une fois, ceci n'est pas propre à la base binaire mais à _n'importe quel base numérique_ (modulo quelques variation pour chaque base).






# L'écriture scientifique binaire

Pour commencer, qu'est ce que l'_écriture scientifique_?

L'écriture scientifique est une façon comme une autre de représenté des nombres.
Elle existe pour chaque base numérique, comme pour la base binaire par exemple.
Le but de cette notation scientifique des nombres est double, le permier objectif est de ne pouvoir représenté les nombres que d'une seule façon, le second permet de simplifier la lecture des grands nombres.
Par exemple, en ___écriture scientifique décimale___ nous pouvons représenté facilement la vitesse approchée d'un photon dans de le vide en m/s $+3.0 \times 10^8$.
Cela peut paraitre plus compliqué à interprété de prime abord, mais comme nous allons le voir, en réalité tout n'est qu'une histoire de vrigule et de puissance.

Le fonctionnement de l'écriture scientifique ne change que peu lorsque nous passons d'une base numérique vers une autre.
Mais ce qui compose la notation scientifique ne change jamais:
  - Un ___signe___
  - Un ___significande___
  - Un ___multiplicande___

Dans l'exemple fournit ci-dessus, le _signe_ est le symbole $+$ qui indique si le nombre est positif ou négatif, le _significande_ (ou ___mantisse___) est la valeur $3.0$ et enfin le multiplicande est le nombre $10$ qui est élevé à la puissance $8$.
En notation scientifique décimale, le significande a une valeur comprise dans l'intervalle $\left[1;10\right[$, et le multiplicande est une puissance de $10$.
De manière plus général, pour une notation scientifique en base $N$, la valeur du significande ne peut être comprise qu'entre $\left[1;N\right[$ et le multiplicande est alors une puissance de $N$.

## Comprendre ce qu'est le significande

Pour écrire en notation scientifique le nombre flottant $F$, il faut partir de son codage en nombre à virgule flottante.

Pour la ___notation scientifique binaire___, le significande est un nombre réel dont la valeur est compris dans l'intervalle suivante $\left[1;2\right[$.
Ce qui veut dire que _la partie entière du significande doit être à_ $1$.
Pour obtenir un significande, _nous déplaçons la virgule du nombre_ $F$ de sa position initial, _jusque devant le MSB1 du nombre_.
Une exception est faite pour le cas où le nombre à virgule flottante $F$ est nul, le significande peut alors lui même être nul.
La raison est qu'il n'y a pas de bit à $1$ dans le champs binaire qui code le nombre flottant $F$ si il est nul, et donc il n'y a pas de _MSB1_.
Cependant, si le _MSB1_ est dans la partie entière du nombre $F$, cela insinue que $\left(F \ge 1\right)$ et que la virgule devra être déplacé vers la gauche (_si elle doit l'être_).
Au contraire, si le _MSB1_ est dans la partie fractionnaire de $F$ alors $\left(F \lt 1\right)$, et la virgule devra être déplacé vers la droite.

Mais le déplacement de la virgule d'un nombre engendre en binaire les même choses qu'en décimale.
Une _division par_ $N$ de $F$ dans le cas d'un déplacement de la virgule de $log_2\left(N\right)$ rangs vers la gauche, et _une multiplication_ de $F$ par le même facteur $N$ pour un décalage de la virgule de $log_2\left(N\right)$ rangs vers la droite.
__Le terme__ $N$ __qui multiplie ou divise__ $F$ __est systèmatiquement une puissance de 2, ce qui suit explique pourquoi__.

### La transformation d'un nombre à virgule flottante en un significande

Pour tout nombre à virgule flottante $F$, nous savons que la virgule se trouve entre le _LSB_ de la partie entière (le bit de poids $0$), et le _MSB_ de la partie fractionnaire (bit de poids $-1$).
Par conséquent, un déplacement de la virgule d'un rang vers la gauche force l'ancien bit de poids $1$ à devenir le bit de poids $0$, l'ancien bit de poids $0$ devient celui de poids $-1$, et celui qu'était de poids $-1$ devient le bit de poids $-2$, etc.
Pour le dire autrement, chaque bit de la partie entière comme de la partie fractionnaire du nombre $F$ voit son poids être _décrémenter_ de $1$. 

L'équation ci-dessous permet de calculer la transformation de n'importe quel nombre à virgule flottante $F$ en un significande.

$$\sum_{i = msb\left(F\right)}^{lsb\left(F\right)} \ \left(F_i \times 2^{\left(i+c\right)}\right) = \ F \times 2^c$$

Le terme $c$ représente le nombre de rang de décalage à induire sur la virgule du nombre $F$.
Qui plus est, lorsque $\left(c \lt 0\right)$ c'est que _la virgule doit être décalée vers la gauche_.
Inversement quand $\left(c \gt 0\right)$, c'est que le décalage de la virgule doit se produire _vers la droite_.

Essayons de comprendre pourquoi est ce qu'un décalage d'un rang vers la gauche de la virgule de $F \ \left(c = -1\right)$, divise la valeur de $F$ par $N = 2$ comme nous le démontre l'équation ci-dessus.
Nous savons que les nombres flottants ont une partie entière et une autre fractionnaire.
Ces deux parties utilisent chaque bit comme un facteur d'une puissance de $2$, des puissances _positives_ pour la partie entière et _négatives_ pour la partie fractionnaire.
Nous pouvons remarquer que dans le codage du nombre $F$, n'importe quel bit de poids $i$ est facteur d'une puissance de $2$ qui est deux fois plus grande que la puissance facteur du bit de poids $i - 1$.
Par exemple $\left(2^0 = 1\right)$ et $\left(2^{-1} = 0.5\right)$. 

Nous avons vu plus haut qu'avec un décalage d'un rang vers la gauche de la virgule de $F$, je cite "chaque bit de la partie entière comme de la partie fractionnaire de $F$ voit son poids être _décrémenter_ de $1$".
Ce qui veut dire que tout bit de $F$ passe de facteur de $2^i$ à $2^\left(i-1\right)$, et _chaque bit_ voit donc son poids être _divisé par_ $2$.
Le calcul de la valeur de $F$ passe de la somme des produits entre les bits de poids $i$ à $1$ et $2^i$, à la somme des produits entre les bits de poids $i$ à $1$ et $2^{\left(i-1\right)}$.
C'est pourquoi la valeur de $F$ est _divisé par_ $2$ avec un décalage d'_un rang vers la gauche_ de la virgule.

Maintenant que nous avons compris pourquoi un décalage d'un rang vers la gauche de la virgule de $F$, engendre une division par $2$ du nombre.
Découvrons désormais les effets d'un décalage d'un rang vers la droite de la virgule de $F$, sur la valeur du nombre lui même.
Dans le cas d'un tel décalage, chaque bit de la partie entière comme fractionnaire de $F$ voit son poids être _incrémenter_ de $1$.
Par conséquent, chaque bit de poids $i$ voit sa valeur doublé car $\left(2^{\left(i+1\right)} = 2 \times 2^i\right)$.
Après décalage de la virgule, nous calculons la valeur de $F$ comme la somme des produits entre les bits de poids $i$ à $1$ et $2^{\left(i+1\right)}$.
Exactement comme le fait la partie gauche de l'équation ci-dessus pour $c = 1$.
Le côté droit de cette équation nous informe que cette opération revient à _multiplié_ le nombre à virgule flottante $F$ par $2$.

Nous sommes désormais capable de comprendre toutes les implications d'un décalage de la virgule du nombre $F$, sur la valeur du nombre lui même. 
Prenons un décalage de $c = x$ comme exemple, où $x \in \left[-\infty ;+\infty \right]$.
Si $\left(c \gt 0\right)$ alors nous n'avons qu'à décalé la virgule de $c$ fois $1$ rang _vers la droite_.
Vu que nous savons qu'un décalage de la virgule de $1$ rang vers la droite engendre une multiplication par $2$ du nombre $F$, alors après $c$ décalages de $1$ rang, nous aurons multiplié $F$ par $2^c$.
Au contraire si $\left(c \lt 0\right)$, alors nous devrons décalé la virgule de $\vert \ c \ \vert$ fois $1$ rang _vers la gauche_.
Un décalage de la virgule d'un rang vers la gauche revient à divisé par $2$ la valeur de $F$.
Par conséquent, après $\vert \ c \ \vert$ décalages de la virgule de $1$ rang vers la gauche, nous aurons divisé $\vert \ c \ \vert$ fois la valeur de $F$ par $2$.
Ou autrement dit, $F$ aura été divisé par $2^{\vert \ c \ \vert}$.

Voilà pourquoi n'_importe quel décalage de la virgule_ d'un nombre flottant $F$, engendre une _multiplication_ ou une _division_ du nombre par une puissance de $2$.

### Détail sur le fonctionnement de l'équation

Je parle de divisé par une puissance de $2$ le nombre flottant $F$ dans le cas d'un décalage de sa virgule vers la gauche, mais malgré que l'équation fonctionne elle ne fait pas usage de division.
Essayons de comprendre pourquoi.
Le côté droit de cette équation $\left(F \times 2^c\right)$ semble être approprié pour un décalage de la virgule vers la droite du nombre $F$.
Je rappelle qu'après un déplacement de la virgule de $c$ rangs vers la droite, le nombre flottant $F$ est mulitplié par une puissance de $2$ qui vaut $2^c$, où $\left(c \gt 0\right)$.

En parallèle, l'équation utilise des valeurs négatives de $c$ lors d'un décalage de virgule vers la gauche.
Pourtant le plus intuitif aurait été de définir les même valeurs de $c$ peu importe la direction des décalages, accompagné de deux équations différentes.
Dans ce qui suit, je vais donc faire usage de $\left(c \gt 0\right)$ pour des décalages vers la gauche.
Les explications suivantes vont nous permettre de comprendre pourquoi il vaut mieux que $c$ soit négatif lors des décalages de la virgule vers la gauche.

Pour un décalage de la virgule de $c$ rangs vers la gauche, nous devrions divisé $c$ fois le nombre $F$ par $2$, ou autrement dit divisé $F$ par $2^c$.
Initialement, nous pourrions penser avoir besoin d'une équation ressemblant à celle-ci $\sum_{i=msb\left(F\right)}^{lsb\left(F\right)} \left(F_i \times 2^{\left(i-c\right)}\right) = \ F \div 2^c$.
J'attire l'attention sur le fait que côté droit de l'équation puisse être modifié pour $\left(F \times \left(1 \div 2^c\right)\right)$.
Mais ce n'est pas fini, car pour trouver l'inverse d'une puissance de $2$ tel que $2^c$, nous pouvons simplement appliqué l'opposé de l'exposant $c$ à la base $2$.
En gros, $1 \div 2^c = \ 2^{-c}$.
Je me permet de rappellé que $\left(c \gt 0\right)$.
Finalement, nous pouvons alors remplacé le membre droit $\left(F \times 2^{-c} \right)$ par $\left(F \times 2^c\right)$, à la condition qu'un décalage de $x$ rangs vers la gauche de la virgule de $F$, soit représenté dans l'équation par une valeur de $c = -x$ et non $c = x$.

Avec $\left(F \times 2^c\right)$ nous retrouvons bien le membre droit de notre équation initial, à la condition que $c$ puisse devenir négatif.
De plus, nous comprenons aussi que le calcul de l'exposant $\left(i + c\right)$ du membre gauche, produit l'équivalent de $\left(i - c\right)$ dans le cas d'un décalage de la virgule de $F$ vers la gauche, car $\left(c \lt 0\right)$.

C'est la raison pour laquelle il vaut mieux que $c$ soit négatif lors d'un décalage vers la gauche de la virgule.
Ceci nous permet de ne faire usage que d'une seule équation pour tout les sens de décalage. 

## Le multiplicande

Nous savons désormais comment obtenir un significande.
Le problème c'est que tout nombre flottant $F$ dont la valeur n'est pas comprise dans l'intervalle $\left[1;2\right[$ doit forcémment subir un décalage de sa virgule, et donc voir sa valeur être multiplié ou divisé par une puissance de $2$.
Pour contrecarrer ce problème, l'écriture scientifique défini ce qui s'appelle un multiplicande. 
Le rôle du multiplicande est de nous permettre de retrouver la valeur initial de $F$ après qu'il ait été transformé en un significande $S$.

En reprenant les termes de l'aquation précédente, dans le cas d'une transformation d'un nombre $F$ en un significande $S$ avec $\left(c \lt 0\right)$, nous savons que la virgule du nombre $F$ a été déplacée vers la gauche et donc sa valeur divisé.
Pour retrouver le nombre à virgule flottante initial $F$, il suffit alors de _multiplié_ le significande $S$ par la puissance de $2$ qui a divisé le nombre $F$.
Comme l'explique le chapitre précédent, l'équation effectue cependant la division du nombre $F$ comme ceci $\left(F \times 2^c\right)$, ce qui est équivalent à $\left(F \div \left(1 \div 2^c\right)\right)$.
Il faut donc multiplié le significande $S$ de la manière suivante afin de retrouver le nombre $F$ d'origine $\left(S \times \left(1 \div 2^c\right)\right)$, ce qui est équivalent à $\left(S \times 2^{-c}\right)$.

Prenons désormais le cas d'une transformation du nombre à virgule flottante $F$ en un significande $S$, avec $\left(c \gt 0\right)$.
La virgule du nombre $F$ est déplacée vers la droite et le nombre est alors multiplié par une puissance de $2$.
Ce coup ci, pour retrouver la valeur initial de $F$ il faudra _divisé_ la valeur du significande $S$ par la puissance de $2$ qui a multiplié le nombre $F$.
Nous savons que $\left(F \times 2^c\right)$, alors nous effectuons $\left(S \div 2^c\right)$ ce qui est équivalent à $\left(S \times \left(1 \div 2^c\right)\right)$ et donc à $\left(S \times 2^{-c}\right)$.

Peu importe la manière dont un significande peut être obtenu, il existe un unique calcul $\left(S \times 2^{-c}\right)$ permettant de retrouver la valeur du flottant $F$ à partir de son significande.
Le multiplicande est le facteur $2^{-c}$. 
Mais il est possible d'exprimé les choses différemment, ce qui va nous servir pour plus tard.

Pour la transformation d'un nombre flottant $F$ en un significande $S$, si la virgule de $F$ a été déplacée de $c$ rangs vers la droite, alors $\left(c \gt 0\right)$.
Pour retrouver depuis le significande $S$ la valeur initial du nombre flottant $F$, le multiplicande doit permettre de déplacé de $c$ rangs vers la gauche la virgule du significande.
Il suffit alors d'inversé le signe de $c$ pour changé la direction du décalage, comme ceci $\left(-\left(+c\right) = -c\right)$.
Le nombre de rang de décalage reste le même mais le sens de décalage passe de la droite avec $\left(c \gt 0\right)$, à la gauche avec $\left(c \lt 0\right)$.

Dans le cas contraire, la transformation du nombre flottant $F$ en un significande $S$ avec $\left(c \lt 0\right)$, engendre un déplacement de la virgule de $F$ de $\vert \ c \ \vert$ rangs vers la gauche.
Pour retrouver la valeur initial du nombre flottant $F$, il faudra alors que la virgule du significande $S$ soit décalée de $\vert \ c \ \vert$ rangs vers la droite.
Une nouvelle fois, la négation de $c$ permettra d'inversé le sens de décalage de la vrigule $\left(-\left(-c\right) = +c\right)$.

Nous comprenons donc que dans les faits déplacé la virgule du significande $S$ de $-c$ rangs, est équivalent à produire le calcul $\left(S \times 2^{-c}\right)$ qui permet de retrouver la valeur de $F$ (comme vu plus haut).




# Le standard IEEE-754

Après cette rapide introduction à l'encodage __Binary Unsigned__, passons au sujet suivant qui est l'encodage __IEEE-754__.

Notre circuit a pour fonction principal de produire des comparaisons entre deux nombres à virgule flottante, de standard _IEEE-754_.
Pour plus d'information sur le circuit électronique, jetez un oeil à la documentation du circuit.
La norme _IEEE-754_ défini trois éléments pour l'encodage de chaque nombre à virgule flottante:
  - Le ___bit de signe___
  - Un champs binaire d'___exposant___
  - Un champs binaire de ___mantisse tronquée___

Il existe plusieurs format de flottant défini par le standard IEEE-754.
Voici la disposition précise de chaque bit de chacun des champs pour le codage d'un nombre au format _Half Precision_, qui est a une taille de $16$ bits:

$$\left[S_{15}, \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

$S$: Sign bit,  $E$: Exponent, $T$: Truncated mantissa

Chaque indice (nombre) compris dans l'intervalle $\left[0;15\right]$ représente le poids d'un bit précis.
En effet, il est très commun d'indicé un bit d'un champs binaire par son poids.

Dans ce qui suit, nous allons nous intéresser aux encodages propre des champs binaires d'exposant et de mantisse tronquée.
Nous allons voir que les encodages des champs de mantisse tronquée et d'exposant partagent les même caractéristiques que le _Binary Unsigned_.
Ce qui permet de traité ces deux champs avec un même processus de calcul, ce qui se reflète sur l'architecture du circuit électronique.

### L'encodage par biais du champs d'exposant

__Le champs d'exposant utilise un encodage par biais__, ce dernier est assez simple à comprendre.
Enfaite, le champs d'exposant est un champs binaire pour lequel nous utilisons un encodage _Binary Unsigned_ qui code une valeur numérique $X$, comme nous l'avons vu précédemment.
A cela, il faut __ajouté ou déduire__ un biais $B$ (un nombre entier naturel) pour obtenir la valeur représenté par le champs binaire.
Dans les faits, la valeur que représente un champs d'exposant est alors issu du calcul $X - B$. 
Peu importe le format de flottant IEEE-754, le champs d'exposant a un biais $B$ qui se calcul de la manière suivante $\left(2^{\left(N-1\right)} - 1\right)$, où $N$ est le nombre de bits du champs d'exposant.

Etant donné que l'encodage par biais se repose sur le _Binary Unsigned_, un champs d'exposant partage alors les même propriétés que cet encodage.
Notamment le fait que dans le champs binaire, la valeur d'un bit à $1$ de poids $i$ soit strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.

### La mantisse tronquée, une histoire de puissance de 2

Comme nous venons de le voir avec le champs d'exposant, le standard IEEE-754 se base sur des champs aux encodages eux même déjà existant.
Je rappelle qu'il existe des nombres binaire à virgule fixe et à virgule flottante (ou nombre flottant).
La seule différence entre les deux est qu'un nombre à virgule flottante peut avoir une quantité variable de bit derrière sa virgule, au contraire d'un nombre à virgule fixe.
L'encodage du champs de mantisse tronquée tel que défini par le standard IEEE-754, se base sur l'encodage des nombres à virgule flottante.
Alors faisons un rapide rappel de cet encodage.

La partie entière d'un nombre flottant utilise du _Binary Unsigned_ pour être codé, tandis que la partie fractionnaire fait usage d'une version modifiée du _Binary Unsigned_.
Chaque bit de la partie fractionnaire est un ___facteur d'une puissance de 2 négative___, et non positive.
Ceci permet à la partie fractionnaire d'un nombre de représenté des valeurs dans l'intervalle $\left[0;1\right[$.
Le poids des puissances négatives de $2$ décroix en fonction de la position du bit.
Prenons l'exemple du nombre $3.875$.

$$3.875 = 11.111_2 = Integer \ Part\left(\left(1 \times 2^1\right) + \left(1 \times 2^0\right)\right) + Fractional \ Part\left(\left(1 \times 2^{-1}\right) + \left(1 \times 2^{-2}\right) + \left(1 \times 2^{-3}\right)\right)$$

Attention car dans les faits, le point dans l'écriture binaire de $3.875$ n'est pas réelement présent dans le codage du nombre, je l'ai rajouté pour simplifier la lecture.
Mais ce qu'il y a d'important à remarquer pour les bits de la partie fractionnaire d'un nombre à virgule flottante, c'est que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Imaginons que nous ayons nommés $F$ le nombre illustré ci-dessus, alors pour le bit à $1$ de poids $-1$ nous savons que $\left(1 \times 2^{-1}\right) \gt \sum_{i=-2}^{-3} \left(F_i \times 2^i\right)$.

Revenons-en au sujet de la mantisse tronquée désormais.
Plus bas, nous verrons comment le champs de mantisse tronquée code tout les bits de la partie entière, ainsi que de la partie fractionnaire d'un nombre à virgule flottante (lorsque c'est possible).
__Vu que la partie entière et fractionnaire d'un nombre à virgule flottante partagent les propriétés du _Binary Unsigned_, c'est aussi le cas de la mantisse tronquée elle même__.
Pour l'instant, c'est tout ce dont nous avons besoin de savoir.

Nous pouvons remarquer que le champs d'exposant et de mantisse tronquée partagent bel et bien les même propriétés que l'encodage _Binary Unsigned_, comme mentionné plus haut.
C'est dans la démonstration mathématique que nous verrons à quel point cela va nous être utile.







# L'ordre de traitement des champs d'opérande

Comme dit plus haut, la tâche primaire du circuit est de comparer deux nombres flottants IEEE-754 (de format ___Half Precision___ qui plus est), que nous nommerons $\alpha$ et $\beta$.
La comparaison en question est une vérification de la supériorité stricte de la valeur absolu de l'un de ces deux opérandes envers l'autre, admettons $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.
Etant donné que le circuit n'a besoin que de la valeur absolu des opérandes $\alpha$ et $\beta$, seul les $15$ bits de poids faible des opérandes sont utiles (le bit de signe est omis).

Il se trouve que le circuit traite les champs d'exposants (que nous nommerons) $E$ de $\alpha$ ainsi que de $\beta$, avant les champs de mantisse tronquée (que nous nommerons $T$) de ces même opérandes.
La raison en est que les champs d'exposant à eux seuls peuvent permettre au circuit d'atteindre un point terminal.
Le circuit électronique atteint un point terminal lorsqu'il est capable de générer le résultat d'une comparaison entre ses deux opérandes, sans avoir besoin d'attendre le traitement de l'entièreté de ses entrées.
Techniquement, un point terminal est atteint si $\left(E_{\alpha} \gt E_{\beta}\right)$ car l'opérande $\vert \ \alpha \ \vert$ est strictement plus grand que $\vert \ \beta \ \vert$, et inversement avec $\left(E_{\alpha} \lt E_{\beta}\right)$.
Nous verrons pourquoi dans le chapitre "_Les points terminaux et non terminaux_".

Cependant, le circuit peut aussi rencontrer un point non terminal.
Le circuit atteint ce dernier lorsque $\left(E_{\alpha} = E_{\beta}\right)$.
Comme nous le verrons plus tard, dans cette situation il n'y a rien dans les champs d'exposant des opérandes $\alpha$ et $\beta$ qui puisse permettre au circuit de déduire le résultat à générer.
Il faudra alors traité les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$, dans un second temps.

__Dans les chapitres suivant, je vais expliqué dans les fondements pourquoi est ce que les champs d'exposant sont traités en priorité par le circuit électronique__.
Pour cela, il va d'abord me falloir abordé le sujet de l'écriture scientifique binaire, alors commençons.

# Le codage d'un nombre flottant IEEE-754

Nous avons déjà parler de l'encodage IEEE-754 en introduction de ce document, plus particulièrement des trois éléments qui le compose ainsi que de l'encodage propre à ceux-ci.
Je rappelle que ces trois éléments sont des champs binaire où chacun code respectivement un __bit de signe__, un __exposant__ ainsi qu'une __mantisse tronquée__.
Par ailleurs, nous venons tout juste de voir dans le détails ce qu'est l'écriture scientifique biniaire d'un nombre, ce qui nous a permis de mettre en lumière ce qu'est un __significande__ (qui peut être appellé __mantisse__), ainsi qu'un __multiplicande__.
Il y a aussi le __signe__ $\pm$ du nombre représenté en écriture scientifique.

Chaque champs binaire de l'encodage IEEE-754, correspond (plus ou moins) à l'un des éléments de l'écriture scientifique binaire d'un nombre.
Par exemple, le champs du _bit de signe_ en IEEE-754 correspond assez logiquement au _signe_ de la notation scientifique binaire d'un nombre.
Nous ne reviendrons pas sur le sujet du bit de signe d'un nombre IEEE-754 car rappelons-le, le circuit électronique ne prend en charge que la valeur absolu (sans bit de signe) de ses opérandes.
Dans ce qui suit, nous allons parlé plus en profondeur du champs d'exposant ainsi que du champs de mantisse tronquée.
Nous avons déjà abordé le sujet de l'encodage de ces champs dans l'introduction, aussi je ne vais pas y revenir.

### Composition du champs de mantisse tronquée et du champs d'exposant

En reprenant ce que nous avons dit précédemment, le champs de la mantisse tronquée correspond au significande (mantisse) d'un nombre représenté en écriture scientifique binaire.
Faisons un rapide rappelle sur ce qu'est un significande.
Le significande $S$ est un nombre dont la valeur se situe dans l'intervalle $\left[1;2\right[$, il est souvent le résultat d'une transformation (un calcul) sur un nombre codé en virgule flottante $F$.
Enfaite, si $F \notin \left[1;2\right[$ alors le flottant sera divisé ou multiplié par une puissance de $2$ que nous nommons $N$.
Ceci nous permet d'obtenir le significande de la notation scientifique binaire de $F$. 

En somme, le significande est un nombre à virgule qui est donc composé d'une partie entière ainsi que d'une partie fractionnaire.
Vu que la valeur du significande doit être strictement inférieur à $2$ ou $10_2$ en binaire, ça veut dire que la partie entière ne peut être que à $0$ ou $1$.
Par conséquent la partie entière du significande n'est codé que sur un bit.
En soit, le bit de la partie entière ne peut pas non plus être à $0$, car il est nécessaire que la valeur du significande soit au moins de $1$.
__Donc le bit de la partie entière est à 1__.
En plus de cela, nous avons la partie fractionnaire du significande à prendre en compte (dont la valeur est logiquement strictement inférieur à $1$).
Finalement, la valeur du significande est la somme de la partie entière et de la partie fractionnaire.

La norme IEEE-754 défini un champs de mantisse tronquée de $x$ bits pour chaque format de nombre flottant. 
Prenons pour exemple le format ___Half Precision___ que gère la FPU Configuration Unit.
Il s'avère que le champs de la mantisse tronquée de ce format est établit à une taille de $10$ bits.
L'entièreté d'un significande $S$ pourrait être enregistré sur ces $10$ bits (si possible), mais ce n'est pas utiles car sa partie entière est toujours composé d'un seul et même bit à $1$, comme nous venons de le voir.
C'est pourquoi la norme ampute le significande (ou mantisse) de sa partie entière, indépendemment du format de flottant.
L'intérêt c'est que le champs champs de mantisse tronquée reste de la même taille ($10$ bits pour un _Half Precision_), tout en gagnant un bit de précision sur le codage d'un significande.
C'est d'ailleurs la raison derrière le nom de ce champs, qui code une mantisse ___tronquée___.

Le standard IEEE-754 défini également un champs d'exposant qui correspond _en partie_ au multiplicande de l'écriture scientifique binaire d'un nombre à virgule flottante $F$.
Je conseille une relecture du chapitre sur le multiplicande si un rafraichissement est nécessaire.
De ce chapitre j'en tire la phrase "_Le multiplicande est le facteur_ $2^{-c}$.".
En réalité le champs d'exposant d'un nombre IEEE-754 ne correspond qu'à l'exposant $-c$ d'un multiplicande, et non à la valeur de la puissance elle même. 
Un champs d'exposant ne code que la valeur $-c$, car il se base sur ce qui a été dit lors des deux derniers paragraphes du chapitre "_Le multiplicande_", afin de déplacé la virgule du significande du bon nombre de rang, ainsi que dans la bonne direction.

Le standard défini une taille de champs d'exposant propre à chaque format de flottant.
Dans le cas du format _Half Precision_, la taille du champs d'exposant est de $5$ bits.








# Les points terminaux et non terminaux

Pour finir, je vais enfin pouvoir expliqué pourquoi le circuit électronique traite les champs d'exposant $E$ des opérandes $\alpha$ et $\beta$, avant les champs de mantisse tronquée $T$ de ces même opérandes.

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
