Ce document contient la démonstration mathématique du circuit électronique ainsi que des explications sur les concepts les plus fondamentaux.

Le document se découpe en plusieurs parties:
  - L'encodage $Binary \ Unsigned$
  - Le standard IEEE-754
    - L'encodage par biais du champs d'exposant
    - La mantisse tronquée, une histoire de puissance de $2$
  - L'écriture scientifique binaire
    - Le significande
    - Le multiplicande
  - Codage des informations dans un nombre flottant IEEE-754
    - Composition du champs de mantisse tronquée et d'exposant
  - Les points terminaux et non terminaux
  - Démonstration mathématique
    - Traitement des champs d'exposant
    - Traitement des champs de mantisses tronquées

# L'encodage Binary Unsigned

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$.
Un champs binaire est quant à lui composé d'un ensemble de bit, c'est à dire d'un ensemble de $0$ et de $1$.

Ce sont ces champs binaire qui nous permettent de représenté des nombres dans un ordinateur, faire des calculs, mémorisé des résultats.
Il y a cependant plein de façon de représenté des nombres dans un champs binaire, nous parlons d'_encodage_ des nombres.
Tout les encodages ne se valent pas car il y en a des plus efficasse que d'autre en fonction des besoins, prenons un exemple.
Pour un champs de $16$ bits nous pourrions calculé la somme des bits pour représenter un nombre.
Avec cet encodage nous ne pourrions codé que des valeurs entre $0$ et $16$ inclus, ce qui n'est pas optimal pour bien des situations.
Mais parmis les multiples encodage existant, le plus connu de tous porte le nom de __Binary Unsigned__.

Le __Binary Unsigned__ utilise les mêmes primitives que la base décimale pour représenter des nombres, _des puissances_.
Pour représenter la valeur $103$ en décimale, nous décomposons en réalité chaque chiffre du nombre pour les multipliés avec une puissance de $10_{10}$ adéquat, ensuite nous faisons la somme des résultats de chaque produit.
Regardez ci-dessous.

$$103 = 1 \times 10^2 + 0 \times 10^1 + 3 \times 10^0$$

Veuillez noté que l'indice $X_{10}$ représente la base numérique dans laquelle le nombre $X$ qui précède est écrit.
Nous retrouvons chacun des chiffres du nombre entrain de multiplié une puissance de $10_{10}$.
Remarquons que le chiffre des unités $3$ est facteur de $10_{10}$ à la puissance $0$, le chiffre d'un ordre plus grand (celui des dizaines) est le facteur de $10_{10}$ à la puissance $1$ et enfin le chiffre des centaines multiplie $10_{10}$ à la puissance $2$.
La valeur des puissances croix en fonction de la position du chiffre, qui plus est, elle commence à $0$ au niveau des unités.

Ceci n'est pas propre à la base décimale mais à _n'importe quel base numérique_, comme la __base binaire__ par exemple.
La base binaire (ou base $2$) utilise les chiffres $0$ et $1$ plutôt que ceux de la base décimale allant de $0$ à $9$.
Par ailleurs, un tel chiffre s'appelle un _bit_, ce qui est la contraction de __Binary Digit__.
Aussi, les puissances de $10_{10}$ sont remplacés par des puissances de $2$.
Nous venons tout juste de décrire ce sur quoi se base l'encodage __Binary Unsigned__.

Un champs binaire pour lequel nous utilisons un encodage _Binary Unsigned_, associe une puissance de $2$ positive ou nulle à chaque bit du champs.
Le champs représente un nombre par la somme des produits entre la valeur de chaque bit et la puissance de $2$ associée.
Techniquement, vu que dans chacun des produits la puissance de $2$ est multiplié par son bit associé, qui est à $0$ ou à $1$, soit le résultat est nul ou soit il est équivalent à la puissance de $2$ associé au bit sous-jacent.
C'est pourquoi vous entenderez souvent dire (ici compris), qu'un champs en _Binary Unsigned_ représente un nombre par la somme des puissances de $2$ dont le bit est à $1$.
Les puissances de $2$ vont de $0$ à $N - 1$, où $N$ est le nombre de bits qui compose le champs binaire.
Par ailleurs, il est souvent dit que chaque bit a un "poids".
Le poids correspond à la valeur de la puissance de $2$ associée à chacun des bits d'un champs, plus cette puissance sera grande et plus un bit aura d'importance dans la représentation d'un nombre. 

