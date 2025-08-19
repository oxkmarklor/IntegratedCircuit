Ce document contient la démonstration mathématique du circuit électronique ainsi que des explications sur les concepts les plus fondamentaux.

Le document se découpe en plusieurs parties:
  - L'encodage $Binary \ Unsigned$
  - Le standard IEEE-754
    - L'encodage par biais du champs d'exposant
    - La mantisse tronquée, une historie de puissance de $2$
  - L'écriture scientifique binaire
    - Le significande
    - Le multiplicande
  - Codage des informations dans un nombre flottant IEEE-754
    - Composition du champs de mantisse tronquée et d'exposant
  - L'ordre de traitement des champs d'exposant et de mantisse tronquée
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

Le __Binary Unsigned__ utilise les mêmes primitives que la base décimale (celle que nous utilisons tous) pour représenter des nombres, _des puissances_.
Pour représenter la valeur $103$ en décimale, nous décomposons en réalité chaque chiffre du nombre pour les multipliés avec une puissance de $10_{10}$ adéquat, ensuite nous faisons la somme des résultats de chaque produit.
Regardez ci-dessous.

$$103 = 1 \times 10^2 + 0 \times 10^1 + 3 \times 10^0$$

Veuillez noté que l'indice $X_{10}$ représente la base numérique dans laquelle le nombre $X$ qui précède est écrit.
Nous retrouvons chacun des chiffres du nombre entrain de multiplié une puissance de $10_{10}$.
Remarquons que le chiffre des unités $3$ est facteur de $10_{10}$ à la puissance $0$, le chiffre d'un ordre plus grand (celui des dizaines) est le facteur de $10_{10}$ à la puissance $1$ et enfin le chiffre des centaines multiplie $10_{10}$ à la puissance $2$.
La valeur des puissances croix en fonction de la position du chiffre, qui plus est, elle commence à $0$.

Ceci n'est pas propre à la base décimale mais à _n'importe quel base numérique_, comme la __base binaire__ par exemple.
La base binaire (ou base $2$) utilise les chiffres $0$ et $1$ plutôt que ceux de la base décimale allant de $0$ à $9$.
Par ailleurs, un tel chiffre s'appelle un _bit_, ce qui est la contraction de __Binary Digit__.
Aussi, les puissances de $10_{10}$ sont remplacés par des puissances de $2$.
Nous venons tout juste de décrire ce sur quoi se base l'encodage __Binary Unsigned__.

Un champs binaire pour lequel nous utilisons un encodage __Binary Unsigned__ associe une puissance de $2$ positive à chaque bit du champs.
Le champs représente un nombre $X$ par la somme des produits entre la valeur de chaque bit et la puissance de $2$ associée.
Techniquement, vu que dans chacun des produits la puissance de $2$ est multiplié par son bit associé, qui est à $0$ ou à $1$, soit le résultat est nulle ou soit il est équivalent à la puissance de $2$ associé au bit sous-jacent.
C'est pourquoi vous entenderez souvent dire (ici compris), qu'un nombre $X$ est représenté par la somme des puissances de $2$ dont le bit est à $1$.
Les puissances de $2$ vont de $0$ à $N - 1$, ou la valeur de $N$ est le nombre de bits qui compose un champs binaire.
Par ailleurs, il est souvent dit que chaque bit a un "poids".
Le poids correspond à la valeur de la puissance de $2$ associée à chacun des bits d'un champs, plus cette puissance sera grande et plus un bit aura d'importance dans la représentation d'un nombre. 

Il est aussi défini ce que nous qualifions de __LSB__ pour __Least Significant Bit__.
Ce terme désigne le bit de poids le plus faible d'un champs, donc le bit de poids $0$ pour un champs dont l'encodage est en __Binary Unsigned__.
Au contraire, le bit de poids le plus fort d'un champs binaire est ce que l'ont appelle le __MSB__ pour __Most Significant Bit__.
Il est aussi possible de faire référence au bit à $1$ de poids le plus faible d'un champs avec le terme __LSB1__, ou à celui de poids le plus fort avec __MSB1__.

### Une propriété importante de l'encodage Binary Unsigned

Avec un champs dont l'encodage est en __Binary Unsigned__, nous donne la certitude que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des valeurs de chaque bit de poids inférieur à $i$.
Autrement dit $\left(1\times 2^i\right) \gt \left(\sum_{i=i-1}^0 2^i\right)$.
Prenons comme exemple un champs binaire de $8$ bits, pour lequel nous n'allons faire attention qu'à un seul bit, celui de poids $6$ admettons.
Disons que ce champs ressemble à ceci $01111111_{2}$, la valeur du nombre représenté est $127$ et le bit qui nous intéresse se trouve être le __MSB1__ du champs.
La somme des valeurs des bits de poids inférieur au bit de poids $6$ donne un résultat strictement inférieur à $1 \times 2^6 = 64$.
Cela est systèmatiquement vrai, même lorsque les $i$ bits de poids inférieur représentent la valeur maximale codable, comme dans cet exemple.

$$\left(1 \times 2^6 = 64\right) \gt \left(63 = \left(1 \times 2^5\right)+\left(1 \times 2^4\right)+\left(1 \times 2^3\right)+\left(1 \times 2^2\right)+\left(1 \times 2^1\right)+\left(1 \times 2^0\right)\right)$$

Encore une fois, ceci n'est pas propre à la base binaire mais à _n'importe quel base numérique_ modulo quelques variation pour chaque base.

# Le standard IEEE-754

Après cette rapide introduction à l'encodage __Binary Unsigned__, passons au sujet suivant qui est l'encodage __IEEE-754__.

Notre circuit a pour fonction principal de produire des comparaisons entre deux opérandes flottants respectant le standard __IEEE-754__.
Pour plus d'information sur le circuit électronique, jetez un oeil à la documentation du circuit.
Cette norme __IEEE-754__ défini trois éléments qui composent chaque nombre à virgule flottante:
- Le ___bit de signe___
- Un champs binaire d'___exposant___
- Un autre champs binaire pour la ___mantisse tronquée___

Dans ce qui suit nous allons nous intéresser aux encodages utilisés dans les champs binaires d'exposant et de mantisse tronquée.
Nous allons voir que les encodages des champs de mantisse tronquée et d'exposant partagent les même caractéristiques que le _Binary Unsigned_.
Ce qui permet le traitement de ces deux champs par un même processus de calcul, ce qui se reflète sur l'architecture du circuit électronique.

### L'encodage par biais du champs d'exposant

__Le champs d'exposant utilise un encodage par biais__, ce dernier est assez simple à comprendre.
Enfaite, le champs d'exposant est un champs binaire pour lequel nous utilisons un encodage _Binary Unsigned_ qui code une valeur numérique $X$, comme nous l'avons vu précédemment.
A cela, il faut ajouté ou déduire un biais $B$ (un nombre entier naturel) pour obtenir la valeur représenté par le champs binaire.
Dans les faits, la valeur que représente le champs d'exposant est alors issu du calcul $X - B$. 
Voici comment se calcul le biais $B$ d'un champs d'exposant $\left(2^{\left(N-1\right)} - 1\right)$, où $N$ est le nombre de bits du champs d'exposant.

Etant donné que l'encodage par biais se base sur le _Binary Unsigned_, le champs d'exposant __partage les même propriétés__ que cet encodage.
Notamment le fait que la valeur d'un bit à $1$ de poids $i$ soit strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.

### La mantisse tronquée, une historie de puissance de 2

Pour le champs de la mantisse tronquée nous avons besoin de faire un rapide rappel sur l'encodage intial d'un nombre flottant.
La partie entière d'un nombre flottant utilise du _Binary Unsigned_ pour être codé, tandis que la partie fractionnaire fait usage d'une version modifiée du _Binary Unsigned_.
Chaque bit de la partie fractionnaire est un ___facteur d'une puissance de 2 négative___ et non positive.
Ceci permet de représenté des valeurs dans l'intervalle $\left[0;1\right[$.
Le poids des puissances négatives de $2$ décroix en fonction de la position du bit.
Prenons l'exemple du nombre $3.75$.

$$3.75 = 11.11_2 = Integer \ Part\left(\left(1 \times 2^1\right) + \left(1 \times 2^0\right)\right) + Fractional \ Part\left(\left(1 \times 2^{-1}\right) + \left(1 \times 2^{-2}\right)\right)$$

