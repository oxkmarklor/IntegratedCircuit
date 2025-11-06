# Annexe: Les différents types de nombres du format Half Precision

Depuis le début de ce document, nous avons toujours évoquer le fait que le FPU Configuration Unit traite deux opérandes de format Half Precision, mais dans les faits cela manque de précision.
Il s'avère que le format Half Precision, à l'instar de tout les autres formats définis par le standard IEEE-754, peut coder des nombres de "type" distinct :

  - des nombres __nomaux__
  - des __NaN__
  - l'__infini positif/négatif__
  - ou encore des nombres __dénormaux__

Chaque type de nombre utilise une plage de codage spécifique du champ d'exposant, et parfois même du champ de mantisse tronquée ; ces dernières seront dévoilés à l'intérieur de chapitres dédiés.
La suite du document explique ce que ces différents types de nombres ainsi que la raison derrière le fait que le circuit ne prenne en charge que certains d'entre eux et pas d'autre.

## Les nombres normaux

Les nombres __normaux__ et __dénormaux__ sont complémentaires, même si ces appellations laissent croire le contraire.
Il y a deux différences entre un nombre __normalisé__ et __dénormalisé__ :

  - la plage de codage du champ d'exposant
  - la valeur du champ de mantisse tronquée

Le codage d'un nombre $F$ est défini comme __normalisé__ lorsque la valeur de son champ de mantisse tronquée correspond à la valeur du significande de sa propre écriture scientifique binaire.
Nous l'aurons compris, dans le cas où le nombre $F$ est défini comme __dénormalisé__ alors la valeur de son champ de mantisse tronquée ne correspond pas à celle d'un significande, qui est rappelons-le comprise dans l'intervalle $\left[1;2\right[$.
Je précise que le champ de mantisse tronquée d'un nombre __dénormalisé__ ne peut pas pour autant interpréter n'importe quel valeur, nous verrons cela dans un chapitre dédié aux nombres __dénormaux__.
En bref, un nombre est défini comme __dénormalisé__ parcqu'il n'est justement pas __normalisé__ par les règles de l'écriture scientifique binaire, particulièrement par celles qui portent sur le significande.

Commençons par voir ce que sont les nombres __normaux__ au travers des deux sections suivantes.

### La plage de codage du champ d'exposant d'un nombre normalisé

Il s'avère que les champs d'exposant des nombres __normaux__ ont une plage de codage restreinte, de sorte à ce qu'un nombre soit considérer comme __normalisé__ dès qu'un champ d'exposant $E$ d'une taille de $N$ bits code une valeur comprise dans l'intervalle $\left[1;2^N - 1\right[$.
Nous parlons bien de la valeur Binary Unsigned que code le champs d'exposant et non de la valeur que le champ interprète, car l'intervalle ci-dessus ne prend pas en compte la déduction du biais des champs d'exposant.
Si vous souhaitez plus de détails, je vous redirige vers le chapitre "_Le champ d'exposant_" et ses sections subséquentes.

### La valeur du champ de mantisse tronquée d'un nombre normalisé

Avant la démonstration mathématique se trouve un chapitre du nom de "_Le champ de mantisse tronquée_", suivi de deux sections.
En l'occurrence, ces lignes cherchent à expliquer, de façon implicite, ce qu'est le champ de mantisse tronquée d'un nombre __normalisé__ codé au format Half Precision.
Je vous redirige vers ces lignes si nécessaire.

Pour résumer, nous savons que la valeur du champ de mantisse tronquée $T$ d'un nombre __normalisé__ $\left(1 + T\right)$, correspond à la valeur du significande de l'écriture scientifique binaire de ce même nombre.
Rappelons que le terme $1$, que l'on additionne à la valeur du champs de mantisse tronquée $T$, représente la valeur du bit implicite (le bit de la partie entière d'un significande).
Les chapitres susmentionnés parlent plus profondément de tout cela.



//

Avant la démonstration mathématique se trouve un chapitre du nom de "_Le champ de mantisse tronquée_", suivi de deux sections.
Ces lignes cherchent à expliquer ce qu'est le champ de mantisse tronquée d'un nombre codé au format Half Precision, mais nous comprenons désormais que cela ne veut pas dire grand chose.
Plus précisément, ces lignes expliquent de façon implicite ce qu'est le champ de mantisse tronquée d'un nombre __normalisé__ de format Half Precision.

//

Au sujet des champs de mantisse tronquée, le chapitre "_Le champ de mantisse tronquée_" et les deux sections suivantes parlent implicitement de la façon dont ces champs sont pris en charge par les nombres __normaux__.
Pour résumer, dans un nombre __normalisé__ le champ de mantisse tronquée $T$ interprète la valeur d'un significande par le calcul $\left(1 + T\right)$.
Rappelons que ce calcul est dû au fait que le champs de mantisse tronquée ne code pas directement le bit de la partie entière d'un significande (_bit implicite_ de son nom), le terme $1$ représente alors la valeur de ce bit qui doit malgré tout être prise en compte par le champs.
Une nouvelle fois, je vous redirige vers les chapitres sus-mentionné si vous souhaitez plus de détails.

