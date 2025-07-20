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
Pour représenter la valeur $103_{10}$ en décimale nous décomposons en réalité chaque chiffre du nombre pour le multiplié avec une puissance de $10$ adéquat, puis il faut faire la somme des résultats de chaque produit.
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
Nous venons tout juste de décrire ce qu'est le $Binary$ $Unsigned$.

Un champs binaire pour lequel nous utilisons un encodage $Binary$ $Unsigned$ associe un puissance de $2$ positive à chaque bit du champs.
Les puissances vont de $0$ jusqu'à $N-1$, où $N$ représente le nombre de bits total dans le champs.
Le bit du champs qui est facteur de $2$ à la puissance $0$ est appellé le $LSB$ pour $Least$ $Significant$ $Bit$, par ailleurs, `plus une puissance de 2 est grande` et `plus son poids est grand`.
La notion de poids est relative à la valeur d'une puissance de $2$, naturellement le $LSB$ est le bit de poids le plus faible d'un champs binaire.
Nous aurons compris qu'avec des puissances de $2$ positives, les nombres encodé en $Binary$ $Unsigned$ sont positifs ou nulle (pas de bit de signe).

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
Globalement, la norme définit trois éléments qui composent chaque nombre à virgule flottante:
- Le bit de signe
- Un champs binaire d'exposant
- Un autre champs binaire pour la mantisse tronquée

Ce sont les encodages utilisés dans les champs binaires d'exposant et de mantisse tronquée que nous allons essayés de comprendre, le comparateur base toute sa logique de comparaison sur les propriétés de ces champs là.

Le champs d'exposant utilise un encodage par biais, ce dernier est assez simple à comprendre.
Enfaite, un champs binaire pour lequel nous utilisons un encodage $Binary$ $Unsigned$ code une valeur numérique $X$, auquel nous ajoutons un biais.
Le biais $B$ est une constante positive ou négative, et la valeur représenté par le champs d'exposant d'un nombre flottant est issu du calcul $X+B$.
A savoir que le biais du champs d'exposant d'un nombre flottant encodé en IEEE-754 est toujours négatif.
Il est qui plus est toujours équivalent à $-\left(2^{N-1}\right)+1$ où $N$ est le nombre de bits du champs d'exposant.

Etant donné que l'encodage par biais se base sur le $Binary$ $Unsigned$, le champs d'exposant partage les mêmes caractéristiques que cet encodage.
Notamment celui qui nous dit que la valeur d'un bit à $1$ de poids $i$ est strictement supérieur à la somme des puissances inférieurs à $i$.

Pour le champs de la mantisse tronquée nous avons besoin de faire un rapide rappel sur l'encodage intial d'un nombre flottant (`le standard IEEE-754 est un enrobage plus qu'autre chose`).
La partie entière d'un nombre flottant utilise du $Binary$ $Unsigned$, tandis que la partie fractionnaire fait usage d'une version modifiée du $Binary$ $Unsigned$.
Chaque bit de la partie fractionnaire est un `facteur d'une puissance de 2 négative` et non positive, ceci permet de représenté des valeurs entre $1$ et $0$.
Le poids des puissances négatives de $2$ décroix en fonction de la position du bit.
Prenons l'exemple du nombre $3.75_{10}$.

$$3.75_{10} = 11.11_2 = IntegerPart\left(\left(1 \times 2^1\right) + \left(1 \times 2^0\right)\right) + FractionalPart\left(\left(1 \times 2^{-1}\right) + \left(1 \times 2^{-2}\right)\right)$$

