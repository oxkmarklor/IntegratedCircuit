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

Le document commence par la démonstration mathématique du traitement des champs d'exposant $E_{\alpha}$ et $E_{\beta}$, puis enchaine avec celle des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Les démonstrations sont verbeuses, pour plus de concision il y a un résumé global qui suit la seconde et dernière démonstration.
Ce résumé est seulement composé d'expression mathématique ainsi que d'une courte description pour chacune d'entre elle.

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

## Rappel sur les champs d'exposant et de mantisse tronquée

Le chapitre "_Ordre de traitement des champs d'exposant et de mantisse tronquée_" explique pourquoi est ce que les champs d'exposant $E$ sont traités avant les champs de mantisse tronquée $T$.
Dans ce chapitre, il y a la formule mathématique $\left(\left(1+T\right) \times 2^E\right)$ qui explique comment est ce qu'un nombre flottant est représenté en IEEE-754.
Nous comprenons que si deux nombres flottants $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$ ont des champs d'exposant égaux $\left(E_{\alpha} = E_{\beta}\right)$, alors la valeur de ces deux nombres $\left(\left(1+T_{\alpha}\right) \times 2^{E_\alpha}\right)$ et $\left(\left(1+T_{\beta}\right) \times 2^{E_\beta}\right)$ ne dépend que de $T_{\alpha}$ et $T_{\beta}$.
C'est pourquoi la première partie de la démonstration ne génère un _point non terminal_ que lorsque $\left(E_{\alpha} = E_{\beta}\right)$, après quoi le traitement se propage aux champs de mantisse tronquée eux même.

Les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$ font l'oeuvre d'une comparaison de supériorité stricte de l'un envers l'autre, comme pour les champs d'exposant.
Je vous rappelle que les champs d'exposant et de mantisse tronquée ont des points communs au niveau de leurs encodages, tout est expliqué dans le chapitre "?TBD?" pour ceux qui ont besoin d'un rafraichissement.
En bref, l'encodage du champs de mantisse tronquée partage les même propriétés que celui d'exposant.
Comme nous allons le voir plus bas, la comparaison entre les champs de mantisse tronquée suit le même processus de traitement que celui des champs d'exposant.

## Le traitement des champs de mantisse tronquée

Voici dans un premier temps de quoi dépend le traitement des champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.
Nous remarquerons que c'est la même opération (à quelques détails prêt) que pour les champs d'exposant $E$ des opérandes $\vert \ \alpha \ \vert$ et $\vert \ \beta \ \vert$.

$$\sum_{i=9}^0 \ Write\left(\tau_i, \ Nimply \ \left(T_{\beta i}, \ T_{\alpha i}\right)\right)$$

La variable $\tau$ est techniquement la même que celle utilisée pour la démonstration précédente.
La seule différence, c'est que le bit de résultat de chaque opération $Nimply$ sur $T_{\beta i}$ et $T_{\alpha i}$ est inscrit dans $\tau_i$, pour $i \in \left[0;9\right]$.

### Un échec certain de la comparaison 

Dans le chapitre "_La mantisse tronquée, une histoire de puissance de 2_" qui parle de l'encodage du champs de mantisse tronquée des nombres flottants IEEE-754.
Il est dit que la valeur d'un bit à $1$ de poids $i$ d'un champs de mantisse tronquée $T$, est _inconditionnellement_ supérieur à la somme de la valeur de chacun de ses bits de poids inférieur à $i$.
Rappellons que c'est aussi le cas pour les champs d'exposant, comme cela a été mentionné dans la première partie de la démonstration.
Le calcul de comparaison entre les champs d'exposant se base sur ces primitives mathématiques, et il en va de même pour les champs de mantisse tronquée $T_{\alpha}$ et $T_{\beta}$.