Mentionnons le fait que durant toute la démonstration mathématique, nous sommes implicitement partis du principe que les champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$ étaient __normalisés__, et par extension les opérandes du circuit $\alpha$ et $\beta$ également.
Il est important de mentionner cela, car nous verrons plus tard que les nombres __dénormaux__ ne prennent pas en charge les champs de mantisse tronquée de la même manière que les nombres __normaux__.
Dans un chapitre dédié aux nombres __dénormaux__, nous verrons pourquoi est-ce que que cela n'invalide pas pour autant la démonstration mathématique.

//

Nous savons que le champs de mantisse tronquée $T$ d'un nombre __normalisé__ correspond au significande de l'écriture scientifique binaire de ce même nombre, ce qui insinue que la valeur de $T \in \left[1;2\right[$.


// le ch "Le champs de mantisse tronquée" fait implicitement allusion aux nombres normaux


//

Il s'avère que les nombres __normaux__ ont une plage de codage restreinte dans le champ d'exposant, de sorte à ce qu'un nombre soit considérer comme __normalisé__ du moment où un champ d'exposant $E$ d'une taille de $N$ bits code une valeur comprise dans l'intervalle $\left[1;2^N - 1\right[$.
Attention, nous ne parlons pas de la valeur qu'interprète le champ d'exposant, car cela impliquerait de déduire un biais à la valeur codée par ledit champ.
Si vous souhaitez plus de détails, je vous redirige vers le chapitre "_Le champ d'exposant_" ainsi que vers les deux sections qui le suivent.
Sachez qu'il n'y a cependant aucune contrainte sur la plage de codage du champ de mantisse tronquée.

Au sujet des champs de mantisse tronquée, le chapitre "_Le champ de mantisse tronquée_" et les deux sections suivantes parlent implicitement de la façon dont ces champs sont pris en charge par les nombres __normaux__.
Pour résumer, dans un nombre __normalisé__ le champ de mantisse tronquée $T$ interprète la valeur d'un significande par le calcul $\left(1 + T\right)$.
Rappelons que ce calcul est dû au fait que le champs de mantisse tronquée ne code pas directement le bit de la partie entière d'un significande (_bit implicite_ de son nom), le terme $1$ représente alors la valeur de ce bit qui doit malgré tout être prise en compte par le champs.
Une nouvelle fois, je vous redirige vers les chapitres sus-mentionné si vous souhaitez plus de détails.

Mentionnons le fait que durant toute la démonstration mathématique, nous sommes implicitement partis du principe que les champs de mantisse tronquée $T_{\alpha}$ ainsi que $T_{\beta}$ étaient __normalisés__, et par extension les opérandes du circuit $\alpha$ et $\beta$ également.
Il est important de mentionner cela, car nous verrons plus tard que les nombres __dénormaux__ ne prennent pas en charge les champs de mantisse tronquée de la même manière que les nombres __normaux__.
Dans un chapitre dédié aux nombres __dénormaux__, nous verrons pourquoi est-ce que que cela n'invalide pas pour autant la démonstration mathématique.

//

Les nombres _normaux_ (ou nombres _normalisés_) utilisent une certaine partie de la plage de codage du champs d'exposant.

N'oublions pas que le champs d'exposant a un encodage quelques peu spéciale, ce qui est le sujet du chapitre "_L'encodage par biais du champs d'exposant_".
En bref, avec l'encodage par biais du champs d'exposant il faut faire la distinction entre le nombre codé dans le champs binaire, et la valeur que représente réelement le champs d'exposant lui même.
Le champs d'exposant $E$ code un nombre en _Binary Unsigned_ et possède un biais dont la valeur se calcul comme $\left(2^{\left(N - 1\right)} - 1\right)$, avec $N$ le nombre de bits qui compose le champs.
Pour obtenir la valeur que représente le champs d'exposant, il faut soustraire le biais du champs au nombre codé dans le champs lui même $\left(E - biais\right)$.

Tout nombre _normalisé_ a un champs d'exposant $E$ dans lequel il peut codé un nombre dont la valeur se situe entre $\left[1;\left(2^N - 1\right)\right[$.
C'est ce que nous appelons _une plage de codage_.
Maintenant que nous connaissons la borne minimal du codage de ce nombre, nous pouvons calculé quel est la plus petite valeur pouvant être représentée par le champs d'exposant avec $\left(1 - \left(2^{\left(N - 1\right)} - 1\right)\right)$.
Nous pouvons aussi faire ceci avec la borne maximal, ce qui donne $\left(\left(2^N - 2\right) - \left(2^{\left(N - 1\right)} - 1\right)\right)$.
Il n'y a pas de plage de codage restreinte pour le champs de mantisse tronquée, les nombres _normaux_ sont identifiés par la seule valeur du champs d'exposant.
Quant au bit de signe, il remplit simplement son rôle en indiquant si le nombre est positif ou négatif (pas de contrainte non plus).

Un nombre est dit "_normalisé_" lorsque dans son codage, le champs de mantisse tronquée est contraint de la même manière que peut l'être un significande de la notation scientifique binaire.
Autrement dit, un nombre est _normalisé_ quand la valeur de son champs de mantisse tronquée oscille entre $\left[1;2\right[$, même si dans les faits ce n'est pas tout à fait le cas.
Le codage d'un nombre _normalisé_ sollicte la notation scientifique binaire de ce nombre.
Le signe $\pm$ de la notation scientifique est convertit en le bit de signe, tandis que le champs d'exposant code la puissance auquel la base $2$ du multiplicande est élevée (dans la mesure du possible car le champs d'exposant doit rappelons-le respecté une plage de codage).
Enfin, le significande binaire est recopié dans le champs de mantisse tronquée.
Un significande est un nombre réel compris dans l'intervalle $\left[1;2\right[$, _sa partie entière est donc toujours composée d'un seul et unique bit à_ $1$.
Pour gagner en précision sur le codage d'un significande, le champs de mantisse tronquée d'un nombre _normalisé_ ne code pas le bit de la partie entière (qui est toujours à $1$), ce bit devient dès lors implicite.
En pratique, le champs de mantisse tronquée $T$ d'un nombre _normalisé_ a une valeur comprise dans l'intervalle $\left[0;1\right[$.
Dans les faits, sa valeur doit être interprétée comme $\left(1 + T\right)$, où le terme $1$ est la valeur du bit rendu implicite.

Comme nous pouvons le remarqué, tout ceci fait échos aux notions des chapitres précédents.

### Les nombres NaN

Pour le standard IEEE-754, il existe des nombres invalides du nom de ___NaN___ (_Not a Number_).
Un nombre _NaN_ est généré par un calcul considéré comme invalide par le standard lui même et/ou par les mathématiques, c'est parfois une sorte d'alternative à la génération d'exception matériel.
Par exemple, le calcul $\left(\left(+ \ \infty\right) \div 0\right)$ génère un _NaN_ pour deux raisons.
Premièrement, divisé par $X$ l'infini positif ne donne pas de résultat concret, de plus si $\left(X = 0\right)$ alors le calcul se retrouve soudainement à être invalide mathématiquement parlant. 
Comme autre exemple, si un calcul se base sur un opérande qui est un _NaN_, alors il génère également _NaN_ en résultat $\left(NaN - 3.5\right) = NaN$.

En bref, le codage d'un _NaN_ recquière que le champs d'exposant code la valeur $\left(2^N - 1\right)$, pour $N$ le nombre de bits qui compose le champs.
Parallèlement, le champs de mantisse tronquée doit être non nul.
Aussi, comme cela a été mentionné ci-dessus, un nombre _NaN_ utilisé en tant qu'opérande d'un calcul arithmétique ne génère jamais de résultat valide.
Donc la FPU Configuration Unit ne prend pas en charge les nombres _NaN_.

### L'infini positif et négatif

Aussi, il se trouve que le standard IEEE-754 définit un moyen de codé un nombre infini positif comme négatif $\left(\pm \ \infty\right)$.
Premièrement, pour le codage de l'infini $\left(\infty\right)$, le champs d'exposant doit codé la valeur est $\left(2^N - 1\right)$, avec $N$ le nombre de bits qui compose le champs.
De plus, il faut que le champs de mantisse tronquée soit nul (composé uniquement de bits à $0$), au contraire d'un nombre _NaN_.
Ce qui permet de différencié efficassement un nombre $\left(\pm \ \infty\right)$, d'un nombre _NaN_.
Ajoutons à cela le bit de signe qui code le signe $\left(\pm\right)$.

Comme nous avons pu le voir avec l'un des exemples de la section précédente "_les nombres NaN_", un calcul arithmétique ayant pour opérande l'infini positif ou négatif génère systèmatiquement un _NaN_ en résultat.
La FPU Configuration Unit ne prend pas non plus en charge les opérandes de valeur infini positif comme négatif.

## La plage de codage des nombres dénormaux

Les nombres _dénormaux_ représentent des nombres concrets, au contraire de l'_infini_ positif/négatif ou encore des nombres _NaN_ que nous avons vu ci-dessus.

Les nombres _dénormaux_ permettent de représenté des nombres très proche de $0$, qui plus est, plus proche de $0$ que ne serait capable de le faire une représentation _normalisé_.
Par conséquent, il existe deux plages de codage des nombres _dénormaux_.
La plage positive qui code des nombres supérieurs à zéro, ainsi que la plage négative des nombres inférieurs à zéro.
La plage de codage des _dénormaux_ positifs se situe entre $0$, et le __plus petit__ nombre positif _normalisé_ pouvant être codé sur n'importe quel format de flottant IEEE-754.
Quant à la plage de codage des _dénormaux_ négatifs, elle va de $0$ au __plus grand__ nombre négatif _normalisé_ pouvant être codé sur ce même format de flottant IEEE-754.

La seule différence dans le codage d'un nombre _dénormalisé_ positif, vis à vis de son opposé (négatif), est la valeur du bit de signe.
Au delà du bit de signe, les plages de codage des nombres _dénormaux_ positif et négatif sont les même.
C'est pourquoi nous ne parlerons ci-bas que des nombres _dénormaux_ positif, cela rendra moins verbeux et complexe les explications suivantes.

Pour commencer, nous allons devoir définir le codage de la borne basse (c'est à dire $0$), ainsi que la borne haute (le plus petit nombre positif _normalisé_) de la plage de codage des _dénormaux_ positifs.
Evidemment, les plages de codage des nombres _dénormaux_ change en fonction du format de flottant IEEE-754.
Donc, les explications ci-dessous ne ciblerons que le format _Half Precision_.

### Le codage du plus petit nombre positif normalisé au format Half Precision (borne haute)

Rappelons que le champs d'exposant $E$ représente une valeur de $\left(E - biais\right)$.
La plage de codage du champs d'exposant d'un nombre _normalisé_ est de $\left[1;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits qui compose le champs.
Quant au biais du champs d'exposant, il se calcul de la manière suivante $\left(2^{\left(N - 1\right)} - 1\right)$.
Le champs d'exposant d'un nombre flottant au format _Half Precision_ est composé de cinq bits $\left(N = 5\right)$.
Par conséquent, la plus petite puissance pouvant être représentée par un tel champs d'exposant est $-14$, ou autrement dit $\left(1 - 15\right)$.
Le plus petit nombre positif _normalisé_ code alors la valeur $1$ sur son champs d'exposant.

Rappelons également que le champs de mantisse tronquée $T$ d'un nombre _normalisé_ a une valeur effective de $\left(1 + T\right)$.
Etant donné que $T \in \left[0;1\right[$, le plus petit nombre positif _normalisé_ a donc un champs de mantisse tronquée nul (composé de dix bits à $0$).

Voici à quoi ressemble le codage du plus petit nombre positif et _normalisé_ pouvant être codé sur un flottant au format _Half Precision_:

$$\left(1\right) \quad \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

### Le codage du zéro positif ainsi que négatif au format Half Precision (borne basse)

Il s'avère que la norme IEEE-754 supporte un zéro positif $\left(+0\right)$ ainsi que négatif $\left(-0\right)$, en fonction de la valeur du bit de signe.
Ceci engendre quelques diffculté pour les comparaisons.
Par exemple, quel résultat générer pour la comparaison suivante $\left(\left(+0\right) = \left(-0\right)\right)$?

En revanche, le circuit n'est pas affecté par cela car rappellons-le, il n'utilise que la valeur absolu de ses opérandes flottants.
Par conséquent, la FPU Configuration Unit est donc capable de prendre en charge des opérandes nuls.
Quant au codage d'un zéro positif comme négatif $\left(\pm 0\right)$, il est nécessaire que le champs d'exposant ainsi que le champs de mantisse tronquée soient nuls.
Le bit de signe est pour sa part fautif du fait de l'existence d'un zéro positif comme négatif, même si ce dernier n'est pas pris en compte par le circuit.

Voici l'illustration du codage d'un zéro positif ou négatif au format _Half Precision_:

$$\left(2\right) \quad \left[S_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

Avec $\left(S = 0\right)$ pour un zéro positif, et $\left(S = 1\right)$ pour un zéro négatif.

## Le codage des nombres dénormaux

Comme dit dans plus haut "_Les nombres dénormaux permettent de représenté des nombres très proche de_ $0$ _, qui plus est, plus proche de_ $0$ _que ne serait capable de le faire une représentation normalisé_".
Ceci est dû a deux différences majeur entre les nombres _normaux_ et _dénormaux_:
  - Le bit implicite du champs de mantisse tronquée
  - L'interprétation que l'on fait de la valeur codé par le champs d'exposant

Nous allons voir que d'une part, le codage du champs d'exposant d'un nombre _dénormalisé_ est à la source d'une interprétation assez spécifique de la valeur qu'il code.
Par ailleurs, nous verrons aussi que la valeur effective du champs de mantisse tronquée des nombres _dénormaux_, est différente de celle des nombres _normaux, sans pour autant que ce soit dû à l'encodage de ce champs.

Avec la section suivante, commençons par parler du codage du champs d'exposant d'un nombre _dénormalisé_, ainsi que de son interprétation.

### Codage et interprétation du champs d'exposant des nombres dénormaux

Rappelons que la plage de codage du champs d'exposant des nombres _normaux_ est de $\left[1;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits du champs.
Le codage d'un nombre _dénormalisé_ se démarque de celui d'un nombre _normalisé_, par le fait que le champs d'exposant doit être obligatoirement nul.
C'est cela qui entraine une interprétation inédite de la valeur que représente le champs d'exposant.

La valeur que représente le champs d'exposant $E$ d'un nombre _normalisé_ se calcul comme suit $\left(E - biais\right)$.
En outre, la valeur qu'interprète le champs d'exposant d'un nombre _dénormalisé_ est la plus petite puissance dont peut faire usage un nombre _normalisé_.
Autrement dit, le champs d'exposant des nombres _dénormaux_ représente la valeur $\left(1 - biais\right)$.
Nous verrons plus bas dans le chapitre du nom de "_La continuité de représentation entre les nombres normaux et dénormaux_", qu'il existe une continuité de codage assez fondamentale entre les nombres _normaux_ et _dénormaux_.
Nous devons cette continuité de codage aux interprétations distinctes que l'ont fait des champs d'exposant des nombres _normaux_ et _dénormaux_.

### Le bit implicite du champs de mantisse tronquée des nombres dénormaux

Comme nous avons pu le voir dans les chapitres précédents, plus particulièrement dans la section "_Les nombres normaux_", le champs de mantisse tronquée $T$ d'un nombre _normalisé_ a une valeur effective de $\left(1 + T\right)$.
Le terme $1$ représente la valeur du bit implicite du champs.
Dans cette section, il est dit je cite "_Le codage d'un nombre normalisé sollicte la notation scientifique binaire de ce nombre._", ce qui n'est absolument pas le cas des nombres _dénormaux_.
Le bit implicite à $1$ du champs de mantisse tronquée des nombres _normaux_, est le bit de la partie entière du significande de la notation scientifique binaire sur laquelle repose le codage des nombres _normaux_.

Rappelons que l'objectif des nombres _dénormaux_ est de pouvoir codé des valeurs plus proche de $0$ que ne sont capable de le faire les nombres _normaux_.
Pour ce faire, le codage d'un nombre _dénormalisé_ ne se repose pas sur la notation scientifique binaire.
Le champs de mantisse tronquée a toujours un bit implicite qui représente la partie entière du nombre interprété par le champs, mais ce dernier est toujours à $0$.
Par conséquent, la valeur effective du champs de mantisse tronquée $T$ d'un nombre _dénormalisé_ est de $\left(0 + T\right)$, et ce champs code une valeur systèmatiquement comprise dans l'intervalle suivante $\left]0;1\right[$.

Plus haut nous avons vu que les nombres _dénormaux_ ont un champs d'exposant nul, permettant ainsi de dissocier le codage de ces derniers vis à vis de celui des nombres _normaux_.
Mais rappelons que dans la section "_Le codage du zéro positif ainsi que négatif au format Half Precision_", nous formalisons le codage des valeurs $\pm \ 0$ qui nécessitent un champs d'exposant et de mantisse tronquée nul.
D'où la nécessité que le champs de mantisse tronquée $T$ des nombres _dénormaux_ soit non nul.
Ceci afin de ne pas confondre le codage d'un nombre _dénormalisé_ avec celui d'un zéro positif ou négatif.

Au terme de ces explications, il est probable que vous ayez une question en tête.
Pourquoi les nombres _dénormaux_ sont capables de codé des nombres plus proche de $0$ que les nombres _normaux_?
En quelque mots, c'est grâce au bit implicite à $0$ du champs de mantisse tronquée des _dénormaux_.
La section suivante démontre formellement ceci. 

### La précision de codage des nombres dénormaux 

Dans une représentation "classique" des nombres _normaux_, le plus petit nombre positif et _normalisé_ qui peut être codé au format _Half Precision_ est $\left(\left(1 + 0.0\right) \times 2^{\left(1 - 15\right)}\right)$ soit $2^{-14}$.
Tout droit tiré de la section "_Le codage du plus petit nombre positif normalisé au format Half Precision_", voici l'illustration du codage du nombre sous-jacent:

$$\left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

Je vais vous demandez un court instant d'imaginer que les nombres _dénormaux_ n'existent pas.
Dans cette hypothèse, la plage de codage du champs d'exposant d'un nombre _normalisé_ pourrait être étendu à $\left[0;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits du champs.
Si nous omettons le conflit que cela poserait avec le codage d'un zéro positif, alors le plus petit nombre positif _normalisé_ serait $\left(\left(1 + 0.0\right) \times 2^{\left(0 - 15\right)}\right)$ soit $2^{-15}$.

Voici l'illustration du codage de cet hypothétique nombre au format _Half Precision_:

$$\left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

Nous voyons clairement que le codage de ce nombre pose un problème car les champs d'exposant et de mantisse tronquée sont tout les deux nuls, à l'instar du codage d'un zéro positif.
Mais tout ceci ne reste qu'une expérience de pensée, alors passons.

Pour conclure, nous pouvons remarqué que $\left(2^{-14} \div 2^{-15}\right) = 2$.
Ce qui veut dire que si les nombres _dénormaux_ n'existaient pas, nous pourrions codé des nombres _normaux_ deux fois plus petit.

Désormais, avec un nombre _dénormalisé_ essayons de codé la valeur $2^{-15}$.
Ce nombre _dénormalisé_ a un champs d'exposant nul, mais conformément à ce qui est dit dans la section "_Codage et interprétation du champs d'exposant des nombres dénormaux_", il faudra interprété la valeur du champs comme étant la plus petite puissance dont peut faire usage un nombre _normalisé_.
C'est à dire $\left(1 - 15\right) = -14$.
Quant au champs de mantisse tronquée, il code la valeur $1000000000_2$. 
Par ailleurs, n'oublions pas que le bit implicite du champs de mantisse tronquée est à $0$, comme l'explique la section "_Le bit implicite du champs de mantisse tronquée des nombres dénormaux_".
Pour finir, la valeur effective du champs de mantisse tronquée est de $\left(0 + 0.1_2\right)$ $=$ $0.5$.

Voici l'illustration du codage du nombre _dénormalisé_ sous-jacent au format _Half Precision_:

$$\left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 1_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

La valeur du nombre est de $\left(\left(0 + 0.1_2\right) \times 2^{\left(1 - 15\right)}\right)$, ou autrement dit $\left(0.5 \times 2^{-14}\right)$ ce qui est égale à $2^{-15}$.
Il est donc possible d'obtenir le même gain de précision avec les nombres _dénormaux_, qu'avec une extension de la plage de codage du champs d'exposant des nombres _normaux_, et même plus!
Dans les faits, pour le plus petit nombre positif _dénormalisé_ au format _Half Precision_, le gain de précision est de $1024$.
Autrement dit, le plus petit nombre positif et _dénormalisé_ au format _Half Precision_ $\left(\left(0 + 0.0000000001\right) \times 2^{-14}\right)$, est $1024$ fois inférieur au plus petit nombre positif _normalisé_ du même format IEEE-754.

Les nombres _dénormaux_ permettent ainsi d'avoir une bien meilleur précision sur le codage des nombres proches de $0$.

## La continuité de représentation entre les nombres normaux et dénormaux

Faisons un très bref rappel de ce que sont les nombres _dénormaux_.
Concrétement, les _dénormaux_ codent des nombres non nul très proche de $0$.
Au format _Half Precision_, les nombres _dénormaux_ positifs sont censés couvrir tout les nombres dont la valeur se situe entre $0$ et le plus petit nombre positif _normalisé_.
Dans les limites de la base binaire, bien évidemment.
Je dit bien censé, car cela n'est vrai que si il y a continuité de codage entre les nombres _normaux_ et _dénormaux_.

Les nombres _normaux_ sont tous consécutifs les uns aux autres (directement, ou indirectement), nous pouvons donc dire qu'il y a continuité dans la représentation des nombres _normaux_.
Il en va de même pour les nombres _dénormaux_ entre eux.
Cependant, ce que nous cherchons à savoir c'est si il y a continuité entre les nombres _normaux_ et _dénormaux_.
Rappelez-vous de ce que dit la section "_Codage et interprétation du champs d'exposant des nombres dénormaux_", au propos de la continuité entre les nombres _normaux_ ainsi que _dénormaux_.
Je cite "_Nous devons cette continuité de codage aux interprétations distinctes que l'ont fait des champs d'exposant des nombres normaux et dénormaux._".

La section suivante va dans un premier temps nous démontrer les problèmes que pose une représentation non continu entre les nombres _normaux_ et _dénormaux_.
Enfin, la dernière section de cet annexe formalisera ce qu'est la continuité de représentation entre les nombres _normaux_ et _dénormaux_.

### Représentation non continu des nombres dénormaux

Commençons par voir les problèmes auquels nous ferions face si le standard IEEE-754 interprétait la valeur du champs d'exposant $E$ d'un nombre _dénormalisé_, comme celle d'un nombre _normalisé_.
C'est à dire, par la soustraction du biais du champs d'exposant à la valeur qu'il code $\left(E - biais\right)$.
Pour cela, nous allons avoir besoin de définir les trois nombres suivants $\alpha$, $\beta$ ainsi que $\tau$.

Nous avons déjà vu l'illustration du plus petit nombre positif _normalisé_ au format _Half Precision_, ce nombre sera connu sous le nom de $\alpha$:

$$\alpha: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 1_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 0_0\right]$$

La valeur du nombre _normalisé_ $\alpha$ est de $\left(\left(1 + 0.0\right) \times 2^{\left(1 - 15\right)}\right)$ ce qui donne $0.00006103515625$.

En prime, voici l'illustration du plus grand nombre positif _dénormalisé_ pouvant être codé au format _Half Precision_, ce nombre sera connu sous le nom de $\beta$:

$$\beta: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 1_9, \ 1_8, \ 1_7, \ 1_6, \ 1_5, \ 1_4, \ 1_3, \ 1_2, \ 1_1, \ 1_0\right]$$

Rappelons que nous allons calculons la puissance que représente le champs d'exposant $E$ des nombres _dénormaux_ avec $\left(E - biais\right)$, comme nous le ferions pour n'importe quel nombre _normalisé_.
Par conséquent, la valeur du nombre _dénormalisé_ $\beta$ est de $\left(\left(0 + 0.9990234375\right) \times 2^{\left(0 - 15\right)}\right)$ ce qui donne $0.0000304877758026123046875$.

Enfin, voici l'illustration du plus petit nombre positif _dénormalisé_ pouvant être codé au format _Half Precision_, ce nombre sera connu sous le nom de $\tau$:

$$\tau: \ \left[0_{15}, \quad 0_{14}, \ 0_{13}, \ 0_{12}, \ 0_{11}, \ 0_{10}, \quad 0_9, \ 0_8, \ 0_7, \ 0_6, \ 0_5, \ 0_4, \ 0_3, \ 0_2, \ 0_1, \ 1_0\right]$$

Par définition, $\tau$ est le plus petit des nombres positif non nul pouvant être codé au format _Half Precision_, c'est le nombre le plus proche de $0$ toute représentation confondu.
Ce nombre, au même titre que $\beta$, est _dénormalisé_.
D'où le fait que nous calculons la puissance que représente le champs d'exposant $E$ de $\tau$ avec $\left(E - biais\right)$, comme nous avons pu le faire pour $\beta$.
La valeur du nombre $\tau$ est de $\left(\left(0 + 0.0009765625\right) \times 2^{\left(0 - 15\right)}\right)$ ce qui donne $0.0000000298023223876953125$.

Pour commencer, remarquons que le ratio suivant $\left(\alpha \div \beta\right)$ a une valeur qui vaut environ $2.0019$.
Voici l'une des phrases d'introduction du chapitre "_La continuité de représentation entre les nombres normaux et dénormaux_", je cite " $\left[..\right]$ _les nombres dénormaux positifs sont censés couvrir tout les nombres dont la valeur se situe entre_ $0$ _et le plus petit nombre positif normalisé._".
Dans les faits, étant donné que le plus grand nombre positif _dénormalisé_ $\beta$ est deux fois inférieur au plus petit nombre positif _normalisé_ $\alpha$, il y a alors une partie des nombres compris dans l'intervalle $\left]0;\alpha\right[$ qui ne sont pas représentable.
Pour cause, tout les nombres qui se situent dans cet intervalle $\left]\beta;\alpha\right[$, sont soit trop grands pour être représenté par des nombres _dénormaux_, soit trop petit pour l'être par des nombres _normaux_.
Parmis tout les nombres de l'intervalle $\left]0;\alpha\right[$, nous pouvons savoir qu'il y a $\left(\beta \div \alpha\right)$ c'est à dire $\approx 49.9$% d'entre eux qui ne peuvent pas être représentés.

Prenons pour exemple le résultat du calcul $\left(\alpha - \tau\right)$, aussi proche que le résultat puisse être de $\alpha$ il n'est pour autant pas représentable.
Nous venons tout juste de voir pourquoi.
Le résultat de ce calcul est $0,0000610053539276123046875$, ce qui est très légèrement inférieur au nombre $\alpha$.
Par conséquent, ce résultat se situe dans l'intervalle $\left]\beta;\alpha\right[$, d'où le fait qu'il ne puisse être ni représenté par un nombre _normalisé_, ni par un nombre _dénormalisé_.
Le résultat de ce calcul fait donc partie d'une sorte de "trou" de valeur non représentable.

Dans les faits, les calculs sur les flottants occasionnent souvent des arrondis, du fait de l'incapacité de toute base numérique à pouvoir représenté certain nombre de manière fini.
Prenez l'exemple de $\left(1 \div 3\right)$ en base décimale, ou encore $\left(1 \div 10\right)$ en binaire pour vous en convaincre.
Le standard IEEE-754 défini des méthodes d'arrondissement que nous ne détaillerons pas dans ce document.
Cependant, le résultat du calcul $\left(\alpha - \tau\right)$ est si proche de la valeur de $\alpha$, qu'il pourrait être possible d'utiliser une méthode d'arrondi par le haut pour obtenir un résultat dont la précision conviendrai à certains cas d'usage.
Malgré tout cela n'est pas toujours possible, ça ne l'est pas pour un nombre comme $0.000045$ par exemple.
Ce nombre est pour ainsi dire aussi lointain du plus grand nombre positif _dénormalisé_ $\beta$, que du plus petit nombre positif _normalisé_ $\alpha$.
N'importe quel méthode d'arrondi transformerait le nombre jusqu'à le rendre trop approximatif pour presque tout les cas d'usage.

Voilà en quoi l'absence de continuité entre la représentation des nombres _normaux_ et _dénormaux_ est un problème sérieux.

### Représentation continu des nombres dénormaux

Nous avons vu dans le chapitre "_Codage et interprétation du champs d'exposant des nombres dénormaux_", comment est ce que le champs d'exposant d'un nombre _dénormalisé_ devait être interprété.
En bref, le champs d'exposant d'un nombre _dénormalisé_ est nul, mais la valeur que représente ce champs est celle de la plus petite puissance dont peut faire usage un nombre _normalisé_.
Le champs d'exposant interprète alors la valeur $\left(1 - biais\right)$.

Dans ce qui suit, nous allons réutiliser les trois nombres $\alpha$, $\beta$ et $\tau$ de la section précédente.
Dans un premier temps, nous allons redéfinir la valeur des nombres _dénormaux_ $\beta$ et $\tau$, avec cette fois-ci une interprétation correcte de leurs champs d'exposant:

  - Le codage de $\beta$ reste le même et une illustration de ce dernier est présent dans la section précédente.
    Quant à sa valeur, le nombre _dénormalisé_ $\beta$ est de $\left(\left(0 + 0.9990234375\right) \times 2^{\left(1 - 15\right)}\right)$ ce qui donne $0.000060975551605224609375$.

  - Il en va de même du codage de $\tau$ qui reste le même, une illustration de ce dernier est aussi présente dans la section précédente.
    Quant à sa valeur, le nombre _dénormalisé_ $\tau$ est de $\left(\left(0 + 0.0009765625\right) \times 2^{\left(1 - 15\right)}\right)$ ce qui donne $0.000000059604644775390625$.

  - Je rappelle simple que le nombre _normalisé_ $\alpha$ ne voit ni son codage ni sa valeur changer, sa valeur est de $0.00006103515625$.

Reprenons l'exemple du calcul $\left(\alpha - \tau\right)$ de la section précédente, seulement, cette fois-ci le résultat du calcul est $\beta$. 
Rappelons-nous du fait que $\tau$ est le plus petit nombre positif _dénormalisé_ pouvant être codé au format _Half Precision_.
Obtenir le plus grand nombre positif _dénormalisé_ $\beta$ en guise de résultat de la soustraction de $\tau$ au plus petit nombre positif _normalisé_ $\alpha$, démontre qu'il y a continuité dans le codage des nombres _normaux_ ainsi que _dénormaux_.

Pour dire les choses autrement.
La valeur effective du champs de mantisse tronquée $T$ des nombres $\alpha$, $\beta$ et $\tau$, est multiplié par $2^{\left(1 - 15\right)}$.
Par conséquent si $\left(\alpha - \tau\right) = \beta$, il faut en conclure que $\left(T_{\alpha} - T_{\tau}\right) = T_{\beta}$ ce qui donne $\left(1.0000000000 - 0.0000000001\right) = 0.1111111111$.
Il devient soudainement plus simple d'observer la continuité de codage entre le plus petit nombre positif _normalisé_ $\alpha$, et le plus grand nombre positif _dénormalisé_ $\beta$.
Au format _Half Precision_, les nombres $\alpha$ et $\beta$ sont donc consécutifs.
Il y a alors continuité de représentation entre les nombres _normaux_ et _dénormaux_, ce qui évite une partie des soucis dont il a été le sujet en fin de section précédente.

Je dit bien une partie seulement, car des méthodes d'arrondis sont toujours nécessaire dans des cas comme la convertion de certains nombres de la base décimale vers la base binaire, ou autre.
Ce qui peut nuire à la précision de représentation des nombres.
Cependant, il n'est pas question de faire quoi que ce soit contre ce manque de précision, car ce dernier est inhérent à la taille du format de flottant IEEE-754 ayant été choisit.
Pour plus de précision, il faut passer a un format de flottant IEEE-754 plus grand.

Enfin, comme nous avons pu le voir durant tout ce document, la FPU Configuration Unit prend en charge les nombres _normaux_.
Mais elle gère aussi les nombres _dénormaux_, car ces derniers peuvent être utilisés dans des soustracteurs flottants que le circuit prend en charge.
Lorsque le comparateur traite deux nombres _dénormaux_, le circuit n'atteint jamais de point terminal car les champs d'exposant des deux opérandes sont nuls. 
D'autre part, le codage des champs de mantisse tronquée des nombres _dénormaux_ suit celui des nombres _normaux_.
Ces derniers subissent alors le même traitement, qui plus est, par l'intermédiaire des même circuits électroniques.

Mais comme nous l'avons vu avec l'exemple du calcul $\left(\alpha - \tau\right)$ de cette section, et de la précédente.
Un soustracteur flottant est tout à fait capable de traité un opérande _normalisé_ ainsi que _dénormalisé_, provenant d'un seul et unique calcul arithmétique.
Cela tombe bien car la FPU Configuration Unit gère la comparaison entre deux nombres d'une part _normalisé_, et de l'autre _dénormalisé_.
Dans une telle situation, le circuit atteint systèmatiquement un état de point terminal.
La raison en est que le champs d'exposant de l'opérande _normalisé_ a une plage de codage qui est rappelons-le de $\left[1;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits du champs.
Tandis que l'opérande _dénormalisé_ a un champs d'exposant nul.
