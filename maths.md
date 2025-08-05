# Démonstration mathématique

Il y a une documentation dédié au circuit pour comprendre sa raison d'être ainsi que son architecture, je conseil de la lire pour plus d'information.

Le rôle du circuit est de généré deux bits de sortie, l'un pour configuré un circuit soustracteur de nombre flottant, et l'autre pour la sortie même de cette unité de calcul.
A cette fin, le circuit nommé __FPU Configuration Unit__ reçoit deux opérandes flottants _Half Precision_ $\alpha$ et $\beta$ en valeur absolu.
Il doit procédé à une comparaison de supériorité stricte de l'un de ces opérandes envers l'autre, admettons $\left(\vert \alpha \vert \gt \vert \beta \vert\right)$.

La démonstration va être scindé en deux parties, l'une pour le traitement des champs d'exposant $E$ et l'autre pour les champs de mantisse tronquée $T$.
Voici où est ce que ce situe les bits de ces champs dans l'encodage d'un nombre IEEE-754 _Half Precision_:

$$\left[S_{15}, \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

__S__: Sign bit, __E__: Exponent field, __T__: Truncated mantissa

Chaque indice de l'intervalle $\left[0;15\right]$ représente le poids de chacun des bits.
C'est en effet très commun d'indicé les bits par leur poids dans un champs binaire, cela permet d'apporté plus de précision aux propos notamment.

Cependant, nous ne considérons que les bits de l'intervalle $\left[0;14\right]$.
Le circuit ne se soucis que de la valeur absolu de $\alpha$ et $\beta$, donc le bit de signe de poids $15$ est omis.
De plus, nous avons vus précédemment que les champs d'exposant $E$ étaient traités avant les champs de mantisse tronquée $T$, car ces derniers peuvent engendré des points terminaux.
C'est pourquoi nous allons commencé par ceux-ci.

Tout d'abord, voici quelques définition d'opération importante pour la démonstration.

$$Write \ \left(x, \ y\right) \rightarrow \ x \ := \ y$$

__Write__ est une fonction d'affectation, le paramètre $y$ _est copié_ dans le paramètre $x$.

$$Nimply \ \left(x, \ y\right) \rightarrow \ x \ \wedge \ \overline{y}$$

__Nimply__ est bien plus connu en tant que porte logique dans les circuits intégrés qu'en tant qu'opération logique en soit.
D'où le fait qu'il n'existe pas de symbole opératoire pour cette opération.

Par ailleurs, $\overline{x}$ est l'opération logique __Not__, qui inverse un bit en son opposé $\left(1 = \overline{0}\right)$ ou $\left(0 = \overline{1}\right)$.
Cette opération possède également sa propre porte logique, qui est d'ailleurs assez élémentaire à la conception de circuit en tout genre.

Pour finir, le symbole $\wedge$ est l'opération logique __And__ qui ne retourne un bit à $1$ que lorsque ses deux bits d'opérande le sont aussi (sinon $0$), et le symbole $\vee$ est l'opération logique __Or__ qui ne retourne un $0$ que si ses deux bits d'opérande le sont également (sinon $1$).

# Traitement du champs d'exposant

La variable $\sigma$ (sigma) servira d'indice pour accéder aux bits $\in \left[10;14\right]$ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.

$$\left(1\right) \quad \sum_{\sigma=14}^{10} \ Write\left(\tau_{\sigma}, \ Nimply \ \left(E_{\beta\sigma}, \ E_{\alpha\sigma}\right)\right)$$

La variable $\tau$ est un champs binaire de $15$ bits dont la représentation est la même que celle illustrée plus haut pour les nombres flottants _Half Precision_ (sans le bit de signe).
Pour chacune des opérations $Nimply$ sur les bits de poids $i \in \left[10;14\right]$ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$, les résultats sont enregistrés dans $\tau_i$.
Ce qui veut dire que $\tau \in \left[10;14\right]$ correspond aux bits de résultat des opérations $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$, comme le montre l'expression qui figure ci-dessus.

-- -

Initialement $\sigma = 10$, mais l'expression changera d'elle même la valeur de $\sigma$.

$$\left(2\right) \quad \sum_{i=14}^{\sigma} Write\left(\sigma, \ \left[\left(\overline{\tau_i} \times 10\right) + \tau_i \times \left(11 + i \ mod \ 5\right)\right]\right)$$

L'objectif de cette expression est d'inscrire dans $\sigma$ la valeur du poids du ___zéro anonyme___ de poids le plus faible, parmis tout ceux qui ont une "importance".
Pour comprendre ce que cette phrase veut dire, nous allons voir ce qu'est un _zéro anonyme_ ainsi que l'importance variable que l'on apporte à chacun d'eux.

Commençons par dire qu'un "_zéro anonyme_" est un terme que j'ai crée dans la documentation du circuit électronique.
Concrètement, un _zéro anonyme_ est un bit de résultat à $0$ sortant d'une opération logique $Nimply$.
Ce nom si étrange est due au fait que l'opération $Nimply$ génère un bit de résultat à $0$ lorsque:
  - Les deux bits de paramètres $\left(x\right)$ et $\left(y\right)$ sont identiques.
  - Le bit sur $\left(y\right)$ est supérieur à celui sur $\left(x\right)$.

Maintenant, revenons un peu à l'expression $\left(1\right)$.
Chaque bit de $\tau \in \left[10;14\right]$ est le résultat de $Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)$ pour le poids $i \in \left[10;14\right]$.
Si $\left(\tau_i = 1\right)$ alors cela veut dire que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$, ou autrement dit $\left(E_{\beta i} \times 2^i\right) \ \gt \ \left(E_{\alpha i} \times 2^i\right)$.

