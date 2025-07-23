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
Pour représenter la valeur $103$ en décimale nous décomposons en réalité chaque chiffre du nombre pour les multipliés avec une puissance de $10$ adéquat, puis il faut faire la somme des résultats de chaque produit.
Regardez ci-dessous.

$$103_{10} = 1 \times 10^2 + 0 \times 10^1 + 3 \times 10^0$$

Veuillez noté que l'indice $X_{10}$ représente la base numérique dans laquelle le nombre $X$ qui précède est écrit.
Nous retrouvons chacun des chiffres du nombre entrain de multiplié une puissance de $10$.
Remarquons que le chiffre des unités $3$ est facteur de $10$ à la puissance $0$, le chiffre d'un ordre plus grand (celui des dizaines) est le facteur de $10$ à la puissance $1$ et enfin le chiffre des centaines multiplie $10$ à la puissance $2$.
La valeur des puissances croix en fonction de la position du chiffre, qui plus est, elle commence à $0$.

Ceci n'est pas propre à la base décimale mais à `nimporte quel base numérique`, comme la base binaire par exemple.
La base binaire (ou base $2$) utilise les chiffres $0$ et $1$ plutôt que ceux de la base décimale allant de $0$ à $9$.
Par ailleurs, un tel chiffre s'appelle un bit, ce qui est la contraction de $Binary$ $Digit$.
Aussi, les puissances de $10$ sont remplacés par des puissances de $2$.
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
Ce champs ressemble à ceci $01111111_{2}$, et la valeur du nombre représenté est $127_{10}$.
La somme des bits de poids inférieur au bit de poids $6$ donne un résultat strictement inférieur à $1 \times 2^6 = 64_{10}$.
Cela est systèmatiquement vrai, même lorsque les $i$ bits de poids inférieur représentent la valeur maximale encodable, comme dans cet exemple.

$$\left(1 \times 2^6 = 64_{10}\right) \gt \left(63_{10} = \left(1 \times 2^5\right)+\left(1 \times 2^4\right)+\left(1 \times 2^3\right)+\left(1 \times 2^2\right)+\left(1 \times 2^1\right)+\left(1 \times 2^0\right)\right)$$

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
Prenons l'exemple du nombre $3.75_{10}$.

$$3.75_{10} = 11.11_2 = Integer \ Part\left(\left(1 \times 2^1\right) + \left(1 \times 2^0\right)\right) + Fractional \ Part\left(\left(1 \times 2^{-1}\right) + \left(1 \times 2^{-2}\right)\right)$$

