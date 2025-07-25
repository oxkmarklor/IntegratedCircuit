poub:  \gg \ll \in 

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
Pour représenter la valeur $103$ en décimale nous décomposons en réalité chaque chiffre du nombre pour les multipliés avec une puissance de $10_{10}$ adéquat, puis il faut faire la somme des résultats de chaque produit.
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
Nous venons tout juste de décrire ce qu'est le $Binary \ Unsigned$.

Un champs binaire pour lequel nous utilisons un encodage $Binary \ Unsigned$ associe une puissance de $2$ positive à chaque bit du champs.
Les puissances vont de $0$ jusqu'à $N-1$, où $N$ représente le nombre de bits total dans le champs.
Le bit du champs qui est facteur de $2$ à la puissance $0$ est appellé le $LSB$ pour $Least \ Significant \ Bit$, par ailleurs, `plus une puissance de 2 est grande` et `plus son poids est grand`.
La notion de poids est relative à la valeur d'une puissance de $2$, naturellement le $LSB$ est le bit de poids le plus faible d'un champs binaire.
Nous aurons compris qu'avec des puissances de $2$ positives, les nombres encodé en $Binary \ Unsigned$ sont positifs ou nulle.

Au contraire, le bit de poids le plus fort d'un champs binaire est ce que l'ont appelle le $MSB$ pour $Most \ Significant \ Bit$.
Il est aussi possible de faire référence au bit à $1$ de poids le plus faible d'un champs avec le terme $LSB1$, ou à celui de poids le plus fort avec $MSB1$.

Il se trouve que le $Binary \ Unsigned$ permet d'être certain que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des bits de poids inférieur à $i$.
Prenons l'exemple d'un champs binaire de $8$ bits pour lequel nous allons faire attention qu'à un seul bit, celui de poids $6$ admettons (le $MSB1$).
Disons que ce champs ressemble à ceci $01111111_{2}$, la valeur du nombre représenté est $127$.
La somme des bits de poids inférieur au bit de poids $6$ donne un résultat strictement inférieur à $1 \times 2^6 = 64$.
Cela est systèmatiquement vrai, même lorsque les $i$ bits de poids inférieur représentent la valeur maximale encodable, comme dans cet exemple.

$$\left(1 \times 2^6 = 64\right) \gt \left(63 = \left(1 \times 2^5\right)+\left(1 \times 2^4\right)+\left(1 \times 2^3\right)+\left(1 \times 2^2\right)+\left(1 \times 2^1\right)+\left(1 \times 2^0\right)\right)$$

Encore une fois, ceci n'est pas propre à la base binaire mais à nimporte quel base numérique.

Le circuit a pour but principal de produire des comparaisons entre deux opérandes flottants, nous allons donc rapidement revenir sur ce que sont les nombres à virgule flottante 
du standard IEEE-754.
Globalement, la norme définit trois éléments qui composent chaque nombre à virgule flottante:
- Le bit de signe
- Un champs binaire d'exposant
- Un autre champs binaire pour la mantisse tronquée

Ce sont les encodages utilisés dans les champs binaires d'exposant et de la mantisse tronquée que nous allons essayés de comprendre, le comparateur base toute sa logique de comparaison sur les propriétés de ces champs là.

Le champs d'exposant utilise un encodage par biais, ce dernier est assez simple à comprendre.
Enfaite, un champs binaire pour lequel nous utilisons un encodage $Binary \ Unsigned$ code une valeur numérique $X$, auquel nous ajoutons un biais.
Le biais $B$ est une constante positive ou négative, et la valeur représenté par le champs d'exposant d'un nombre flottant est issu du calcul $X+B$.
A savoir que le biais du champs d'exposant d'un nombre flottant IEEE-754 est toujours négatif.
Il est qui plus est toujours équivalent à $-\left(2^{N-1}\right)+1$ où $N$ est le nombre de bits du champs d'exposant.

