poub:  \gg \ll \in 

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$.
Un champs binaire est quant à lui composé d'un ensemble de bit, c'est à dire d'un ensemble de $0$ et de $1$.

Ce sont ces champs binaire qui nous permettent de représenté des nombres dans un ordinateur, faire des calculs, mémorisé des résultats.
Il y a cependant plein de façon de représenté des nombres dans un champs binaire, nous parlons d'encodage des nombres.
Tout les encodages ne se valent pas car il y en a des plus efficasse que d'autre en fonction des besoins, prenons un exemple.
Pour un champs de $16$ bits nous pourrions calculé la somme des bits pour représenter un nombre.
Avec cet encodage nous ne pourrions codé que des valeurs entre $0$ et $16$ inclus, ce qui n'est pas optimal pour bien des situations.
Mais parmis les multiples encodage existant, le plus connu de tous porte le nom de $Binary$ $Unsigned$.

Le $Binary$ $Unsigned$ utilise les mêmes primitives que la base décimale (celle que nous utilisons tous) pour représenter des nombres, `des puissances`.
Pour représenter la valeur $103_{10}$ en décimale nous décomposons en réalité chaque chiffre du nombre pour le multiplié avec une puissance de $10$ adéquat, regardez ci-dessous.

$$103_{10} = 1 \times 10^2 + 0 \times 10^1 + 3 \times 10^0$$

Veuillez noté que l'indice $X_{10}$ représente la base numérique dans laquelle le nombre $X$ qui précède est écrit.
Nous retrouvons chacun des chiffres du nombre entrain de multiplié une puissance de $10$.
Remarquons que le chiffre des unités $3$ est facteur de $10$ à la puissance $0$, le chiffre d'un ordre plus grand (celui des dizaines) est le facteur de $10$ à la puissance $1$ et enfin le chiffre des centaines multiplie $10$ à la puissance $2$.
La valeur des puissances croix en fonction de la position du chiffre, qui plus est, elle commence à $0$.

Ceci n'est pas propre à la base décimale mais à `nimporte quel base numérique`, comme la base binaire par exemple.
La base binaire (ou base $2$) utilise des chiffres $0$ et $1$ plutôt que ceux de la base décimale allant de $0$ à $9$.
Par ailleurs, un tel chiffre s'appelle un bit, ce qui est la contraction de $Binary$ $Digit$.
Aussi, les puissances de $10$ sont remplacés par des puissances de $2$.
Nous venons tout juste de décrire ce qu'est le $Binary$ $Unsigned$.

Un champs binaire pour lequel nous utilisons un encodage $Binary$ $Unsigned$ associe un puissance de $2$ positive à chaque bit du champs.
Les puissances vont de $0$ jusqu'à $N-1$, $N$ qui représente le nombre de bits total dans le champs.
Le bit du champs qui est facteur de $2$ à la puissance $0$ est appellé le $LSB$ pour $Least$ $Significant$ $Bit$, par ailleurs, `plus une puissance de 2 est grande` et `plus son poids est grand`.
La notion de poids est relative à la valeur d'une puissance de $2$, naturellement le $LSB$ est le bit de poids le plus faible d'un champs binaire.

Au contraire, le bit de poids le plus fort d'un champs binaire est ce que l'ont appelle le $MSB$ pour $Most$ $Significant$ $Bit$.
Il est aussi possible de faire référence au bit à $1$ de poids le plus faible d'un champs avec le terme $LSB1$, ou à celui de poids le plus fort avec $MSB1$.

Il se trouve que le $Binary$ $Unsigned$ permet d'être certain que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des bits de poids inférieur à $i$.
Prenons l'exemple d'un champs binaire de $8$ bits pour lequel nous allons faire attention qu'à un seul bit, celui de poids $6$ admettons (le $MSB1$).
Ce champs ressemble à ceci $01111111_{2}$, et la valeur du nombre représenté est $127_{10}$.
La somme des bits de poids inférieur au bit de poids $6$ donne un résultat strictement inférieur à $1 \times 2^6 = 64_{10}$.
Cela est systèmatiquement vrai, même lorsque les $i$ bits de poids inférieur représentent la valeur maximale encodable, comme dans cet exemple.

