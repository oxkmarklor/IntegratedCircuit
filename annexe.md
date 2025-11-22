# Annexe : Approfondissement du standard IEEE-754

Je vous rappelle que nous avons déjà parlé du standard IEEE-754 dans ce document, cependant j'ai expressément survolé beaucoup de choses à ce sujet afin que les chapitres et sections concernés soient le plus lisible possible.
Nous avons, entre autre, pas parlé du fait que les formats définis par le standard IEEE-754 ne codent que des nombres __normaux__ ou __dénormaux__, à quelques exception près.

Dans la théorie, tout nombre codé en IEEE-754 est initialement censé être __normalisé__, les nombres __normaux__ portent une telle appellation car l'écriture scientifique binaire normalise ces derniers.
Cependant, dans la pratique, certains nombres assez spéciaux codés en IEEE-754 peuvent être __dénormalisés__.
Il s'avère que le codage des nombres __dénormaux__ ainsi que l'interprétation qu'ils ont de la valeur de leurs champs d'exposant et de mantisse tronquée, diverge en tout point des bases __normalisées__ sur lesquels reposent les nombres __normaux__.
Ces nombres portent cette appellation car ils ne sont pas __normalisé__ par l'écriture scientifique binaire, au contraire des nombres __normaux__.

Dans les faits, les nombres __normaux__ sont très polyvalents : ils offrent la possibilité de coder des valeurs plus ou moins grandes/petites ainsi que plus ou moins proche de $0$.
Les nombres __dénormaux__, quant à eux, sont beaucoup plus spécifiques car ils ne permettent que le codage de valeurs très proche de $0$, plus proche que ne pourrait l'être n'importe quel nombre __normalisé__.
Pour parvenir à ceci, les nombres __dénormaux__ doivent interpréter la valeur d'un champ de mantisse tronquée différemment de la normale, comme cela a été mentionné dans le paragraphe précédent.

Veuillez remarquer que les nombres __normaux__ comme __dénormaux__ ne peuvent pas coder le chiffre $0$.
Il s'avère que le codage d'une valeur nulle a été prédéfini par le standard IEEE-754 lui-même, mais ce n'est pas la seule valeur à être dans ce cas.
Les __NaN__ ainsi que l'__infini positif__ et l'__infini négatif__ sont des valeurs qui ne peuvent pas non plus être codées par les nombres __normaux__ et __dénormaux__, leurs codages a également été prédéfini par le standard IEEE-754.
Nous parlerons dans le détails de ces excpetions à la fin de cet annexe.

## Les nombres normaux

Rappelons que dans la théorie, tout nombre que code un format IEEE-754 est initialement censé être __normalisé__, cependant dans la pratique nous savons que ces même formats peuvent coder des valeurs exceptionnelles ainsi que des nombres __dénormaux__.
De ce fait, la plage de codage des nombres __normaux__, et plus particulièrement celle du champ d'exposant, en est directement impacté.

En outre, les nombres __normaux__ suivent une approche standard dans l'interprétation de la valeur de leurs champs d'exposant et de mantisse tronquée, nous verrons ci-bas en quoi ces approches sont standard.
N'oubliez pas que l'une des différences majeur entre les nombres __normaux__ et __dénormaux__ est l'interprétation de la valeur de ces champs, cela consiste donc en un sujet important à comprendre.

Nous allons parler de tout cela au travers de deux sections dédiés au champ d'exposant et au champ de mantisse tronquée des nombres __normaux__.

### Plage de codage et interprétation de la valeur du champ d'exposant des nombres normaux

Pour coder un nombre __normalisé__, il faut s'assurer du fait que le champ d'exposant qui compose tout format IEEE-754, d'une taille de $N$ bits, code l'un des entiers de l'intervalle $\left[1;2^N - 1\right[$.
Comme cela a été mentionné en amont, cette plage de codage semble bien restreinte par les nombres __dénormaux__ ainsi que par les quelques valeurs exceptionnelles que peuvent coder ces même formats.

Cherchons désormais à comprendre en quoi les nombres __normaux__ suivent une approche standard dans l'interprétation de la valeur de leur champ d'exposant.
Pour cela, nous allons avoir besoin de revenir sur le chapitre "_Le standard IEEE-754_", et plus particulièrement sur l'une de ses sections qui porte le nom de "_L'encodage par biais du champ d'exposant_".

