Ce document contient la démonstration mathématique du circuit électronique ainsi que des explications sur les concepts les plus fondamentaux.

Le document se découpe en plusieurs parties:
  - L'encodage $Binary \ Unsigned$
  - Le standard IEEE-754
    - L'encodage par biais du champs d'exposant
    - La mantisse tronquée, une historie de puissance de $2$ négative
  - L'écriture scientifique binaire
    - Le significande
    - Le multiplicande
  - Codage des informations dans un nombre flottant IEEE-754
    - Composition du champs de mantisse tronquée et d'exposant
  - Le circuit traite les champs d'exposant avant ceux des mantisses tronquées
  - Démonstration mathématique
    - Traitement des champs d'exposant
    - Traitement des champs de mantisses tronquées

# I. L'encodage $Binary \ Unsigned$

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$.
Un champs binaire est quant à lui composé d'un ensemble de bit, c'est à dire d'un ensemble de $0$ et de $1$.

Ce sont ces champs binaire qui nous permettent de représenté des nombres dans un ordinateur, faire des calculs, mémorisé des résultats.
Il y a cependant plein de façon de représenté des nombres dans un champs binaire, nous parlons d'encodage des nombres.
Tout les encodages ne se valent pas car il y en a des plus efficasse que d'autre en fonction des besoins, prenons un exemple.
Pour un champs de $16$ bits nous pourrions calculé la somme des bits pour représenter un nombre.
Avec cet encodage nous ne pourrions codé que des valeurs entre $0$ et $16$ inclus, ce qui n'est pas optimal pour bien des situations.
Mais parmis les multiples encodage existant, le plus connu de tous porte le nom de $Binary \ Unsigned$.

Le $Binary \ Unsigned$ utilise les mêmes primitives que la base décimale (celle que nous utilisons tous) pour représenter des nombres, `des puissances`.
Pour représenter la valeur $103$ en décimale, nous décomposons en réalité chaque chiffre du nombre pour les multipliés avec une puissance de $10_{10}$ adéquat, ensuite nous faisons la somme des résultats de chaque produit.
Regardez ci-dessous.

$$103_{10} = 1 \times 10^2 + 0 \times 10^1 + 3 \times 10^0$$

Veuillez noté que l'indice $X_{10}$ représente la base numérique dans laquelle le nombre $X$ qui précède est écrit.
Nous retrouvons chacun des chiffres du nombre entrain de multiplié une puissance de $10_{10}$.
Remarquons que le chiffre des unités $3$ est facteur de $10_{10}$ à la puissance $0$, le chiffre d'un ordre plus grand (celui des dizaines) est le facteur de $10_{10}$ à la puissance $1$ et enfin le chiffre des centaines multiplie $10_{10}$ à la puissance $2$.
La valeur des puissances croix en fonction de la position du chiffre, qui plus est, elle commence à $0$.

Ceci n'est pas propre à la base décimale mais à `nimporte quel base numérique`, comme la base binaire par exemple.
La base binaire (ou base $2$) utilise les chiffres $0$ et $1$ plutôt que ceux de la base décimale allant de $0$ à $9$.
Par ailleurs, un tel chiffre s'appelle un bit, ce qui est la contraction de $Binary$ $Digit$.
Aussi, les puissances de $10_{10}$ sont remplacés par des puissances de $2$.
Nous venons tout juste de décrire ce sur quoi se base l'encodage $Binary \ Unsigned$.

Un champs binaire pour lequel nous utilisons un encodage $Binary \ Unsigned$ associe une puissance de $2$ positive à chaque bit du champs.
Le champs représente un nombre $X$ par la somme des produits entre la valeur de chaque bit et la puissance de $2$ associée.
Techniquement, vu que dans chacun des produits la puissance de $2$ est multiplié par son bit associé, qui est à $0$ ou à $1$, soit le résultat est nulle ou soit il est équivalent à la puissance de $2$ associé au bit sous-jacent.
C'est pourquoi vous entenderez souvent dire (ici compris), qu'un nombre $X$ est représenté par la somme des puissances de $2$ dont le bit est à $1$.
Les puissances de $2$ vont de $0$ à $N - 1$, ou la valeur de $N$ est le nombre de bits qui compose un champs binaire.
Par ailleurs, il est souvent dit que chaque bit a un "poids".
Les poids correspondent aux exposants des puissances de $2$ associées à chacun des bits, plus cet exposant sera grand et plus un bit aura d'importance dans la représentation d'un nombre. 

Il est aussi définit ce que nous qualifions de $LSB$ pour $Least \ Significant \ Bit$.
Ce terme désigne le bit de poids le plus faible d'un champs, donc le bit de poids $0$ pour un champs dont l'encodage est en $Binary \ Unsigned$.
Au contraire, le bit de poids le plus fort d'un champs binaire est ce que l'ont appelle le $MSB$ pour $Most \ Significant \ Bit$.
Il est aussi possible de faire référence au bit à $1$ de poids le plus faible d'un champs avec le terme $LSB1$, ou à celui de poids le plus fort avec $MSB1$.