Prenons le cas de $\left(\tau_i = \tau_9 = 1\right)$.
En sachant que $\left(\tau_9 = 1\right)$ nous savons que $\left(T_{\beta 9} = 1\right)$ tandis que $\left(T_{\alpha 9} = 0\right)$, et par conséquent $\left(T_{\beta 9} \times 2^9\right) \gt \sum_{i=9}^0 \ \left(T_{\alpha i} \times 2^i\right)$.
Etant donné que $T_{\alpha 9}$ ainsi que $T_{\beta 9}$ sont les _MSB_ des champs de mantisse tronquée $T$, nous pouvons alors conclure que $\left(T_{\alpha} \lt T_{\beta}\right)$. 
La condition initial que teste le circuit $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ __échoue__.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$, alors $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \lt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$, ou dit autrement $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$.

N'oublions pas qu'il peut y avoir des _zéros anonymes_ dans $\tau \in \left[0;8\right]$, mais ces derniers ne peuvent être que de poids inférieur au _MSB1_ de $\tau$ qui est $\tau_9$.
Ces potentiels bit à $0$ sont alors des _zéros anonymes non capitaux_, sans importance.

### Lorsque les zéros anonymes rendent la génération du résultat incertain

Je tient à rappellé qu'il faudra gardé en tête le fait que $\left(E_{\alpha} = E_{\beta}\right)$ pour toute les expressions qui figurent ci-dessous.

Prenons désormais le cas dans lequel le poids du _MSB1_ de $\tau$ est $\tau_i$ pour $i \in \left[0;8\right]$.
Dans cette situation, nous sommes certains qu'il y a _au moins_ le bit $\tau_{\left(i+1\right)}$ qui est d'un poids supérieur à $\tau_i$.
Les bits de poids supérieur à $\tau_i$ sont tous des _zéros anonymes capitaux_, nous devons donc y prêté attention.

Nous calculons la somme de la valeur des bits de $T_{\beta}$ d'un poids supérieur à $\tau_i$ avec $\ Write \ \left(\lambda, \ \sum_{i=i+1}^9 \ \left(T_{\beta i} \times 2^i\right)\right)$, et nous enregistrons le résultat dans $\lambda$.
Puis nous faisons le même calcul pour $T_{\alpha}$, et vérifions si le résultat obtenu est égale à ce que nous avons trouvé pour $T_{\beta}$, c'est à dire $\lambda$.
Dans le cas où $\left(\lambda \ = \ \sum_{i=i+1}^9 \ \left(T_{\alpha i} \times 2^i\right)\right)$ alors $\left(T_{\alpha} \lt T_{\beta}\right)$ car $\left(T_{\beta i} \times 2^i\right) \gt \sum_i^0 \ \left(T_{\alpha i} \times 2^i\right)$.
Par conséquent, la condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ __échoue__ car $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \lt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$, ou autrement dit $\left(\vert\alpha\vert \lt \vert\beta\vert\right)$.

Cependant, si $\left(\lambda \ \lt \ \sum_{i=i+1}^9 \ \left(T_{\alpha i} \times 2^i\right)\right)$ alors $\left(T_{\alpha} \gt T_{\beta}\right)$ car $\left(\sum_{i=i+1}^9 \ \left(T_{\alpha i} \times 2^i\right) \ \gt \ \sum_{i=9}^0 \ \left(T_{\beta i} \times 2^i\right)\right)$.
Autrement dit, la somme de la valeur des bits de poids supérieur à $\tau_i$ (le _MSB1_ de $\tau \in \left[0;8\right]$) dans $T_{\alpha}$, est strictement supérieur à la valeur totale du champs de mantisse tronquée $T_{\beta}$.
La condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ ___réussie___ car $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$.

Pour finir, il est possible qu'il n'y ait pas de _MSB1_ dans $\tau \in \left[0;9\right]$, tout les bits composant $\tau$ sont alors des _zéros anonymes capitaux_.
Nous pouvons donc affirmé que $\left(T_{\alpha} \ge T_{\beta}\right)$.

Dans le cas où $\left(T_{\alpha i} \gt T_{\beta i}\right)$ pour $i \in \left[0;9\right]$, alors $\left(T_{\alpha} \gt T_{\beta}\right)$.
La condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ ___réussie___ car $\left(\sum_i^9 \ \left(T_{\alpha i} \times 2^i\right) \ \gt \ \sum_{i=9}^0 \ \left(T_{\beta i} \times 2^i\right)\right)$, ou autrement dit $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \gt \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$.

