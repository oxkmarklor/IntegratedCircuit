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

$$Xor \ \left(x, \ y\right) \rightarrow \ \left(x \ \wedge \ \overline{y}\right) \ \vee \ \left(\overline{x} \ \wedge \ y\right)$$

__Xor__ est une opération logique très connu et qui a sa propre porte logique.
L'opération possède son propre symbole opératoire attitré $\oplus$, mais j'utiliserai la fonction $Xor$ pour plus de clareté.

Par ailleurs, $\overline{x}$ est l'opération logique __Not__, qui inverse un bit en son opposé $\left(1 = \overline{0}\right)$ ou $\left(0 = \overline{1}\right)$.
Cette opération possède également sa propre porte logique, qui est d'ailleurs assez élémentaire à la conception de circuit en tout genre.

Pour finir, le symbole $\wedge$ est l'opération logique __And__ qui ne retourne un bit à $1$ que lorsque ses deux bits d'opérande le sont aussi (sinon $0$), et le symbole $\vee$ est l'opération logique __Or__ qui ne retourne un $0$ que si ses deux bits d'opérande le sont également (sinon $1$).

# Traitement du champs d'exposant

La variable $\sigma$ (sigma) servira d'indice pour accéder aux bits $\in \left[10;14\right]$ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.

$$\left(1\right) \quad \sum_{\sigma=14}^{10} \ Write\left(\tau_{\sigma}, \ Nimply \ \left(E_{\beta\sigma}, \ E_{\alpha\sigma}\right)\right)$$

La variable $\tau$ est un champs binaire de $15$ bits dont la représentation est la même que celle illustrée plus haut pour les nombres flottants _Half Precision_ (sans le bit de signe).
Pour chacun des traitements des bits de poids $E_{\alpha i}$ et $E_{\beta i}$, les résultats sont enregistrés dans $\tau_i$.
Ce qui veut dire que $\tau \in \left[10;14\right]$ correspond aux bits de résultat des opérations sur les bits $E_{\alpha i}$ et $E_{\beta i}$.

-- -

$$\left(2\right) \quad \sigma = 10, \ \left(\sum_{i=14}^{\sigma} Write\left(\sigma, \ \left[\left(\overline{\tau_i} \times 10\right) + \tau_i \times \left(11 + i \ mod \ 5\right)\right]\right)\right)$$

L'expression numéro $\left(1\right)$ effectue une opération $Nimply$ sur les bits de poids $i \in \left[10;14\right]$ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.
Dans la documentation du circuit, un bit de résultat à $0$ sortant d'une opération $Nimply$ est appellé "__zéro anonyme__".
La raison en est que l'opération $Nimply$ génère un $0$ dans deux situations, une égalité entre les bits d'opérandes, ou une supériorité stricte du bit sur le paramètre $\left(y\right)$ vis à vis de celui sur $\left(x\right)$.
En bref, nous savons que $\left(E_{\alpha} \ge E_{\beta}\right)$ si l'ensemble des bits de résultats dans $\tau \in \left[10;14\right]$ sont à $0$.
Mais nous savons aussi que $\left(E_{\alpha} \neq E_{\beta}\right)$ si dans $\tau$ l'un des bits mentionné dans l'intervalle ci-dessus est à $1$.
Dans ce second cas, nous pouvons être sûre que le circuit atteindra un point terminal, au contraire du premier cas.

Si tout ou partie des bits de $\tau \in \left[10;14\right]$ sont à $1$, l'expression numéro $\left(2\right)$ enregistre dans $\sigma$ la valeur du poids $+1$ du _MSB1_ de $\tau$.
Cependant, si l'ensemble des bits de $\tau \in \left[10;14\right]$ sont à $0$, alors $\left(\sigma = 10\right)$.

Ceci nous permet ainsi de faire la différenciation entre une situation qui mène quoi qu'il arrive vers un point terminal, d'une situation pour laquelle ce n'est pas forcé d'être le cas:
  - Si $\left(\sigma = 10\right)$ alors $Goto\left(?\right)$.
  - Sinon si $\left(\sigma \in \left[11;15\right]\right)$ alors $Goto\left(3\right)$.