$$ 101_2 = 1 \times 2^2 + 0 \times 2^1 + 1 \times 2^0$$

Il est aussi défini ce que nous qualifions de __LSB__ pour __Least Significant Bit__.
Ce terme désigne le bit de poids le plus faible d'un champs, en l'occurence le bit le plus à droite de l'illustration ci-dessus et dont la puissance est $2^0$.
Au contraire, le bit de poids le plus fort d'un champs binaire est ce que l'ont appelle le __MSB__ pour __Most Significant Bit__.
Dans l'illustration il s'agit du bit dont la puissance est $2^2$, ou autrement dit le bit de poids $2$.
Il est aussi possible de faire spécifiquement référence au bit à $1$ de poids le plus faible d'un champs avec le terme __LSB1__, ou à celui de poids le plus fort avec __MSB1__.

### Une propriété importante de l'encodage Binary Unsigned

Avec un champs dont l'encodage est en __Binary Unsigned__, nous avons la certitude que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des valeurs de chaque bit de poids inférieur à $i$.
Prenons comme exemple un champs binaire de $8$ bits, pour lequel nous n'allons faire attention qu'à un seul bit, celui de poids $6$ admettons.
Disons que ce champs ressemble à ceci $01111111_{2}$, la valeur du nombre représenté est $127$ et le bit qui nous intéresse se trouve être le __MSB1__ du champs.
La somme des valeurs des bits de poids inférieur au bit de poids $6$ donne un résultat strictement inférieur à $1 \times 2^6 = 64$.
Cela est systèmatiquement vrai, même lorsque tout les bits de poids inférieur sont à $1$, comme dans cet exemple.

$$\left(1 \times 2^6 = 64\right) \gt \left(63 = 1 \times 2^5 + 1 \times 2^4 + 1 \times 2^3 + 1 \times 2^2 + 1 \times 2^1 + 1 \times 2^0\right)$$

Encore une fois, ceci n'est pas propre à la base binaire mais à _n'importe quel base numérique_ (modulo quelques variation pour chaque base).

# Le standard IEEE-754

Après cette rapide introduction à l'encodage __Binary Unsigned__, passons au sujet suivant qui est l'encodage __IEEE-754__.

Notre circuit a pour fonction principal de produire des comparaisons entre deux nombres à virgule flottante, de standard _IEEE-754_.
Pour plus d'information sur le circuit électronique, jetez un oeil à la documentation du circuit.
La norme _IEEE-754_ défini trois éléments pour l'encodage de chaque nombre à virgule flottante:
  - Le ___bit de signe___
  - Un champs binaire d'___exposant___
  - Un champs binaire de ___mantisse tronquée___

Dans ce qui suit, nous allons nous intéresser aux encodages propre des champs binaires d'exposant et de mantisse tronquée.
Nous allons voir que les encodages des champs de mantisse tronquée et d'exposant partagent les même caractéristiques que le _Binary Unsigned_.
Ce qui permet de traité ces deux champs avec un même processus de calcul, ce qui se reflète sur l'architecture du circuit électronique.

### L'encodage par biais du champs d'exposant

__Le champs d'exposant utilise un encodage par biais__, ce dernier est assez simple à comprendre.
Enfaite, le champs d'exposant est un champs binaire pour lequel nous utilisons un encodage _Binary Unsigned_ qui code une valeur numérique $X$, comme nous l'avons vu précédemment.
A cela, il faut __ajouté ou déduire__ un biais $B$ (un nombre entier naturel) pour obtenir la valeur représenté par le champs binaire.
Dans les faits, la valeur que représente le champs d'exposant est alors issu du calcul $X - B$. 
Peu importe le format de flottant IEEE-754, le champs d'exposant a un biais $B$ qui se calcul de la manière suivante $\left(2^{\left(N-1\right)} - 1\right)$, où $N$ est le nombre de bits du champs d'exposant.

Etant donné que l'encodage par biais se repose sur le _Binary Unsigned_, le champs d'exposant partage alors les même propriétés que cet encodage.
Notamment le fait que dans le champs binaire, la valeur d'un bit à $1$ de poids $i$ soit strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.