A l'inverse, il est tout aussi probable que $\left(T_{\alpha} = T_{\beta}\right)$.
Pour cela il faudrait que chaque bit de même poids des champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$ soient identiques, ou pour le dire autrement $\forall \left(i \in \left[0;9\right]\right)$ alors $\left(T_{\alpha i} = T_{\beta i}\right)$.
Sachant que $\left(E_{\alpha} = E_{\beta}\right)$ et que $\left(T_{\alpha} = T_{\beta}\right)$, alors $\left(\left(1+T_{\alpha}\right) \times 2^{E_{\alpha}}\right) \ = \ \left(\left(1+T_{\beta}\right) \times 2^{E_{\beta}}\right)$ et la condition initial $\left(\vert\alpha\vert \gt \vert\beta\vert\right)$ ___échoue___.

# Conclusion et Nombre dénormaux

Les processus de traitement des champs d'exposant et des mantisses tronquées pour les opérandes $\alpha$ et $\beta$ sont identiques, comme nous avons pu le voir dans les démonstrations qui figurent ci-dessus.
Dans les faits, le circuit électronique n'est composé que d'un seul sous ensemble de circuit logique, permettant ainsi le traitement de chacun des champs des opérandes flottants.
La documentation du circuit aborde l'architecture de ces sous ensembles de circuit logique.

Je tient aussi à rappellé que tout ce que nous avons vu ici n'est qu'une démonstration de la faisabilité de la logique de comparaison du circuit.
Pour autant, les opérations que produit réelement le circuit ne sont pas les même que celle qui sont utilisées dans la démonstration.
Par exemple, le circuit ne contient pas d'additionneur.
La raison est simplement qu'il est plus optimal de faire différemment.
De plus, le circuit génère deux bits de sortie dont l'un par le biais du comparateur directement.
Le circuit de comparaison est donc architecturé d'une certaine manière de sorte à pouvoir généré un bit à $0$ en cas d'échec de la comparaison, et $1$ pour une réussite.
L'autre bit de sortie est généré par un autre circuit (une porte logique $Xor$), qui aggrège la sortie du comparateur avec un autre bit dont il est trop difficile d'expliqué l'origine ici.
Tout ces sujets sont abordés dans la documentation elle même.

## Les différents nombres du standard IEEE-754

Il me faut aussi précisé que le standard IEEE-754 peut représenté plusieurs type de nombre:
  - Les nombres _normaux_ (ceux dont il est le sujet durant tout le document)
  - Les nombres ___dénormaux___
  - Les _NaN_ (Not a Number)
  - L'_infini_ positif ou négatif

Commençons par définir ce qu'est un nombre _normalisé_.
Les nombres _normaux_ ont une valeur d'exposant comprise entre $-\left(2^{\left(N - 1\right)}\right) + 2$ et $2^{\left(N - 1\right)} - 1$ inclus, où $N$ représente le nombre de bits qui compose le champs d'exposant $E$ d'un flottant.
Rappellons que le champs d'exposant d'un flottant _Half precision_ est de $5$ bits, et que le biais de ce dernier est de $2^{N-1} - 1$.
Un nombre _normalisé_ a donc un champs d'exposant biaisé dont la valeur oscille entre $\left(-\left(2^{\left(N - 1\right)}\right) + 2 + biais = 00001_2\right)$ et $\left(2^{\left(N - 1\right)} - 1 + biais = 11110_2\right)$ inclus.
Pour le dire autrement, un nombre _normalisé_ a un champs d'exposant biaisé non nul et strictement inférieur à la valeur maximal encodable sur $N$ bits, c'est à dire $2^N - 1$.
Par ailleurs, le champs de mantisse tronquée peut codé n'importe quel valeur.
Les nombres _normaux_ sont ceux dont je parle implicitement en début de document, ainsi que dans les démonstrations.