Mais ce n'est pas tout.
Rappellez vous du chapitre "_Encodage par biais_" qui parle de l'encodage du champs d'exposant des nombres flottants IEEE-754.
Nous avons appris dans ce chapitre que la valeur d'un bit à $1$ de poids $i$ dans un champs d'exposant, est inconditionnellement supérieur à la somme de la valeur de chacun des bits de poids inférieur à $i$.
Autrement dit, nous pouvons en déduire que $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i=i-1}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.

Nous pouvons donc en conclure que si $\left(\tau_i = 1\right)$ alors $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.
Pour autant, ceci ne veut pas dire que $\left(E_{\beta} \gt E_{\alpha}\right)$.

Si $\left(\tau_{14} = 1\right)$ alors un point terminal $\left(E_{\alpha} \lt E_{\beta}\right)$ est atteint car $\left(E_{\beta 14} \times 2^{14}\right) \ \gt \ \sum_{i=14}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.
Il s'avère que les bits $E_{\alpha 14}$ et $E_{\beta 14}$ sont les _MSB_ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.
Pour que $\left(E_{\alpha} \gt E_{\beta}\right)$ il faudrait déjà que ces champs d'exposant disposent d'un bit de poids plus fort que $14$, ce qui n'est pas le cas.
En bref, nous comprenons que même si $\tau_{\left(i-1\right)}$ est un _zéro anonyme_ alors cela ne changera rien au fait que $\left(E_{\alpha} \lt E_{\beta}\right)$.

Cependant, c'est différent dans le cas où le poids du _MSB1_ est $\tau_i$ pour $i \in \left[10;13\right]$.
Nous pouvons donc être certain qu'il y a au moins un bit de poids plus fort que $\tau_i$ dans $\tau \in \left[10;14\right]$, mais ce ou ces bits sont des _zéros anonymes_.
Le problème avec le bit $\tau_{\left(i+1\right)}$ c'est qu'il est un _zéro anonyme_ qui par définition ne permet pas de savoir si $\left(E_{\alpha\left(i+1\right)} = E_{\beta\left(i+1\right)}\right)$ ou si $\left(E_{\alpha\left(i+1\right)} \gt E_{\beta\left(i+1\right)}\right)$.
Le ou les _zéros anonymes_ de poids supérieur à $\tau_i$ sont décisifs car si $\left(\sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right) \ = \ \sum_{i=i+1}^{14} \ \left(E_{\beta i} \times 2^i\right)\right)$ alors $\left(E_{\alpha} \lt E_{\beta}\right)$ car $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.
En revanche si $\left(\sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right) \ \gt \ \sum_{i=i+1}^{14} \ \left(E_{\beta i} \times 2^i\right)\right)$ alors $\left(E_{\alpha} \gt E_{\beta}\right)$.

Par conséquent, les _zéros anonymes_ "important" ont un poids d'une valeur supérieur à celui de poids $i$ du _MSB1_ de $\tau \in \left[10;14\right]$, ceux qui ne le sont pas ont un poids inférieur à $\tau_i$. 

Jusqu'ici nous avions un _MSB1_ dans $\tau \in \left[10;14\right]$, mais il pourrait ne pas y en avoir.
Chacun bit des bits de $\tau_i$ sont alors des _zéros anonymes_, ce qui nous permet d'affirmé que $\left(E_{\alpha} \ge E_{\beta}\right)$.
Si jamais $\left(E_{\alpha} \gt E_{\beta}\right)$, c'est obligatoirement que $\left(E_{\alpha i} \gt E_{\beta i}\right)$ pour $i \in \left[10;14\right]$, mais si $\left(E_{\alpha} = E_{\beta}\right)$, alors cela veut dire que tout les bits de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ sont identiques.

L'expression inscrit donc dans $\sigma$ la valeur du poids $+1$ du _MSB1_ lorsqu'il y en a un dans $\tau \in \left[10;14\right], et $10$ si il n'y a pas de _MSB1_.

Pour poursuivre la démonstration, il va vous falloir passé de cette expression à la suivante en fonction de la valeur de $\sigma$ :
  - Si $\left(\sigma = 15\right)$ alors veuillez vous rendre sur l'expression suivante (celle de numéro $\left(3\right)$ ).
  - Sinon si $\left(\sigma \in \left[10;14\right]\right)$ alors rendez-vous vers l'expression numéro $\left(4\right)$.

-- -

$$\left(3\right) \quad \left(E_{\beta\left(\sigma - 1\right)} \times 2^{\left(\sigma - 1\right)}\right) \gt \sum_{\sigma=\left(\sigma-1\right)}^{10} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right)$$

-- -

$$\left(4\right) \quad \lambda = \sum_{\sigma}^{14} \ \left(E_{\beta\sigma} \times 2^{\sigma}\right)$$

-- -

$$\left(5\right) \quad \left[\left(\sum_{\sigma}^{14} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right) = \lambda\right), \ Goto\left(7\right)\right] \ \vee \ \left[\left(\sum_{\sigma}^{14} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right) > \lambda\right), \ Goto\left(8\right)\right]$$

-- -

$$\left(6\right) \quad \left(E_{\alpha} \lt E_{\beta}\right)$$


-- -

$$\left(7\right) \quad \left(E_{\alpha} \gt E_{\beta}\right)$$

-- -

$$\left(8\right) \quad \left[\left(T_{\alpha} \gt T_{\beta}\right), \ \left(\vert \alpha \vert \gt \vert \beta \vert\right)\right] \ \vee \ 
\left[\left(\left(T_{\alpha} = T_{\beta}\right) \ \vee \ \left(T_{\alpha} < T_{\beta}\right)\right), \ \left(\vert \alpha \vert \le \vert \beta \vert\right)\right]$$