$$(1 \times 2^6 = 64_{10}) \gt (63_{10} = (1 \times 2^5)+(1 \times 2^4)+(1 \times 2^3)+(1 \times 2^2)+(1 \times 2^1)+(1 \times 2^0))$$

Encore une fois, ceci n'est pas propre à la base binaire mais à nimporte quel base numérique.

Le circuit a pour but principal de produire des comparaisons entre deux opérandes flottants, nous allons donc rapidement revenir sur ce que sont les nombres à virgule flottante 
du standard IEEE-754.
Globalement, la norme définit trois éléments qui composent chaque nombre à virgule flottante (qu'importe sa taille):
- Le bit de signe
- Un champs binaire d'exposant
- Un autre champs binaire pour la mantisse tronquée

Ce sont les encodages utilisés dans les champs binaires d'exposant et de mantisse tronquée que nous allons essayés de comprendre, le comparateur base toute sa logique de comparaison sur les propriétés de ces champs là.

Le champs d'exposant utilise un encodage par biais, ce dernier est assez simple à comprendre.
Enfaite, un champs binaire pour lequel nous utilisons un encodage $Binary$ $Unsigned$ représente une valeur numérique $X$, auquel nous ajoutons un biais.
Le biais $B$ est une constante positive ou nulle, et la valeur représenté par le champs d'exposant d'un nombre flottant est issu du calcul $X-B$.

Pour le champs de la mantisse tronquée nous avons besoin de faire un rapide rappel sur l'encodage intial d'un nombre flottant (`le standard IEEE-754 est un enrobage plus qu'autre chose`).
La partie entière d'un nombre flottant utilise du $Binary$ $Unsigned$, tandis que la partie fractionnaire fait usage d'une version modifiée du $Binary$ $Unsigned$.
Chaque bit de la partie fractionnaire est un `facteur d'une puissance de 2 négative` et non positive, ceci permet de représenté des valeurs entre $1$ et $0$.
Le poids des puissances négative de $2$ décroix en fonction de la position du bit.
Prenons l'exemple du nombre $3.75_{10}$.

$$3.75_{10} = 11.11_2 = IntegerPart((1 \times 2^1) + (1 \times 2^0)) + FractionalPart((1 \times 2^{-1}) + (1 \times 2^{-2}))$$

Précisons que la virgule n'est là que pour facilité la lecture du nombre, `elle n'est pas réelement présente dans le codage des nombres flottants`.
Ce qu'il y a d'important à remarqué pour la partie fractionnaire, c'est que la valeur d'un bit à $1$ de poids $i$, est toujours strictement supérieur à la somme des bits de poids inférieur à $i$.
Autrement dit, pour le nombre fractionnaire $N$ dont ont ne prête attention qu'au bit à $1$ de poids $i$, la $\sum_{weight=0}^{i-1} (N_{weight} \times 2^{weight})$ est strictement inférieur à $1 \times 2^i$.

Nous verrons que le comparateur se sert de cela pour effectué les comparaisons.
Pour en revenir au sujet de la mantisse tronquée, elle est composée des bits de la partie entière et de la partie fractionnaire d'un nombre flottant.

// biais négtif champs exp
// binary unsigned uniquement nombre entier positif

// l'encodage de la mantisse tronquée et du champs d'exposant.

Plus une puissance de $2$ est grande, plus elle confère 

Définissons la mantisse tronquée $X$ est: 

$$X_{TruncatedMantissa} \in BinaryUnsigned = \sum_{weight=0}^9\left(X_{weight}\times2^{weight}\right)$$

Par ailleurs, la mantisse non normalisé de $X$ est: 

$$X_{Mantissa} \in BinaryUnsigned = \sum_{weight=0}^9\left(X_{weight}\times2^{weight}\right) + 1\times2^{10}$$