### La mantisse tronquée, une histoire de puissance de 2

Comme nous venons de le voir avec le champs d'exposant, le standard IEEE-754 se base sur des encodages eux même déjà existant.
Je rappelle qu'il existe des nombres à virgule fixe et à virgule flottante (ou nombre flottant).
La seule différence entre les deux est qu'un nombre à virgule flottante peut avoir une quantité variable de bit derrière sa virgule, au contraire d'un nombre à virgule fixe.
L'encodage du champs de mantisse tronquée tel que défini par le standard IEEE-754, se base sur l'encodage des nombres à virgule flottante.
Alors faisons un rapide rappel que cet encodage.

La partie entière d'un nombre flottant utilise du _Binary Unsigned_ pour être codé, tandis que la partie fractionnaire fait usage d'une version modifiée du _Binary Unsigned_.
Chaque bit de la partie fractionnaire est un ___facteur d'une puissance de 2 négative___ et non positive.
Ceci permet de représenté des valeurs dans l'intervalle $\left[0;1\right[$.
Le poids des puissances négatives de $2$ décroix en fonction de la position du bit.
Prenons l'exemple du nombre $3.75$.

$$3.75 = 11.11_2 = Integer \ Part\left(\left(1 \times 2^1\right) + \left(1 \times 2^0\right)\right) + Fractional \ Part\left(\left(1 \times 2^{-1}\right) + \left(1 \times 2^{-2}\right)\right)$$

Attention car dans les faits, le point dans l'écriture binaire du nombre $3.75$ n'est pas réelement présent dans le codage du nombre.
Ce qu'il y a d'important à remarquer pour la partie fractionnaire d'un nombre à virgule flottante, c'est que la valeur d'un bit à $1$ de poids $i$ est toujours strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Autrement dit, pour le bit à $1$ de poids $i$ du nombre fractionnaire $F \in \left[0;1\right[$, alors $\left(1 \times 2^i\right) \gt \left(\sum_{i=i-1}^{lsb\left(F\right)} \left(F_i \times 2^i\right)\right)$.

Revenons-en au sujet de la mantisse tronquée désormais.
Plus bas, nous verrons comment le champs de mantisse tronquée code tout les bits de la partie entière ainsi que de la partie fractionnaire d'un nombre à virgule flottante (lorsque c'est possible).
__Vu que la partie entière et fractionnaire d'un nombre à virgule flottante partagent les propriétés du _Binary Unsigned_, c'est aussi le cas de la mantisse tronquée elle même__.
Pour l'instant, c'est tout ce qui nous importe de comprendre.

Nous pouvons remarquer que le champs d'exposant et de mantisse tronquée partagent bel et bien les même propriétés que l'encodage _Binary Unsigned_, comme mentionné plus haut.
C'est dans la démonstration mathématique que nous verrons à quel point cela va nous être utile.

# L'ordre de traitement des champs d'opérande

