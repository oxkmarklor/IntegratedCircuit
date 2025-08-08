# Démonstration mathématique

Il y a une documentation dédié au circuit pour comprendre sa raison d'être ainsi que son architecture, je conseil de la lire pour plus d'information.

Le circuit se nomme __FPU Configuration Unit__.
Le rôle du circuit est de généré deux bits de sortie, l'un pour configuré un circuit soustracteur de nombre flottant, et l'autre pour la sortie même de cette unité de calcul.
A cette fin, le circuit reçoit deux opérandes flottants _Half Precision_ $\alpha$ et $\beta$, en valeur absolu.
Il doit procédé à une comparaison de supériorité stricte de l'un de ces opérandes envers l'autre.
La démonstration mathématique va se basé sur le teste de la condition suivante $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$.

### Les opérandes Half Precision

Je rappelle que l'encodage IEEE-754 définit trois éléments dans chacun des nombres flottants du standard, le _bit de signe_, le _champs d'exposant_ ainsi que le _champs de mantisse tronquée_.
Voici la disposition précise de chaque bit de chacun de ces champs pour l'encodage d'un nombre _Half Precision_ ($16$ bits):

$$\left[S_{15}, \quad E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}, \quad T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right]$$

__S__: Sign bit,  __E__: Exponent field,  __T__: Truncated mantissa

Chaque indice (nombre) compris dans l'intervalle $\left[0;15\right]$ représente le poids d'un bit précis.
C'est en effet très commun d'indicé les bits d'un champs binaire par leur poids.

Cependant, dans les démonstrations mathématique nous ne considérerons que les bits de l'intervalle $\left[0;14\right]$.
N'oublions pas que le circuit ne se soucis que de la valeur absolu de ses opérandes $\left(\vert\alpha\vert\right)$ et $\left(\vert\beta\vert\right)$, donc le bit de signe de poids $15$ est omis.
De plus, nous avons vus dans le chapitre "_Ordre de traitement des champs d'exposant et de mantisse tronquée_", que les champs d'exposant $E$ des opérandes étaient traités avant les champs de mantisse tronquée $T$.
Rappellons que ceci est dû au fait que le circuit profite des *___points terminaux___ que les champs d'exposant génèrent souvent.
C'est pourquoi le document commence par traité le cas des champs d'exposant $E_{\alpha}$ et $E_{\beta}$, puis ensuite celui des champs de mantisse tronquée $T$.

*Un ___point terminal___ est le fait que le circuit soit capable de déduire un résultat correct, par le seul traitement des champs d'exposant.
Le circuit court-circuite alors les traitements qui concerne les champs de mantisse tronquée, et génère le résultat final plus rapidement.

### L'organisation du document

!!!!
Le document commence par la démonstration mathématique du traitement des champs d'exposant $E_{\alpha}$ et $E_{\beta}$, puis enchaine avec celle des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Les démonstrations sont verbeuses, pour plus de concision il y a un résumé pour chacune d'entre elles.
Les résumés sont composés seulement des expressions mathématique, ainsi que d'une courte description de chaque expression.

### Quelques définitions

__Write__ est une fonction d'affectation, le paramètre $\left(y\right)$ _est copié_ dans le paramètre $\left(x\right)$.

$$Write \ \left(x, \ y\right) \rightarrow \ x \ := \ y$$

__Nimply__ est une fonction qui reproduit le comportement d'une porte logique du même nom.
C'est une opération logique assez peu connu, d'où le fait qu'il n'existe pas de symbole opératoire qui lui est propre.

$$Nimply \ \left(x, \ y\right) \rightarrow \ x \ \wedge \ \overline{y}$$

De plus, $\overline{y}$ est l'opération logique __Not__ qui inverse un bit en son opposé $\left(1 = \overline{0}\right)$ ou $\left(0 = \overline{1}\right)$.
Cette opération possède également sa propre porte logique, qui est d'ailleurs assez élémentaire pour la conception de circuit en tout genre.

Pour finir, le symbole $\wedge$ est l'opération logique __And__ qui ne retourne un bit à $1$ que lorsque ses deux bits d'opérande le sont aussi (sinon $0$), et le symbole $\vee$ est l'opération logique __Or__ qui ne retourne un $0$ que si ses deux bits d'opérande le sont également (sinon $1$).

## Le traitement des champs d'exposant

La démonstration démarre par ceci:

$$\sum_{i=14}^{10} \ Write\left(\tau_i, \ Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)\right)$$

La variable $\tau$ est un champs binaire de $15$ bits dont la représentation est la même que celle illustrée plus haut dans "_Les opérandes Half Precision_", mais sans le bit de signe.
Chaque résultat d'une opération $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$ pour $i \in \left[10;14\right]$, est inscrit dans $\tau_i$.
Ce qui veut dire que $\tau \in \left[10;14\right]$ correspond aux bits de résultat des opérations $Nimply$ sur $E_{\beta i}$ et $E_{\alpha i}$.