Il se trouve que le $Binary \ Unsigned$ nous donne la certitude que la valeur d'un bit à $1$ de poids $i$, est strictement supérieur à la somme des bits de poids inférieur à $i$.
Autrement dit $\left(1\times 2^i\right) \gt \left(\sum_{i=i-1}^0 2^i\right)$.
Prenons comme exemple un champs binaire de $8$ bits, pour lequel nous n'allons faire attention qu'à un seul bit, celui de poids $6$ admettons.
Disons que ce champs ressemble à ceci $01111111_{2}$, la valeur du nombre représenté est $127$ et le bit qui nous intéresse se trouve être le $MSB1$ du champs.
La somme des bits de poids inférieur au bit de poids $6$ donne un résultat strictement inférieur à $1 \times 2^6 = 64$.
Cela est systèmatiquement vrai, même lorsque les $i$ bits de poids inférieur représentent la valeur maximale encodable, comme dans cet exemple.

$$\left(1 \times 2^6 = 64\right) \gt \left(63 = \left(1 \times 2^5\right)+\left(1 \times 2^4\right)+\left(1 \times 2^3\right)+\left(1 \times 2^2\right)+\left(1 \times 2^1\right)+\left(1 \times 2^0\right)\right)$$

Encore une fois, ceci n'est pas propre à la base binaire mais à nimporte quel base numérique modulo quelques variation pour chaque base.

# II. Le standard IEEE-754

Après cette rapide introduction à l'encodage $Binary \ Unsigned$, passons au sujet suivant qui est l'encodage IEEE-754.

Notre circuit a pour fonction principal de produire des comparaisons entre deux opérandes flottants respectant le standard IEEE-754.
Pour plus d'information sur le circuit électronique, jetez un oeil à la documentation du circuit.
Cette norme IEEE-754 définit trois éléments qui composent chaque nombre à virgule flottante:
- Le bit de signe
- Un champs binaire d'exposant
- Un autre champs binaire pour la mantisse tronquée

Dans ce qui suit nous allons nous intéressé aux encodages utilisés dans les champs binaires d'exposant et de mantisse tronquée.
La raison en est que bien évidemment le fonctionnement du circuit en dépend.

# III. L'encodage par biais du champs d'exposant

Le champs d'exposant utilise un encodage par biais, ce dernier est assez simple à comprendre.
Enfaite, le champs d'exposant est un champs binaire pour lequel nous utilisons un encodage $Binary \ Unsigned$ qui code une valeur numérique $X$, comme nous l'avons vu précédemment.
A cela, il faut ajouté un biais $B$ pour obtenir la valeur représenté par le champs d'exposant.
Dans les faits le biais est une constante qui peut être positive ou négative.
La valeur que représente le champs d'exposant est alors issu du calcul $X + B$, cependant, dans le cadre de ce champs le biais $B$ est une constante systèmatiquement négative.
Elle se calcul de la manière suivante $-\left(2^{\left(N-1\right)}\right)+1$, où $N$ est le nombre de bits du champs d'exposant.

Etant donné que l'encodage par biais se base sur le $Binary \ Unsigned$, le champs d'exposant partage les même propriétés que cet encodage.
Notamment le fait que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des bits de poids inférieur à $i$.

# IV. La mantisse tronquée, une historie de puissance de $2$ négative

Pour le champs de la mantisse tronquée nous avons besoin de faire un rapide rappel sur l'encodage intial d'un nombre flottant.
La partie entière d'un nombre flottant utilise du $Binary \ Unsigned$ pour être codé, tandis que la partie fractionnaire fait usage d'une version modifiée du $Binary \ Unsigned$.
Chaque bit de la partie fractionnaire est un `facteur d'une puissance de 2 négative` et non positive.
Ceci permet de représenté des valeurs dans l'intervalle $\left[0;1\right[$.
Le poids des puissances négatives de $2$ décroix en fonction de la position du bit.
Prenons l'exemple du nombre $3.75$.

$$3.75 = 11.11_2 = Integer \ Part\left(\left(1 \times 2^1\right) + \left(1 \times 2^0\right)\right) + Fractional \ Part\left(\left(1 \times 2^{-1}\right) + \left(1 \times 2^{-2}\right)\right)$$

