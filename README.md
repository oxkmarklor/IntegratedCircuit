poub:  \gg \ll \in 

Commençons par la base.

Un bit est symboliquement représenté par un $0$ ou un $1$.
Un champs binaire est quant à lui composé d'un ensemble de bit, c'est à dire d'un ensemble de $0$ et de $1$.

Ce sont ces champs binaire qui nous permettent de représenté des nombres dans un ordinateur, faire des calculs, mémorisé des résultats.
Il y a cependant plein de façon de représenté des nombres dans un champs binaire, mais il y a des manières plus efficasse que d'autre de le faire en fonction des besoins, prenons un exemple.
Pour un champs de $16$ bits nous pourrions calculé la somme des bits pour représenter un nombre.
Ceci s'appelle un encodage, et avec ce dernier nous ne pourrions codé que des valeurs entre $0$ et $16$ inclus, ce qui n'est pas optimal pour bien des situations.
Mais parmis les multiples encodage existant, le plus connu de tous porte le nom de $Binary$ $Unsigned$.

Le $Binary$ $Unsigned$ utilise les mêmes primitives que la base décimale (celle que nous utilisons tous) pour représenter des nombres, des puissances.
Pour représenter la valeur $103$ nous décomposons en réalité chaque chiffre du nombre pour le multiplié avec une puissance de $10$ adéquat, regardez ci-dessous.

$$103 = 1 \times 10^2 + 0 \times 10^1 + 3 \times 10^0$$

Nous retrouvons chacun des chiffres du nombre entrain de multiplié une puissance de $10$.
Remarquons que le chiffre des unités $3$ est facteur de $10$ à la puissance $0$, le chiffre d'une ordre plus grand (des dizaines) est le facteur de $10$ à la puissance $1$ et enfin le chiffre des centaines multiplie $10$ à la puissance $2$.
La valeur des puissances croix en fonction de la position du chiffre, qui plus est, elle commence à $0$.

Ceci n'est pas propre à la base décimale mais à nimporte quel base numérique, comme la base binaire par exemple.
La base binaire (ou base $2$) utilise des chiffres $0$ et $1$ plutôt que ceux de la base décimale allant de $0$ à $9$.
Par ailleurs, un tel chiffre s'appelle un bit, ce qui est la contraction de $Binary$ $Digit$.
Aussi, les puissances de $10$ sont remplacés par des puissances de $2$.
Nous venons tout juste de décrire ce qu'est le $Binary$ $Unsigned$.

Un champs binaire pour lequel nous utilisons un encodage $Binary$ $Unsigned$ associe un puissance de $2$ positive à chaque bit du champs.
Les puissances vont de $0$ jusqu'à $N-1$, $N$ qui représente le nombre de bits total dans le champs.
Le bit du champs qui est facteur de $2$ à la puissance $0$ est appellé le $LSB$ pour $Least$ $Significant$ $Bit$, par ailleurs, plus une puissance de $2$ est grande et plus son poids est grand.
La notion de poids est relative à l'ordre de grandeur d'une puissance de $2$, naturellement le $LSB$ est le bit de poids le plus faible d'un champs binaire.







//

Plus une puissance de $2$ est grande, plus elle confère 

Définissons la mantisse tronquée $X$ est: 

$$X_{TruncatedMantissa} \in BinaryUnsigned = \sum_{weight=0}^9\left(X_{weight}\times2^{weight}\right)$$

Par ailleurs, la mantisse non normalisé de $X$ est: 

$$X_{Mantissa} \in BinaryUnsigned = \sum_{weight=0}^9\left(X_{weight}\times2^{weight}\right) + 1\times2^{10}$$