### L'opération logique Nimply

L'opération $Nimply$ a été définit plus haut.
Selon cette définition, nous comprenons que le bit de résultat d'une telle opération ne peut être à $1$ que lorsque le bit sur le paramètre $\left(x\right)$ est $1$, et que celui sur $\left(y\right)$ est à $0$.
Pour tout les autres cas, si $\left(x = y\right)$ ou $\left(x \lt y\right)$ alors le bit de résultat sera $0$.
La documentation du circuit électronique définit le terme de "___zéro anonyme___" pour désigné tout bit de résultat à $0$, sortant d'une opération $Nimply$.

### Qu'est ce qu'un zéro anonyme?

Nous venons de voir à quoi correspond le terme de "_zéro anonyme_".

Cependant, nous ne connaissons pas la raison d'un nom aussi bizarre.
Le circuit se sert des sorties des portes logiques $Nimply$ pour faire des déductions sur les entrées de celle-ci, et la démonstration mathématique en fait de même.
Mais lorsque le bit de sortie d'une opération $Nimply$ est à $0$, il nous est impossible de savoir si cette sortie a été générée par deux bits d'opérandes identique sur les paramètres $\left(x\right)$ et $\left(y\right)$, ou si le bit sur le paramètre $\left(y\right)$ est strictement supérieur à celui sur $\left(x\right)$.
En bref, ce flou pose un problème que nous dervons géré dans la démonstration mathématique (le circuit doit lui aussi résoudre ce problème).

Petite précision d'ailleurs.
Il existe en réalité des _zéros anonymes_ ___capitaux___ et ___non capitaux___, nous définirons plus bas la différence entre ces deux types de _zéro anonyme_.

## Les deux facettes des zéros anonymes (introduction)

Revenons à l'expression qui figure ci-dessus.
Chaque bit $\tau_i$ est le résultat de $\ Nimply \ \left(E_{\beta i}, \ E_{\alpha i}\right)$ pour le poids $i \in \left[10;14\right]$.
Si $\left(\tau_i = 1\right)$ alors nous savons que $\left(E_{\beta i} = 1\right)$ tandis que $\left(E_{\alpha i} = 0\right)$, ou autrement dit $\left(E_{\beta i} \times 2^i\right) \ \gt \ \left(E_{\alpha i} \times 2^i\right)$.

De plus, rappellez vous du chapitre "_Encodage par biais_" qui parle de l'encodage du champs d'exposant des nombres flottants IEEE-754.
Dans ce chapitre, il est dit que la valeur d'un bit à $1$ de poids $i$ d'un champs d'exposant $E$, est _inconditionnellement_ supérieur à la somme de la valeur de chacun de ses bits de poids inférieur à $i$.
Autrement dit, si nous prenons au pied de la lettre la phrase ci-dessus, alors $\forall \left(i \in \left[11;14\right]\right)$ nous obtenons $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i=i-1}^{10} \ \left(E_{\beta i} \times 2^i\right)$.
Mais nous pouvons arrangé cette expression à notre champs d'exposant $E_{\alpha}$, alors $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i=i-1}^{10} \ \left(E_{\alpha i} \times 2^i\right)$. 

Au final, nous pouvons en conclure que si $\left(\tau_i = 1\right)$ alors $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$.

Ce qui vient d'être dit ci-dessus ne démontre pas pour autant que $\left(E_{\alpha} \lt E_{\beta}\right)$.

### Les zéros anonymes non capitaux

Prenons le cas de $\left(\tau_i = \tau_{14} = 1\right)$.
Dans se cas spécifique, nous avons affaire a un point terminal car $\left(E_{\alpha} \lt E_{\beta}\right)$.
Ce point terminal est atteint par le fait que $\left(E_{\beta 14} \times 2^{14}\right) \ \gt \ \sum_{i=14}^{10} \ \left(E_{\alpha i} \times 2^i\right)$, sachant que $E_{\alpha 14}$ et $E_{\beta 14}$ sont les _MSB_ des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.

En bref, nous comprenons que même si $\left(\tau_{\left(i-1\right)} = \tau_{13} = 0\right)$, alors ce _zéro anonyme_ ne changera rien au fait que $\left(E_{\alpha} \lt E_{\beta}\right)$.
La raison en est que l'inéquation qui figure ci-dessus reste valide qu'importe la valeur du bit $E_{\alpha 13}$, qu'il soit à $0$ ou à $1$.

De manière plus général, nous pouvons en déduire que tout _zéro anonyme_ de poids inférieur au _MSB1_ de $\tau \in \left[11;14\right]$, est _non capital_.