Etant donné que l'encodage par biais se base sur le $Binary \ Unsigned$, le champs d'exposant partage les mêmes caractéristiques que cet encodage.
Notamment celui qui nous dit que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des puissances inférieurs à $i$.
Ce qui est normal vu que nous ne faisons qu'ajouté un biais constant à chaque nombre encodé en $Binary \ Unsigned$.

Pour le champs de la mantisse tronquée nous avons besoin de faire un rapide rappel sur l'encodage intial d'un nombre flottant (`le standard IEEE-754 est un enrobage plus qu'autre chose`).
La partie entière d'un nombre flottant utilise du $Binary \ Unsigned$ pour être codé, tandis que la partie fractionnaire fait usage d'une version modifiée du $Binary \ Unsigned$.
Chaque bit de la partie fractionnaire est un `facteur d'une puissance de 2 négative` et non positive, ceci permet de représenté des valeurs dans l'intervalle $\left]1;0\right]$.
Le poids des puissances négatives de $2$ décroix en fonction de la position du bit.
Prenons l'exemple du nombre $3.75$.

$$3.75 = 11.11_2 = Integer \ Part\left(\left(1 \times 2^1\right) + \left(1 \times 2^0\right)\right) + Fractional \ Part\left(\left(1 \times 2^{-1}\right) + \left(1 \times 2^{-2}\right)\right)$$