Précisons que sous la forme binaire du nombre $3.75$, le point n'est là que pour facilité la lecture du nombre.
Dans les faits, le point n'est pas réelement présent dans le codage des nombres flottants.
Ce qu'il y a d'important à remarquer pour la partie fractionnaire, c'est que la valeur d'un bit à $1$ de poids $i$ est toujours strictement supérieur à la somme des valeurs des bits de poids inférieur à $i$.
Autrement dit, pour le bit à $1$ de poids $i$ du nombre fractionnaire $F \in \left[0;1\right[$, alors $\left(1 \times 2^i\right) \gt \left(\sum_{i=i-1}^{lsb\left(F\right)} \left(F_i \times 2^i\right)\right)$.

Pour en revenir au sujet de la mantisse tronquée, elle est composée des bits de la partie entière et de la partie fractionnaire d'un nombre flottant.
Vu que la partie entière et fractionnaire d'un nombre flottant partagent les propriétés du _Binary Unsigned_, c'est aussi le cas de la mantisse tronquée elle même.

Nous pouvons remarquer que le champs d'exposant et de mantisse tronquée partagent bel et bien les même propriétés que l'encodage _Binary Unsigned_, comme mentionné plus haut.
C'est dans la démonstration mathématique que nous verrons à quel point cela va nous être utile.

# Le traitement des opérandes flottants

La tâche primaire du circuit est de comparer deux nombres flottants ___Half Precision___ de $16$ bits, nous les nommerons $\alpha$ et $\beta$.
La comparaison en question est une vérification de la supériorité stricte de la valeur absolu de l'un de ces deux opérandes envers l'autre, admettons $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.
Etant donné que le circuit n'a besoin que de la valeur absolu des opérandes $\alpha$ et $\beta$, seul les $15$ bits de poids faible sont utiles (omission du bit de signe).

Il se trouve que le circuit traite les champs d'exposants $E$ de $\alpha$ ainsi que de $\beta$, avant les champs de mantisse tronquée de ces même opérandes.
La raison en est que les champs d'exposant à eux seuls peuvent permettre au circuit d'atteindre un point terminal.
Le circuit électronique atteint un point terminal lorsqu'il est capable de générer le résultat d'une comparaison entre ses deux opérandes, sans avoir besoin d'attendre le traitement de l'entièreté de ses entrées.
Techniquement, un point terminal est atteint si $\left(E_{\alpha} \gt E_{\beta}\right)$ car l'opérande $\vert \ \alpha \ \vert$ est strictement plus grand que $\vert \ \beta \ \vert$, et inversement avec $\left(E_{\alpha} \lt E_{\beta}\right)$.
Nous verrons pourquoi dans le chapitre "_?L'ordre de traitement des champs d'exposant et de mantisse tronquée?_".

Cependant, le circuit peut aussi rencontrer un point non terminal.
Le circuit atteint ce dernier lorsque $\left(E_{\alpha} = E_{\beta}\right)$.
Comme nous le verrons plus tard, dans cette situation il n'y a rien dans les champs d'exposant des opérandes $\alpha$ et $\beta$ qui puisse permettre au circuit de savoir quel résultat générer sans procédé à d'autre calcul.
Il faudra alors traité les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$, dans un second temps.

Dans les chapitres suivant, je vais expliqué dans les fondements pourquoi est ce que les champs d'exposant sont traités en priorité par le circuit électronique.
Pour cela, il va d'abord me falloir abordé le sujet de l'écriture scientifique binaire, alors commençons.

# L'écriture scientifique binaire

Pour commencer, qu'est ce que l'_écriture scientifique_?

L'écriture scientifique est une manière de représenté les nombres.
Elle existe pour chaque base numérique, comme pour la base binaire.
Le but de cette notation scientifique des nombres est double, le permier objectif est de ne pouvoir représenté un nombre que d'une seule façon, le second permet de simplifier la lecture des grands nombres.
Par exemple, en ___écriture scientifique décimale___ nous pouvons représenté facilement la vitesse approchée de la lumière en km/s $+3.0 \times 10^5$.
Cela peut paraitre plus compliqué à interprété de prime abord, mais comme nous le verrons dans les chapitres ci-dessous, en réalité tout n'est qu'une histoire de puissance et de virgule.

Le fonctionnement de l'écriture scientifique ne change que peu lorsque nous passons d'une base numérique vers une autre.
La composition elle, reste la même peut importe la base numérique:
  - Le ___signe___
  - Un ___significande___
  - Un ___multiplicande___

Dans l'exemple fournit ci-dessus, le signe est évidemment le symbole $+$ qui indique si le nombre est positif ou négatif, la valeur $3.0$ est le significande qui peut également être appellé ___mantisse___, et enfin le multiplicande qui est la base $10$ élevé à la puissance $5$.
En notation scientifique décimale, le significande peut être toute valeur comprise dans l'intervalle $\left[1;10\right[$, et le multiplicande est une puissance de $10$.
De manière général, pour une notation scientifique en base $N$.
Le significande ne peut être compris qu'entre $\left[1;N\right[$, et le multiplicande est alors une puissance de $N$.
C'est ce que nous allons repectivement voir dans les chapitres du nom de "_Le signficiande_" et "_Le multiplicande_" qui suivent.

Pour terminer, si nous parlons de l'écriture scientifique ce n'est pas pour rien.
Chaque élément de l'encodage _IEEE-754_ ayant été défini plus tôt, correspond à l'un des éléments de l'écriture scientifique binaire d'un nombre.
Nous verrons cela dans des chapitres ultérieurs.

## Le significande

Pour écrire en notation scientifique le nombre flottant $F$, il faut partir du codage binaire "classique" du nombre $F$.

Pour la ___notation scientifique binaire___, le significande est un nombre réel dont la valeur est compris dans l'intervalle suivante $\left[1;2\right[$.
Ce qui veut dire que _la partie entière du significande doit être à_ $1$.
Pour obtenir un significande, _nous déplaçons la virgule du nombre_ $F$ (codé en binaire normal) de sa position initial, _jusque devant le MSB1 du nombre_.
Ce qui tout logiquement modifie la valeur de $F$.
Une exception est faite pour le cas où le flottant $F$ est nul, le significande peut alors lui même être nul (c'est une exception).
La raison est qu'il n'y a pas de bit à $1$ dans le champs binaire qui code le flottant $F$ si il est nul, et donc il n'a y a pas de _MSB1_.
Cependant, si le _MSB1_ est dans la partie entière du flottant $F$, cela insinue que $\left(F \ge 1\right)$ et que la virgule devra être déplacé vers la gauche (_si elle doit l'être_).
Au contraire, si le _MSB1_ est dans la partie fractionnaire alors $\left(F \lt 1\right)$, et la virgule sera déplacé vers la droite.

Mais le déplacement de la virgule d'un nombre engendre en binaire les même choses qu'en décimale.
Une _division par_ $N$ de $F$ dans le cas d'un déplacement de la virgule de $log_2\left(N\right)$ rangs vers la gauche, et _une multiplication_ de $F$ par le même facteur $N$ pour un décalage de la virgule de $log_2\left(N\right)$ rangs vers la droite.
Le terme $N$ qui multiplie ou divise $F$ est __systèmatiquement une puissance de 2__, ce qui suit explique pourquoi.

### Comprendre pourquoi F est multiplié/divisé par une puissance de 2 lorsque sa virgule est déplacée

Pour tout nombre flottant $F$, nous savons que la virgule se trouve entre le _LSB_ de la partie entière (le bit de poids $0$), et le _MSB_ de la partie fractionnaire (bit de poids $-1$).
Par conséquent, un déplacement de la virgule d'un rang vers la gauche force le bit de poids $1$ à devenir le bit de poids $0$, le bit de poids $0$ devient celui de poids $-1$, et celui de poids $-1$ devient le bit de poids $-2$, etc.
Pour le dire autrement, chaque bit de la partie entière comme de la partie fractionnaire de $F$ voit son poids être _décrémenter_ de $1$. 
C'est pourquoi après le déplacement de la virgule d'un rang vers la gauche, le nombre $F$ vaut ce qui suit pour $c = -1$:

$$\left(\sum_{i = msb\left(F\right)}^{lsb\left(F\right)} \ \left(F_i \times 2^{\left(i+c\right)}\right)\right) = \left(F\times 2^c\right)$$

Le terme $c$ représente le nombre de rang de décalage à induire sur la virgule du nombre $F$.
Qui plus est, lorsque $\left(c \lt 0\right)$ c'est que _la virgule doit être décalée vers la gauche_, et inversement quand $\left(c \gt 0\right)$ c'est que le décalage doit se produire _vers la droite_.

Essayons de comprendre pourquoi est ce qu'un décalage d'un rang vers la gauche de la virgule de $F$, divise la valeur de $F$ par $N = 2$, comme nous le démontre l'équation ci-dessus.
Nous savons que les nombres flottants ont une partie entière et une autre fractionnaire.
Ces deux parties utilisent chaque bit comme un facteur d'une puissance de $2$, des puissances _positives_ pour la partie entière et _négatives_ pour la partie fractionnaire.
Nous pouvons remarquer que dans le codage du nombre $F$, n'importe quel bit de poids $i$ est facteur d'une puissance de $2$.
Elle même deux fois plus grande que la puissance facteur du bit de poids $i - 1$.
Par exemple $\left(2^0 = 1\right)$ et $\left(2^{-1} = 0.5\right)$. 

Nous avons vus plus haut qu'avec un décalage d'un rang vers la gauche de la virgule de $F$, je cite "chaque bit de la partie entière comme de la partie fractionnaire de $F$ voit son poids être _décrémenter_ de $1$".
Ce qui veut dire que tout bit de $F$ passe de facteur de $2^i$ à $2^\left(i-1\right)$, et _chaque bit_ voit donc son poids être _divisé par_ $2$.
Le calcul de la valeur de $F$ passe de la somme des valeurs des bits de poids $i$ à $1$ (qui multiplient $2^i$), à la somme des valeurs des bits de poids $i$ à $1$ (qui multiplient $2^{\left(i-1\right)}$).
C'est pourquoi la valeur de $F$ est _divisé par_ $2$ avec un décalage d'_un rang vers la gauche_ de la virgule.

Maintenant que nous avons compris pourquoi un décalage d'un rang vers la gauche de la virgule de $F$, engendre une division par $2$ du nombre.
Découvrons désormais les effets d'un décalage d'un rang vers la droite de la virgule de $F$, sur la valeur du nombre lui même.
Dans le cas d'un tel décalage, chaque bit de la partie entière comme fractionnaire de $F$ voit son poids être _incrémenter_ de $1$.
Par conséquent, chaque bit de poids $i$ voit sa valeur doublé car $\left(2^{\left(i+1\right)} = 2 \times 2^i\right)$.
Nous calculons la valeur de $F$ après décalage, comme la somme des valeurs des bits de poids $i$ à $1$ qui multiplient $2^{\left(i+1\right)}$.
Exactement comme le fait la partie gauche de l'équation ci-dessus pour un décalage $c = 1$.
Le côté droit de cette équation nous informe que cette opération revient à _multiplié_ le nombre $F$ par $2$.

Nous sommes désormais capable de comprendre toutes les implications d'un décalage de la virgule du nombre $F$, sur la valeur du nombre lui même. 
Prenons un décalage de $c = x$ comme exemple, où $x \in \left[-\infty ;+\infty \right]$.
Si $\left(c \gt 0\right)$ alors nous n'avons qu'à décalé la virgule de $c$ fois $1$ rang _vers la droite_.
Vu que nous savons qu'un décalage de la virgule de $1$ rang vers la droite engendre une multiplication par $2$ du nombre $F$, alors après $c$ décalages de $1$ rang, nous aurons multiplié $F$ par $2^c$.
Au contraire si $\left(c \lt 0\right)$, alors nous devrons décalé la virgule de $\vert \ c \ \vert$ fois $1$ rang _vers la gauche_.
Un décalage d'un rang vers la gauche revient à divisé par $2$ la valeur de $F$.
Par conséquent, après $\vert \ c \ \vert$ décalages de la virgule de $1$ rang vers la gauche, nous aurons divisé $\vert \ c \ \vert$ fois la valeur de $F$ par $2$.
Ou autrement dit, $F$ aura été divisé par $2^{\vert \ c \ \vert}$.

Voilà pourquoi n'_importe quel décalage de la virgule_ d'un nombre flottant $F$, engendre une _multiplication_ ou une _division_ du nombre par une puissance de $2$.

### Détail sur le fonctionnement de l'équation

Cependant, je parle de divisé par une puissance de $2$ le nombre flottant $F$ dans le cas d'un décalage de sa virgule vers la gauche, mais malgré que l'équation précédente fonctionne, elle ne fait pas usage de division.
Essayons de comprendre pourquoi.
Le côté droit de cette équation $\left(F\times 2^c\right)$ est approprié pour un décalage de la virgule vers la droite.
Car je rappelle qu'après un déplacement de la virgule de $c$ rangs vers la droite, le nombre flottant $F$ est mulitplié par une puissance de $2$ qui vaut $2^c$, où $\left(c \gt 0\right)$.

En outre, nous avons jusqu'ici utilisé des valeurs négatives de $c$ lors des décalages de virgule vers la gauche.
Pourtant le plus intuitif aurait été de définir les même valeurs de $c$ peu importe la direction des décalages, alors faisons comme si c'était le cas depuis le début.
Dans ce qui suit je vais donc faire usage de $\left(c \gt 0\right)$ pour des décalages vers la gauche.
Les explications suivantes vont nous permettre de comprendre pourquoi, pour nos explications, il vaut mieux que $c$ soit négatif pour les décalages de la virgule vers la gauche.

Pour un décalage de la virgule de $c$ rangs vers la gauche, nous devrions divisé $c$ fois le nombre $F$ par $2$, ou autrement dit, divisé $F$ par $2^c$.
Initialement, nous pourrions penser avoir besoin d'une équation ressemblant à celle-ci $\left(\sum_{i=msb\left(F\right)}^{lsb\left(F\right)} \left(F_i \times 2^{\left(i-c\right)}\right)\right) = \left(F\div 2^c\right)$.
J'attire l'attention sur le fait que côté droit de l'équation peut être modifié pour $\left(F\times \left(1\div 2^c\right)\right)$.
Mais ce n'est pas fini, car pour trouver l'inverse d'une puissance de $2$ tel que $2^c$, nous pouvons simplement appliqué l'opposé de l'exposant $c$ à la base $2$.
En gros, $\left(1\div 2^c\right) = \ 2^{-c}$.
Je me permet de rappellé que $\left(c \gt 0\right)$.
Finalement, nous pouvons alors remplacé $\left(F\times 2^{-c}\right)$ par $\left(F\times 2^c\right)$, à la condition qu'un décalage de $x$ rangs vers la gauche de la virgule de $F$, soit représenté dans l'équation par une valeur de $c = -x$.

Avec $\left(F\times 2^c\right)$ nous retrouvons bien le membre droit de notre équation initial, à la condition que $c$ puisse devenir négatif.
De plus, nous comprenons aussi que le calcul de l'exposant $\left(i+c\right)$ du membre gauche, produit l'équivalent de $\left(i-c\right)$ dans le cas d'un décalage de la virgule de $F$ vers la gauche, car $\left(c \lt 0\right)$.

C'est la raison pour laquelle il vaut mieux que $c$ soit négatif lors d'un décalage vers la gauche de la virgule.
Ceci nous permet de ne faire usage que d'une seule équation pour tout les sens de décalage. 

## Le multiplicande

// contextualisation sur le résultat que génère l'équation vu plus haut

// apporté plus de contexte avec des références au chapitre qui précède

Nous savons désormais comment obtenir un significande.
Le problème c'est que tout nombre flottant $F$ dont la valeur n'est pas comprise dans l'intervalle $\left[1;2\right[$ doit forcémment subir un décalage de sa virgule, et donc voir sa valeur être multiplié ou divisé par une puissance de $2$.
Pour contrecarrer ce problème, l'écriture scientifique défini ce qui s'appelle un multiplicande. 
Le rôle du multiplicande est de nous permettre de retrouver la valeur initial de $F$ après qu'il ait été transformé en un significande $S$, voyons comment.

Pour commencer, le nombre flottant $F$ est transformé en un significande $S$.
Cette transformation passe par un décalage vers la droite ou vers la gauche de la virgule de $F$.
Comme nous l'avons vus plus haut, alors la valeur de $F$ est divisé ou multiplié par une puissance de $2$.
Imaginons que pour la transformation de $F$ en un significande $S$ nous ayons défini $\left(c \lt 0\right)$, alors la virgule du nombre $F$ a été déplacé vers la gauche et donc le nombre divisé.
Pour retrouver $F$, il suffit alors de multiplié $S$ par la puissance de $2$ qui a divisé le nombre $F$.
Mais rappellons que la division de $F$ s'effectue comme ceci $\left(F\times 2^c\right)$, ce qui est équivalent à $\left(F\div \left(1\div 2^c\right)\right)$.
Il faut donc multiplié le significande $S$ de la manière suivante afin de retrouver le nombre $F$ d'origine $\left(S\times \left(1\div 2^c\right)\right)$, soit $\left(S\times 2^{-c}\right)$.
Mais dans le cas inverse, pour la transformation du nombre $F$ en significande $S$ avec $\left(c \gt 0\right)$.
La virgule de $F$ est déplacé vers la droite et le nombre a alors été multiplié par une puissance de $2$.
Ce coup ci, pour retrouver $F$, il faudra divisé la valeur de $S$ par la puissance de $2$ qui a multiplié $F$.
Nous savons que $S = \left(F \times 2^c\right)$, alors nous divisons $S$ par $2^c$ comme ceci $\left(S\div 2^c\right)$ et nous obtenons une nouvelle fois sur $\left(S \times 2^{-c}\right)$.

Nous retombons dans tout les cas sur le même calcul $\left(S\times 2^{-c}\right)$ permettant de retrouver le nombre d'origine $F$, le multiplicande est le facteur $2^{-c}$. 
Mais nous pouvons exprimé les choses de manière différentes, ce qui nous servira pour plus tard.

Pour la transformation d'un nombre flottant $F$ en un significande $S$, si la virgule de $F$ a été déplacé de $c$ rangs vers la droite alors $\left(c \gt 0\right)$.
Afin de retrouver la valeur du nombre $F$ d'origine depuis le significande, le multiplicande doit permettre de déplacé de $c$ rangs vers la gauche la virgule du significande $S$.
Il suffit alors d'inversé le signe de $c$ pour changé la direction du décalage.
Nous faisons cela en négationnant $c$, comme ceci $\left(-\left(+c\right) = -c\right)$.
Le nombre de rang de décalage reste le même mais le sens de décalage passe de la droite $\left(c \gt 0\right)$, à la gauche avec $\left(c \lt 0\right)$.

Dans le cas contraire, la transformation de $F$ en $S$ est due à un décalage de la virgule de $F$ de $\vert \ c \ \vert$ rangs vers la gauche, où $\left(c \lt 0\right)$.
Pour obtenir la valeur du nombre $F$ d'origine, il faudra alors que le significande $S$ voit sa virgule être décalée de $\vert \ c \ \vert$ rangs vers la droite.
Rappellons que $c$ est négatif, la négation de $c$ permettra donc une nouvelle fois d'inversé le sens de décalage $\left(-\left(-c\right) = \vert \ c \ \vert\right)$.

Il est donc simple de comprendre que déplacé la virgule du significande $S$ de $-c$ rangs, vers la gauche si $\left(c \lt 0\right)$ ou vers la droite si $\left(c \gt 0\right)$, est équivalent à multiplié $S$ par $2^{-c}$.
Comme nous l'indique les premières explications de cette section.

# IX. Codage des informations dans un nombre flottant IEEE-754

Nous avons déjà parler de l'encodage IEEE-754 en introduction de ce document, plus particulièrement des éléments qui le compose et de leur encodage propre.
Je rappelle que ces $3$ éléments sont des champs binaire qui codent chacun un __bit de signe__, un __exposant__ ainsi qu'une __mantisse tronquée__.
Par ailleurs, nous venons tout juste de voir dans le détails ce qu'est l'écriture scientifique biniaire d'un nombre, ce qui nous a permis de mettre en lumière ce qu'est un __significande__ (qui peut être appellé __mantisse__), ainsi qu'un __multiplicande__.
Il y a aussi un élément de l'écriture scientifique dont j'ai peu parlé, c'est le __signe__ $\pm$ du nombre représenté.

Chaque champs binaire de l'encodage IEEE-754 correspond plus ou moins à l'un des éléments de l'écriture scientifique binaire d'un nombre.
Par exemple, le champs du _bit de signe_ correspond assez logiquement au _signe_ de la notation scientifique d'un nombre, mais nous ne nous intéresserons pas plus que ça au bit de signe dans ce qui suit.
Je rappelle que le circuit électronique ne travaille qu'avec la valeur absolu de ses opérandes flottants.
En revanche, nous allons parlé plus en profondeur du champs d'exposant ainsi que de la mantisse tronquée.
Nous avons déjà abordé le sujet de l'encodage de ces champs dans l'introduction, aussi je ne vais pas y revenir.

# X. Composition du champs de mantisse tronquée et du champs d'exposant

En reprenant ce que nous avons dit précédemment, le champs de la mantisse tronquée correspond au significande d'un nombre représenté en écriture scientifique binaire.
Faisons un rapide rappelle sur ce qu'est le significande.
Le significande $S$ est un nombre dont la valeur se trouve être dans l'intervalle de $\left[1;2\right[$, il est souvent le résultat d'une transformation (un calcul) sur un nombre flottant $F$ à représenté en écriture scientifique.
Enfaite, si $F \notin \left[1;2\right[$ alors le nombre sera divisé ou multiplié par une puissance de $2$ que nous nommons $N$, cette transformation permet l'obtention d'un significande. 

En somme, le significande est un nombre à virgule qui est donc composé d'une partie entière ainsi que d'une partie fractionnaire.
Vu que la valeur du significande doit être strictement inférieur à $2$ ou $10_2$ en binaire, ça veut dire que l'encodage de la partie entière ne peut être que inférieur à $10_2$, c'est à dire $0$ ou $1$.
Par conséquent la partie entière du significande n'est codé que sur un bit.
Mais en soit le bit de la partie entière ne peut pas être à $0$, car il est nécessaire que la valeur du significande soit comprise dans l'intervalle mentionné ci-dessus.
__Donc le bit de la partie entière est à 1__.
Pour résumer $\left(S \ge 1\right) \wedge \left(S \lt 2\right)$ nous force à dire que la partie entière du significande $S$ est à $1$.
En plus de cela, nous avons la partie fractionnaire du significande à prendre en compte, qui est logiquement strictement inférieur à $1$.
Finalement, la valeur du significande est la somme de la partie entière et de la partie fractionnaire, ce qui veut dire que $\left(S \lt 2\right)$.

La norme IEEE-754 définit un champs de mantisse tronquée de $X$ bits pour chaque format de nombre flottant. 
Prenons pour exemple le format ___Half Precision___ que gère le circuit électronique en cours d'étude.
Il s'avère que le champs de la mantisse tronquée de ce format est établit à une taille de $10$ bits.
L'entièreté d'un significande $S$ pourrait être enregistré sur ces $10$ bits (si possible), mais ce n'est pas utiles car la partie entière est toujours composé d'un seul et même bit à $1$, comme nous venons de le voir.
C'est pourquoi la norme ampute le significande de sa partie entière, indépendemment du format de flottant.
L'intérêt c'est que le champs reste de la même taille ($10$ bits pour un _Half Precision_), mais que nous gagnons un bit de précision sur le codage d'un nombre.
C'est d'ailleurs la raison derrière le nom de mantisse tronquée.

Le standard IEEE-754 définit aussi un champs d'exposant, qui correspond _en partie_ au multiplicande de l'écriture scientifique binaire d'un nombre $F$.
Je conseille une relecture du chapitre sur le multiplicande si un rafraichissement est nécessaire.
Je cite "_le multiplicande est le facteur_ $2^{-c}$.", mais en réalité le champs d'exposant lui ne correspond qu'à l'exposant $-c$ du multiplicande, et non à la puissance de $2$ en sa totalité. 
Rappellons que la variable $c$ représente le nombre de rang par lequel décalé la virgule du nombre $F$ pour le transformé en un significande, et que lorsque $\left(c \lt 0\right)$ cela témoigne d'un décalage vers la gauche, inversement pour $\left(c \gt 0\right)$.
Le champs d'exposant se base sur ce qui a été dit dans les trois derniers paragraphes du chapitre sur le multiplicande, pour déplacé la virgule du significande du bon nombre de rang, et dans la bonne direction.

En gros, le champs d'exposant code le résultat de la négation de $c$, ce qui a pour effet d'inversé le sens de décalage de la virgule, sans touché pour autant au nombre de rang à décalé.
Si la valeur de l'exposant codé dans le champs est négative, alors le décalage de la virgule du significande se fera vers la gauche et du nombre de rang codé par le champs.
Au contraire, si cette valeur est positive, la virgule du significande sera décalée vers la droite et sera toujours déplacé du nombre de rang codé par le champs.
Ce champs d'exposant possède néanmoins une taille $X$ qui varie en fonction du format des nombres flottants.
Pour un format _Half Precision_ sa taille est de $5$ bits.

# XI. L'ordre de traitement des champs d'exposant et de mantisse tronquée

Pour finir, je vais enfin pouvoir expliqué pourquoi le circuit électronique traite les champs d'exposant $E$ des opérandes $\alpha$ et $\beta$, avant les champs de mantisse tronquée $T$ de ces même opérandes.

Nous avons vu dans le chapitre sur le multiplicande, comment est ce qu'en écriture scientifique binaire nous pouvions obtenir le nombre d'origine $F$ depuis le significande $S$. Précisémment au travers du calcul suivant $F = \left(S\times 2^{-c}\right)$.
En tenant compte de tout ce qui a déjà été dit plus haut, nous en déduisons qu'en IEEE-754 le nombre $F$ vaut alors $\left(\left(1 + T\right)\times 2^E\right)$.

N'oublions pas que si le significande $S$ de l'écriture scientifique binaire du nombre $F$ est compris dans l'intervalle de valeur suivante $\left[1;2\right[$.
Ce n'est pas le cas du champs de mantisse tronquée $T$, __car le bit à 1 de sa partie entière est rendu implicite__.
D'où le fait que pour représenté la valeur réel du champs, il nécessaire d'ajouté ce que vaut la partie entière à la mantisse tronquée $\left(1 + T\right)$.

Au travers du calcul suivant $\left(\left(1 + T\right)\times 2^E\right)$ nous observons quelques chose d'intéressant.
Par essence $\left(2^i = 2\times 2^{\left(i-1\right)}\right)$.
Etant donné que la valeur codé dans le champs de la mantisse tronquée $T$ est strictement inférieur à $1$, alors même pour $T = 0.99..9$ nous obtenons avec $\left(\left(1 + T\right)\times 2^E\right)$ une valeur __strictement inférieur__ à $\left(1\times 2^{\left(E+1\right)}\right)$.
Nous voyons ici pourquoi est ce que les champs d'exposant $E$ des opérandes sont traités avant les champs de mantisse tronquée $T$.

Imaginons que pour deux opérandes $\alpha$ et $\beta$ nous voulions vérifié $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$:
  - Si $\left(E_{\alpha} \lt E_{\beta}\right)$ alors nous savons que la comparaison __échoue__ car $\left(\left(1 + T_{\alpha}\right)\times 2^{E_{\alpha}}\right) \lt \left(\left(1 + T_{\beta}\right)\times 2^{E_{\beta}}\right)$ et ce même si $T_{\alpha} = 0.99..9$ lorsque $T_{\beta} = 0$.

  - Si $\left(E_{\alpha} \gt E_{\beta}\right)$ alors la comparaison __réussie__ cette fois-ci, car $\left(\left(1 + T_{\alpha}\right)\times 2^{E_{\alpha}}\right) \gt \left(\left(1 + T_{\beta}\right)\times 2^{E_{\beta}}\right)$ et ce même si $T_{\alpha} = 0$ lorsque $T_{\beta} = 0.99..9$.

Dans les deux cas qui figurent ci-dessus, __le circuit atteint un point terminal__.
Mais il existe une troisième possibilité à laquelle le circuit peut être confronté $\left(E_{\alpha} = E_{\beta}\right)$.

Dans ce cas spécifique, les produits $\left(\left(1 + T_{\alpha}\right)\times 2^{E_{\alpha}}\right)$ ainsi que $\left(\left(1 + T_{\beta}\right)\times 2^{E_{\beta}}\right)$ ont un facteur commun, la puissance de $2$.
Par conséquent, tout ne repose que sur les champs $T_{\alpha}$ et $T_{\beta}$ pour pouvoir départagé si $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ ou non.

Nous comprenons désormais pourquoi le circuit électronique, qui a pour but de vérifié la supériorité stricte d'un opérande envers l'autre, traite d'abord les champs d'exposant $E$ des opérandes $\alpha$ et $\beta$, puis ensuite les champs de mantisse tronquée $T$ si nécessaire.

# XII. Démonstration mathématique

Il y a une documentation dédié au circuit pour comprendre sa raison d'être ainsi que son architecture, je conseil de la lire pour plus d'information.

Le circuit se nomme __FPU Configuration Unit__.
Le rôle du circuit est de généré deux bits de sortie, l'un pour configuré un circuit soustracteur de nombre flottant, et l'autre pour la sortie même de cette unité de calcul.
A cette fin, le circuit reçoit deux opérandes flottants _Half Precision_ $\alpha$ et $\beta$, en valeur absolu.
Il doit procédé à une comparaison de supériorité stricte de l'un de ces opérandes envers l'autre.
La démonstration mathématique va se basé sur le teste de la condition suivante $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.

### Les opérandes Half Precision

Je rappelle que l'encodage IEEE-754 définit trois éléments dans chacun des nombres flottants du standard, le _bit de signe_, le _champs d'exposant_ ainsi que le _champs de mantisse tronquée_.
Voici la disposition précise de chaque bit de chacun de ces champs pour l'encodage d'un nombre _Half Precision_ ($16$ bits):

$$\left(1\right) \quad \left[S_{15}, \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

__S__: Sign bit,  __E__: Exponent field,  __T__: Truncated mantissa

Chaque indice (nombre) compris dans l'intervalle $\left[0;15\right]$ représente le poids d'un bit précis.
C'est en effet très commun d'indicé les bits d'un champs binaire par leur poids.

Cependant, dans les démonstrations mathématique nous ne considérerons que les bits de l'intervalle $\left[0;14\right]$.
N'oublions pas que le circuit ne se soucis que de la valeur absolu de ses opérandes $\left(\vert\alpha\vert\right)$ et $\left(\vert\beta\vert\right)$, donc le bit de signe de poids $15$ est omis.
De plus, nous avons vus dans le chapitre "_Ordre de traitement des champs d'exposant et de mantisse tronquée_", que les champs d'exposant $E$ des opérandes étaient traités avant les champs de mantisse tronquée $T$.
Rappellons que ceci est dû au fait que le circuit profite des *___points terminaux___ que les champs d'exposant génèrent souvent.
C'est pourquoi le document commence par traité le cas des champs d'exposant $E_{\alpha}$ et $E_{\beta}$, puis ensuite celui des champs de mantisse tronquée $T$.

*Un ___point terminal___ est le fait que le circuit soit capable de déduire un résultat correct, par le seul traitement des champs d'exposant.
Le circuit court-circuite alors les traitements qui concerne les champs de mantisse tronquée, et génère le résultat final plus rapidement.

### L'organisation du document

Le document commence par la démonstration mathématique du traitement des champs d'exposant $E_{\alpha}$ et $E_{\beta}$, puis enchaine avec celle des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Les démonstrations sont verbeuses, pour plus de concision il y a un résumé global qui suit la seconde et dernière démonstration.
Ce résumé est seulement composé d'expression mathématique ainsi que d'une courte description pour chacune d'entre elle.

### Quelques définitions

__Write__ est une fonction d'affectation, le paramètre $\left(y\right)$ _est copié_ dans le paramètre $\left(x\right)$.

$$Write \ \left(x, \ y\right) \rightarrow \ x \ := \ y$$

__Nimply__ est une fonction qui reproduit le comportement d'une porte logique du même nom.
C'est une opération logique assez peu connu, d'où le fait qu'il n'existe pas de symbole opératoire qui lui est propre.

$$Nimply \ \left(x, \ y\right) \rightarrow \ x \ \wedge \ \overline{y}$$

De plus, $\overline{y}$ est l'opération logique __Not__ qui inverse un bit en son opposé $\left(1 = \overline{0}\right)$ ou $\left(0 = \overline{1}\right)$.
Cette opération possède également sa propre porte logique, qui est d'ailleurs assez élémentaire pour la conception de circuit en tout genre.

Pour finir, le symbole $\wedge$ est l'opération logique __And__ qui ne retourne un bit à $1$ que lorsque ses deux bits d'opérande le sont aussi (sinon $0$), et le symbole $\vee$ est l'opération logique __Or__ qui ne retourne un $0$ que si ses deux bits d'opérande le sont également (sinon $1$).

## Le traitement des champs d'exposant

La démonstration démarre par ceci:

$$\sum_{i=14}^{10} \ Write\left(\tau_i, \ Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)\right)$$

La variable $\tau$ est un champs binaire de $15$ bits dont la représentation est la même que celle illustrée plus haut dans "_Les opérandes Half Precision_", mais sans le bit de signe.
Chaque résultat d'une opération $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$ pour $i \in \left[10;14\right]$, est inscrit dans $\tau_i$.
Ce qui veut dire que $\tau \in \left[10;14\right]$ correspond aux bits de résultat des opérations $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$.

### L'opération logique Nimply

L'opération $Nimply$ a été définit plus haut.
Selon cette définition, nous comprenons que le bit de résultat d'une telle opération ne peut être à $1$ que lorsque le bit sur le paramètre $\left(x\right)$ est $1$, et que celui sur $\left(y\right)$ est à $0$.
Pour tout les autres cas, si $\left(x = y\right)$ ou $\left(x \lt y\right)$ alors le bit de résultat sera $0$.
La documentation du circuit électronique définit le terme de "___zéro anonyme___" pour désigné tout bit de résultat à $0$, sortant d'une opération $Nimply$.

### Qu'est ce qu'un zéro anonyme?

Nous venons de voir à quoi correspond le terme de "_zéro anonyme_".

Cependant, nous ne connaissons pas la raison d'un nom aussi bizarre.
Le circuit se sert des sorties des portes logiques $Nimply$ pour faire des déductions sur les entrées de celle-ci, et la démonstration mathématique en fait de même.
Mais lorsque le bit de sortie d'une opération $Nimply$ est à $0$, il nous est impossible de savoir si cette sortie a été générée par deux bits d'opérandes identique sur les paramètres $\left(x\right)$ et $\left(y\right)$, ou si le bit sur le paramètre $\left(y\right)$ est strictement supérieur à celui sur $\left(x\right)$.
En bref, ce flou pose un problème que nous dervons géré dans la démonstration mathématique (le circuit doit lui aussi résoudre ce problème).

Petite précision d'ailleurs.
Il existe en réalité des _zéros anonymes_ ___capitaux___ et ___non capitaux___, nous définirons plus bas la différence entre ces deux types de _zéro anonyme_.

## Les deux facettes des zéros anonymes (introduction)

Revenons à l'expression qui figure ci-dessus.
Chaque bit $\tau_i$ est le résultat de $\ Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)$ pour le poids $i \in \left[10;14\right]$.
Si $\left(\tau_i = 1\right)$ alors nous savons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$, ou autrement dit $\left(E_{\beta i} \times 2^i\right) \ \gt \ \left(E_{\alpha i} \times 2^i\right)$.

De plus, rappellez vous du chapitre "_Encodage par biais_" qui parle de l'encodage du champs d'exposant des nombres flottants IEEE-754.
Dans ce chapitre, il est dit que la valeur d'un bit à $1$ de poids $i$ d'un champs d'exposant $E$, est _inconditionnellement_ supérieur à la somme de la valeur de chacun de ses bits de poids inférieur à $i$.
Autrement dit, si nous prenons au pied de la lettre la phrase ci-dessus, alors $\forall \left(i \in \left[11;14\right]\right)$ nous obtenons $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i=i-1}^{10} \ \left(E_{\beta i} \times 2^i\right)$.
Mais nous pouvons arrangé cette expression à notre champs d'exposant $E_{\alpha}$, alors $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i=i-1}^{10} \ \left(E_{\alpha i} \times 2^i\right)$. 

Au final, nous pouvons en conclure que si $\left(\tau_i = 1\right)$ alors $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.

Ce qui vient d'être dit ci-dessus ne démontre pas pour autant que $\left(E_{\alpha} \lt E_{\beta}\right)$.

### Les zéros anonymes non capitaux

Prenons le cas de $\left(\tau_i = \tau_{14} = 1\right)$.
Dans se cas spécifique, nous avons affaire a un point terminal car $\left(E_{\alpha} \lt E_{\beta}\right)$.
Ce point terminal est atteint par le fait que $\left(E_{\beta 14} \times 2^{14}\right) \ \gt \ \sum_{i=14}^{10} \ \left(E_{\alpha i} \times 2^i\right)$, sachant que $E_{\alpha 14}$ et $E_{\beta 14}$ sont les _MSB_ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.

En bref, nous comprenons que même si $\left(\tau_{\left(i-1\right)} = \tau_{13} = 0\right)$, alors ce _zéro anonyme_ ne changera rien au fait que $\left(E_{\alpha} \lt E_{\beta}\right)$.
La raison en est que l'inéquation qui figure ci-dessus reste valide qu'importe la valeur du bit $E_{\alpha 13}$, qu'il soit à $0$ ou à $1$.

De manière plus général, nous pouvons en déduire que tout _zéro anonyme_ de poids inférieur au _MSB1_ de $\tau \in \left[11;14\right]$, est _non capital_.

### Les zéros anonymes capitaux

Prenons désormais le cas dans lequel le poids du _MSB1_ de $\tau$ est $\tau_i$ pour $i \in \left[10;13\right]$.
Dans cette situation, nous sommes certains qu'il y a _au moins_ le bit $\tau_{\left(i+1\right)}$ qui est d'un poids supérieur à $\tau_i$.
Par définition, $\tau_{\left(i+1\right)}$ (et les autres bits de poids supérieur à $\tau_i$, si il y en a) sont des _zéros anonymes_.
Le problème des bits de poids supérieur à $\tau_i$, c'est que par essence ils ne permettent pas de déduire la valeur des bits de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.
Par exemple pour $\tau_{\left(i+1\right)}$, nous ne savons pas si $\left(E_{\alpha\left(i+1\right)} = E_{\beta\left(i+1\right)}\right)$ ou si $\left(E_{\alpha\left(i+1\right)} \gt E_{\beta\left(i+1\right)}\right)$.

Ceci pose problème car les _zéros anonymes_ de poids supérieur à $\tau_i$ sont décisifs dans le processus de génération d'un point terminal.
A ce stade nous savons que $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$, mais comme mentionné plus haut ceci ne suffit pas à dire que $\left(E_{\alpha} \lt E_{\beta}\right)$.

Par conséquent, si $\ Write \ \left(\lambda, \ \sum_{i=i+1}^{14} \ \left(E_{\beta i} \times 2^i\right)\right)$ et que $\left(\lambda \ = \ \sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right)\right)$, alors le point terminal était et reste $\left(E_{\alpha} \lt E_{\beta}\right)$.
Cependant, si $\left(\lambda \ \lt \ \sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right)\right)$ alors le point terminal change pour $\left(E_{\alpha} \gt E_{\beta}\right)$, car $\left(\sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right) \ \gt \ \sum_{i=14}^{10} \ \left(E_{\beta i} \times 2^i\right)\right)$.
Autrement dit, la somme de la valeur des bits de poids supérieur à $\tau_i$ (le _MSB1_ de $\tau$) dans $E_{\alpha}$, est strictement supérieur à la valeur totale du champs $E_{\beta}$.

Nous comprenons donc que les _zéros anonymes_ de poids supérieur au _MSB1_ de $\tau \in \left[10;13\right]$ devront être traités avec attention par le circuit, ce sont des _zéros anonymes capitaux_.

### L'obtention d'un point non terminal

Jusqu'ici nous avions un _MSB1_ dans $\tau \in \left[10;14\right]$, mais il pourrait ne pas y en avoir.
Dans cette situation, chacun des bits de $\tau \in \left[10;14\right]$ est un _zéro anonyme_, ce qui nous permet d'affirmé que $\left(E_{\alpha} \ge E_{\beta}\right)$.

Nous pourrions atteindre un point terminal si jamais $\left(E_{\alpha} \gt E_{\beta}\right)$, mais ce point terminal ne pourrait s'obtenir qu'à la condition que $\left(E_{\alpha i} \gt E_{\beta i}\right)$ pour $i \in \left[10;14\right]$.
A l'inverse, nous pourrions aussi atteindre une situation inédite de *___point non terminal___ si $\left(E_{\alpha} = E_{\beta}\right)$.
Cette situation inédite ne peut se produire que dans le cas où l'ensemble des bits de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ sont identiques, ou autrement dit $\forall \left(i \in \left[10;14\right]\right)$ alors $\left(E_{\alpha i} = E_{\beta i}\right)$.

Pour conclure, les _zéros anonymes_ qui composent l'entièreté du champs $\tau \in \left[10;14\right]$ sont capitaux à prendre en compte pour le circuit, car ils ont chacun le pouvoir de faire basculé un _point non terminal_ en un _point terminal_.

*Je rappelle qu'un _point terminal_ se définit comme suit: "_Un point terminal est le fait que le circuit soit capable de déduire un résultat correct, par le seul traitement des champs d'exposant._".
Un ___point non terminal___ est tout logiquement l'inverse.
Lorsque les champs d'exposant $E_{\alpha}$ et $E_{\beta}$ sont égaux, il n'est pas possible de généré un résultat correct avec le seul traitement de ces champs.
Le circuit doit alors poursuivre les traitements sur les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Ce que nous allons faire prochainement.

## Rappel sur les champs d'exposant et de mantisse tronquée

Le chapitre "_Ordre de traitement des champs d'exposant et de mantisse tronquée_" explique pourquoi est ce que les champs d'exposant $E$ sont traités avant les champs de mantisse tronquée $T$.
Dans ce chapitre, il y a la formule mathématique $\left(\left(1+T\right) \times 2^E\right)$ qui explique comment est ce qu'un nombre flottant est représenté en IEEE-754.
Nous comprenons que si deux nombres flottants $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$ ont des champs d'exposant égaux $\left(E_{\alpha} = E_{\beta}\right)$, alors la valeur de ces deux nombres $\left(\left(1+T_{\alpha}\right) \times 2^{E_\alpha}\right)$ et $\left(\left(1+T_{\beta}\right) \times 2^{E_\beta}\right)$ ne dépend que de $T_{\alpha}$ et $T_{\beta}$.
C'est pourquoi la première partie de la démonstration ne génère un _point non terminal_ que lorsque $\left(E_{\alpha} = E_{\beta}\right)$, après quoi le traitement se propage aux champs de mantisse tronquée eux même.

Les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$ font l'oeuvre d'une comparaison de supériorité stricte de l'un envers l'autre, comme pour les champs d'exposant.
Je vous rappelle que les champs d'exposant et de mantisse tronquée ont des points communs au niveau de leurs encodages, tout est expliqué dans le chapitre "?TBD?" pour ceux qui ont besoin d'un rafraichissement.
En bref, l'encodage du champs de mantisse tronquée partage les même propriétés que celui d'exposant.
Comme nous allons le voir plus bas, la comparaison entre les champs de mantisse tronquée suit le même processus de traitement que celui des champs d'exposant.

## Le traitement des champs de mantisse tronquée

Voici dans un premier temps de quoi dépend le traitement des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Nous remarquerons que c'est la même opération (à quelques détails prêt) que pour les champs d'exposant $E$ des opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$.

$$\sum_{i=9}^0 \ Write\left(\tau_i, \ Nimply \ \left(T_{\beta i}, \ T_{\alpha i}\right)\right)$$

La variable $\tau$ est techniquement la même que celle utilisée pour la démonstration précédente.
La seule différence, c'est que le bit de résultat de chaque opération $Nimply$ sur $T_{\beta i}$ et $T_{\alpha i}$ est inscrit dans $\tau_i$, pour $i \in \left[0;9\right]$.

### Un échec certain de la comparaison 

Dans le chapitre "_La mantisse tronquée, une histoire de puissance de 2_" qui parle de l'encodage du champs de mantisse tronquée des nombres flottants IEEE-754.
Il est dit que la valeur d'un bit à $1$ de poids $i$ d'un champs de mantisse tronquée $T$, est _inconditionnellement_ supérieur à la somme de la valeur de chacun de ses bits de poids inférieur à $i$.
Rappellons que c'est aussi le cas pour les champs d'exposant, comme cela a été mentionné dans la première partie de la démonstration.
Le calcul de comparaison entre les champs d'exposant se base sur ces primitives mathématiques, et il en va de même pour les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.

Prenons le cas de $\left(\tau_i = \tau_9 = 1\right)$.
En sachant que $\left(\tau_9 = 1\right)$ nous savons que $\left(T_{\beta 9} = 1\right)$ tandis que $\left(T_{\alpha 9} = 0\right)$, et par conséquent $\left(T_{\beta 9} \times 2^9\right) \gt \sum_{i=9}^0 \ \left(T_{\alpha i} \times 2^i\right)$.
Etant donné que $T_{\alpha 9}$ ainsi que $T_{\beta 9}$ sont les _MSB_ des champs de mantisse tronquée $T$, nous pouvons alors conclure que $\left(T_{\alpha} \lt T_{\beta}\right)$. 
La condition initial que teste le circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ __échoue__.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$, alors $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \lt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$, ou dit autrement $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$.

N'oublions pas qu'il peut y avoir des _zéros anonymes_ dans $\tau \in \left[0;8\right]$, mais ces derniers ne peuvent être que de poids inférieur au _MSB1_ de $\tau$ qui est $\tau_9$.
Ces potentiels bit à $0$ sont alors des _zéros anonymes non capitaux_, sans importance.

### Lorsque les zéros anonymes rendent la génération du résultat incertain

Je tient à rappellé qu'il faudra gardé en tête le fait que $\left(E_{\alpha} = E_{\beta}\right)$ pour toute les expressions qui figurent ci-dessous.

Prenons désormais le cas dans lequel le poids du _MSB1_ de $\tau$ est $\tau_i$ pour $i \in \left[0;8\right]$.
Dans cette situation, nous sommes certains qu'il y a _au moins_ le bit $\tau_{\left(i+1\right)}$ qui est d'un poids supérieur à $\tau_i$.
Les bits de poids supérieur à $\tau_i$ sont tous des _zéros anonymes capitaux_, nous devons donc y prêté attention.

Nous calculons la somme de la valeur des bits de $T_{\beta}$ d'un poids supérieur à $\tau_i$ avec $\ Write \ \left(\lambda, \ \sum_{i=i+1}^9 \ \left(T_{\beta i} \times 2^i\right)\right)$, et nous enregistrons le résultat dans $\lambda$.
Puis nous faisons le même calcul pour $T_{\alpha}$, et vérifions si le résultat obtenu est égale à ce que nous avons trouvé pour $T_{\beta}$, c'est à dire $\lambda$.
Dans le cas où $\left(\lambda \ = \ \sum_{i=i+1}^9 \ \left(T_{\alpha i} \times 2^i\right)\right)$ alors $\left(T_{\alpha} \lt T_{\beta}\right)$ car $\left(T_{\beta i} \times 2^i\right) \gt \sum_i^0 \ \left(T_{\alpha i} \times 2^i\right)$.
Par conséquent, la condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ __échoue__ car $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \lt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$, ou autrement dit $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$.

Cependant, si $\left(\lambda \ \lt \ \sum_{i=i+1}^9 \ \left(T_{\alpha i} \times 2^i\right)\right)$ alors $\left(T_{\alpha} \gt T_{\beta}\right)$ car $\left(\sum_{i=i+1}^9 \ \left(T_{\alpha i} \times 2^i\right) \ \gt \ \sum_{i=9}^0 \ \left(T_{\beta i} \times 2^i\right)\right)$.
Autrement dit, la somme de la valeur des bits de poids supérieur à $\tau_i$ (le _MSB1_ de $\tau \in \left[0;8\right]$) dans $T_{\alpha}$, est strictement supérieur à la valeur totale du champs de mantisse tronquée $T_{\beta}$.
La condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ ___réussie___ car $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$.

Pour finir, il est possible qu'il n'y ait pas de _MSB1_ dans $\tau \in \left[0;9\right]$, tout les bits composant $\tau$ sont alors des _zéros anonymes capitaux_.
Nous pouvons donc affirmé que $\left(T_{\alpha} \ge T_{\beta}\right)$.

Dans le cas où $\left(T_{\alpha i} \gt T_{\beta i}\right)$ pour $i \in \left[0;9\right]$, alors $\left(T_{\alpha} \gt T_{\beta}\right)$.
La condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ ___réussie___ car $\left(\sum_i^9 \ \left(T_{\alpha i} \times 2^i\right) \ \gt \ \sum_{i=9}^0 \ \left(T_{\beta i} \times 2^i\right)\right)$, ou autrement dit $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$.

A l'inverse, il est tout aussi probable que $\left(T_{\alpha} = T_{\beta}\right)$.
Pour cela il faudrait que chaque bit de même poids des champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$ soient identiques, ou pour le dire autrement $\forall \left(i \in \left[0;9\right]\right)$ alors $\left(T_{\alpha i} = T_{\beta i}\right)$.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$ et que $\left(T_{\alpha} = T_{\beta}\right)$, alors $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \ = \ \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$ et la condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ ___échoue___.

# Conclusion des démonstrations

Les processus de traitement des champs d'exposant et des mantisses tronquées pour les opérandes $\alpha$ et $\beta$ sont identiques, comme nous avons pu le voir dans les démonstrations qui figurent ci-dessus.
Dans les faits, le circuit électronique n'est composé que d'un seul sous ensemble de circuit logique, permettant ainsi le traitement de chacun des champs des opérandes flottants.
La documentation du circuit aborde l'architecture de ces sous ensembles de circuit logique.

Je tient aussi à rappellé que tout ce que nous avons vu ici n'est qu'une démonstration de la faisabilité de la logique de comparaison du circuit.
Pour autant, les opérations que produit réelement le circuit ne sont pas les même que celle qui sont utilisées dans la démonstration.
Par exemple, le circuit ne contient pas d'additionneur.
La raison est simplement qu'il est plus optimal de faire différemment.
De plus, le circuit génère deux bits de sortie dont l'un par le biais du comparateur directement.
Le circuit de comparaison est donc architecturé d'une certaine manière de sorte à pouvoir généré un bit à $0$ en cas d'échec de la comparaison, et $1$ pour une réussite.
L'autre bit de sortie est généré par un autre circuit (une porte logique $Xor$), qui aggrège la sortie du comparateur avec un autre bit dont il est trop difficile d'expliqué l'origine ici.
Tout ces sujets sont abordés dans la documentation elle même.

# Les différents nombres du standard IEEE-754

Il me faut aussi précisé que le standard IEEE-754 peut représenté plusieurs "type" de nombre:
  - Les nombres _normaux_ (ceux dont nous parlons implicitement depuis le début du document)
  - Les nombres ___dénormaux___
  - Les _NaN_ (Not a Number)
  - L'_infini_ positif ou négatif

Chacun de ces types de nombre peut être codé dans n'importe quel format IEEE-754.
Chaque type de nombre se différencie des autres en utilisant une certaine plage de codage dans le champs d'exposant et de mantisse tronquée, plage de codage qui a été préfixé par le standard IEEE-754.

### Les nombres normaux

Comme dit plus haut, les nombres _normaux_ (ou _normalisés_) utilisent une certaine partie de la plage de codage du champs d'exposant.
Mais n'oublions pas que ce champs a un encodage quelques peu spéciale dont nous avons parlé en début de document, dans le chapitre du nom de "_L'encodage par biais du champs d'exposant_".
Je vous invite à relire ce chapitre si nécessaire.
En bref, avec cet encodage par biais il faut faire la distinction entre la valeur codé dans le champs binaire d'exposant, et la valeur que représente le champs d'exposant lui même.

Le champs d'exposant a un biais qui se calcul de la manière suivante $\left(2^{\left(N - 1\right)} - 1\right)$, avec $N$ le nombre de bits qui compose le champs.
Ce biais est la raison pour laquelle nous devons différencié la valeur codé dans le champs d'exposant, de la valeur qu'il représente.
Plus particulièrement, le champs d'exposant code une valeur en _Binary Unsigned_ et représente une puissance équivalente à la valeur codé dans le champs, moins le biais.
Pour un champs d'exposant $E$, la valeur qu'il représente est $\left(E - biais\right)$.

Tout nombre _normalisé_ a un champs d'exposant dont la plage de codage se situe entre $\left[1;\left(2^N - 1\right)\right[$.
Maintenant que nous connaissons les bornes minimal et maximal pour le codage de l'exposant de tout nombre _normalisé_, nous pouvons calculé la valeur du plus petit exposant avec $\left(1 - \left(2^{\left(N - 1\right)} - 1\right)\right)$, et du plus grand exposant avec $\left(\left(2^N - 2\right) - \left(2^{\left(N - 1\right)} - 1\right)\right)$.
En parallèle, le champs de mantisse tronquée n'a pas de plage de codage définit par le standard.
Un nombre _normalisé_ peut être reconnu à la seul valeur de son champs d'exposant.
Le bit de signe permet simplement d'indiqué si le nombre est positif ou négatif.

L'encodage d'un nombre _normalisé_ $F$ se base sur l'écriture scientifique binaire du nombre $F$.
Plus particulièrement, le champs de mantisse tronquée doit avoir une valeur équivalente au significande du nombre à représenté, même si dans les faits ce n'est pas le cas.
Le significande en _écriture scientifique binaire_ a une valeur comprise entre $\left[1;2\right[$.
La partie entière de ce dernier est donc toujours composé d'un unique bit à $1$.
Par conséquent, le champs de mantisse tronquée d'un nombre _normalisé_ ne code pas le bit de la partie entière du significande, car nous savons que ce dernier est à $1$.
Le champs de mantisse tronquée ne représente donc que la partie fractionnaire du significande, et rendre la partie entière implicite permet de gagner un bit de précision dans le codage de la partie fractionnaire.

### Les nombres NaN

Pour le standard IEEE-754, il existe des nombres invalides du nom de ___NaN___ (_Not a Number_).
Un nombre _NaN_ est généré par un calcul considéré comme invalide par le standard lui même et/ou par les mathématiques, c'est en quelque sorte une alternative à la génération d'exception matériel.
Par exemple le calcul $\left(\left(+ \ \infty\right) \div 0\right)$ génère un _NaN_ pour deux raisons.
Premièrement, divisé par $X$ l'infini positif ne donne pas de résultat concret, de plus si $\left(X = 0\right)$ alors le calcul se retrouve soudainement à être invalide mathématiquement parlant. 
Comme autre exemple, si un calcul a un opérande qui est un _NaN_, alors il génère également _NaN_ en résultat $\left(3.5 - NaN\right)$.

En bref, l'encodage d'un _NaN_ nécessite que le champs d'exposant biaisé code la valeur $\left(2^N - 1\right)$, pour $N$ le nombre de bits qui compose le champs.
Parallèlement, le champs de mantisse tronquée peut codé n'importe quel valeur tant qu'elle est non nul.
Aussi, comme dit plus haut un nombre _NaN_ ne génère jamais de résultat valide lorsqu'il est utilisé en tant qu'opérande pour un calcul arithmétique.
Par conséquent, il n'y a pas d'utilité à ce que l'Unité de Configuration de la FPU ne les prennent pas en charge.

### L'infini positif et négatif

Il se trouve que le standard IEEE-754 définit un moyen de codé un nombre infini positif, ou négatif $\left(\pm \ \infty\right)$.
Le codage de l'infini $\left(\infty\right)$ nécessite un champs d'exposant biaisé dont la valeur est $\left(2^N - 1\right)$, avec $N$ le nombre de bits qui compose le champs.
Par ailleurs, il faut que le champs de mantisse tronquée soit nul (composé uniquement de bits à $0$), sinon le nombre serait _NaN_.
Seul le bit de signe permet de passé de l'infini positif à négatif $\left(\pm\right)$.

Comme nous avons pu le voir avec l'exemple ci-dessus, un calcul arithmétique avec un opérande de valeur infini positif ou négatif génère un _NaN_ en résultat.
Ceci est vrai pour n'importe quel type de calcul arithmétique.
A l'instar des nombres _NaN_, L'unité de Configuration de la FPU n'a donc aucun intérêt à prendre en charge les opérandes de valeur infini positif comme négatif.

## La plage de codage des nombres dénormaux

Pour finir, les nombres _dénormaux_ représentent de vrai valeur concrète, au contraire des nombres _NaN_ ou $\pm \ \infty$.
Les nombres _dénormaux_ ont une plage de codage qu'il va être interéssante d'étudier, voyons d'abord ce qu'est un nombre _dénormalisé_.

Un nombre _dénormalisé_ a une valeur qui se situe entre $0$, et le plus petit nombre positif _normalisé_ pouvant être codé sur n'importe quel format de flottant IEEE-754.
Le rôle des nombres _dénormaux_ est de pouvoir codé des nombres très proche de $0$, qui plus est des nombres plus proche de $0$ que ne serait capable de le faire une représentation _normalisé_.
La plage de codage des nombres _dénormaux_ change en fonction du format de flottant, mais concentrons nous sur le format _Half Precision_.

### Le codage du plus petit nombre positif et normalisé au format Half Precision

Commençons par définir la valeur du champs d'exposant.
La section "_Les nombres normaux_" qui figure ci-dessus, nous dit que la plus petite valeur qui peut être codé sur le champs d'exposant d'un nombre _normalisé_ est $1$.
Je cite "_Tout nombre normalisé a un champs d'exposant dont la plage de codage se situe entre_ $\left[1;\left(2^N - 1\right)\right[$.".
Rappelons que le biais du champs d'exposant vaut $\left(2^{\left(N - 1\right)} - 1\right)$, où $N$ est le nombre de bits qui composent le champs d'exposant au format _Half Precision_, c'est à dire $5$.
Le champs d'exposant représente alors la puissance $\left(1 - 15\right) = -14$.

Le champs de mantisse tronquée est nul quant à lui.
En prenant en compte le bit implicite à $1$ de la partie entière du champs de mantisse tronquée d'un nombre _normalisé_, nous calculons la valeur réel de la mantisse qui est $\left(1 + Truncated \ Mantissa\right)$.
Le plus petit nombre positif et _normalisé_ pouvant être codé au format _Half Precision_ est donc $\left(\left(1 + 0.0000000000_2\right) \times 2^{-14}\right)$, et voici l'illustration de son codage:

$$\left(2\right) \quad \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

### Le codage du zéro positif ainsi que négatif

Il s'avère que la norme IEEE-754 supporte un zéro positif $\left(+0\right)$ ainsi que négatif $\left(-0\right)$, en fonction de la valeur du bit de signe.
Ceci engendre quelques diffculté de comparaison, par exemple quel résultat générer pour la condition $\left(\left(+0\right) = \left(-0\right)\right)$?
En revanche, le circuit n'est pas affecté par cela car rappellons-le, il n'utilise que la valeur absolu de ses opérandes flottants.
L'Unité de Configuration de la FPU est donc capable de prendre en charge des opérandes nuls.
Pour le codage d'un zéro positif comme négatif, il est nécessaire que le champs d'exposant biaisé soit nul.
Le champs de mantisse tronquée doit aussi être nul, sous peine que le codage d'un zéro positif ou négatif se confonde avec celui d'un nombre _dénormalisé_ (comme nous le verrons dans le chapitre suivant).
Quant au bit de signe, il est fautif du fait qu'il existe un zéro positif et négatif.

Voici une illustration du codage d'un zéro positif/négatif au format _Half Precision_ :

$$\left(3\right) \quad \left[S_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

Avec $S$ le bit de signe qui vaut $0$ pour un zéro positif, et $1$ pour un zéro négatif.

## Le codage des nombres dénormaux

Il y a deux différences entre les nombres _normalisés_ et _dénormalisés_ :
  - Le bit implicite de la mantisse tronquée
  - L'interprétation que l'on fait de la valeur codé par le champs d'exposant

Commençons par comprendre ce qui concerne le bit implicite.

### Lorsque le bit implicite de la mantisse tronquée est à 0

La représentation en IEEE-754 d'un nombre _normalisé_ $F$, doit respecter les règles de l'écriture scientifique binaire.
Particulièrement, la valeur du champs de mantisse tronquée doit correspondre à celle du significande de l'écriture scientifique binaire du nombre $F$.
Dans les faits, nous avons vus plus haut que la mantisse tronquée ne représente que la partie fractionnaire d'un significande.
Le bit de la partie entière du significande étant toujours à $1$, il est rendu implicite pour les nombres _normaux_ afin de gagner un bit de précision sur le codage des nombres.

Cependant, le codage d'un nombre IEEE-754 dit "_dénormalisé_" ne respecte pas les règles de l'écriture scientifique binaire.
Le champs de mantisse tronquée n'a plus de lien avec un quelconque significande.
Effectivement, les nombres _dénormaux_ définissent un bit implicite à $0$ pour le champs de mantisse tronquée.
Le bit implicite est toujours perçu comme la partie entière d'un nombre (qui est _dénormalisé_ cette fois-ci), quant au champs de mantisse tronquée il en interprète la partie fractionnaire.
C'est pourquoi la valeur réel du champs de mantisse tronquée d'un nombre _dénormalisé_ est alors directement celle du champs lui même $\left(0 + Truncated \ Mantissa\right)$.
La partie entière est nul, c'est logique.

Jusqu'ici, la seule différence entre un nombre _normalisé_ et _dénormalisé_ est la valeur du bit implicite du champs de mantisse tronquée.
Cependant, le bit implicite n'étant pas lui même réelement codé dans le champs binaire de ces deux "type" de nombre, il faut alors trouver un autre moyen pour départager les nombres _normaux_ des nombres _dénormaux_.
Le standard IEEE-754 définit alors le champs d'exposant d'un nombre _dénormalisé_ comme devant être nul, afin que la séparation entre nombre _normalisé_ et _dénormalisé_ devienne explicite.
Mais n'oublions pas la section "_Le codage du zéro positif et négatif_", dans laquelle il est dit que le nombre $\pm \ 0$ est codé avec un champs d'exposant et de mantisse tronquée nuls.
Ce qui contraint le codage des nombres _dénormaux_ à avoir un champs de mantisse tronquée non nul.

Mais rappelons que "_Le rôle des nombres _dénormaux_ est de pouvoir codé des nombres très proche de_ $0$", et non $0$ lui même.
Pour cela, il est dans tout les cas nécessaire que le champs de mantisse tronquée d'un nombre _dénormalisé_ soit différent de $0$.
Ce qui empêche de confondre les zéros positifs ou négatifs, avec les nombres _dénormaux_.

### L'interprétation de la valeur du champs d'exposant des nombres dénormaux

Au delà du fait que le champs d'exposant permet de savoir si un nombre est _normalisé_ ou _dénormalisé_, il permet bien évidemment de codé une puissance. 
Cependant, pour un nombre _dénormalisé_ la valeur que représente le champs d'exposant n'est pas interprétée comme d'habitude.

Les nombres _normaux_ et _dénormaux_ sont deux représentations distinctes des nombres dans un flottant IEEE-754.
Il s'avère que les nombres d'une même représentation sont consécutifs.
Dans l'idéal, il faudrait qu'en passant d'une représentation à une autre les nombres de ces deux représentations le soient aussi.
C'est ce que l'on appelle le principe de continuité, nous verrons cela plus en détails dans le chapitre qui suit "_La continuité de représentation des nombres dénormaux_".
Il faut savoir que cette continuité est atteignable entre les nombres _normaux_ et _dénormaux_, qu'à la condition que l'interprétation de la valeur du champs d'exposant d'un nombre _dénormalisé_ ne suive pas celle d'un nombre _normalisé_.

Pour un champs d'exposant biaisé $E$ d'un nombre _normalisé_, nous calculons la valeur que représente le champs avec $\left(E - biais\right)$.
Nous soustrayons simplement le biais du champs d'exposant, à la valeur qu'il code.
Cependant, pour qu'une continuité soit possible entre nombres _normaux_ et _dénormaux_, il faut que la valeur que représente le champs d'exposant nul d'un nombre _dénormalisé_ soit perçu comme la plus petite puissance dont peut faire usage un nombre _normalisé_.
Autrement dit, un nombre _dénormalisé_ a un champs d'exposant $\left(E = 0\right)$ dont la puissance interprétée est équivalente à $\left(1 - biais\right)$.

## La continuité de représentation des nombres dénormaux

Réintroduisons en une phrase ce que sont les nombres _dénormaux_.
Un nombre _dénormalisé_ représente une valeur qui se situe entre $0$ et le plus petit nombre positif _normalisé_ pour un format donné.
Dans la théorie, tout nombre dont la valeur se situe entre $0$ et le plus petit nombre positif _normalisé_, devrait pouvoir être représenté sous la forme d'un nombre _dénormalisé_.
Dans les limites de la représentation binaire bien entendu.
Si c'est le cas, alors les représentations des nombres _normaux_ et _dénormaux_ sont continues. 
Cependant, pour cela il faut que le champs d'exposant des nombres _dénormaux_ soit interprété de la bonne manière, interprétation dont il est le sujet dans le chapitre précédent.

Nous allons démontrer cela mathématiquement au travers des deux chapitres prochain.

### Représentation non continu des nombres dénormaux

Commençons par voir les problèmes auquels nous ferions face si le standard IEEE-754 interprétait la valeur du champs d'exposant d'un nombre _dénormalisé_, comme celle d'un nombre _normalisé_.
C'est à dire, par la soustraction du biais du champs d'exposant à la valeur qu'il code.  
Ce chapitre et le suivant auront tout deux besoin des trois nombres $\alpha$, $\beta$ et $\tau$, qui ont ci-dessous chacun une illustration de leur codage au format _Half Precision_.

Pour commencer, je recopie ici même l'illustration numéro $\left(2\right)$ du plus petit nombre positif et _normalisé_ pouvant être codé au format _Half Precision_.
Ce nombre _normalisé_ sera désormais connu sous le nom de $\alpha$.

$$\alpha: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

Pour calculer la valeur de $\alpha$ nous avons besoin de la formule $\left(\left(1 + Truncated \ Mantissa\right) \times 2^{\left(Exponent - bias\right)}\right)$, ce qui donne $\left(\left(1 + 0.0\right) \times 2^{\left(1 - 15\right)}\right) = 0.00006103515625$.

Je présente également l'illustration du plus grand nombre positif et _dénormalisé_ pouvant être codé au format _Half Precision_.
Ce nombre _dénormalisé_ sera connu sous le nom de $\beta$.

$$\beta: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 1_9, \ 1_8, \ 1_7, \ 1_6, \ 1_5, \ 1_4, \ 1_3, \ 1_2, \ 1_1, \ 1_0\right]$$

Nous allons calculer la puissance que représente le champs d'exposant $E$ de $\beta$ avec $\left(E - biais\right)$, comme nous le ferions pour n'importe quel nombre _normalisé_.
Pour calculer la valeur de $\beta$ nous avons alors besoin de la formule $\left(\left(0 + Truncated \ Mantissa\right) \times 2^{\left(Exponent - bias\right)}\right)$, ce qui donne $\left(\left(0 + 0.9990234375\right) \times 2^{\left(0 - 15\right)}\right) = 0.0000304877758026123046875$.

Et pour finir, voici l'illustration du plus petit nombre positif et _dénormalisé_ pouvant être codé au format _Half Precision_.
Par définition, ce nombre est alors le plus petit des nombres positif non nul codable sur un _Half Precision_, c'est le plus proche de $0$.
Il portera le nom de $\tau$.

$$\tau: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 1_0\right]$$

Nous devons faire attention à la même chose pour $\tau$ que pour $\beta$.
Cela veut dire que nous devons calculé la puissance que représente le champs d'exposant $E$ de $\tau$ avec $\left(E - biais\right)$, comme nous le ferions pour n'importe quel nombre _normalisé_.
C'est un nombre _dénormalisé_ tout comme $\beta$ après tout.
La valeur de $\tau$ se calcul de la manière suivante $\left(\left(0 + 0,0009765625\right) \times 2^{\left(0 - 15\right)}\right) = 0.0000000298023223876953125$.

Nous pouvons remarquer quelque chose d'intéressant $\left(\alpha \div \beta\right) \approx 2.0019$.
Autrement dit, $\beta$ qui représente le plus grand nombre positif _dénormalisé_ est deux fois plus petit que $\alpha$ qui représente le plus petit nombre positif _normalisé_.
Rappelons ce qui a été dit plus haut, je cite "_Dans la théorie, tout nombre dont la valeur se situe entre_ $0$ _et le plus petit nombre positif normalisé, devrait pouvoir être représenté sous la forme d'un nombre dénormalisé_.".
Dans les faits, nous nous rendons compte que ce n'est pas le cas.
La raison en est que la plage de codage des nombres _dénormaux_ comporte un "trou" de valeur ne pouvant pas être représenté.

Par exemple, pour le calcul $\left(\alpha - \tau\right)$ aussi proche de $\alpha$ que puisse être le résultat, il n'est pourtant pas représentable.
Le résultat de ce calcul est $0,0000610053539276123046875$ et figure dans l'intervalle de valeur $\left]\beta;\alpha\right[$ ne pouvant donc être codé ni par des nombres _normaux_, ni par des nombres _dénormaux_.

Dans les faits, les calculs sur les flottants occasionnent souvent des arrondis, du fait de l'incapacité de toute base numérique à pouvoir représenté certain nombre de manière fini.
Prenez l'exemple de $\left(1 \div 3\right)$ en base décimale, ou encore $\left(1 \div 10\right)$ en binaire pour vous en convaincre.
Le standard IEEE-754 défini des méthodes d'arrondissement que nous ne détaillerons pas dans ce document.
Pour autant, le résultat de $\left(\alpha - \tau\right)$ est si proche de la valeur de $\alpha$, qu'il pourrait être possible d'utiliser une méthode d'arrondi par le haut pour obtenir un résultat suffisamment précis pour certain type de calcul.
Mais si cela est vrai pour $\tau$, ça ne l'est pas pour un nombre comme $0.000045$.
Le nombre $0.000045$ a une valeur aussi lointaine de $\alpha$ que de $\beta$ pour ainsi dire.
N'importe quel méthode d'arrondi transformerait beaucoup ce nombre jusqu'à le rendre trop approximatif pour presque tout les cas d'usage.

Voilà le problème que pose l'absence de continuité entre la représentation des nombres _normaux_ et _dénormaux_.

### Représentation continu des nombres dénormaux

Dans le chapitre du nom de "_L'interprétation de la valeur du champs d'exposant des nombres dénormaux_", nous avons vu comment est ce que le champs d'exposant des nombres _dénormaux_ devait être interprété.
Un nombre _dénormalisé_ a obligatoirement un champs d'exposant nul, ce dernier interprète une puissance dont la valeur est alors équivalente au résultat de $\left(1 - biais\right)$.
Ce qui veut dire que:
  - $\beta = \left(\left(0 + 0.9990234375\right) \times 2^{\left(1 - 15\right)}\right) = 0,000060975551605224609375 = 0.000000000000001111111111_2$
  - $\tau = \left(\left(0 + 0,0009765625\right) \times 2^{\left(1 - 15\right)}\right) = 0,000000059604644775390625 = 0.000000000000000000000001_2$

Si nous reprenons l'exemple de $\left(\alpha - \tau\right)$, le résultat est alors $\beta$.
Rappelons nous que $\tau$ représente le plus petit nombre pouvant être codé au format _Half Precision_.
En obtenant $\beta$ par la soustraction de $\tau$ à $\alpha$, nous démontrons que les plages de codage des nombres _normaux_ ainsi que _dénormaux_ sont continues.
Le fait est que nous ne pourrions pas soustraire une plus petite valeur que $\tau$, au plus petit des nombres _normaux_ que représente $\alpha$.
Le résultat obtenu est tout juste le plus grand des nombres _dénormaux_ pouvant être codé au format _Half Precision_.
Il n'y a donc par définition aucun "trou" de valeur non représentable dans la plage de codage des nombres _dénormaux_.
D'où le terme de continuité.

Pour voir les choses autrement.
Le résultat $T$ de l'addition entre le bit implicite et le champs de mantisse tronquée de chaque nombre $\alpha$, $\beta$ et $\tau$, est multiplié par $2^{\left(1 - 15\right)}$.
La puissance de $2^{-14}$ déplace de $14$ rangs vers la gauche la virgule du champs de mantisse tronquée de nos trois nombres.
Mais nous pouvons faire le choix de ne pas effectuer ces décalages identique les uns aux autres.
Par conséquent si $\left(\alpha - \tau\right) = \beta$ alors $\left(T_{\alpha} - T_{\tau}\right) = T_{\beta}$, ce qui donne $\left(1.0000000000 - 0.0000000001\right) = 0.1111111111$.
Constatez le fait que le plus petit nombre _normalisé_ $\alpha$, et le plus grand nombre _dénormalisé_ $\beta$ soient consécutifs.
Il y a donc continuité de représentation entre les nombres _normaux_ et _dénormaux_.

Pour finir, l'Unité de Configuration de la FPU prend en charge les nombres _normaux_ bien évidemment, mais aussi les nombres _dénormaux_.
Le codage des champs de mantisse tronquée d'un nombre _normalisé_ comme _dénormalisé_ est identique, c'est ce qui permet au circuit de prendre en charge les comparaisons entre deux nombres _dénormaux_.
Mais ce n'est pas tout, il est techniquement possible pour le circuit de traité un nombre _normalisé_ avec un nombre _dénormalisé_.
N'oublions pas qu'un nombre _dénormalisé_ code un champs d'exposant nul, tandis que le champs d'exposant d'un nombre _normalisé_ code une valeur comprise dans $\left[1;\left(2^N - 1\right)\right[ (la valeur codé est non nul).
En bref, peu importe la condition initial, nous sommes certains que le circuit génèrera un point terminal en traitant les champs d'exposant de ses deux opérandes.