Comme dit plus haut, la tâche primaire du circuit est de comparer deux nombres flottants ___Half Precision___ (d'une taille de $16$ bits), nous les nommerons $\alpha$ et $\beta$.
La comparaison en question est une vérification de la supériorité stricte de la valeur absolu de l'un de ces deux opérandes envers l'autre, admettons $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.
Etant donné que le circuit n'a besoin que de la valeur absolu des opérandes $\alpha$ et $\beta$, seul les $15$ bits de poids faible sont utiles (le bit de signe est omis).

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

# L'écriture scientifique binaire

Pour commencer, qu'est ce que l'_écriture scientifique_?

L'écriture scientifique est une manière de représenté les nombres.
Elle existe pour chaque base numérique, comme pour la base binaire.
Le but de cette notation scientifique des nombres est double, le permier objectif est de ne pouvoir représenté un nombre que d'une seule façon, le second permet de simplifier la lecture des grands nombres.
Par exemple, en ___écriture scientifique décimale___ nous pouvons représenté facilement la vitesse approchée de la lumière en km/s $+3.0 \times 10^5$.
Cela peut paraitre plus compliqué à interprété de prime abord, mais comme nous le verrons dans les chapitres ci-dessous, en réalité tout n'est qu'une histoire de vrigule et de puissance.

Le fonctionnement de l'écriture scientifique ne change que peu lorsque nous passons d'une base numérique vers une autre.
La composition de la notation scientifique d'un nombre reste cependant la même peut importe la base numérique:
  - Le ___signe___
  - Un ___significande___
  - Un ___multiplicande___

Dans l'exemple fournit ci-dessus, le signe est évidemment le symbole $+$ qui indique si le nombre est positif ou négatif, la valeur $3.0$ est le significande qui peut également être appellé ___mantisse___, et enfin le multiplicande qui est la base $10$ élevé à la puissance $5$.
En notation scientifique décimale, le significande peut être toute valeur comprise dans l'intervalle $\left[1;10\right[$, et le multiplicande est une puissance de $10$.
De manière général, pour une notation scientifique en base $N$.
Le significande ne peut être compris qu'entre $\left[1;N\right[$, et le multiplicande est alors une puissance de $N$.
C'est ce que nous allons respectivement voir dans les chapitres du nom de "_Le significande_" et "_Le multiplicande_" qui suivent.

Pour terminer, précisons que si nous parlons de l'écriture scientifique ce n'est pas pour rien.
Nous verrons d'ici quelques chapitre que chacun des éléments de l'encodage _IEEE-754_ ayant été défini plus tôt, correspond à l'un des éléments de l'écriture scientifique binaire d'un nombre.

## Le significande

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
Finalement, nous pouvons alors remplacé le membre droit $\left(F \times 2^{-c} \right)$ par $\left(F \times 2^c\right)$, à la condition qu'un décalage de $x$ rangs vers la gauche de la virgule de $F$, soit représenté dans l'équation par une valeur de $c = -x$.

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

# Démonstration mathématique

Nous pouvons désormais plongé dans le coeur de ce document, la démonstration mathématique du circuit.
Il y a une documentation dédié au circuit pour comprendre son utilité ainsi que son architecture, si vous souhaitez plus d'information concrète sur ce dernier je vous conseille de la lire.

Je rappelle que le circuit se nomme __FPU Configuration Unit__.
En bref, il génère deux bits de sortie.
L'un pour configuré un circuit soustracteur de nombre flottant, et l'autre pour la sortie même de cette unité de calcul.
A cette fin, le circuit reçoit deux opérandes IEEE-754 en valeur absolu (au format _Half Precision_ dans le cas du circuit schématisé), que nous nommerons $\alpha$ et $\beta$.
Pour le circuit, la génération de ses deux bits de sortie passe par une comparaison entre les deux opérandes $\alpha$ et $\beta$.
La démonstration mathématique ne va abordé que la logique de comparaison du circuit, ce qui représente la plus grosse partie de la logique du circuit tout de même.
Pour expliqué le reste de la logique, il faudrait en dire plus sur le circuit, ce qui est le cas de la documentation vers laquelle je vous renvoie une nouvelle fois.
__En outre, tout ce qui va suivre de la démonstration se base sur la verification de la condition suivante__ $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.

### Rapide survol du format Half Precision

L'encodage IEEE-754 défini trois éléments pour chaque format de flottant pris en charge par la norme, le _bit de signe_, le _champs d'exposant_ ainsi que le _champs de mantisse tronquée_.
Voici la disposition précise de chaque bit de chacun de ces champs pour le codage d'un nombre _Half Precision_ ($16$ bits):

$$\left(1\right) \quad \left[S_{15}, \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

$S$: Sign bit,  $E$: Exponent, $T$: Truncated mantissa

Chaque indice (nombre) compris dans l'intervalle $\left[0;15\right]$ représente le poids d'un bit précis.
En effet, il est très commun d'indicé les bits d'un champs binaire par leur poids.
Cependant, dans la démonstration mathématique nous ne considérerons que les bits dont les indices sont dans l'intervalle $\left[0;14\right]$.
N'oublions pas que le circuit ne se soucis que de la valeur absolu des opérandes $\alpha$ et $\beta$ qu'il reçoit en entrée, donc le bit de signe de ces opérandes (bit de poids $15$) est omis.

### Définition de quelques opérations fondamentales à la démonstration 

Pour commencer, l'opération __Write__ sert de fonction d'affectation.
L'argument sur le paramètre $\left(y\right)$ est copié dans l'argument sur le paramètre $\left(x\right)$.
Il n'y a pas de taille ni de "type" d'argument, cette fonction est abstraite et vise simplement à faire comprendre qu'il faut retenir $\left(y\right)$ dans $\left(x\right)$.

$$Write \ \left(x, \ y\right) \rightarrow \ x \ := \ y$$

Passons désormais à l'opération logique __Nimply__.
Nous formalisons cette opération en tant que fonction, cette dernière n'étant pas très connu elle ne possède pas son propre symbole calculatoire.
C'est une opération de logique bit à bit, par conséquent, la fonction ne prend que deux bits d'opérande comme paramètre.
Elle ne retourne un $1$ que si le bit sur son paramètre $\left(y\right)$ vaut $0$ lorsque celui sur $\left(x\right)$ vaut $1$, autrement l'opération retourne $0$.
Par ailleurs, en électronique cette opération ce décline directement en une porte logique.

$$Nimply \ \left(x, \ y\right) \rightarrow \ x \ \wedge \ \overline{y}$$

De plus, $\overline{y}$ est l'opération logique __Not__ qui inverse un bit en son opposé $\left(1 = \overline{0}\right)$ ou $\left(0 = \overline{1}\right)$.
Cette opération possède également sa propre porte logique, qui est d'ailleurs assez élémentaire pour la conception de circuit électronique en tout genre.

Pour finir, le symbole $\wedge$ est l'opération logique __And__ qui ne retourne un bit à $1$ que lorsque ses deux bits d'opérande le sont aussi, sinon $0$.
Tandis que le symbole $\vee$ est l'opération logique __Or__ qui ne retourne un $0$ que si ses deux bits d'opérande le sont également, autrement elle renvoie un $1$.
Ce sont des opérations de logique bit à bit qui ne manipulent donc que des bits en entrée, je le précise une nouvelle fois.

# Le traitement des champs d'exposant

$$\forall \ i \in \left[10;14\right], \quad Write \ \left(\tau_i, \ Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)\right)$$

Nous effectuons l'opération logique $Nimply$ sur tout les bits de poids $i$ des champs d'exposant $E$ des opérandes $\alpha$ et $\beta$.
La variable $\tau$ (tau) est un champs binaire de $15$ bits, dont les poids vont de $0$ à $14$.
Chaque bit de résultat d'une opération $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$ pour $i \in \left[10;14\right]$, est inscrit dans $\tau_i$.
Ce qui veut dire que $\tau \in \left[10;14\right]$ correspond aux bits de résultat des opérations $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$.

### L'opération logique Nimply

L'opération logique $Nimply$ a déjà été défini plus haut dans la section "_Définition de quelques opérations fondamentales à la démonstration_".
Selon cette définition, nous observons que le bit de résultat d'une telle opération ne peut être à $1$ que lorsque le bit sur le paramètre $\left(x\right)$ est lui même à $1$, et que celui sur $\left(y\right)$ est à $0$.
Pour tout les autres cas, si $\left(x = y\right)$ ou $\left(x \lt y\right)$ alors le bit de résultat sera $0$.
Dans la documentation du circuit électronique, nous définissons le terme de "___zéro anonyme___" pour désigné tout bit de résultat à $0$, provenant d'une opération logique $Nimply$.

### Qu'est ce qu'un zéro anonyme?

Nous venons de voir à quoi correspond le terme de "_zéro anonyme_".
Cependant, nous ne connaissons pas la raison derrière un nom aussi bizarre.

Nous avons dit plus haut que l'opération logique $Nimply$ pouvait être facilement décliné en une porte logique (un petit circuit électronique).
Dans les faits, le FPU Configuration Unit se sert des sorties de ses portes logiques $Nimply$, pour faire des déductions sur les entrées de celle-ci.
La démonstration mathématique en fait autant.
Pourtant, lorsqu'une opération logique $Nimply$ renvoie un bit de résultat à $0$, il nous est impossible de savoir si les deux bits d'opérandes sont identiques $\left(x = y\right)$, ou si le bit sur le paramètre $\left(y\right)$ est strictement supérieur à celui sur $\left(x\right)$.
Autrement dit, un $0$ en guise de bit de résultat d'une opération logique $Nimply$ rend anonyme les bits d'opérande, d'où le terme de "_zéro anonyme_".
En bref, les _zéros anonymes_ posent un problème que nous allons devoir prendre en charge dans la démonstration mathématique.

Petite précision par ailleurs, il existe en réalité des _zéros anonymes_ dit ___capitaux___, et d'autre ___non capitaux___.
Nous allons définir ci-bas la différence entre ces deux types là.

## Les deux facettes des zéros anonymes (introduction)

Revenons à l'expression qui figure au tout début du chapitre "_Le traitement des champs d'exposant_".
Chaque bit $\tau_i$ est le résultat de $\ Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)$ pour le poids $i \in \left[10;14\right]$.
Si $\left(\tau_i = 1\right)$ alors nous savons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$, ou autrement dit $\left(E_{\beta i} \times 2^i\right) \ \gt \ \left(E_{\alpha i} \times 2^i\right)$.