Dans les grandes lignes, cette section explique ce qu'est l'__encodage par biais__ au travers d'un cas concret, celui du champ d'exposant de tout format défini par le standard IEEE-754.
Rappelez-vous en, nous y apprenons que tout champ d'exposant $E$ code un entier naturel auquel nous soustrayons un biais $B$, qui est un autre entier naturel, afin de calculer la valeur que le champ d'exposant interprète $\left(E - B\right)$.
Il s'avère que les nombres __normaux__ se contentent d'interprèter tel quel la valeur de leur champ d'exposant, ce qui n'est pas le cas des nombres __dénormaux__ mais nous en parlerons plus tard ; d'où le fait qu'il soit dit que les nombres __normaux__ interprètent de façon standard la valeur des champs d'exposant.

### Plage de codage et interprétation de la valeur du champ de mantisse tronquée des nombres normaux

Il s'avère que les nombres __normaux__ se distinguent des autres nombres par la valeur que code leur champ d'exposant, ce qui insinue que la plage de codage du champ mantisse tronquée des nombres __normaux__ n'est pas restreinte.
Pour le dire autrement, un nombre __normalisé__ a un champ de mantisse tronquée qui, dans les limites du possible, peut coder n'importe quel nombre réel de l'intervalle $\left[0;1\right[$.

Cherchons désormais à comprendre en quoi les nombres __normaux__ suivent une approche standard dans l'interprétation de la valeur de leur champ de mantisse tronquée.
Pour cela, nous allons avoir besoin de revenir une nouvelle fois sur le chapitre "_Le standard IEEE-754_", et plus particulièrement sur l'une de ses sections qui porte le nom de "_Le codage du champ de mantisse tronquée_".

Ladite section explique comment est-ce qu'un nombre code et interprète la valeur d'un champ de mantisse tronquée, nombre implicitement considéré comme __normalisé__ car à ce stade du document nous ne parlions pas encore de nombres __normaux__, __dénormaux__ et autres...

Pour résumer, cette section nous apprend que le champ de __mantisse tronquée__ d'un nombre __normalisé__ ne correspond qu'à une partie du __significande__ de l'écriture scientifique binaire de ce même nombre.
Ce qui n'exempte pas le fait de prendre en compte la valeur manquante du __significande__ $\left(1 \times 2^0\right)$, lors de l'évaluation de la valeur du champ de __mantisse tronquée__.
Par conséquent, nous en concluons que la valeur du champ de __mantisse tronquée__ $T$ de tout nombre __normalisé__ est de $\left(1 + T\right)$, avec nous l'aurons compris, le terme $1$ qui représente la valeur de la partie manquante du significande.

Cette interprétation de la valeur du champ de mantisse tronquée permet de représenter la valeur d'un significande, d'où le fait qu'elle soit décrite comme standard.
Nous verrons plus tard que ce n'est pas le cas du champ de mantisse tronquée des nombres __dénormaux__.
Je vous redirige vers la section susmentionné dans le cas où vous souhaiteriez plus de détails.

Clôturons cette section par une précision à propos de la démonstration mathématique.
Remarquez que dans cette dernière, les champs de __mantisse tronquée__ $T_{\alpha}$ ainsi que $T_{\beta}$ ont respectivement une valeur de $\left(1 + T_{\alpha}\right)$ et $\left(1 + T_{\beta}\right)$.
En somme, cela insinue que la démonstration mathématique part du principe que les opérandes $\alpha$ et $\beta$ codent des nombres __normaux__ de format Half Precision.
Cela ne rend pas pour autant invalide et/ou ambigu la démonstration mathématique vis à vis des autres types de nombres que prend en charge le circuit, nous expliquerons pourquoi dans les chapitres et sections dédiés à ces types nombres.

## Les nombres dénormaux

Outre le codage de nombres __normaux__, nous savons que les formats définis par le standard IEEE-754 peuvent également coder des nombres __dénormaux__.
Comme cela a été mentionné en introduction de l'annexe, ces derniers codent des valeurs plus proche de $0$ que ne le permettent les nombres __normaux__ eux même, d'où leur intérêt.
Cependant, pour parvenir à de tel résultat, il faut que les nombres __dénormaux__ interprètent différemment des nombres __normaux__ la valeur que codent les champs de mantisse tronquée composant tout format IEEE-754.

A propos du codage, les nombres __dénormaux__ font usage d'une plage de codage pour le champ d'exposant et pour le champ de mantisse tronquée qui sont toutes les deux restreintes.

En outre, il s'avère que les nombres __dénormaux__ permettent d'avoir une __continuité de codage__ avec les nombres __normaux__.
La __continuité de codage__ est le fait que l'écriture binaire des nombres __dénormaux__ soit contingüe à celle des nombres __normaux__, nous verrons cela plus dans le détails avec les chapitres suivants.
Néanmoins, sachez que nous devons cette __continuité de codage__ au fait que les nombres __dénormaux__ interprètent différemment des nombres __normaux__ la valeur d'un champ d'exposant.

Au travers des deux sections suivantes, nous abordons le sujet du codage ainsi que de l'interprétation de la valeur que code le champ d'exposant et le champ de mantisse tronquée des nombres __dénormaux__.

### Plage de codage et interprétation de la valeur du champ d'exposant des nombres dénormaux

Le codage des nombres __dénormaux__ requière entre autre que le champ d'exposant composant tout format IEEE-754 soit nul.

Par ailleurs, ci-dessus il a été fait mention du fait que : les nombres __dénormaux__ interprètent différemment des nombres __normaux__ la valeur d'un champ d'exposant.

De plus, comme cela a été mentionné ci-dessus : les nombres __dénormaux__ interprètent différemment des nombres __normaux__ la valeur d'un champ d'exposant.
Rappelons que les nombres __normaux__ interprètent la valeur de leur champ d'exposant $E$ suivant le calcul $\left(E - B\right)$, avec le terme $B$ qui représente le biais du champ d'exposant.
Si les nombres __dénormaux__ suivaient la même interprétation, alors le champ d'exposant de ces nombres interpréterait la valeur $\left(0 - B\right)$ ; car je vous rappelle que le champ d'exposant des __dénormaux__ ne peut qu'être nul.

Cependant, pour des raisons que je ne peux fondamentalement expliquer ici, les nombres __dénormaux__ interprètent la valeur de leur champ d'exposant par le calcul suivant $\left(1 - B\right)$, où le terme $B$ représente toujours le biais tout à fait ordinaire d'un champ d'exposant.
Pour le dire autrement, c'est comme si le champ d'exposant des nombres __dénormaux__ interprétait le plus petit exposant pouvant être interprété par le champ d'exposant d'un nombre __normalisé__.

En bref, rappelez-vous du fait que la __continuité de codage__ entre les nombres __normaux__ et __dénormaux__ est à la base de cette interprétation distincte de la valeur du champ d'exposant des nombres __dénormaux__.
Nous verrons dans un chapitre ultérieur en quoi consiste réelement cette __continuité de codage__ et en quoi est-ce que cette interprétation est absolument indispensable.


//

Rappelons que la plage de codage du champs d'exposant des nombres _normaux_ est de $\left[1;\left(2^N - 1\right)\right[$, avec $N$ le nombre de bits du champs.
Le codage d'un nombre _dénormalisé_ se démarque de celui d'un nombre _normalisé_, par le fait que le champs d'exposant doit être obligatoirement nul.

La valeur que représente le champs d'exposant $E$ d'un nombre _normalisé_ se calcul comme suit $\left(E - biais\right)$.
En outre, la valeur qu'interprète le champs d'exposant d'un nombre _dénormalisé_ est la plus petite puissance dont peut faire usage un nombre _normalisé_.
Autrement dit, le champs d'exposant des nombres _dénormaux_ représente la valeur $\left(1 - biais\right)$.
Nous verrons plus bas dans le chapitre du nom de "_La continuité de représentation entre les nombres normaux et dénormaux_", qu'il existe une continuité de codage assez fondamentale entre les nombres _normaux_ et _dénormaux_.
Nous devons cette continuité de codage aux interprétations distinctes que l'ont fait des champs d'exposant des nombres _normaux_ et _dénormaux_.

//

Dans les grandes lignes, cette section explique ce qu'est l'__encodage par biais__ au travers d'un cas concret, celui du champ d'exposant de tout format défini par le standard IEEE-754.
Rappelez-vous en, nous y apprenons que tout champ d'exposant $E$ code un entier naturel auquel nous soustrayons un biais $B$, qui est un autre entier naturel, afin de calculer la valeur que le champ d'exposant interprète $\left(E - B\right)$.
Il s'avère que les nombres __normaux__ se contentent d'interprèter tel quel la valeur de leur champ d'exposant, ce qui n'est pas le cas des nombres __dénormaux__ mais nous en parlerons plus tard ; d'où le fait qu'il soit dit que les nombres __normaux__ interprètent de façon standard la valeur des champs d'exposant.

### Plage de codage et interprétation de la valeur du champ de mantisse tronquée des nombres dénormaux


// la continuité de codage entre normaux et dénormaux

## Comprendre ce qu'est un NaN ainsi que l'infini positif et négatif

Le premier paragraphe du chapitre précédent explique que tout format défini par le standard IEEE-754 peut coder des valeurs spéciales, parmis lesquels figure les __NaN__ ainsi que l'__infini positif__ et l'__infini négatif__.
Cependant, nous savons que le codage de ces valeurs spéciales a lui-même été prédéfini par le standard IEEE-754, alors commençons par voir quel est le codage des valeurs susmentionnés avant de parler de quoique ce soit d'autre à leurs sujets.

Pour coder une valeur __NaN__ ou bien l'__infini positif__ et __négatif__, il faut déjà que le champ d'exposant qui compose tout format IEEE-754, d'une taille de $N$ bits, code le nombre $\left(2^N - 1\right)$.
En outre, il s'avère que le codage de ces valeurs se départage par le biais du champ de mantisse tronquée, la plage de codage de ce champ n'est pas la même d'une valeur à l'autre.
C'est pourquoi nous parlons du codage du champ de mantisse tronquée ainsi que de l'origine de ces valeurs dans les sections dédiés suivantes.

### Codage du champ de mantisse tronquée et origine des NaN

A ce stade, nous savons ce que le codage d'un __NaN__ requière d'un champ d'exposant, mais il faut ajouter à cela le fait que le champ de mantisse tronquée associé, d'une taille de $K$ bits, doive coder un nombre de l'intervalle $\left]0;2^K - 1\right]$.
En bref, le champ de mantisse tronquée doit être non nul.

Par ailleurs, aussi étrange que cela puisse être, le standard IEEE-754 considère que la valeur __NaN__ n'est pas un nombre, d'où le fait que cette dernière ait pour nom l'abréviation de __Not a Number__.
Dans les faits, les valeurs __NaN__ sont générées à la place des résultats des calculs considérés comme invalide par le standard IEEE-754 et/ou par les mathématiques elles même.
Prenons pour exemple le quotient suivant $\left(NaN \div 0\right)$.
Nous savons d'ores et déjà que le calcul est mathématiquement invalide, du fait de la divison par $0$.
De surcroît, la division d'une valeur qui n'est pas un nombre (la valeur __NaN__) donne lieu à un calcul arithmétique ambigu ; ce sont les raisons pour lesquelles ce quotient mène quoiqu'il arrive à la génération d'un __NaN__ en guise de résultat.

Pour conclure cette section, il va me falloir donner quelques détails à propos du FPS Configuration Unit, ce qui m'amène à vous redirigé vers la documentation dédiée au circuit si vous souhaitez plus d'information.
Dans les grandes lignes, sachez que l'une des fonctionnalités du circuit est d'acheminer convenablement les opérandes d'un calcul arithmétique flottant, sur les bonnes entrées d'un soustracteur flottant.
Néanmoins, nous savons que ledit soustracteur flottant génère __NaN__ sur sa sortie lorsque l'un de ses opérandes est un __NaN__, et ce, indépendamment du positionnement des opérandes sur les entrées de l'unité de calcul.
Par conséquent, nous en déduisons que le soustracteur flottant ne peut pas commettre d'erreur de calcul dû au mauvais placement de ses opérandes, lorsqu'un d'entre eux est un __NaN__ ; ce qui justifie le fait que le circuit n'ait pas besoin de prendre en charge les valeurs __NaN__.

Pour la culture général, sachez que le codage d'un __NaN__ peut varier en fonction de la valeur que l'on donne au bit de signe.
Pourtant, rappelons qu'un __NaN__ n'est pas un nombre, ce qui insinue qu'il n'existe pas de __NaN__ positif d'une part et négatif de l'autre.
En outre, l'ensemble des circuits d'un ordinateur interprètent de la même manière deux valeurs __NaN__ aux codages distincts.

### Codage du champ de mantisse tronquée et origine de l'infini positif/négatif

A ce stade, nous savons ce que le codage de l'__infini positif__ et __négatif__ requière d'un champ d'exposant, mais il faut ajouter à cela le fait que le champ de mantisse tronquée associé doive être nul.
En outre, le codage d'un __infini positif__ ou __négatif__ se joue sur la valeur du bit de signe.

Il ne vous surprendra pas d'apprendre que l'__infini positif__ comme __négatif__ ne représentent que des nombres ayant pour point commun le fait d'être trop grand, ou trop petit, pour pouvoir être codé.
Le standard IEEE-754 ne considère pas pour autant l'__infini positif__ ou __négatif__ comme des nombres, ce qui a pour effet de rendre invalide tout calcul arithmétique sur ceux-ci.
Seulement, grâce à la section précédente, nous savons que tout calcul arithmétique invalide génère un résultat __NaN__, et ces calculs là n'y échappent pas.

En définitive, rappelez vous du fait que l'une des fonctionnalités du FPS Configuration Unit est d'acheminer convenablement les opérandes d'un calcul arithmétique flottant, sur les bonnes entrées d'un soustracteur flottant.
Néanmoins, nous savons que ledit soustracteur flottant génère un __NaN__ lorsqu'un de ses opérandes est __infini__, nonobstant le positionnement desdites opérandes.
Par conséquent, nous en déduisons que pour de tels calculs, le soustracteur flottant ne peut pas commettre d'erreur dû au mauvais placement des opérandes ; justifiant ainsi le fait que le FPS Configuration Unit n'ait pas besoin de prendre en charge l'__infini positif/négatif__.

///

# Annexe: Les différents types de nombres du format Half Precision

Depuis le début de ce document, nous avons toujours évoquer le fait que le Floating Point Configuration Unit traite deux opérandes de format Half Precision, mais dans les faits ces propos manquent de précision.
Il s'avère que le format Half Precision, à l'instar de tout les autres formats définis par le standard IEEE-754, peut coder des nombres de ' type ' distincts :

  - des nombres __nomaux__
  - des nombres __dénormaux__
  - des __NaN__
  - l'__infini positif/négatif__
  - ou encore le __zéro positif/négatif__

Chaque type de nombre utilise une plage de codage spécifique du champ d'exposant, et parfois même du champ de mantisse tronquée ; ces dernières seront dévoilés à l'intérieur de chapitres dédiés.
La suite du document explique ce que sont ces différents types de nombres, ainsi que la raison derrière le fait que le circuit ne prenne en charge que certains d'entre eux et pas d'autre.

## Les NaN

L'introduction explique que les formats définis par le standard IEEE-754, tel que le Half Precision, peuvent coder différents types de nombres parmis lesquels figurent les __NaN__.
Commençons par parler de la plage de codage du champ d'exposant et du champ de mantisse tronquée d'un nombre __NaN__, avant de parler de quoique ce soit d'autre.

Les plages de codage des champs d'exposant et de mantisse tronquée d'un nombre __NaN__ sont toutes les deux restreintes. 
Pour qu'un nombre soit considéré comme __NaN__, il faut que son champ d'exposant d'une taille de $N$ bits code la valeur $\left(2^N - 1\right)$, et que son champ de mantisse tronquée soit non nul.

Aussi étrange que cela puisse être, le standard IEEE-754 considère qu'un nombre __NaN__ n'est pas un nombre, d'où le fait que ces derniers aient pour nom l'abréviation de __Not a Number__.
Dans les faits, les nombres __NaN__ sont générés à la place des résultats des calculs considérés comme invalide par le standard IEEE-754 et/ou par les mathématiques elles même.
Prenons pour exemple le quotient suivant $\left(NaN \div 0\right)$.
Nous savons d'ores et déjà que le calcul est mathématiquement invalide, du fait de la divison par $0$.
De surcroît, la division d'un nombre qui n'en est pas un (un nombre __NaN__) donne lieu à un calcul arithmétique ambigu ; ce sont les raisons pour lesquelles ce quotient mène quoiqu'il arrive à la génération d'un nombre __NaN__ en guise de résultat.

Pour conclure ce chapitre, il va me falloir donner quelques détails à propos du Floating Point Configuration Unit, ce qui m'amène à vous redirigé vers la documentation dédiée au circuit si vous souhaitez plus d'information.
Dans les grandes lignes, sachez que l'une des fonctionnalités du circuit est d'acheminer convenablement les opérandes d'un calcul arithmétique flottant, sur les bonnes entrées d'un soustracteur flottant.
Néanmoins, nous savons que ledit soustracteur flottant génère un nombre __NaN__ lorsque l'un de ses opérandes est __NaN__, et ce, indépendamment du positionnement des opérandes sur les entrées de l'unité de calcul.
Par conséquent, nous en déduisons que pour de tels opérations, le soustracteur flottant ne peut pas commettre d'erreur dû au mauvais placement des opérandes ; ce qui justifie le fait que le FPC Unit n'ait pas besoin de prendre en charge les nombres __NaN__.

Pour la culture général, sachez que le codage d'un nombre __NaN__ peut varier en fonction de la valeur que l'on donne au bit de signe.
Pourtant, rappelons qu'un __NaN__ n'est pas un nombre, ce qui insinue qu'il n'existe pas de __NaN__ positif d'une part et négatif de l'autre.
En outre, l'ensemble des circuits d'un ordinateur interprètent de la même manière deux nombres __NaN__ aux codages distincts.

## L'infini positif/négatif

Les formats définis par le standard IEEE-754, tel que le Half Precision, peuvent également coder deux valeurs assez spéciales : l'__infini positif__ ainsi que l'__infini négatif__.
Faisons de même que le chapitre précédent et commençons d'abord par parler de la plage de codage du champ d'exposant ainsi que du champ de mantisse tronquée d'un nombre __infini positif/négatif__, avant d'aborder le reste.

Les plages de codage des champs d'exposant et de mantisse tronquée des nombres __infinis positifs__ et __infinis négatifs__ sont toutes les deux restreintes. 
Pour qu'un nombre soit considéré comme l'__infini positif__ ou __négatif__, il faut que son champ d'exposant d'une taille de $N$ bits code la valeur $\left(2^N - 1\right)$, et que son champ de mantisse tronquée soit nul.
En outre, le codage d'un __infini positif__ ou __négatif__ se joue sur la valeur du bit de signe.

Il ne vous surprendra pas d'apprendre que l'__infini positif__ comme __négatif__ ne représentent que des nombres ayant pour point commun le fait d'être trop grand, ou trop petit, pour pouvoir être codé.
A l'instar des __NaN__, le standard IEEE-754 ne considère pas vraiment l'__infini positif__ ou __négatif__ comme des nombres ; ce qui a pour effet de rendre invalide tout calcul arithmétique sur ceux-ci.
Par conséquent, il s'avère qu'un nombre __NaN__ est généré à la place du résultat de tout calcul ayant pour opérande l'__infini__, et ce, indépendamment du positionnement des opérandes dans le calcul.

Pour finir, rappelez vous du fait que l'une des fonctionnalités du FPC Unit est d'acheminer convenablement les opérandes d'un calcul arithmétique flottant, sur les bonnes entrées d'un soustracteur flottant.
Néanmoins, nous savons que ledit soustracteur flottant génère un nombre __NaN__ lorsque l'un de ses opérandes est __infini__, nonobstant le positionnement des opérandes.
Cela veut dire que dans une telle situation, le soustracteur flottant ne peut pas commettre d'erreur dû au mauvais placement des opérandes ; justifiant ainsi le fait que le Floating Point Configuration Unit n'ait pas besoin de prendre en charge les nombres __infinis positifs/négatifs__.

## Les nombres normaux

Les nombres __normaux__ font partie des ' types ' de nombres que peuvent coder les divers formats définis par le standard IEEE-754.

Plus en amont, ce document aborde déjà le sujet des nombres __normaux__.
Rappelez vous du chapitre "_Le standard IEEE-754_" et de ses sections subséquentes, ces lignes expliquent ce qu'est le format Half Precision du standard IEEE-754


Il s'avère que, plus en amont, ce document abordre le sujet du standard IEEE-754 par le biais du format Half Precision, et plus particulièrement par le biais des nombres __normaux__ que peut coder un Half Precision. 

//

En outre, faisons mention du fait qu'avant la démonstration mathématique se trouve un chapitre du nom de "_Le champ de mantisse tronquée_", suivi de deux sections.
Ces sections expliquent ce qu'est le champ de __mantisse tronquée__ d'un nombre Half Precision, nombre implicitement considéré comme __normalisé__ car à ce stade du document nous ne parlions pas encore de nombres __normaux__, __dénormaux__ et autres...

//

Il s'avère que nous avons déjà longuement parlé des nombres __normaux__ dans les chapitres qui précèdent la démonstration mathématique, mais de manière totalement implicite car à ce stade de la documentation nous ne parlions que des formats du standard IEEE-754, tel que le Half Precision.
En effet, dans les chapitres évoquant le standard IEEE-754 nous expliquons ce qu'est le format Half Precision, et plus précisément ce qu'est un nombre __normalisé__ codé au format Half Precision.
Voyons cela plus en détails dans les sections suivantes.

### La plage de codage du champ d'exposant des nombres normaux



/// nouveau test

## Les nombres normaux et dénormaux

Les nombres __normaux__ et __dénormaux__ sont deux autres types de nombres que peuvent coder les divers formats définis par le standrard IEEE-754.
Sachez que la plage de codage des champs d'exposant de ces nombres sont toutes les deux restreintes mais pas de la même manière, a contrario, les plages de codage des champs de mantisse tronquée de ces même nombres ne le sont pas.

En outre, les nombres __normaux__ interprètent les valeurs des champs d'exposant et de mantisse tronquée différemment des nombres __dénormaux__.

Commençons par le cas des nombres __normaux__.

### Interprétation et plage de codage du champ d'exposant des nombres normaux

Comme dit précédemment, les plages de codage des champs d'exposant des nombres __normaux__ sont restreintes.
Pour qu'un nombre soit considéré comme __normalisé__, il faut que son champ d'exposant d'une taille de $N$ bits code une valeur comprise dans l'intervalle $\left[1;2^N - 1\right[$.

En somme, le chapitre du nom de "_L'encodage par biais du champ d'exposant_" parle entre autre de l'interprétation de la valeur du champ d'exposant d'un nombre Half Precision, nombre implicitement considéré comme __normalisé__ car à ce stade du document nous ne parlions pas encore de nombres __normaux__, __dénormaux__ et autres.


//

En outre, faisons mention du fait qu'avant la démonstration mathématique se trouve un chapitre du nom de "_Le champ de mantisse tronquée_", suivi de deux sections.
Ces sections expliquent ce qu'est le champ de __mantisse tronquée__ d'un nombre Half Precision, nombre implicitement considéré comme __normalisé__ car à ce stade du document nous ne parlions pas encore de nombres __normaux__, __dénormaux__ et autres...

### L'interprétation du champ de mantisse tronquée des nombres normaux

### Interprétation et plage de codage du champ d'exposant des nombres dénormaux

### L'interprétation du champ de mantisse tronquée des nombres dénormaux

////////

## Les nombres normaux

Les nombres __normaux__ et __dénormaux__ sont complémentaires, même si ces appellations laissent croire le contraire.
Ces nombres se différencient par une plage de codage distincte de leurs champs d'exposant, ainsi que par deux interprétations différentes de la valeur que code leurs champs d'exposant et de mantisse tronquée.

Le codage d'un nombre $F$ est défini comme __normalisé__ lorsque la valeur de son champ de mantisse tronquée correspond à la valeur du significande de sa propre écriture scientifique binaire.
Nous l'aurons compris, dans le cas où le nombre $F$ est défini comme __dénormalisé__ alors la valeur de son champ de mantisse tronquée ne correspond pas à celle d'un significande, qui est rappelons-le comprise dans l'intervalle $\left[1;2\right[$.
Je précise que le champ de mantisse tronquée d'un nombre __dénormalisé__ ne peut pas pour autant interpréter n'importe quel valeur, nous verrons cela dans un chapitre dédié aux nombres __dénormaux__.
En bref, un nombre est défini comme __dénormalisé__ parcqu'il n'est justement pas __normalisé__ par les règles de l'écriture scientifique binaire, particulièrement par celles qui portent sur le significande.

Nous reviendrons séparément au sujet 

### Plage de codage et interprétation du champ d'exposant d'un nombre normalisé

Il s'avère qu'il n'y a que la plage de codage du champ d'exposant d'un nombre __normalisé__ qui est restreinte, celle du champ de mantisse tronquée ne l'est pas.
Pour qu'un nombre soit considéré comme __normalisé__, il faut que son champ d'exposant $E$ d'une taille de $N$ bits code une valeur comprise dans l'intervalle $\left[1;2^N - 1\right[$.

En amont de la démonstration mathématique se trouve une section du nom de "_L'encodage par biais du champ d'exposant_"

//

Il s'avère que les champs d'exposant des nombres __normaux__ ont une plage de codage restreinte, de sorte à ce qu'un nombre soit considéré comme __normalisé__ dès qu'un champ d'exposant $E$ d'une taille de $N$ bits code une valeur comprise dans l'intervalle $\left[1;2^N - 1\right[$.
Nous parlons bien de la valeur Binary Unsigned que code le champs d'exposant et non de la valeur que le champ interprète, car l'intervalle ci-dessus ne prend pas en compte la déduction du biais des champs d'exposant.
Si vous souhaitez plus de détails, je vous redirige vers le chapitre "_Le champ d'exposant_" et ses sections subséquentes.


### La valeur du champ de mantisse tronquée d'un nombre normalisé

Pour commencer, précisons que le champ de __mantisse tronquée__ d'un nombre __normalisé__ n'a pas de plage de codage restreinte, au contraire du champ d'__exposant__ de ce même type de nombre.

En outre, faisons mention du fait qu'avant la démonstration mathématique se trouve un chapitre du nom de "_Le champ de mantisse tronquée_", suivi de deux sections.
Ces sections expliquent ce qu'est le champ de __mantisse tronquée__ d'un nombre Half Precision, nombre implicitement considéré comme __normalisé__ car à ce stade du document nous ne parlions pas encore de nombres __normaux__, __dénormaux__ et autres...

Pour résumer, nous y apprenons que la valeur du champ de __mantisse tronquée__ d'un nombre __normalisé__ correspond à la valeur du __significande__ de l'écriture scientifique binaire de ce même nombre.
De plus, à des fins d'optimisation, un champ de __mantisse tronquée__ ne code pas le bit de la partie entière d'un __significande__ (__bit implicite__ de son nom).
Ce qui n'exempte pas le fait de prendre en compte la valeur de ce bit $\left(1 \times 2^0\right)$ lors de l'évaluation de la valeur d'un champ de __mantisse tronquée__.
Par conséquent, nous en concluons que la valeur du champ de __mantisse tronquée__ $T$ d'un nombre __normalisé__ est de $\left(1 + T\right)$, avec nous l'aurons compris, le terme $1$ qui représente la valeur du __bit implicite__.
Seul un __significande__ nul entraine une exceptionnelle dérogation à la règle.

Je vous redirige vers les chapitres susmentionnés dans le cas où vous souhaiteriez plus de détails.

Clôturons cette section par une précision à propos de la démonstration mathématique.
Remarquez que dans cette dernière, les champs de __mantisse tronquée__ $T_{\alpha}$ ainsi que $T_{\beta}$ ont respectivement une valeur de $\left(1 + T_{\alpha}\right)$ et $\left(1 + T_{\beta}\right)$, ce qui insinue que la démonstration mathématique part de l'hypothèse que les opérandes $\alpha$ et $\beta$ codent des nombres __normaux__ de format Half Precision.
Cela ne rend pas pour autant ambigu la démonstration mathématique vis à vis des autres types de nombres, nous expliquerons pourquoi dans les chapitres suivants.

//

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