Pour le standard, il existe des nombres invalides du nom de ___NaN___ pour _Not a Number_.
Le standard IEEE-754 génère un _NaN_ depuis un calcul considéré comme invalide par le standard lui même, ainsi que par les mathématiques. 
Comme par exemple $\left(\infty - \infty\right)$, ou encore par un calcul dont l'un des opérandes est un _NaN_, l'opération arithmétique $\left(3.5 - NaN\right)$ n'a aucun sens et génère un _NaN_ à son tour.
En bref, l'encodage d'un _NaN_ nécessite que le champs d'exposant biaisé ait la valeur $2^N - 1$ (les $N$ bits de ce dernier sont à $1$), tandis que le champs de mantisse tronquée est de n'importe quel valeur tant qu'il est non nul.
Comme dit plus haut, un nombre _NaN_ ne génère jamais de résultat valide lorsqu'il est utilisé en tant qu'opérande pour un calcul arithmétique quel qu'il soit.
Par conséquent, l'Unité de Configuration de la FPU ne les prends pas en charge.

Le standard IEEE-754 définit aussi un moyen de codé l'infini positif ou négatif $\left(\pm \ \infty\right)$.
Le codage de l'infini $\left(\infty\right)$ nécessite un champs d'exposant biaisé dont la valeur est $2^N - 1$ (la valeur maximal encodable sur $N$ bits), et dont le champs de mantisse tronquée est obligatoirement nul (uniquement composé de bits à $0$).
Seul le bit de signe permet de passé de l'infini positif à négatif $\left(\pm\right)$.
Comme nous avons pu le voir avec l'exemple ci-dessus, un calcul arithmétique avec l'infini comme opérande génère un _NaN_ en résultat.
C'est d'autant plus vrai pour n'importe quel type de calcul arithmétique, comme $\left(3.5 \times \infty\right)$ pour donné un autre exemple.
L'unité de Configuration de la FPU ne prend donc pas en charge les opérandes infini positif ou négatif.

### La plage de codage des nombres dénormaux (Nombre dénormalisé et plage de codage)

Les nombres _dénormaux_ sont les derniers "type" de nombre pouvant être représenté dans un flottant IEEE-754.
Les nombres _dénormalisés_ sont dans un intervalle de valeur se situant entre la plus petite valeur positive codable sur un nombre _normalisé_, ainsi que $\left(\vert \pm 0 \ \vert\right)$.
Un même intervalle de nombre _dénormaux_ cette fois-ci négatif existe également, il suffit simplement de faire passé le bit de signe de $0$ à $1$.

La représentation du nombre positif _normalisé_ le plus petit qu'il soit, utilise la valeur du champs d'exposant la plus petite qu'il soit pour un nombre _normalisé_.
C'est à dire $1$, comme nous l'avons vu plus haut.
Le codage du champs d'exposant est toujours le même qu'importe la taille du flottant (_Half Precision, Simple Precision, etc._), l'ensemble des bits du champs sont à $0$, sauf le _LSB_ qui est à $1$.
Mais attention, la valeur que représente le champs d'exposant biaisé change bel est bien d'une taille de flottant à une autre, car le biais n'est pas le même.

De plus, l'ensemble des bits du champs de mantisse tronquée sont à $0$, ce qui encore une fois ne change pas d'un flottant de taille $X$ à $Y$.
Rappellez vous du chapitre "_Composition du champs d'exposant et de la mantisse tronquée_", où il est expliqué pourquoi est ce que la valeur réel du champs de mantisse tronquée est $\left(1 + Truncated \ Mantissa\right)$.
Le $1$ que l'on additionne à la mantisse tronquée est la valeur du bit de la partie entière du significande, rendu implicite.
Par conséquent, la plus petite valeur positive qui puisse être codé sur un nombre normalisé _Half Precision_ est $\left(\left(1 + 0\right) \times 2^{-14} = 0.00000000000001_2 = 0.00006103515625_{10}\right)$.

Illustration de la représentation de la plus petite valeur (positive comme négative) codable d'un nombre normalisé, pour le champs binaire d'un _Half Precision_ :