Précisons que sous la forme binaire du nombre, le point n'est là que pour facilité sa lecture (`dans les faits, il n'est pas réelement présent dans le codage des nombres flottants`).
Ce qu'il y a d'important à remarqué pour la partie fractionnaire, c'est que la valeur d'un bit à $1$ de poids $i$, est toujours strictement supérieur à la somme des bits de poids inférieur à $i$.
Autrement dit, pour le nombre fractionnaire $N$ dont ont ne prête attention qu'au bit à $1$ de poids $i$, la $\sum_{weight=0}^{i-1} (N_{weight} \times 2^{weight})$ est strictement inférieur à $1 \times 2^i$.

Pour en revenir au sujet de la mantisse tronquée, elle est composée des bits de la partie entière et de la partie fractionnaire d'un nombre flottant.
Vu que la partie entière et fractionnaire d'un nombre flottant partagent les caractéristiques du $Binary$ $Unsigned$, c'est aussi le cas de la mantisse tronquée.
Nous verrons que le comparateur se sert de cela pour effectué les comparaisons.

-- -

La démonstration mathématique du circuit est donc assez simple, après toutes ces explications.

Nous définissons deux nombres flottants $Half$ $Precision$ ($16$ bits) $\alpha$ et $\beta$, la comparaison à produire sera $\left|\alpha\right|\gt\left|\beta\right|$.
Les opérandes ne font alors plus que $15$ bits (omission du bit de signe).

Commençons par chercher si le champs d'exposant $E$ de $\beta$ est plus grand que celui de $\alpha$.
Nous avons vu que ce champs binaire de $5$ bits code la valeur de l'exposant avec un encodage par biais, mais aussi que cet encodage partage toutes les caractéristiques de l'encodage $Binary$ $Unsigned$.

- Voici une opération logique dont nous allons nous servir $Nimply\left(X,Y\right)\mapsto X \wedge Not(Y)$

Nous effectuons l'opération $Nimply\left(\beta_{\sigma},\alpha_{\sigma}\right)$ où $\sigma$ représente le poids des bits d'opérande, jusqu'à obtenir $1$ ou $\sigma = 10$.

- $\sum_{\sigma=14}^{10} Nimply\left(\beta_{\sigma}, \alpha_{\sigma}\right)$

Avec un $1$ en résultat nous savons que $E_{\beta\sigma} \gt E_{\alpha\sigma}$ et par conséquent $\left(E_{\beta\sigma}\times 2^{\sigma}\right) \gt \left(\sum_{\sigma}^{10} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

Si $\sigma \lt 14$ alors le fait d'avoir perçu un $1$ n'assure pas pour autant que $E_{\beta} \gt E_{\alpha}$ car $\left(\sum_{\sigma+1}^{14} E_{\beta\sigma} \times 2^{\sigma}\right) \le \left(\sum_{\sigma+1}^{14} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

Dans le cas d'une égalité des bits de poids supérieur à $\sigma$ pour nos deux champs d'exposant $E_{\beta}$ et $E_{\alpha}$ alors nous affirmons que $E_{\beta} \gt E_{\alpha}$, mais dans le cas d'une infériorité stricte $E_{\beta} \lt E_{\alpha}$.

Le circuit départage ces deux cas avec une opération $XOR$ entre chaque bit de même poids compris entre $\left]\sigma ; 14\right]$ des opérandes $\beta$ et $\alpha$.

$$\left(\left(\sum_{\sigma+1}^{14} \left(\beta_{\sigma} \oplus \alpha_{\sigma}\right) = 0 \right) = \left(E_{\beta} \gt E_{\alpha}\right)\right) \vee \left(\left(\sum_{\sigma+1}^{14} \left(\beta_{\sigma} \oplus \alpha_{\sigma}\right) \gt 0 \right) = \left(E_{\beta} \lt E_{\alpha}\right)\right)$$

Cependant, si $\sigma = 10$ sans avoir reçu de $1$ en résultat, alors $\left(\sum_{\sigma=14}^{10} E_{\beta\sigma} \times 2^{\sigma}\right) \le \left(\sum_{\sigma=14}^{10} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

// Que faire se passe t'il si sigma = 10 qu'avec des 0.




Plus une puissance de $2$ est grande, plus elle confère 

Définissons la mantisse tronquée $X$ est: 

$$X_{TruncatedMantissa} \in BinaryUnsigned = \sum_{weight=0}^9\left(X_{weight}\times2^{weight}\right)$$

Par ailleurs, la mantisse non normalisé de $X$ est: 

$$X_{Mantissa} \in BinaryUnsigned = \sum_{weight=0}^9\left(X_{weight}\times2^{weight}\right) + 1\times2^{10}$$