Précisons que sous la forme binaire du nombre $3.75$, le point n'est là que pour facilité sa lecture.
Dans les faits le point n'est pas réelement présent dans le codage des nombres flottants.
Ce qu'il y a d'important à remarqué pour la partie fractionnaire, c'est que la valeur d'un bit à $1$ de poids $i$, est toujours strictement supérieur à la somme des bits de poids inférieur à $i$.
Autrement dit, pour le nombre fractionnaire $F \in \left[0;1\right[$ dont ont ne prête attention qu'au bit à $1$ de poids $i$ alors $\left(1 \times 2^i\right) \gt \left(\sum_{i=i-1}^{lsb\left(F\right)} \left(F_i \times 2^i\right)\right)$.

Pour en revenir au sujet de la mantisse tronquée, elle est composée des bits de la partie entière et de la partie fractionnaire d'un nombre flottant.
Vu que la partie entière et fractionnaire d'un nombre flottant partagent les propriétés du $Binary \ Unsigned$, c'est aussi le cas de la mantisse tronquée elle même.

Nous verrons dans la démonstration mathématique, qu'il est utile que l'encodage des champs d'exposant et de mantisse tronquée aient les même propriétés que le $Binary \ Unsigned$, tout particulièrement.

# V. L'ordre de traitement des opérandes flottantes

Le circuit électronique compare la stricte supériorité des nombres flottants entre eux, nommons ces opérandes $\alpha$ et $\beta$.

Il se trouve que le circuit traite les champs d'exposants de $\alpha$ ainsi que de $\beta$, avant les champs de mantisse tronquée de ces même opérandes.
La raison en est que les champs d'exposant seul, peuvent permettre au circuit d'atteindre un point terminal.
Un point terminal n'est atteint que lorsque le circuit peut être sûre du résultat qu'il génère.
Techniquement, si $\left(E_{\alpha} \gt E_{\beta}\right)$ alors l'opérande $\alpha$ est strictement plus grand que $\beta$, nous verrons pourquoi ci dessous.
Cette conclusion permet au circuit de généré le bon résultat, en fonction de la comparaison à vérifié.

Il existe bien un cas non terminal cependant, celui où $E_{\alpha} = E_{\beta}$.
Dans cette situation, le circuit ne peut rien tiré des champs d'exposants des opérandes $\alpha$ et $\beta$, car ils sont égaux.
Alors, le circuit atteindra un point terminal en traitant les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$ en cas de dernier recours.

Dans les chapitres suivant, je vais expliqué dans les fondements pourquoi est ce que les champs d'exposant sont traités en priorité par le circuit électronique.
Pour cela, il va d'abord me falloir abordé le sujet de l'écriture scientifique binaire, commençons.

# VI. L'écriture scientifique binaire

La démonstration mathématique du circuit est donc assez simple, après toutes ces explications.

Nous définissons deux nombres flottants $Half$ $Precision$ ($16$ bits) $\alpha$ et $\beta$, la comparaison à produire sera $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.
Les opérandes ne font alors plus que $15$ bits (omission du bit de signe).

Nous allons commencé par chercher si le champs d'exposant $E$ de $\beta$ est plus grand que celui de $\alpha$ $\left(E_{\beta} \gt E_{\alpha}\right)$.
Pour comprendre pourquoi nous traitons les champs d'exposants $E_{\alpha}$ et $E_{\beta}$ avant ceux des mantisses tronquées $T$ de nos opérandes, il faut se penché sur l'écriture scientifique binaire sur lequel repose l'encodage IEEE-754 des nombres flottants.

// peut être faire une réf à l'introduction
# VI. Le significande

Pour transformé le nombre flottant $F$ codé classiquement en binaire en son écriture scientifique, il faut convertir le nombre en un significande ou mantisse binaire.
Le significande est un nombre dont la valeur est compris dans l'intervalle suivante $\left[1;2\right[$, ce qui veut dire que la partie entière du significande doit être à $1$.
Qui plus est, le significande s'obtient en déplaçant la virgule du nombre $F$ de sa position initial jusque devant le $MSB1$ du nombre, ce qui a pour effet de modifié la valeur de $F$.
Une exception est faite pour $F = 0$, alors le significande peut être exceptionellement nulle.
Si le $MSB1$ est dans la partie entière de $F$, ça veut dire que $\left(F \ge 1\right)$ et que la virgule devra être déplacé vers la gauche (si elle doit l'être), au contraire si le $MSB1$ est dans la partie fractionnaire alors $\left(F \lt 1\right)$ et la virgule sera déplacé vers la droite.
Mais le déplacement de la virgule engendre en binaire les même choses qu'en décimale.
Une division par $N$ de $F$ dans le cas d'un déplacement de la virgule de $log_2\left(N\right)$ rangs vers la gauche, et une multiplication de $F$ par le même facteur $N$ pour un décalage de la virgule de $log_2\left(N\right)$ rangs vers la droite.
Le terme $N$ multiplie ou divise $F$ et est systèmatiquement une puissance de $2$, ce qui suit explique pourquoi.

Pour tout nombre flottant $F$, nous savons que la virgule se trouve entre le $LSB$ de la partie entière (le bit de poids $0$), et le $MSB$ de la partie fractionnaire (bit de poids $-1$).
Par conséquent, un déplacement de la virgule d'un rang vers la gauche force le bit de poids $1$ à devenir le bit de poids $0$, le bit de poids $0$ devient celui de poids $-1$, et celui de poids $-1$ devient le bit de poids $-2$, etc.
Pour le dire autrement, chaque bit de la partie entière comme de la partie fractionnaire de $F$ voit son poids être décrémenter de $1$. 
C'est pourquoi après le déplacement de la virgule d'un rang vers la gauche, le nombre $F$ vaut ce qui suit pour $c = -1$:

$$\left(\sum_{i=msb\left(F\right)}^{lsb\left(F\right)} \left(F_i \times 2^{\left(i+c\right)}\right)\right) = \left(F\times 2^c\right)$$

Le terme $c$ représente le nombre de rang de décalage à induire sur la virgule du nombre $F$.
Qui plus est, lorsque $\left(c \lt 0\right)$ c'est que la virgule doit être décalée vers la gauche, et inversement quand $\left(c \gt 0\right)$, c'est que le décalage doit se produire vers la droite.

Essayons de comprendre pourquoi est ce qu'un décalage d'un rang vers la gauche de la virgule de $F$, divise la valeur de $F$ par $N = 2$, comme nous le démontre l'équation ci-dessus.
Nous savons que les nombres flottants ont une partie entière et une autre fractionnaire.
Ces deux parties utilisent chaque bit comme un facteur d'une puissance de $2$, des puissances positives pour la partie entière et négatives pour la partie fractionnaire.
Il n'empêche que dans notre nombre flottant $F$, n'importe quel bit de poids $i$ est facteur d'une puissance de $2$, deux fois plus grande que le bit de poids $\left(i-1\right)$.
Par exemple $\left(2^0 = 1\right)$ et $\left(2^{-1} = 0.5\right)$ ou autrement dit, pour tout nombre flottant $F$ alors $\sum_{i=msb\left(F\right)}^{lsb\left(F\right)} \left(2^i \div 2 = 2^\left(i-1\right)\right)$.
Nous avons vus plus haut qu'avec un décalage d'un rang vers la gauche de la virgule de $F$, je cite "chaque bit de la partie entière comme de la partie fractionnaire de $F$ voit son poids être décrémenter de $1$".
Ce qui veut dire que tout bit de $F$ passe de facteur de $2^i$ à $2^\left(i-1\right)$, et chaque bit voit donc sa valeur être divisé par $2$.
Le calcul de la valeur de $F$ passe de la somme des bits de poids $i$ à $1$ qui multiplient $2^i$, à la somme des bits de poids $i$ à $1$ qui multiplient $2^\left(i-1\right)$.
Pour le dire autrement, nous faisons la somme de tous les bits de $F$ après que leur poids (valeur) ait été divisé par $2$ avec le décalage.
C'est pourquoi la valeur de $F$ est divisé par $2$ avec un décalage d'un rang vers la gauche de la virgule.

Nous avons compris pourquoi un décalage d'un rang vers la gauche de la virgule de $F$, engendre une division par $2$ de $F$.
Découvrons désormais les effets d'un décalage d'un rang vers la droite de la virgule de $F$, sur la valeur du nombre lui même.
Dans le cas d'un tel décalage, chaque bit de la partie entière comme fractionnaire de $F$ voit son poids être incrémenter de $1$.
Par conséquent, chaque bit voit sa valeur doublé car $\left(2^{\left(i+1\right)} = 2 \times 2^i\right)$.
Nous calculons la valeur de $F$ après décalage comme la somme des bits de poids $i$ à $1$ qui multiplient $2^{\left(i+1\right)}$.
Exactement comme le fait la partie gauche de l'équation ci-dessus pour un décalage $c = 1$.
Le côté droit de cette équation nous informe que cette opération revient à multiplié le nombre $F$ par $2$.

Nous sommes désormais capable de comprendre toutes les implications d'un décalage de la virgule du nombre $F$, sur la valeur du nombre lui même. 
Prenons un décalage de $c = x$ comme exemple, où $x \in \left[-\infty ;+\infty \right]$.
Si $\left(c \gt 0\right)$ alors nous n'avons qu'à décalé la virgule de $c$ fois $1$ rang vers la droite.
Vu que nous savons qu'un décalage de la virgule de $1$ rang vers la droite engendre une multiplication par $2$ du nombre $F$, alors après $c$ décalages de $1$ rang, nous aurons multiplié $F$ par $2^c$.
Au contraire si $\left(c \lt 0\right)$, alors nous devrons décalé la virgule de $\vert c \vert$ fois $1$ rang vers la gauche.
Ce décalage d'un rang vers la gauche revient à divisé par $2$ la valeur de $F$.
Par conséquent, après $\vert c \vert$ décalages de $1$ rang vers la gauche nous aurons divisé $\vert c \vert$ fois la valeur de $F$ par $2$.
Ou autrement dit, $F$ aura été divisé par $2^{\vert c \vert}$.

Voilà pourquoi n'importe quel décalage de la virgule d'un nombre flottant $F$, engendre une multiplication ou une division du nombre par une puissance de $2$.

Cependant, je parle de divisé par une puissance de $2$ le nombre flottant $F$ dans le cas d'un décalage de sa virgule vers la gauche, mais malgré que l'équation précédente fonctionne, elle ne fait pas usage de division.
Comprenons pourquoi.
Le côté droit de cette équation $\left(F\times 2^c\right)$ est approprié pour un décalage de la virgule vers la droite.
Car après un déplacement de la virgule de $c$ rangs vers la droite, le nombre flottant $F$ est mulitplié par une puissance de $2$ qui vaut $2^c$, où je rappelle que $\left(c \gt 0\right)$.

Nous avons jusqu'ici utilisé des valeurs négatives de $c$ lors des décalages de virgule vers la gauche.
Pourtant le plus intuitif aurait été de définir les même valeurs de $c$ peu importe la direction des décalages, alors faisons comme si c'était le cas depuis le début.
Dans ce qui suit je vais donc faire usage de $\left(c \gt 0\right)$ pour des décalages vers la gauche.
Les explications suivantes vont nous permettre de comprendre pourquoi, dans les faits, il vaut mieux que $c$ soit négatif pour les décalages vers la gauche.

Pour un décalage de la virgule de $c$ rangs vers la gauche, nous devrions divisé $c$ fois le nombre $F$ par $2$, ou autrement dit, divisé $F$ par $2^c$.
Initialement, nous pourrions penser avoir besoin d'une équation ressemblant à celle-ci $\left(\sum_{i=msb\left(F\right)}^{lsb\left(F\right)} \left(F_i \times 2^{\left(i-c\right)}\right)\right) = \left(F\div 2^c\right)$.
J'attire l'attention sur le fait que côté droit de l'équation peut être modifié pour $\left(F\times \left(1\div 2^c\right)\right)$.
Mais ce n'est pas fini car pour trouver l'inverse d'une puissance de $2$ tel que $2^c$, nous pouvons simplement appliqué l'opposé de l'exposant $c$ à la base $2$.
En gros, $\left(1\div 2^c\right) = \ 2^{-c}$.
Je me permet de rappellé que $\left(c \gt 0\right)$.
Mais nous pouvons alors remplacé $\left(F\times 2^{-c}\right)$ par $\left(F\times 2^c\right)$, à la condition qu'un décalage de $x$ rangs vers la gauche de la virgule de $F$ soit représenté dans l'équation par une valeur de $c = -x$, c'est à dire négative.

Avec $\left(F\times 2^c\right)$ nous retrouvons bien le membre droit de notre équation initial.
De plus, nous comprenons aussi que le calcul de l'exposant $\left(i+c\right)$ du membre gauche, produit l'équivalent de $\left(i-c\right)$ dans le cas d'un décalage de la virgule de $F$ vers la gauche, car $\left(c \lt 0\right)$.

Voici la raison pour laquelle il vaut mieux que $c$ soit négatif lors d'un décalage vers la gauche de la virgule, ceci nous permet de ne faire usage que d'une seule équation pour tout les sens de décalage. C'est cool.

# VII. Le multiplicande

Nous savons désormais comment obtenir un significande.
Le problème c'est que tout nombre flottant $F$ dont la valeur n'est pas comprise dans l'intervalle $\left[1;2\right[$ doit forcémment subir un décalage de sa virgule, et donc voir sa valeur être multiplié ou divisé par une puissance de $2$.
Pour contrecarrer ce problème, l'écriture scientifique binaire définit ce qui s'appelle un multiplicande. 
Le rôle du multiplicande est de nous permettre de retrouver la valeur initial de $F$ après qu'il ait été transformé en un significande $S$, voyons comment.

Pour commencer, le nombre flottant $F$ est transformé en un significande $S$.
Cette transformation passe peut être par un décalage de la virgule de $F$ vers la droite ou vers la gauche.
Admettons que ce soit le cas, alors la valeur de $F$ est divisé ou multiplé par une puissance de $2$.
Imaginons que pour la transformation de $F$ en un significande nous ayons définit $\left(c \lt 0\right)$, alors la virgule du nombre $F$ a été déplacé vers la gauche et donc le nombre divisé.
Pour retrouver $F$, il suffit alors de multiplié $S$ par la puissance de $2$ qui a divisé le nombre $F$.
Mais rappellons que la division de $F$ s'effectue comme ceci $\left(F\times 2^c\right)$, ce qui est équivalent à $\left(F\div \left(1\div 2^c\right)\right)$.
Il faut donc multiplié le significande $S$ de la manière suivante afin de retrouver le nombre $F$ d'origine $\left(S\times \left(1\div 2^c\right)\right)$, soit $\left(S\times 2^{-c}\right)$.
Mais dans le cas inverse, pour la transformation du nombre $F$ en significande $S$ avec $\left(c \gt 0\right)$.
La virgule de $F$ est déplacé vers la droite et le nombre a alors été multiplié par une puissance de $2$.
Ce coup ci, pour retrouver $F$, il faudra divisé la valeur de $S$ par la puissance de $2$ qui a multiplié $F$.
Nous savons que $S = \left(F\times 2^c\right)$, alors nous divisons $S$ par $2^c$ comme ceci $\left(S\div 2^c\right)$ et nous tombons une nouvelle fois sur $\left(S\times 2^{-c}\right)$.

Nous retombons dans tout les cas le même calcul $\left(S\times 2^{-c}\right)$ permettant de retrouver le nombre d'origine $F$, le multiplicande est le facteur $2^{-c}$. 
Mais nous pouvons exprimé les choses de manière différentes, ce qui nous servira pour plus tard.

Pour la transformation d'un nombre flottant $F$ en un significande $S$, si la virgule de $F$ a été déplacé de $c$ rangs vers la droite alors $\left(c \gt 0\right)$.
Afin de retrouver la valeur du nombre d'origine $F$ depuis le significande, le multiplicande doit permettre de déplacé de $c$ rangs vers la gauche la virgule du significande $S$.
Il suffit alors d'inversé le signe de $c$ pour changé la direction du décalage $\left(-c\right)$.
Le nombre de rang de décalage reste le même mais le sens de décalage passe de la droite $\left(c \gt 0\right)$, à la gauche avec $\left(c \lt 0\right)$.

Dans le cas contraire, la transformation de $F$ en $S$ est due à un décalage de la virgule de $F$ de $\vert c \vert$ rangs vers la gauche, où $\left(c \lt 0\right)$.
Pour obtenir la valeur du nombre d'origine $F$, il faudra alors que le significande $S$ voit sa virgule être décalée de $\vert c \vert$ rangs vers la droite.
Rappellons que $c$ est négatif, la négation de $c$ permettra donc une nouvelle fois d'inversé le sens de décalage $\left(-c\right)$.

Nous remarquons que les deux explications donne les même conclusions, il faut négationné $c$.

# VIII. Codage des informations dans un nombre flottant IEEE-754

Nous avons déjà parler de l'encodage IEEE-754 en introduction de ce document, plus particulièrement des éléments qui le compose.
Je rappelle que ces $3$ éléments sont des champs binaire qui codent chacun un `bit de signe`, un `exposant` ainsi qu'une ` mantisse tronquée`.
Par ailleurs, nous venons tout juste de voir dans le détails ce qu'est l'écriture scientifique biniaire d'un nombre, ce qui nous a permis de mettre en lumière ce qu'est un `significande` (qui peut être appellé `mantisse`), ainsi qu'un `multiplicande`.
Il y a aussi un élément de l'écriture scientifique dont je n'ai pas parlé, c'est le `signe` du nombre représenté, qui est soit $+$ soit $-$ bien entendu.

Chaque champs binaire de l'encodage IEEE-754 correspond plus ou moins à l'un des éléments de l'écriture scientifique binaire d'un nombre.
Par exemple, le champs du bit de signe correspond assez logiquement au signe d'un nombre, mais nous ne nous intéresserons pas plus que ça au bit de signe dans ce qui suit.
En revanche, nous allons parlé plus en profondeur du champs d'exposant ainsi que de la mantisse tronquée.
Nous avons déjà abordé le sujet de l'encodage de ces champs dans l'introduction, aussi je ne vais pas y revenir.

# IX. Composition du champs de mantisse tronquée et du champs d'exposant

Le champs de la mantisse tronquée correspond au significande d'un nombre représenté en écriture scientifique binaire.
Faisons un rapide rappelle sur ce qu'est le significande.
Le significande $S$ est un nombre dont la valeur se trouve être dans l'intervalle de $\left[1;2\right[$, il est souvent le résultat d'une transformation (un calcul) sur un nombre $F$ à représenté en écriture scientifique.
Si jamais $\left(F \ge 2\right) \vee \left(F \lt 1\right)$ alors le nombre $F$ sera divisé ou multiplié par une puissance de $2$ que nous nommons $N$, et qui permettra donc de généré un significande $S$ qui respecte l'intervalle stipulé ci-dessus.

En somme, le significande est un nombre à virgule qui est donc composé d'une partie entière ainsi que d'une partie fractionnaire.
Vu que la valeur du significande doit être strictement inférieur à $2$ ou $10_2$ en binaire, ça veut dire que l'encodage de la partie entière ne peut être que inférieur à $10_2$, c'est à dire $0$ ou $1$.
Par conséquent la partie entière du significande n'est codé que sur un bit.
Mais en soit le bit de la partie entière ne peut pas être à $0$, car il est nécessaire que la valeur du significande soit comprise dans l'intervalle mentionné ci-dessus.
Donc le bit de la partie entière est à $1$.
Pour résumer $\left(S \ge 1\right) \wedge \left(S \lt 2\right)$ nous force à dire que la partie entière du significande $S$ est à $1$.
En plus de cela, nous avons la partie fractionnaire du significande à prendre en compte, qui est logiquement strictement inférieur à $1$.
Finalement, la valeur du significande est la somme de la partie entière et de la partie fractionnaire, ce qui veut dire que $\left(S \lt 2\right)$.

La norme IEEE-754 définit un champs de mantisse tronquée de $X$ bits pour chaque format de nombre flottant. 
Prenons pour exemple le format $Half \ Precision$ que gère le circuit électronique en cours d'étude.
Il s'avère que le champs de la mantisse tronquée de ce format est établit à une taille de $10$ bits.
L'entièreté d'un significande $S$ pourrait être enregistré sur ces $10$ bits (si possible), mais ce n'est pas utiles car la partie entière est toujours composé d'un seul et même bit à $1$, comme nous venons de le voir.
C'est pourquoi la norme ampute le significande de sa partie entière, indépendemment du format de flottant.
L'intérêt c'est que le champs reste de la même taille ($10$ bits pour un $Half \ Precision$), mais que nous gagnons un bit de précision sur le codage d'un nombre.
C'est d'ailleurs la raison derrière le nom de mantisse tronquée.

Le standard IEEE-754 définit aussi un champs d'exposant, qui correspond en partie au multiplicande de l'écriture scientifique binaire d'un nombre $F$.
Je conseille une relecture du chapitre sur le multiplicande si un rafraichissement est nécessaire.
Je cite "le multiplicande est le facteur $2^{-c}$.", mais en réalité le champs d'exposant lui ne correspond qu'à l'exposant $-c$ du multiplicande, et non à la puissance de $2$ en sa totalité. 
Rappellons que la variable $c$ représente le nombre de rang par lequel décalé la virgule du nombre $F$ pour le transformé en un significande, et que lorsque $\left(c \lt 0\right)$ cela témoigne d'un décalage vers la gauche, inversement pour $\left(c \gt 0\right)$.
Le champs d'exposant se base sur ce qui a été dit dans les deux derniers paragraphes du chapitre sur le multiplicande, pour déplacé la virgule du significande du bon nombre de rang, et dans la bonne direction.

En gros, le champs d'exposant code le résultat de la négation de $c$, ce qui a pour effet d'inversé le sens de décalage de la virgule, sans touché pour autant au nombre de rang à décalé.
Si la valeur de l'exposant codé dans le champs est négative, alors le décalage de la virgule du significande se fera vers la gauche et du nombre de rang codé par le champs.
Au contraire, si cette valeur est positive, la virgule du significande sera décalée vers la droite et sera toujours déplacé du nombre de rang codé par le champs.
Ce champs d'exposant possède néanmoins une taille $X$ qui varie en fonction du format des nombres flottants, pour un format $Half \ Precision$ sa taille est de $5$ bits.

# X. Le circuit traite les champs d'exposant avant ceux des mantisses tronquées

Pour finir, je vais enfin pouvoir expliqué pourquoi le circuit électronique traite les champs d'exposant $E$ des opérandes $\alpha$ et $\beta$, avant les champs de mantisse tronquée $T$ de ces même opérandes.

Nous avons vu dans le chapitre sur le multiplicande, comment est ce qu'en écriture scientifique binaire nous pouvions obtenir le nombre d'origine $F$ depuis le significande $S$. Précisémment au travers du calcul suivant $F = \left(S\times 2^{-c}\right)$.
En tenant compte de tout ce qui a déjà été dit plus haut, nous en déduisons qu'en IEEE-754 le nombre $F$ vaut alors $\left(T\times 2^E\right)$.

N'oublions pas que si le significande de l'écriture scientifique du nombre $F$ est compris dans l'intervalle de valeur suivante $\left[1;2\right[$.
Alors c'est aussi le cas du champs de la mantisse tronquée $T$ (en prenant en compte le bit à $1$ de la partie entière qui est rendu implicite).

Au travers du calcul suivant $\left(T\times 2^E\right)$ nous observons quelques chose d'intéressant.
Par essence $\left(2^i = 2\times 2^{\left(i-1\right)}\right)$.
Etant donné que la valeur codé dans le champs de la mantisse tronquée $T$ est strictement inférieur à $2$, alors même pour $T = 1.99..9$ nous obtenons pour $\left(T\times 2^E\right)$ une valeur strictement inférieur à $\left(1\times 2^{\left(E+1\right)}\right)$.
Nous voyons ici pourquoi est ce que les champs d'exposant $E$ des opérandes sont traités avant les champs de mantisse tronquée $T$.

Imaginons que pour deux opérandes $\alpha$ et $\beta$ nous voulions vérifié si $\left(\vert\beta\vert \gt \vert\alpha\vert\right)$:
  - Si $\left(E_{\alpha} \gt E_{\beta}\right)$ alors nous savons que la comparaison échoue car $\left(T_{\alpha}\times 2^{E_{\alpha}}\right) \gt \left(T_{\beta}\times 2^{E_{\beta}}\right)$ et ce même si $T_{\alpha} = 1.0_{10}$ pendant que $T_{\beta} = 1.99..9$.

  - Si $\left(E_{\alpha} \lt E_{\beta}\right)$ alors la comparaison réussie cette fois-ci, car $\left(T_{\alpha}\times 2^{E_{\alpha}}\right) \lt \left(T_{\beta}\times 2^{E_{\beta}}\right)$ et ce même si $T_{\beta} = 1.0_{10}$ pendant que $T_{\alpha} = 1.99..9$.

Mais il existe une troisième possibilité bien évidemment, $\left(E_{\alpha} = E_{\beta}\right)$.
Dans ce cas spécifique, les produits $\left(T_{\alpha}\times 2^{E_{\alpha}}\right)$ ainsi que $\left(T_{\beta}\times 2^{E_{\beta}}\right)$ ont un facteur commun qui est la puissance de $2$.
Par conséquent tout ne dépend que des champs $T_{\alpha}$ et $T_{\beta}$ pour pouvoir départagé si $\vert\beta\vert$ est strictement supérieur à $\vert\alpha\vert$ ou non.

Nous comprenons désormais pourquoi le circuit électronique, qui a pour but de vérifié la supériorité stricte d'un opérande envers l'autre, traite d'abord les champs d'exposant $E$ des opérandes $\alpha$ et $\beta$, puis ensuite les champs de mantisse tronquée $T$ si nécessaire.

# XI. Démonstration mathématique

// pourquoi traité E avant T

Nous avons vu que ce champs binaire de $5$ bits code la valeur de l'exposant avec un encodage par biais, mais aussi que cet encodage partage toutes les caractéristiques de l'encodage $Binary$ $Unsigned$.

# XII. Traitement des champs d'exposant

- Voici la définition de l'opération logique $Nimply\left(X,Y\right)\mapsto X \wedge Not(Y)$ dont nous allons nous sevrir.

Nous effectuons l'opération $Nimply\left(E_{\beta\sigma},E_{\alpha\sigma}\right)$ où $\sigma$ est le poids des bits du champs d'exposant $E$ des opérandes $\beta$ et $\alpha$.

  $$\sum_{\sigma=14}^{10} Nimply\left(E_{\beta\sigma}, E_{\alpha\sigma}\right) = \tau$$

// plus d'explication sur les résultats terminaux

Le résultat est un champs binaire $\tau$ composé de $5$ bits, et si l'un (ou plusieurs) d'entre eux est à $1$, alors:

  - Nous affectons le poids du $MSB1$ du champs $\tau$ à la variable $\sigma$.

  - Nous savons que $\left(E_{\beta\sigma} \gt E_{\alpha\sigma}\right)$ et par conséquent $\left(E_{\beta\sigma} \times 2^{\sigma}\right) \gt \left(\sum_{\sigma}^{10} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

    - Dans le cas où $\sigma \neq 14$, ce qui figure ci-dessus ne prouve pas que $\left(E_{\beta} \gt E_{\alpha}\right)$ car $\left(\sum_{\sigma+1}^{14} E_{\beta\sigma} \times 2^{\sigma}\right) \le \left(\sum_{\sigma+1}^{14} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

      - Nous savons que la somme des bits de poids supérieur à $\sigma$ pour $E_{\beta}$ est $\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \times 2^{\sigma}\right) = r\right)$.

      - Par conséquent si $\left(\sum_{\sigma+1}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) = r \right)$ alors $\left(E_{\beta} \gt E_{\alpha}\right)$, sinon  $\left(\sum_{\sigma+1}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt r \right)$ et $\left(E_{\beta} \lt E_{\alpha}\right)$.

    - Autrement $\sigma = 14$ et nous sommes certains que $\left(E_{\beta} \gt E_{\alpha}\right)$ car $Nimply\left(E_{\beta 14},E_{\alpha 14}\right) = 1$.

Cependant, si le champs $\tau$ n'est composé que de $5$ bits à $0$, alors:

  - Nous savons que l'opération $Nimply\left(E_{\beta\sigma}, E_{\alpha\sigma}\right)$ ne renvoie $0$ que si $\left(E_{\beta\sigma} = E_{\alpha\sigma}\right)$, ou si $\left(E_{\beta\sigma} = 0\right)$ alors que $\left(E_{\alpha\sigma} = 1\right)$.

  - Par conséquent vu que $\tau$ n'est composé que de bits à $0$, nous savons que $\left(\sum_{\sigma=10}^{14} E_{\beta\sigma} \times 2^{\sigma}\right) \le \left(\sum_{\sigma=10}^{14} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

  - En reprenant en partie ce qui a été dit plus haut, la valeur du champs d'exposant $E_{\beta}$ est $\left(\sum_{\sigma=10}^{14} \left(E_{\beta\sigma} \times 2^{\sigma}\right) = r\right)$:

    - Dans le cas où $\left(\sum_{\sigma=10}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt r\right)$ alors $\left(E_{\beta} \lt E_{\alpha}\right)$.

    - Cependant si $\left(\sum_{\sigma=10}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) = r\right)$ alors $\left(E_{\beta} = E_{\alpha}\right)$.

// Explication sur le cas non terminal Ealpha = Ebeta

Pour résumer, si les champs $E_{\beta} \neq E_{\alpha}$ alors soit $\left(E_{\beta} \gt E_{\alpha}\right)$ et nous savons que $\left(\left|\alpha\right| \lt \left|\beta\right|\right)$, ou alors $\left(E_{\beta} \lt E_{\alpha}\right)$ et $\left(\left|\alpha\right| \gt \left|\beta\right|\right)$.

// Explication du traitement de la mantisse tronquée?

# XIII. Traitement des champs des mantisses tronquées 

Mais rappellons que quand bien même $\left(E_{\beta} = E_{\alpha}\right)$, ce n'est pas pour autant que la comparaison $\left(\left|\alpha\right| \gt \left|\beta\right|\right)$ échoue.
Les opérandes flottants $\alpha$ et $\beta$ ont aussi un champs de mantisse tronquée $T$ de $10_{10}$ bits.
Si les champs d'exposant $E$ des deux opérandes sont égaux, il faut encore que les champs binaires $T_{\beta}$ et $T_{\alpha}$ le soient aussi pour que $\left(\left|\alpha\right| = \left|\beta\right|\right)$ et que la comparaison échoue.


// Expliqué que les poids des bits du champs d'exposant sont plus grands que ceux de la mantisse tronquée, d'où le fait de traité en priorité ces derniers.

// Explication de l'expression
   
$$\left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) = 0 \right) = \left(E_{\beta} \gt E_{\alpha}\right)\right) \vee \left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) \gt 0 \right) = \left(E_{\beta} \lt E_{\alpha}\right)\right)$$
