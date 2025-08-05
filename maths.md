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

L'objectif de cette expression est d'inscrire dans $\sigma$ la valeur du poids du ___zéro anonyme___ de poids le plus faible, parmis tout ceux qui ont une importance.
Pour comprendre ce que cette phrase veut dire, nous allons voir ce qu'est un _zéro anonyme_ ainsi que l'"importance" variable que l'on apporte à chacun d'eux.

Commençons par dire qu'un "_zéro anonyme_" est un terme que j'ai crée dans la documentation du circuit électronique.
Concrètement, un _zéro anonyme_ est un bit de résultat à $0$ sortant d'une opération logique $Nimply$.
Ce nom si étrange est due au fait que l'opération $Nimply$ génère un bit de résultat à $0$ quand:
  - Les deux bits de paramètres $\left(x\right)$ et $\left(y\right)$ sont identiques.
  - Ou lorsque le bit sur $\left(y\right)$ est supérieur à celui sur $\left(x\right)$.

Maintenant, revenons un peu à l'expression $\left(1\right)$.
Chaque bit de $\tau \in \left[10;14\right]$ est le résultat de $Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)$ pour le poids $i \in \left[10;14\right]$.
Si $\left(\tau_i = 1\right)$ alors cela veut dire que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$, ou autrement dit $\left(E_{\beta i} \times 2^i\right) \ \gt \ \left(E_{\alpha i} \times 2^i\right)$.

Mais ce n'est pas tout.
Rappellez vous du chapitre "_Encodage par biais_" qui parle de l'encodage du champs d'exposant des nombres flottants IEEE-754.
Nous avons appris dans ce chapitre que la valeur d'un bit à $1$ de poids $i$ dans un champs d'exposant, est strictement supérieur à la somme de la valeur de chacun des bits de poids inférieur à $i$.
Autrement dit, ceci donne $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i=i-1}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.

Si $\left(\tau_i = 1\right)$ nous pouvons donc en conclure que $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$, nous verrons à nouveau ceci dans l'expression $\left(3\right)$.

Imaginons maintenant que $\left(\tau_i = 1\right)$ soit le _MSB1_ du champs binaire $\tau \in \left[10;14\right]$.
Si $\left(i = 14\right)$ alors un point terminal est atteint avec $\left(E_{\beta} \gt E_{\alpha}\right)$.
La raison en est qu'il n'y a pas de bit de poids supérieur à $14$ dans les champs d'exposant $E_{\alpha}$ et $E_{\beta}$, permettant de faire passé $E_{\alpha}$ de strictement inférieur à strictement supérieur à $E_{\beta}$.

Mais si $\left(i \in \left[10;13\right]\right)$, alors c'est une autre histoire.
Il y a alors dans $\tau$ au moins un bit de poids plus fort que son _MSB1_, ce qui pose problème car c'est un _zéro anonyme_.
Autrement dit, la seule chose dont nous puissions être sûre c'est que $\left(E_{\alpha} \ge E_{\beta}\right)$ car si $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$ pour $i\in \left[10;13\right]$, alors $\left(\sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right) \ \ge \ \sum_{i=i+1}^{14} \ \left(E_{\beta i} \times 2^i\right)\right)$.
// lien avec les zéros anonymes

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