### Les zéros anonymes capitaux

Prenons désormais le cas dans lequel le poids du _MSB1_ de $\tau$ est $\tau_i$ pour $i \in \left[10;13\right]$.
Dans cette situation, nous sommes certains qu'il y a _au moins_ le bit $\tau_{\left(i+1\right)}$ qui est d'un poids supérieur à $\tau_i$.
Par définition, $\tau_{\left(i+1\right)}$ (et les autres bits de poids supérieur à $\tau_i$, si il y en a) sont des _zéros anonymes_.
Le problème des bits de poids supérieur à $\tau_i$, c'est que par essence ils ne permettent pas de déduire la valeur des bits de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$.
Par exemple pour $\tau_{\left(i+1\right)}$, nous ne savons pas si $\left(E_{\alpha\left(i+1\right)} = E_{\beta\left(i+1\right)}\right)$ ou si $\left(E_{\alpha\left(i+1\right)} \gt E_{\beta\left(i+1\right)}\right)$.

Ceci pose problème car les _zéros anonymes_ de poids supérieur à $\tau_i$ sont décisifs dans le processus de génération d'un point terminal.
A ce stade nous savons que $\left(E_{\beta i} \times 2^i\right) \ \gt \ \sum_{i}^{10} \ \left(E_{\alpha i} \times 2^i\right)$, mais comme mentionné plus haut ceci ne suffit pas à dire que $\left(E_{\alpha} \lt E_{\beta}\right)$.

Par conséquent, si $\ Write \ \left(\lambda, \ \sum_{i=i+1}^{14} \ \left(E_{\beta i} \times 2^i\right)\right)$ et que $\left(\lambda \ = \ \sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right)\right)$, alors le point terminal était et reste $\left(E_{\alpha} \lt E_{\beta}\right)$.
Cependant, si $\left(\lambda \ \lt \ \sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right)\right)$ alors le point terminal change pour $\left(E_{\alpha} \gt E_{\beta}\right)$, car $\left(\sum_{i=i+1}^{14} \ \left(E_{\alpha i} \times 2^i\right) \ \gt \ \sum_{i=14}^{10} \ \left(E_{\beta i} \times 2^i\right)\right)$.
Autrement dit, la somme de la valeur des bits de poids supérieur à $\tau_i$ (le _MSB1_ de $\tau$) dans $E_{\alpha}$, est strictement supérieur à la valeur totale du champs $E_{\beta}$.

Nous comprenons donc que les _zéros anonymes_ de poids supérieur au _MSB1_ de $\tau \in \left[10;13\right]$ devront être traités avec attention par le circuit, ce sont des _zéros anonymes capitaux_.

### L'obtention d'un point non terminal

Jusqu'ici nous avions un _MSB1_ dans $\tau \in \left[10;14\right]$, mais il pourrait ne pas y en avoir.
Dans cette situation, chacun des bits de $\tau \in \left[10;14\right]$ est un _zéro anonyme_, ce qui nous permet d'affirmé que $\left(E_{\alpha} \ge E_{\beta}\right)$.

Nous pourrions atteindre un point terminal si jamais $\left(E_{\alpha} \gt E_{\beta}\right)$, mais ce point terminal ne pourrait s'obtenir qu'à la condition que $\left(E_{\alpha i} \gt E_{\beta i}\right)$ pour $i \in \left[10;14\right]$.
A l'inverse, nous pourrions aussi atteindre une situation inédite de *___point non terminal___ si $\left(E_{\alpha} = E_{\beta}\right)$.
Cette situation inédite ne peut se produire que dans le cas où l'ensemble des bits de même poids des champs d'exposant $E_{\alpha}$ et $E_{\beta}$ sont identiques, ou autrement dit $\forall \left(i \in \left[10;14\right]\right)$ alors $\left(E_{\alpha i} = E_{\beta i}\right)$.

Pour conclure, les _zéros anonymes_ qui composent l'entièreté du champs $\tau \in \left[10;14\right]$ sont capitaux à prendre en compte pour le circuit, car ils ont chacun le pouvoir de faire basculé un _point non terminal_ en un _point terminal_.

*Je rappelle qu'un _point terminal_ se définit comme suit: "_Un point terminal est le fait que le circuit soit capable de déduire un résultat correct, par le seul traitement des champs d'exposant._".
Un ___point non terminal___ est tout logiquement l'inverse.
Lorsque les champs d'exposant $E_{\alpha}$ et $E_{\beta}$ sont égaux, il n'est pas possible de généré un résultat correct avec le seul traitement de ces champs.
Le circuit doit alors poursuivre les traitements sur les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Ce que nous allons faire prochainement.