Précisons que sous la forme binaire du nombre, le point n'est là que pour facilité sa lecture (`dans les faits, il n'est pas réelement présent dans le codage des nombres flottants`).
Ce qu'il y a d'important à remarqué pour la partie fractionnaire, c'est que la valeur d'un bit à $1$ de poids $i$, est toujours strictement supérieur à la somme des bits de poids inférieur à $i$.
Autrement dit, pour le nombre fractionnaire $N$ dont ont ne prête attention qu'au bit à $1$ de poids $i$ alors $\left(1 \times 2^i \gt \left(\sum_{i-1}^{lsb} N_i \times 2^i\right)\right)$.

Pour en revenir au sujet de la mantisse tronquée, elle est composée des bits de la partie entière et de la partie fractionnaire d'un nombre flottant.
Vu que la partie entière et fractionnaire d'un nombre flottant partagent les caractéristiques du $Binary \ Unsigned$, c'est aussi le cas de la mantisse tronquée elle même.
Nous verrons que le comparateur se sert de cela pour effectué les comparaisons.

-- -

La démonstration mathématique du circuit est donc assez simple, après toutes ces explications.

Nous définissons deux nombres flottants $Half$ $Precision$ ($16$ bits) $\alpha$ et $\beta$, la comparaison à produire sera $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.
Les opérandes ne font alors plus que $15$ bits (omission du bit de signe).

Nous allons commencé par chercher si le champs d'exposant $E$ de $\beta$ est plus grand que celui de $\alpha$ $\left(E_{\beta} \gt E_{\alpha}\right)$.
Pour comprendre pourquoi nous traitons les champs d'exposants $E_{\alpha}$ et $E_{\beta}$ avant ceux des mantisses tronquées $T$ de nos opérandes, il faut se penché sur l'écriture scientifique binaire sur lequel repose l'encodage IEEE-754 des nombres flottants.

Pour transformé au format IEEE-754 le nombre flottant $F$ codé classiquement en binaire, il faut convertir le nombre en un significande ou mantisse binaire.
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
Pour le dire autrement, nous faisons la somme de tous les bits de $F$ après que leur valeur ait été divisé par $2$ avec le décalage.
C'est pourquoi $F$ est divisé par $2$ avec un décalage d'un rang vers la gauche de la virgule.

Nous avons compris pourquoi un décalage d'un rang vers la gauche de la virgule de $F$, engendre une division par $2$ de $F$.
Découvrons désormais les effets d'un décalage d'un rang vers la droite de la virgule de $F$, sur la valeur du nombre lui même.
Dans le cas d'un tel décalage, chaque bit de la partie entière comme fractionnaire de $F$ voit son poids être incrémenter de $1$.
Par conséquent, chaque bit voit sa valeur doublé car $\left(2^{\left(i+1\right)} = 2 \times 2^i\right)$.
Nous calculons la valeur de $F$ après décalage comme la somme des bits de poids $i$ à $1$ qui multiplient $2^{\left(i+1\right)}$.
Exactement comme le fait la partie gauche de l'équation ci-dessus pour un décalage $c = 1$.
Le côté droit de cette équation nous informe que cette opération revient à multiplié le nombre $F$ par $2$.

Nous sommes désormais capable de comprendre toutes les implications d'un décalage de la virgule du nombre $F$, sur la valeur du nombre lui même. 
Prenons un décalage de $c = x$ comme exemple où $x \in \left[-\infty ;+\infty \right]$.
Si $\left(c \gt 0\right)$ alors nous n'avons qu'à décalé la virgule de $c$ fois $1$ rang vers la droite.
Vu que nous savons qu'un décalage de la virgule de $1$ rang vers la droite engendre une multiplication par $2$ du nombre $F$, alors après $c$ décalages de $1$ rang, nous aurons multiplié $F$ par $2^c$.
Au contraire si $\left(c \lt 0\right)$, alors nous devrons décalé la virgule de $\vert c \vert$ fois $1$ rang vers la gauche.
Ce décalage d'un rang vers la gauche revient à divisé par $2$ la valeur de $F$.
Par conséquent, après $\vert c \vert$ décalages de $1$ rang vers la gauche nous aurons divisé $\vert c \vert$ fois la valeur de $F$ par $2$.
Ou autrement dit, $F$ aura été divisé par $2^{\vert c \vert}$.

Voilà pourquoi n'importe quel décalage de la virgule d'un nombre flottant $F$, engendre une multiplication ou une division du nombre par une puissance de $2$.

Cependant, je parle de divisé par une puissance de $2$ un nombre flottant $F$ dans le cas d'un décalage de sa virgule vers la gauche, mais malgré que l'équation précédente fonctionne, elle ne fait pas usage de division.
Comprenons pourquoi.
Le côté droit de cette équation $\left(F\times 2^c\right)$ est approprié pour un décalage de la virgule vers la droite.
Car après un déplacement de la virgule de $c$ rangs vers la droite, le nombre flottant $F$ est mulitplié par $2^c$.
Je rappelle ici que pour un décalage vers la droite $\left(c \gt 0\right)$.

Nous avons jusqu'ici utilisé des valeurs négatives de $c$ lors des décalages de virgule vers la gauche.
Pourtant le plus intuitif aurait été de définir les même valeurs de $c$ peu importe la direction des décalages, alors faisons comme si c'était le cas depuis le début.
Dans ce qui suit je vais donc faire usage de $\left(c \gt 0\right)$ pour des décalages vers la gauche.
Les explications suivantes vont nous permettre de comprendre pourquoi, dans les faits, il vaut mieux que $c$ soit négatif pour les décalages vers la gauche.

Pour un décalage de la virgule de $c$ rangs vers la gauche, nous devrions divisé $c$ fois le nombre $F$ par $2$, ou autrement dit, divisé $F$ par $2^c$.
Initialement, nous pourrions penser avoir besoin d'une équation ressemblant à celle-ci $\left(\sum_{i=msb\left(F\right)}^{lsb\left(F\right)} \left(F_i \times 2^{\left(i-c\right)}\right)\right) = \left(F\div 2^c\right)$.
J'attire l'attention sur le fait que côté droit de l'équation peut être modifié pour $\left(F\times \left(1\div 2^c\right)\right)$.
Mais ce n'est pas fini car pour trouver l'inverse d'une puissance de $2$ tel que $2^c$, nous pouvons simplement appliqué l'opposé de l'exposant $c$ à la base $2$.
En gros, $\left(1\div 2^c\right) = \ 2^{-c}$.
Je me permet de rappellé que $c$ est positif.
Mais nous pouvons alors remplacé $\left(F\times 2^{-c}\right)$ par $\left(F\times 2^c\right)$, à la condition que $c$ puisse être négatif lors d'un décalage vers la gauche de la virgule de $F$.

Voici la raison pour laquelle il vaut mieux que $c$ soit négatif lors d'un décalage vers la gauche d'une virgule, ceci nous permet de ne faire usage que d'une seule équation pour tout type de décalage, c'est cool.

-- -

Nous savons désormais comment obtenir un significande.
Le problème c'est que tout nombre flottant $F$ dont la valeur n'est pas comprise dans l'intervalle $\left[1;2\right[$ doit forcémment subir un décalage de sa virgule, et donc voir sa valeur être multiplié ou divisé par une puissance de $2$.
Pour contrecarrer ce problème, l'écriture scientifique binaire définit ce qui s'appelle un multiplicande. 
Le rôle du multiplicande est de nous permettre de retrouver la valeur initial de $F$ après qu'il ait été transformé en un significande $S$, voyons comment.

Pour commencer, le nombre flottant $F$ est transformé en un significande $S$.
Cette transformation passe peut être par un décalage de la virgule de $F$ vers la droite ou vers la gauche.
Admettons que ce soit le cas, alors la valeur de $F$ est divisé ou multiplé par une puissance de $2$.
Imaginons que pour la transformation de $F$ en un significiande nous ayons définit $\left(c \lt 0\right)$, alors la virgule du nombre $F$ a été déplacé vers la gauche et donc le nombre divisé.
Pour retrouver $F$, il suffit alors de multiplié $S$ par la puissance de $2$ qui a divisé le nombre $F$.
Mais rappellons que la division de $F$ s'effectue comme ceci $\left(F\times 2^c\right)$ ce qui est équivalent à $\left(F\div \left(1\div 2^c\right)\right)$.
Il faut donc multiplié le significande $S$ de la manière suivante afin de retrouver le nombre $F$ d'origine $\left(S\times \left(1\div 2^c\right)\right)$, soit $\left(S\times 2^{-c}\right)$.
Mais dans le cas inverse où la transformation du nombre $F$ en significande $S$ engendre $\left(c \gt 0\right)$.
La virgule de $F$ est déplacé vers la droite et le nombre a alors été multiplié par une puissance de $2$.
Ce coup ci, pour retrouver $F$, il faudra divisé la valeur de $S$ par la puissance de $2$ qui a multiplié $F$.
Nous savons que $S = \left(F\times 2^c\right)$, alors nous divisons $S$ par $2^c$ comme ceci $\left(S\div 2^c\right)$ et nous tombons une nouvelle fois sur $\left(S\times 2^{-c}\right)$.

Le multiplicande est le facteur $2^{-c}$ qui multiplie le significande $S$. 
Mais nous pouvons exprimé les choses de manière différentes, ce qui nous servira pour plus tard.

Pour la transformation d'un nombre flottant $F$ en un significande $S$, si la virgule de $F$ a été déplacé de $c$ rangs vers la droite alors $\left(c \gt 0\right)$.
Afin de retrouver la valeur du nombre d'origine $F$ depuis le significande, le multiplicande doit permettre de déplacé de $c$ rangs vers la gauche la virgule du significande $S$.
Il suffit alors d'inversé le signe de $c$ pour changé la direction du décalage $\left(-c\right)$.
Le nombre de rang de décalage reste le même mais le sens de décalage passe de la droite $\left(c \gt 0\right)$, à la gauche avec $\left(c \lt 0\right)$.
Le multiplicande sera donc de $2^{-c}$ pour pouvoir procédé à la multiplication du significande $S$ avec $\left(S\times 2^{-c}\right)$, ce qui engendre bel et bien un décalage de la virgule de $\left(\vert \ log_2\left(2^{-c}\right) \vert = \vert -c \ \vert\right)$ rangs vers la gauche.

Dans le cas contraire, la transformation de $F$ en $S$ est due à un décalage vers la gauche de la virgule de $F$ de $\vert c \vert$ rangs, alors $\left(c \lt 0\right)$.
Pour obtenir la valeur du nombre d'origine $F$, il faudra alors que le significande $S$ voit sa virgule être décalée de $\vert c \vert$ rangs vers la droite.
Rappellons que $c$ est négatif, la négation de $c$ permettra donc d'inversé le sens de décalage $\left(-c\right)$.
Nous retombons donc sur un multiplicande qui vaut $2^{-c}$ afin de multiplié $S$. Ce qui équivaut à $\left(log_2\left(2^{-c}\right) = -c\right)$ décalages de la virgule vers la droite.

-- -

// faire le pont entre l'écriture scientifique binaire et l'encodage IEEE-754 des nombres flottants.

// expliquer pourquoi le champs d'exposant E d'un flottant IEEE-754 doit être prit en compte par le circuit avant le champs T lui même.

-- -

// pourquoi traité E avant T

Nous avons vu que ce champs binaire de $5$ bits code la valeur de l'exposant avec un encodage par biais, mais aussi que cet encodage partage toutes les caractéristiques de l'encodage $Binary$ $Unsigned$.

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

    - Autrement $\sigma = 14$ et $\sigma$ est le $MSB1$ ainsi que le $MSB$ du champs $\tau$.
      Comme $\left(E_{\beta\sigma} \gt E_{\alpha\sigma}\right)$ nous sommes certains que $\left(E_{\beta} \gt E_{\alpha}\right)$.

Cependant, si le champs $\tau$ n'est composé que de $5$ bits à $0$, alors:

  - Nous savons que l'opération $Nimply\left(E_{\beta\sigma}, E_{\alpha\sigma}\right)$ ne renvoie $0$ que si $\left(E_{\beta\sigma} = E_{\alpha\sigma}\right)$, ou si $\left(E_{\beta\sigma} = 0\right)$ alors que $\left(E_{\alpha\sigma} = 1\right)$.

  - Par conséquent vu que $\tau$ n'est composé que de bits à $0$, nous savons que $\left(\sum_{\sigma=10}^{14} E_{\beta\sigma} \times 2^{\sigma}\right) \le \left(\sum_{\sigma=10}^{14} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

  - En reprenant en partie ce qui a été dit plus haut, la valeur du champs d'exposant $E_{\beta}$ est $\left(\sum_{\sigma=10}^{14} \left(E_{\beta\sigma} \times 2^{\sigma}\right) = r\right)$:

    - Dans le cas où $\left(\sum_{\sigma=10}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt r\right)$ alors $\left(E_{\beta} \lt E_{\alpha}\right)$.

    - Cependant si $\left(\sum_{\sigma=10}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) = r\right)$ alors $\left(E_{\beta} = E_{\alpha}\right)$.

// Explication sur le cas non terminal Ealpha = Ebeta

Pour résumer, si les champs $E_{\beta} \neq E_{\alpha}$ alors soit $\left(E_{\beta} \gt E_{\alpha}\right)$ et nous savons que $\left(\left|\alpha\right| \lt \left|\beta\right|\right)$, ou alors $\left(E_{\beta} \lt E_{\alpha}\right)$ et $\left(\left|\alpha\right| \gt \left|\beta\right|\right)$.

// Explication du traitement de la mantisse tronquée?

Mais rappellons que quand bien même $\left(E_{\beta} = E_{\alpha}\right)$, ce n'est pas pour autant que la comparaison $\left(\left|\alpha\right| \gt \left|\beta\right|\right)$ échoue.
Les opérandes flottants $\alpha$ et $\beta$ ont aussi un champs de mantisse tronquée $T$ de $10_{10}$ bits.
Si les champs d'exposant $E$ des deux opérandes sont égaux, il faut encore que les champs binaires $T_{\beta}$ et $T_{\alpha}$ le soient aussi pour que $\left(\left|\alpha\right| = \left|\beta\right|\right)$ et que la comparaison échoue.


// Expliqué que les poids des bits du champs d'exposant sont plus grands que ceux de la mantisse tronquée, d'où le fait de traité en priorité ces derniers.

// Explication de l'expression
   
$$\left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) = 0 \right) = \left(E_{\beta} \gt E_{\alpha}\right)\right) \vee \left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) \gt 0 \right) = \left(E_{\beta} \lt E_{\alpha}\right)\right)$$
