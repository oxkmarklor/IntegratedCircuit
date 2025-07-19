poub:  \gg \ll \in 


Voyons comment un nombre flottant half precision est codé sur un champs binaire de 16 bits $X$.

Le $BinaryUnsigned$ est une représentation entière et non signé (positive) des nombres dans un ordinateur.
Cet encodage utilise chaque bit d'un nombre comme d'un facteur d'une puissance positive de $2$, par ailleurs, il est définit ce qui s'appelle un $LSB$ pour $LeastSignificantBit$ et un $MSB$ pour $MostSignificantBit$. 
Ces termes sont souvent repris pour divers raisons, mais il faut comprendre qu'ils proviennent à l'origine du $BinaryUnsigned$.

A savoir que chacun bits possède un "poids".
Ce poids n'est autre que l'importance qu'a le bit dans la représentation d'un nombre, le $LSB$ est le bit de poids le plus faible.
En effet, le $LSB$ est rattaché à la plus petite puissance positive de $2$ qu'est $0$, ou autrement dit il s'agit du bit le moins significatif comme son nom l'indique.
Je précise que le poids ne fait pas référence à la valeur d'un bit, seulement à la puissance de $2$ qu'il multiplie.

Le $MSB$ est naturellement l'inverse du $LSB$, il référence le bit de poids le plus fort, celui dont la puissance de $2$ est la plus importante du nombre. 


//

Plus une puissance de $2$ est grande, plus elle confère 

Définissons la mantisse tronquée $X$ est: 

$$X_{TruncatedMantissa} \in BinaryUnsigned = \sum_{weight=0}^9\left(X_{weight}\times2^{weight}\right)$$

Par ailleurs, la mantisse non normalisé de $X$ est: 

$$X_{Mantissa} \in BinaryUnsigned = \sum_{weight=0}^9\left(X_{weight}\times2^{weight}\right) + 1\times2^{10}$$