De plus, rappellez vous du chapitre du nom de "_L'encodage par biais du champs d'exposant_".
Dans ce chapitre, il est dit que la valeur d'un bit à $1$ de poids $i$ d'un champs d'exposant $E$, est _inconditionnellement_ supérieur à la somme de la valeur de chacun des bits de poids inférieur à $i$.
En prenant au pied de la lettre ce que nous venons tout juste de dire, pour $i \in \left[11;14\right]$ si $\left(\beta_i = 1\right)$ nous obtenons alors $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{\sigma=i-1}^{10} \ \left(E_{\beta\sigma} \times 2^{\sigma}\right)$.
Ceci peut également être arrangé au champs d'exposant $E_{\alpha}$, alors $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{\sigma=i-1}^{10} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$. 

Au final, nous pouvons en conclure que si $\left(\tau_i = 1\right)$ alors $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.

Ce qui vient d'être dit ci-dessus ne démontre pas pour autant que $\left(E_{\alpha} \lt E_{\beta}\right)$.

### Les zéros anonymes non capitaux

Prenons le cas de $\left(\tau_i = \tau_{14} = 1\right)$.
Dans se cas spécifique, nous avons affaire a un point terminal car $\left(E_{\alpha} \lt E_{\beta}\right)$.
Ce point terminal est atteint par le fait que $\left(E_{\beta 14} \times 2^{14}\right) \ \gt \ \sum_{i=14}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.
Sachant que $E_{\alpha 14}$ et $E_{\beta 14}$ sont les _MSB_ respectifs des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.