L'instruction ___Goto___ demande simplement au lecteur de poursuivre la démonstration vers le numéro de l'expression indiqué dans ses "_paramètres_".

-- -

$$\left(3\right) \quad \left(E_{\beta\left(\sigma - 1\right)} \gt E_{\alpha\left(\sigma - 1\right)}\right)$$

Nous savons que $\left(\sum_{\sigma = 14}^{10} \ \tau_{\sigma} \gt 0\right)$, et donc que $\left(\tau_{\left(\sigma - 1\right)} = 1\right)$.
Dans l'expression numéro $\left(1\right)$, je cite "_Pour chacun des traitements des bits de poids_ $E_{\alpha i}$ _et_ $E_{\beta i}$ _, les résultats sont enregistrés dans_ $\tau_i$.".
Toujours selon cette expression, si $\left(\tau_{\left(\sigma - 1\right)} = 1\right)$ sachant que $\tau_i = Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)$, cela veut dire que $\left(E_{\beta\left(\sigma - 1\right)} = 1\right)$ pendant que $\left(E_{\alpha\left(\sigma - 1\right)} = 0\right)$.

Dans le cas où $\left(\sigma = 15\right)$ alors $\left(\tau_{\left(\sigma - 1\right)} = \tau_{14} = 1\right)$.
Nous savons que $\tau_{14}$ est le _MSB_ (et le _MSB1_) de $\tau \in \left[10;14\right]$, ce qui veut dire que les _MSB_ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ valent chacun ceci $\left(E_{\beta 14} = 1\right)$ ainsi que $\left(E_{\alpha 14} = 0\right)$.
Auquel cas, $Goto\left(4\right)$.

Si $\left(\sigma \neq 15\right)$ alors $\sigma \in \left[11;14\right]$. 
Ceci veut dire que le poids du _MSB1_ dans $\tau \in \left[10;13\right]$ et il y a donc au moins un bit (celui de poids $\sigma$, ou plus) dont le poids est strictement supérieur à celui du _MSB1_.
Les bits de poids supérieur au _MSB1_ de $\tau$ ($\sigma$ et plus) sont des zéros anonymes, il nous faut alors savoir à quoi est ce qu'ils sont due.
Par conséquent $Goto\left(5\right)$.


-- -

$$\left(4\right) \quad \left(E_{\beta\left(\sigma - 1\right)} \times 2^{\left(\sigma - 1\right)}\right) \gt \left[\sum_{\sigma = \sigma - 1}^{10} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right]$$

En reprenant ce que nous venons de dire plus haut, nous constatons que $\left(E_{\beta 14} \times 2^{14}\right) \gt \left(E_{\alpha 14} \times 2^{14}\right)$.
Mais rappellons nous du chapitre "Encodage par biais" qui vise à expliqué l'encodage du champs d'exposant d'un nombre IEEE-754.
Nous avons appris dans ce chapitre que cet encodage partage les même propriétés que le __Binary Unsigned__.
Ou autrement dit, _la valeur d'un bit à_ $1$ _de poids_ $i$ dans le champs d'exposant est _strictement supérieur à la somme des valeurs des bits de poids inférieur à_ $i$.
Par conséquent, nous savons aussi que $\left(E_{\beta 14} \times 2^{14}\right) \gt \left[\sum_{\sigma = 13}^{10} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right]$, ce qui nous amène à l'expression numéro $\left(5\right)$ pour $\left(\sigma = 15\right)$ je le rappelle.

Etant donné que $\tau_{14}$ est le _MSB_ (et le _MSB1_) de $\tau \in \left[10;14\right]$, alors ce bit de résultat est issu du traitement du _MSB_ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.
Nous en déduisons alors que $\left(E_{\alpha} \lt E_{\beta}\right)$.

Mais rappellons nous que dans le chapitre "_Ordre de traitement des champs d'exposant et de mantisse tronquée_" nous avons vus qu'un ___point terminal___ était atteint si $\left(E_{\alpha} \neq E_{\beta}\right)$.
Plus particulièrement, nous avons vus qu'avec $\left(E_{\alpha} \lt E_{\beta}\right)$ alors $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$, et la condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ __échoue__.