Précisons que sous la forme binaire du nombre, le point n'est là que pour facilité sa lecture (`dans les faits, il n'est pas réelement présent dans le codage des nombres flottants`).
Ce qu'il y a d'important à remarqué pour la partie fractionnaire, c'est que la valeur d'un bit à $1$ de poids $i$, est toujours strictement supérieur à la somme des bits de poids inférieur à $i$.
Autrement dit, pour le nombre fractionnaire $N$ dont ont ne prête attention qu'au bit à $1$ de poids $i$ alors $\left(1 \times 2^i \gt \left(\sum_{i-1}^{lsb} N_{i} \times 2^{i}\right)\right)$.

Pour en revenir au sujet de la mantisse tronquée, elle est composée des bits de la partie entière et de la partie fractionnaire d'un nombre flottant.
Vu que la partie entière et fractionnaire d'un nombre flottant partagent les caractéristiques du $Binary \ Unsigned$, c'est aussi le cas de la mantisse tronquée elle même.
Nous verrons que le comparateur se sert de cela pour effectué les comparaisons.

-- -

La démonstration mathématique du circuit est donc assez simple, après toutes ces explications.

Nous définissons deux nombres flottants $Half$ $Precision$ ($16$ bits) $\alpha$ et $\beta$, la comparaison à produire sera $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.
Les opérandes ne font alors plus que $15$ bits (omission du bit de signe).

Nous allons commencé par chercher si le champs d'exposant $E$ de $\beta$ est plus grand que celui de $\alpha$ $\left(E_{\beta} \gt E_{\alpha}\right)$.
Pour comprendre pourquoi nous traitons les champs d'exposants $E_{\alpha}$ et $E_{\beta}$ avant ceux des mantisses tronquées $T$ de nos opérandes, il faut se penché sur l'écriture scientifique binaire sur lequel repose l'encodage IEEE-754 des nombres flottants.

Pour transformé le nombre flottant $F$ codé classiquement en binaire, il faut convertir le nombre en un significande ou mantisse binaire.
Le significande est un nombre dont la valeur est compris dans l'intervalle suivante $\left[1;2\right[$, ça veut dire que la partie entière du nombre doit être strictement inférieur à $2$, tout en étant supérieur ou égale à $1$.
Qui plus est, le significande s'obtient en déplaçant la virgule du nombre $F$ de sa position initial jusque devant le $MSB1$ du nombre, ce qui a pour effet de modifié la valeur de $F$.
Une exception est faite pour $F = 0$ et le significande peut exceptionellement être nulle.
Si le $MSB1$ est dans la partie entière de $F$, ça veut dire que $\left(F \ge 1\right)$ et que la virgule devra être déplacé vers la gauche, au contraire si le $MSB1$ est dans la partie fractionnaire alors $\left(F \lt 1\right)$ et la virgule sera déplacé vers la droite.
Mais le déplacement de la virgule engendre en binaire les même choses qu'en décimale.
Une division par $N$ de $F$ dans le cas d'un déplacement de la virgule de $log_2\left(N\right)$ rangs vers la gauche, et une multiplication de $F$ d'un même facteur pour un même nombre de rang de décalage, c'est à dire $log_2\left(N\right)$ décalage vers la droite.
Le coefficient $N$ est systèmatiquement une puissance de $2$, une explication à cela serait la suivante.

Pour tout nombre flottant $F$, nous savons que la virgule se trouve entre le $LSB$ de la partie entière (le bit de poids $0$), et le $MSB$ de la partie fractionnaire (bit de poids $-1$).
Par conséquent, un déplacement de la virgule d'un rang vers la gauche force le bit de poids $1$ à devenir le bit de poids $0$, le bit de poids $0$ devient celui de poids $-1$, et celui de poids $-1$ devient le bit de poids $-2$, etc.
Pour le dire autrement, chaque bit de la partie entière comme de la partie fractionnaire de $F$ voit son poids être décrémenter de $1$. 
C'est pourquoi après le déplacement de la virgule d'un rang vers la gauche, le nombre $F$ vaut $\left(\sum_{i=msb}^{lsb} \left(F_i \times 2^{\left(i+c\right)}\right) = \left(F\times 2^c\right)\right)$ où $c = -1$.
Par ailleurs, lorsque $\left(c \lt 0\right)$ c'est que la virgule doit être décalée vers la gauche, et inversement quand $\left(c \gt 0\right)$, c'est que le décalage est vers la droite.

Je tient à attirer l'attention sur le terme de droite de l'équation qui précède $\left(F \times 2^c\right)$.
Plus particulièrement sur $2^c$ car ce terme là est notre coefficient $N$ dont je parlais plus haut.
Dans ce terme nous avons l'exposant $c$ qui je le rappelle représente le nombre de rang de décalage à induire sur la virgule, mais aussi l'information de la direction du décalage.
Décalage vers la gauche pour $\left(c \lt 0\right)$ et décalage vers la droite avec $\left(c \gt 0\right)$.
Par conséquent, si $c$ est négatif le décalage se fera vers la gauche et nous diviserons alors le nombre $F$ d'un coefficient diviseur $N$ où $\left(N = 2^c\right)$.
Ceci explique pourquoi $N$ est toujours une puissance de $2$.
De plus, nous divisons $F$ par $N$ lorsque nous déplaçons la virgule du nombre $F$ de $log_2\left(N\right)$ rangs vers la gauche, ou autrement dit de $log_2\left(2^c\right) = c$ rangs vers la gauche.
D'où le lien mentionné plus haut entre l'unité de décalage de la virgule $c$, et le coefficient diviseur ou multiplicateur $N$.
Mais nous ne savons pas pourquoi $\left(N = 2^c\right)$, alors tâchons de le comprendre avec ce qui suit.

Dans les faits si nous prenons la base décimale comme référence, déplacé la virgule d'un nombre $F = 103.5_{10}$ de $\vert c \vert$ rangs vers la gauche revient à divisé $F$ par $10^{\vert c \vert}$.
Prenons $c = -2$ comme exemple, alors $\left(F \div 10^{\vert c \vert}\right) = 1.035_{10}$ et nous voyons bien que la virgule a été décalé de deux rangs vers la gauche, comme attendu.
En base binaire il se passe la même chose, mais les puissances de $10_{10}$ sont remplacés par des puissances de $2$.
Le nombre $103.5$ vaut $1100111.1_2$ en binaire et si $c$ est toujours égale à $-2$ alors $\left(F \div 2^{\vert c \vert}\right) = 11001.111_2 = 25.875$.
Comme prévu nous avons déplacés la virgule de deux rangs vers la gauche en divisant notre nombre $F$ par $2^{\vert c \vert}$, ou autrement dit en multipliant $F$ par $2^c$ tel que le terme droit $\left(F \times 2^c\right)$ de l'équation précédente.

Maintenant, quel est la raison fondamentale derrière le fait qu'un décalage de la virgule de $F$ de $log_2\left(N\right)$ rangs vers la gauche, engendre une division de $F$ par $N$.
Nous savons que les nombres flottants ont une partie entière et une autre fractionnaire.
Ces deux parties utilisent chaque bit comme un facteur d'une puissance de $2$, des puissances positives pour la partie entière et négatives pour la partie fractionnaire.
Il n'empêche que dans notre nombre flottant $F$, n'importe quel bit de poids $i$ est facteur d'une puissance de $2$, deux fois plus grande que le bit de poids $\left(i-1\right)$.
Par exemple $\left(2^0 = 1\right)$ et $\left(2^{\left(-1\right)} = 0.5\right)$ ou autrement dit, pour tout nombre flottant $F$ alors $\sum_{i=msb}^{lsb} \left(2^i = 2 \times 2^\left(i-1\right)\right)$.
Nous avons vus plus haut qu'avec un décalage d'un rang vers la gauche de la virgule de $F$, je cite "chaque bit de la partie entière comme de la partie fractionnaire de $F$ voit son poids être décrémenter de $1$".
Ce qui veut dire que tout bit de $F$ passe de facteur de $2^i$ à $2^\left(i-1\right)$, et chaque bit voit donc sa valeur être divisé par $2$.
La valeur de $F$ qui est je le rappelle, la somme des bits de poids $i$ à $1$ qui multiplient $2^i$, est donc divisé par $2$ suite au décalage de la virgule.
Exactement comme le démontre notre équation $\left(\sum_{i=msb}^{lsb} \left(F_i \times 2^{\left(i+c\right)}\right) = \left(F\times 2^c\right)\right)$ avec $c = -1$.

Cependant, si après cette première opération de décalage d'un rang vers la gauche nous ré-itérons l'opération.
Alors la valeur initial de $F$ aura été divisé par $2$, deux fois, ou autrement dit divisé par $4$.
Si nous ré-itérons une troisième fois cette opération, nous diviserons par $2$ la valeur de $F$ déjà divisé par $4$, ce qui donne $\left(F \div 8\right)$.
En bref vous aurez compris que pour un décalage vers la droite c'est exactement l'inverse qui se produit, l'ensemble des bits de $F$ voit leur poids être incrémenté du nombre de rang de décalage $c$ et $F$ est multiplié par $N$.
Nous comprenons alors que $N = 2^c$ car un décalage de la virgule de $\vert c \vert$ rangs vers la gauche engendre une multiplication de $F$ par $N$, équivalent à $F \ \div \left(1 \div N\right)$.
En parallèle, un décalage de $c$ rangs vers la droite engendre aussi une multiplication de $F$ par $N$.
Voilà pourquoi un décalage de la virgule de $log_2 \left(N\right)$ qui est égale à $\ log_2 \left(2^c\right) = \ c$ engendre une division ou une multiplication de $F$ par $N$.

// faire passé c -> C
// supprimer les indices de base numérique lorsqu'ils ne sont pas nécessaire

// raison fondamentale derrière le fait qu'un décalage de la virgule de log2(N) vers la gauche induise une divison de F par N


/*
Par exemple, si $F = 5.25_{10} = 101.01_2$ alors nous procédons à $c = -2$ décalages vers la gauche, le résultat selon l'équation ci-dessus est $1.3125_{10} \quad = \quad 5.25_{10} \times 2^c\quad = \quad 5.25_{10} \div 2^{\left(-c\right)}\quad = \quad 1.0101_2$.
Dans cet exemple nous avons eu besoin de déplacé la virgule de $F$ de deux rangs vers la gauche, nous divisons donc $F$ par $2^2$ en le multipliant par $2^{-2}$ comme le montre  $5.25_{10} \times 2^c$ qui est égale à $5.25_{10} \div 2^{\left(-c\right)}$.
Remarquons que notre coefficient diviseur $N$ est donc $2^{-c} = 2^{-\left(-2\right)} = 4_{10}$ et que le nombre de rang de décalage vers la gauche est de $log_2\left(N\right) = 2_{10}$.
Finalement, le déplacement de la virgule a globalement le même effet en binaire qu'en décimale.
Un décalage de la virgule de $F$ engendre une division ou une multiplication de $F$ par une puissance de $2$ (et non de $10$), en fonction de la direction du décalage.
*/


Si le nombre $F$ change pour devenir un significande licite, il est nécessaire de compensé exactement ces transformations, nous faisons ceci à l'aide du multiplicande.
Le multiplicande élève $2$ à la puissance $N$.

  



Voici un exemple avec le nombre $3.75_{10} = 11.11_{2}$ que nous avons utilisés plus haut.
De base, la virgule ne se situe pas devant le $MSB1$ du nombre $11.11_{2}$, cela veut dire qu'il faut induire un décalage vers la gauche, un décalage de $1$ qui plus est.

// pourquoi traité E avec T

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
Les opérandes flottants $\alpha$ et $\beta$ ont aussi un champs de mantisse tronquée $T$ de $10$ bits.
Si les champs d'exposant $E$ des deux opérandes sont égaux, il faut encore que les champs binaires $T_{\beta}$ et $T_{\alpha}$ le soient aussi pour que $\left(\left|\alpha\right| = \left|\beta\right|\right)$ et que la comparaison échoue.


// Expliqué que les poids des bits du champs d'exposant sont plus grands que ceux de la mantisse tronquée, d'où le fait de traité en priorité ces derniers.

// Explication de l'expression
   
$$\left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) = 0 \right) = \left(E_{\beta} \gt E_{\alpha}\right)\right) \vee \left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) \gt 0 \right) = \left(E_{\beta} \lt E_{\alpha}\right)\right)$$