$$\left[S_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

$S$ est le bit de signe qui vaut $0$ ou $1$, et qui n'est pas pris en compte par le circuit.

Par ailleurs, il s'avère que la norme IEEE-754 supporte un zéro positif $\left(+0\right)$ ainsi que négatif $\left(-0\right)$, en fonction de la valeur du bit de signe ($0$ ou $1$).
Ceci engendre quelques diffculté de comparaison, par exemple quel résultat générer pour $\left(\left(+0\right) \lt \left(-0\right)\right)$?
En revanche, le circuit n'est pas affecté par cela car rappellons-le il n'utilise que la valeur absolu de ses opérandes flottants, il est donc capable de prendre en charge des opérandes nuls.
Pour le codage d'un zéro positif comme négatif, il est nécessaire que le champs d'exposant biaisé ainsi que celui de mantisse tronquée soient intièrement composés de bits à $0$.
Le bit de signe fait le reste.

Illustration d'un zéro positif comme négatif pour le champs binaire d'un _Half Precision_ :

$$\left[S_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

$S$ est le bit de signe qui vaut $0$ ou $1$, mais comme dit ci-dessus le circuit ne prend pas en compte le bit de signe.

### Le codage des nombres dénormaux

Il y a deux différences entre les nombres _normalisés_ et _dénormalisés_ :
  - Le bit implicite de la mantisse tronquée
  - La considération de la valeur codé par le champs d'exposant

Si la représentation IEEE-754 d'un nombre $F$ est "_normalisé_", alors elle doit respecter les règles de l'écriture scientifique binaire.
La mantisse tronquée doit notamment avoir une valeur réel comprise entre $\left[1;2\right[$, équivalente à la valeur du significande de l'écriture scientifique binaire du nombre $F$.
Il se trouve que nous avons vu dans le chapitre précédent que les nombres _normaux_ ont une mantisse tronquée dont la valeur réel est $\left(1 + Truncated \ Mantissa\right)$.
Ceci veut dire que le bit implicite du champs de mantisse tronquée est donc à $1$ pour tout nombre _normalisé_.
Si cela n'est pas clair pour vous, veuillez relire la section précédente.

Cependant, un nombre dit "_dénormalisé_" ne respecte pas les règles de l'écriture scientifique binaire.
La raison est que le champs de mantisse tronquée a un bit implicite à $0$ plutôt qu'à $1$.
La valeur réel du champs de mantisse tronquée d'un nombre _dénormalisé_ est alors directement celle du champs lui même $\left(0 + Truncated \ Mantissa\right)$.
Par conséquent, la représentation IEEE-754 du nombre ne suit plus les normes de l'écriture scientifique binaire car le champs de mantisse tronquée ne peut pas être dans l'intervalle $\left[1;2\right[$.
C'est pourquoi la représentation est dites _dénormalisé_.

// verif

Pour différencier un nombre _normalisé_ d'un nombre _dénormalisé_, nous nous servons alors du champs d'exposant.
Lorsque le champs d'exposant est nul (l'ensemble de ses bits sont à $0$), alors le nombre codé dans le champs binaire est soit un zéro positif ou négatif $\left(\pm 0\right)$, soit un nombre _dénormalisé_.
Comme nous l'avons vu dans la section précédente, un zéro positif comme négatif est codé avec des champs d'exposant et de mantisse tronquée nuls.
Les nombres _dénormaux_ quant à eux sont codés par un champs d'exposant nul, tandis que le champs de mantisse tronquée est non nul (au moins un bit du champs est à $1$).


// 




Dans n'importe quel format de flottant IEEE-754, un nombre _dénormalisé_ nécessite un champs d'exposant nul (l'ensemble des bits à $0$).
La raison à cela est double.
Pour commencer, cela influe sur la valeur du bit 


Dans la théorie le champs $E$ est nul et il est sensé représenté la valeur d'exposant $\left(0 - biais\right)$, ce qui donne $\left(0 - 15\right)$ dans le cas d'un _Half Precision_.
Cependant, dans la pratique le champs d'ex représente 


