-- -

$$\left(5\right) \quad \lambda = \sum_{\sigma}^{14} \ \left(E_{\beta\sigma} \times 2^{\sigma}\right)$$

En nous retrouvant ici nous savons que $\left(\sigma \in \left[11;14\right]\right)$.
Comme nous l'avons vu plus haut dans l'expression $\left(3\right)$, il existe au moins un bit $\tau_{\sigma}$ qui est d'un poids strictement supérieur à celui du _MSB1_ de $\tau \in \left[10;13\right]$.
Ce ou ces bits de poids $\sigma$ (et plus) sont des _zéros anonymes_, ce qui veut dire que $\left(\sum_{\sigma}^{14} \ E_{\alpha\sigma}\right) \ \ge \ \left(\sum_{\sigma}^{14} \ E_{\beta\sigma}\right)$.
Cette expression calcul alors la somme de la valeur de chaque bit dans $E_{\beta}$ de poids supérieur au _MSB1_ de $\tau$, afin de comparé ce résultat $\left(\lambda\right)$ avec celui obtenu par le même calcul sur $E_{\alpha}$.
Ou autrement dit, $\left(\lambda \ = \ \left(E_{\beta} \in \left[\sigma;14\right]\right)\right)$.

-- -

$$\left(6\right) \quad \left[\left(\sum_{\sigma}^{14} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right) = \lambda\right), \ Goto\left(7\right)\right] \ \vee \ \left[\left(\sum_{\sigma}^{14} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right) > \lambda\right), \ Goto\left(8\right)\right]$$

A ce stade nous savons que $\left(E_{\alpha} \in \left[10; \left(\sigma - 1\right)\right]\right) \ \lt \ \left(E_{\beta} \in \left[10; \left(\sigma - 1\right)\right]\right)$.
Donc que sur une partie des bits de poids faible de $E_{\alpha}$ et $E_{\beta}$, le champs d'exposant $\left(E_{\alpha} \lt E_{\beta}\right)$.

Il y a maintenant deux possibilités vis à vis des _zéros anonymes_ de poids $\sigma$ et plus. 
En bref, si la valeur de $E_{\alpha}$ compris dans l'intervalle de $\left[\sigma;14\right]$ est égale à $\left(E_{\beta} \in \left[\sigma;14\right]\right)$ alors nous sommes certains que $\left(E_{\alpha} \lt E_{\beta}\right)$.
Dans le cas contraitre, si $\left(E_{\alpha} \in \left[\sigma;14\right]\right) \ \gt \ \left(E_{\beta} \in \left[\sigma;14\right]\right)$ nous savons alors que $\left(E_{\alpha} \gt E_{\beta}\right)$.

L'expression calcul $E_{\alpha} \in \left[\sigma;14\right]$, de la même manière que ne le fait l'expression $\left(6\right)$ pour $E_{\beta}$.

-- -

$$\left(7\right) \quad \left(E_{\alpha} \lt E_{\beta}\right)$$

Nous venons de voir plus haut dans l'expression $\left(6\right)$, que $\left(E_{\alpha} \lt E_{\beta}\right)$ si jamais je cite "_la valeur de_ $E_{\alpha}$ _compris dans l'intervalle de_ $\left[\sigma;14\right]$ _est égale à_ $\left(E_{\beta} \in \left[\sigma;14\right]\right)$".
Mais comme nous l'avons vus dans le chapitre du nom de "_Ordre de traitement des champs d'exposant et de mantisse tronquée_", lorsque $\left(E_{\alpha} \neq E_{\beta}\right)$ alors nous sommes obligatoirement face à un point terminal.
En l'occurence, nous avons vu dans ce chapitre que si $\left(E_{\alpha} \lt E_{\beta}\right)$ alors $\left(\vert\alpha\vert \ \lt \ \vert\beta\vert\right)$.
Ce qui consiste alors en un ___point terminal___ possible, et la condition initial $\left(\vert\alpha\vert \ \gt \ \vert\beta\vert\right)$ __échoue__.