En bref, nous comprenons que même si $\left(\tau_{\left(i-1\right)} = \tau_{13} = 0\right)$, alors ce _zéro anonyme_ ne changera rien au fait que $\left(E_{\alpha} \lt E_{\beta}\right)$.
La raison en est que l'inéquation qui figure ci-dessus reste valide qu'importe la valeur du bit $E_{\alpha 13}$, qu'il soit à $0$ ou à $1$.

De manière plus général, nous pouvons en déduire que tout _zéro anonyme_ compris dans l'intervalle $\left[10;14\right[$ et de poids inférieur au _MSB1_ de $\tau \in \left]10;14\right]$, est ___non capital___.

Grâce au chapitre "_Les points terminaux et non terminaux_", nous savons que si $\left(E_{\alpha} \lt E_{\beta}\right)$ alors $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$.
N'oublions pas que la démonstration mathématique du circuit se base sur la vérification de la condition suivante $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$, qui se solde donc par un échec.

### Les zéros anonymes capitaux

Prenons désormais le cas dans lequel le poids du _MSB1_ de $\tau$ est $\tau_i$ pour $i \in \left[10;13\right]$.
Dans cette situation, nous sommes certains qu'il y a _au moins_ le bit $\tau_{\left(i+1\right)}$ qui est d'un poids supérieur à $\tau_i$.
Par définition, $\tau_{\left(i+1\right)}$ et les autres bits de poids supérieur à $\tau_i$ (si il y en a), sont des _zéros anonymes_.
Par exemple pour $\tau_{\left(i+1\right)}$, nous ne savons pas si $\left(E_{\alpha\left(i+1\right)} = E_{\beta\left(i+1\right)}\right)$ ou si $\left(E_{\alpha\left(i+1\right)} \gt E_{\beta\left(i+1\right)}\right)$.
Il en va de même pour tout autre bit de poids supérieur à $\tau_i$.