## Le traitement des champs de mantisse tronquée

Voici dans un premier temps de quoi dépend le traitement des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Nous remarquerons que c'est la même opération (à quelques détails prêt) que pour les champs d'exposant $E$ des opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$.

$$\sum_{i=9}^0 \ Write\left(\tau_i, \ Nimply \ \left(T_{\beta i}, \ T_{\alpha i}\right)\right)$$

La variable $\tau$ est techniquement la même que celle utilisée pour la démonstration précédente.
La seule différence, c'est que le bit de résultat de chaque opération $Nimply$ sur $T_{\beta i}$ et $T_{\alpha i}$ est inscrit dans $\tau_i$, pour $i \in \left[0;9\right]$.

### Rappel sur les champs d'exposant et de mantisse tronquée

Dans le chapitre "_La mantisse tronquée, une histoire de puissance de_ $2$" qui parle de l'encodage du champs de mantisse tronquée des nombres flottants IEEE-754.
Il est dit que la valeur d'un bit à $1$ de poids $i$ d'un champs de mantisse tronquée $T$, est _inconditionnellement_ supérieur à la somme de la valeur de chacun de ses bits de poids inférieur à $i$.
Rappellons que c'est aussi le cas des champs d'exposant, comme cela a été mentionné dans la première partie de la démonstration.

En bref, l'origine de ces points communs est que les champs d'exposant et de mantisse tronquée utilisent des encodages dont les primitives sont celles du _Binary Unsigned_, c'est à dire des puissances de $2$.
Je vous conseille une relecture du début du document si ceci n'est pas clair pour vous.

Pour finir, le chapitre "_Ordre de traitement des champs d'exposant et de mantisse tronquée_" explique pourquoi est ce que les champs d'exposant $E$ sont traités avant les champs de mantisse tronquée $T$ (prise en compte des _points terminaux_).
Dans ce chapitre, il y a la formule mathématique $\left(\left(1+{\color{LightGreen}T}\right) \times 2^{\color{LightGreen}E}\right)$ qui explique comment est ce qu'un nombre flottant est représenté en IEEE-754.
Les termes ${\color{LightGreen}T}$ et ${\color{LightGreen}E}$ sont respectivement le champs de mantisse tronquée et d'exposant d'un nombre flottant quelconque.
Le chapitre explique que si deux nombres flottants $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$ ont des champs d'exposant égaux $\left({\color{LightGreen}E_{\alpha}} = {\color{LightGreen}E_{\beta}}\right)$, alors les produits $\left(\left(1+{\color{LightGreen}T_{\alpha}}\right) \times 2^{\color{LightGreen}E_{\alpha}}\right)$ et $\left(\left(1+{\color{LightGreen}T_{\beta}}\right) \times 2^{\color{LightGreen}E_{\beta}}\right)$ ne peuvent être différent que si $\left({\color{LightGreen}T_{\alpha}} \neq {\color{LightGreen}T_{\beta}}\right)$.
Qui plus est, si $\left({\color{LightGreen}T_{\alpha}} \gt {\color{LightGreen}T_{\beta}}\right)$ alors $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ et dans le cas inverse si $\left({\color{LightGreen}T_{\alpha}} \lt {\color{LightGreen}T_{\beta}}\right)$ alors $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$.

Pour comparé la stricte supériorité des champs d'exposant entre eux et détecté la présence de point terminaux, nous utilisons les même primitives communes que celle partagé avec les champs de mantisse tronquée (les primitives du _Binary Unsigned_).
A la fin de la démonstration du traitement des champs d'exposant, nous en avons conclus qu'en cas d'égalité des champs d'exposant nous aurions besoin de traité les champs de mantisse tronquée.
Nous savons désormais quel est ce traitement et comment l'effectué.



















-- -

$$\left(1\right) \quad \sum_{\sigma=14}^{10} \ Write\left(\tau_{\sigma}, \ Nimply \ \left(E_{\beta\sigma}, \ E_{\alpha\sigma}\right)\right)$$

-- -

Initialement $\left(\sigma = 10\right)$, mais l'expression changera d'elle même la valeur de $\sigma$.

$$\left(2\right) \quad \sum_{i=14}^{\sigma} Write\left(\sigma, \ \left[\left(\overline{\tau_i} \times 10\right) + \tau_i \times \left(11 + i \ mod \ 5\right)\right]\right)$$

L'objectif de cette expression est d'inscrire dans $\sigma$ la valeur du poids du ___zéro anonyme___ de poids le plus faible, parmis tout ceux qui ont une importance.
Pour comprendre ce que cette phrase veut dire, nous allons voir ce qu'est un _zéro anonyme_ ainsi que l'"importance" variable que l'on apporte à chacun d'eux.

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