-- -

$$\left(8\right) \quad \left(E_{\alpha} \gt E_{\beta}\right)$$

Nous venons de voir plus haut dans l'expression $\left(6\right)$, que $\left(E_{\alpha} \gt E_{\beta}\right)$ si la valeur de $E_{\alpha}$ compris dans l'intervalle $\left[\sigma;14\right]$ est strictement supérieur à $\left(E_{\beta} \in \left[\sigma;14\right]\right)$.
Mais comme nous l'avons vus dans le chapitre du nom de "_Ordre de traitement des champs d'exposant et de mantisse tronquée_", lorsque $\left(E_{\alpha} \neq E_{\beta}\right)$ alors nous sommes obligatoirement face à un point terminal.
En l'occurence, nous avons vu dans ce chapitre que si $\left(E_{\alpha} \gt E_{\beta}\right)$ alors $\left(\vert\alpha\vert \ \gt \ \vert\beta\vert\right)$.
Ce qui consiste alors en un ___point terminal___ possible, et la condition initial $\left(\vert\alpha\vert \ \gt \ \vert\beta\vert\right)$ __réussie__.

-- -

$$\left(6\right) \quad \lambda = \sum_{\sigma=10}^{14} \ \left(E_{\beta\sigma} \times 2^{\sigma}\right)$$


$$\left(9\right) \quad \left[\left(\left(\sum_{\sigma = 14}^{10} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right) \ \gt \ \left(\sum_{\sigma = 14}^{10} \ \left(E_{\beta\sigma} \times 2^{\sigma}\right)\right)\right), \ Goto\left(9\right)\right] \ \vee \ \left[\left(\sum_{\sigma = 14}^{10} \ \left(\left(E_{\alpha\sigma} \times 2^{\sigma}\right) \ = \ \left(E_{\beta\sigma} \times 2^{\sigma}\right)\right)\right), \ Goto\left(11\right)\right]$$


-- -

$$\left(10\right) \quad \left[\left(T_{\alpha} \gt T_{\beta}\right), \ \left(\vert \alpha \vert \gt \vert \beta \vert\right)\right] \ \vee \ 
\left[\left(\left(T_{\alpha} = T_{\beta}\right) \ \vee \ \left(T_{\alpha} < T_{\beta}\right)\right), \ \left(\vert \alpha \vert \le \vert \beta \vert\right)\right]$$


// continuer le traitement avec T











Cependant, si le champs $\tau$ n'est composé que de $5$ bits à $0$, alors:

  - Nous savons que l'opération $Nimply\left(E_{\beta\sigma}, E_{\alpha\sigma}\right)$ ne renvoie $0$ que si $\left(E_{\beta\sigma} = E_{\alpha\sigma}\right)$, ou si $\left(E_{\beta\sigma} = 0\right)$ alors que $\left(E_{\alpha\sigma} = 1\right)$.

  - Par conséquent vu que $\tau$ n'est composé que de bits à $0$, nous savons que $\left(\sum_{\sigma=10}^{14} E_{\beta\sigma} \times 2^{\sigma}\right) \le \left(\sum_{\sigma=10}^{14} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

  - En reprenant en partie ce qui a été dit plus haut, la valeur du champs d'exposant $E_{\beta}$ est $\left(\sum_{\sigma=10}^{14} \left(E_{\beta\sigma} \times 2^{\sigma}\right) = r\right)$:

    - Dans le cas où $\left(\sum_{\sigma=10}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt r\right)$ alors $\left(E_{\beta} \lt E_{\alpha}\right)$.

    - Cependant si $\left(\sum_{\sigma=10}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) = r\right)$ alors $\left(E_{\beta} = E_{\alpha}\right)$.

$$\left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) = 0 \right) = \left(E_{\beta} \gt E_{\alpha}\right)\right) \vee \left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) \gt 0 \right) = \left(E_{\beta} \lt E_{\alpha}\right)\right)$$