Ceci pose problème car les _zéros anonymes_ de poids supérieur à $\tau_i$ sont décisifs dans le processus de génération d'un point terminal.
A ce stade nous savons que $\left(\tau_i = 1\right)$, par conséquent $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.
Mais comme mentionné plus haut ceci ne suffit pas à dire que $\left(E_{\alpha} \lt E_{\beta}\right)$, voyons pourquoi.

Pour commencer, calculons la somme $\lambda$ (lambda) de la valeur des bits de poids supérieur à $i$ du champs d'exposants $E_{\alpha}$ avec $\left(\lambda_{\alpha} = \sum_{\sigma=i+1}^{14} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right)$, puis de $E_{\beta}$ avec $\left(\lambda_{\beta} = \sum_{\sigma=i+1}^{14} \ \left(E_{\beta\sigma} \times 2^{\sigma}\right)\right)$.
Nous savons que $\left(\lambda_{\alpha} \ge \lambda_{\beta}\right)$ car pour tout $\sigma$ (sigma) compris dans l'intervalle $\left]i;14\right]$, il existe $\left(E_{\alpha\sigma} \ge E_{\beta\sigma}\right)$.

Si jamais $\left(\lambda_{\alpha} = \lambda_{\beta}\right)$ alors $\left(E_{\alpha} \lt E_{\beta}\right)$ car $\left(\lambda_{\beta} + E_{\beta i} \times 2^i\right) \ \gt \ \left(\lambda_{\alpha} + \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)\right)$.
La vérification de la condition sur laquelle se base la démonstration mathématique du circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$, se solde par un nouvel échec.

Cependant, si $\left(\lambda_{\alpha} \gt \lambda_{\beta}\right)$ alors cela veut dire que parmis les bits de poids $\sigma \in \left]i;14\right]$, il y a une ou plusieurs occurrence de $\left(E_{\alpha\sigma} \gt E_{\beta\sigma}\right)$.
Par conséquent, nous savons que pour chacune de ces occurences $\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \ \gt \ \sum_{\sigma}^{10} \ \left(E_{\beta\sigma} \times 2^{\sigma}\right)$ et donc $\left(E_{\alpha} \gt E_{\beta}\right)$.
Comme le démontre le chapitre "_Les points terminaux et non terminaux_", lorsque $\left(E_{\alpha} \gt E_{\beta}\right)$ alors $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ et la vérification de la condition sur laquelle repose la démonstration mathématique du circuit est réussie.

Nous comprenons donc que les _zéros anonymes_ de l'intervalle $\left]10;14\right]$ et de poids supérieur au _MSB1_ de $\tau \in \left[10;14\right[$, sont des _zéros anonymes_ ___capitaux___.

### L'obtention d'un point non terminal

Jusqu'ici nous avions un _MSB1_ dans $\tau \in \left[10;14\right]$, mais il pourrait ne pas y en avoir.
Dans cette situation, chaque bit de $\tau \in \left[10;14\right]$ est un _zéro anonyme_, ce qui nous permet d'affirmé que $\left(E_{\alpha} \ge E_{\beta}\right)$.

Nous pourrions atteindre un point terminal si jamais $\left(E_{\alpha} \gt E_{\beta}\right)$, mais ce point terminal ne pourrait s'obtenir qu'à la condition qu'au moins une occurence $\left(E_{\alpha i} \gt E_{\beta i}\right)$ existe pour $i \in \left[10;14\right]$.
Alors, la vérification de la condition sur laquelle se base la démonstration mathématique du circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ réussie.

Mais nous pourrions aussi atteindre une situation inédite de ___point non terminal___, si $\left(E_{\alpha} = E_{\beta}\right)$.
Cette situation inédite ne peut se produire que dans le cas où l'ensemble des paires de bits de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ sont identiques, ou autrement dit $\forall \ i \in \left[10;14\right]$ alors $\left(E_{\alpha i} = E_{\beta i}\right)$.
A ce stade, le circuit n'est pas capable de savoir si la condition $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ va réussir ou échoué, il faut encore traité les champs de mantisse tronquée $T$.

