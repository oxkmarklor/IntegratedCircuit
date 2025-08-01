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

La variable $\tau$ interprète un champs binaire tampon de $15$ bits.
Les bits de résultats sont stockés dans $\tau$ sur les même poids que les bits d'opérande des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.

$$\left(2\right) \quad \left[\left(\sum_{\sigma=14}^{10} \ \tau_{\sigma} \gt 0\right), \ Goto\left(3\right)\right] \ \vee \ \left[\left(\sum_{\sigma=14}^{10} \ \tau_{\sigma} = 0\right), \ Goto\left(?\right)\right]$$

L'instruction __Goto__ demande simplement au lecteur de poursuivre la démonstration vers le numéro indiqué dans ses "_paramètres_".

$$\left(3\right) \quad \sigma = 10, \ \left(\sum_{i=14}^{\sigma} Write\left(\sigma, \ \left[\left(\overline{\tau_i} \times 10\right) + \tau_i \times \left(11 + i \ mod \ 5\right)\right]\right)\right)$$

Nous savons qu'il y a au moins un des bits de $\tau \in \left[10;14\right]$ qui est à $1$.
Cette expression a pour but d'enregistré dans $\sigma$ la valeur du poids de ce _MSB1_, incrémenté de $1$.

 __Si__ $\left(\sigma = 15\right)$ __alors nous atteigons un point terminal et__ $Goto\left(4\right)$, sinon $Goto\left(?\right)$.

$$\left(4\right) \quad \left(E_{\beta\left(\sigma - 1\right)} \gt E_{\alpha\left(\sigma - 1\right)}\right)$$

Pour commencer, nous savons que $\left(\sigma - 1 = 14\right)$ est le poids du _MSB_ des champs d'exposant $E$.
Nous savons que $\left(4\right)$ est correct car $\tau_{14} = 1$ et où je rappelle que $\tau_{14} = Nimply \ \left(E_{\beta 14}, \ E_{\alpha 14}\right)$.

$$\left(5\right) \quad \left[\left(E_{\beta\left(\sigma - 1\right)} \times 2^{\left(\sigma - 1\right)}\right) \gt \left(\sum_{\sigma = \sigma - 1}^{10} \ \left(E_{\alpha\sigma} \times 2^{\sigma}\right)\right)\right]$$

L'expression $\left(5\right)$ est elle aussi correct car nous savons que les champs d'exposant $E$ des nombres IEEE-754 utilisent un encodage par biais.
Encodage par biais qui partage les même propriétés que l'encodage _Binary Unsigned_.
Autrement dit, dans un tel encodage la valeur d'un bit à $1$ qui multiplie son poids $i$, est strictement supérieur à la somme des résultats des produits entre les bits de poids inférieur $i$ et leur poids respectif.
De plus, grâce à $\left(4\right)$ nous savons que $E_{\alpha 14} = 0$, d'où le fait que sa prise en compte dans le calcul de somme en $\left(5\right)$ n'est pas un problème.

Pour conclure, grâce à $\left(5\right)$ nous savons que $\left(E_{\beta} \gt E_{\alpha}\right)$ et comme nous l'avons vus plus haut, cela permet de dire que $\left(\vert \alpha \vert \lt \vert \beta \vert\right)$.
La comparaison échoue.







  - Dans le cas où $\sigma \neq 14$, ce qui figure ci-dessus ne prouve pas que $\left(E_{\beta} \gt E_{\alpha}\right)$ car $\left(\sum_{\sigma+1}^{14} E_{\beta\sigma} \times 2^{\sigma}\right) \le \left(\sum_{\sigma+1}^{14} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

    - Nous savons que la somme des bits de poids supérieur à $\sigma$ pour $E_{\beta}$ est $\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \times 2^{\sigma}\right) = r\right)$.

    - Par conséquent si $\left(\sum_{\sigma+1}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) = r \right)$ alors $\left(E_{\beta} \gt E_{\alpha}\right)$, sinon  $\left(\sum_{\sigma+1}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt r \right)$ et $\left(E_{\beta} \lt E_{\alpha}\right)$.


Cependant, si le champs $\tau$ n'est composé que de $5$ bits à $0$, alors:

  - Nous savons que l'opération $Nimply\left(E_{\beta\sigma}, E_{\alpha\sigma}\right)$ ne renvoie $0$ que si $\left(E_{\beta\sigma} = E_{\alpha\sigma}\right)$, ou si $\left(E_{\beta\sigma} = 0\right)$ alors que $\left(E_{\alpha\sigma} = 1\right)$.

  - Par conséquent vu que $\tau$ n'est composé que de bits à $0$, nous savons que $\left(\sum_{\sigma=10}^{14} E_{\beta\sigma} \times 2^{\sigma}\right) \le \left(\sum_{\sigma=10}^{14} E_{\alpha\sigma} \times 2^{\sigma}\right)$.

  - En reprenant en partie ce qui a été dit plus haut, la valeur du champs d'exposant $E_{\beta}$ est $\left(\sum_{\sigma=10}^{14} \left(E_{\beta\sigma} \times 2^{\sigma}\right) = r\right)$:

    - Dans le cas où $\left(\sum_{\sigma=10}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) \gt r\right)$ alors $\left(E_{\beta} \lt E_{\alpha}\right)$.

    - Cependant si $\left(\sum_{\sigma=10}^{14} \left(E_{\alpha\sigma} \times 2^{\sigma}\right) = r\right)$ alors $\left(E_{\beta} = E_{\alpha}\right)$.

$$\left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) = 0 \right) = \left(E_{\beta} \gt E_{\alpha}\right)\right) \vee \left(\left(\sum_{\sigma+1}^{14} \left(E_{\beta\sigma} \oplus E_{\alpha\sigma}\right) \gt 0 \right) = \left(E_{\beta} \lt E_{\alpha}\right)\right)$$