Pour conclure, les _zéros anonymes_ qui composent l'entièreté du champs $\tau \in \left[10;14\right]$ sont ___capitaux___.
La raison d'une telle considération de ces _zéros anonymes_ est capacité propre de chacun d'eux, a pouvoir de faire basculé le circuit d'un état de _point non terminal_, à celui de _point terminal_.

## Rappel sur les champs d'exposant et de mantisse tronquée

Le chapitre "_Les points terminaux et non terminaux_" explique pourquoi est ce que les champs d'exposant $E$ des opérandes $\alpha$ et $\beta$, sont traités avant les champs de mantisse tronquée $T$ de ces même opérandes.
Dans ce chapitre, il y a la formule mathématique $\left(\left(1+T\right) \times 2^E\right)$ qui explique comment est ce que la valeur d'un nombre à virgule flottante est représenté en IEEE-754.
Nous y apprenons que si deux nombres flottants $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$ ont des champs d'exposant égaux $\left(E_{\alpha} = E_{\beta}\right)$, alors la valeur de ces deux nombres $\left(\left(1+T_{\alpha}\right) \times 2^{E_\alpha}\right)$ et $\left(\left(1+T_{\beta}\right) \times 2^{E_\beta}\right)$ ne dépend que de $T_{\alpha}$ et $T_{\beta}$.
Le circuit atteint alors un _point non terminal_.
C'est à dire qu'il est incapable de généré le résultat d'une comparaison entre les opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$, par le seul traitement de leur champs d'exposant $E$.
Par conséquent, le circuit électronique bascule le traitement sur les champs de mantisse tronquée $T$.

Qui plus est, le traitement des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$ est le même que celui des champs d'exposant.
Pour ceux qui ont besoin d'un rafraichissement, je vous rappelle que l'encodage des champs d'exposant et de mantisse tronquée ont des points en commun.
Tout a été expliquer dans le chapitre "_Le standard IEEE-754_".
En bref, l'encodage du champs de mantisse tronquée et du champs d'exposant partagent les même propriété que le _Binary Unsigned_.
Grâce à cela, le processus de comparaison entre les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$, suit la même procédure de traitement que les champs d'exposant $E$.
Nous allons pouvoir remarqué ça dans le chapitre suivant.

# Le traitement des champs de mantisse tronquée

Voici dans un premier temps de quoi dépend le traitement des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Nous remarquerons que c'est la même opération (à quelques détails prêt), que pour les champs d'exposant $E$ des opérandes $\alpha$ et $\beta$.

$$\forall \ i \in \left[0;9\right], \quad Write \ \left(\tau_i, \ Nimply \ \left(T_{\beta i}, \ T_{\alpha i}\right)\right)$$

La variable $\tau$ est techniquement la même que celle utilisée dans la première partie de la démonstration "_Le traitement des champs d'exposant_".
La seule différence, c'est que le bit de résultat de chaque opération logique $Nimply$ sur $T_{\beta i}$ et $T_{\alpha i}$ est inscrit dans $\tau_i$, pour $i \in \left[0;9\right]$.

N'oublions pas que depuis le début, la démonstration mathématique du circuit se base entièrement sur la verification de la condition suivante $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.
Dans les prochaines sections de ce chapitre, nous allons définir tout les cas de réussite et d'échec de cette condition que l'on peut rencontré au travers du traitement des champs de mantisse tronquée.

### Un échec certain de la condition 

Dans le chapitre "_La mantisse tronquée, une histoire de puissance de 2_", qui introduit à l'encodage du champs de mantisse tronquée.
Il est dit que la valeur d'un bit à $1$ de poids $i$ d'un champs de mantisse tronquée $T$, est _inconditionnellement_ supérieur à la somme de la valeur de chacun des bits de poids inférieur à $i$.
Rappellons que c'est aussi le cas des champs d'exposant, comme cela a été mentionné dans le chapitre "_L'encodage par biais du champs d'exposant_".
Comme nous avons pu le voir dans la première partie de la démonstration, la comparaison entre les champs d'exposant utilise ces primitives mathématiques.
Ci-dessous, nous allons voir qu'il en va de même pour la comparaison entre les champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$. 

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
