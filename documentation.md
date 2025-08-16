
Pendant que j'étudiais l'électronique, plus particulièrement la microarchitecture des ordinateurs, j'ai pu comprendre à quel point l'écart entre 
les algorithmes logiciels et matériels était faible.

Ce document est là pour expliqué le fonctionnement d'un circuit intégré que j'ai moi même conceptualisé après avoir étudié le standard IEEE-754 ainsi 
que les unités de calcul pour les nombres à virgule flottante (FPU) et les unités SIMD.

Je n'ai pas crée ce circuit dans le but qu'il soit nécessairement réaliste dans son implémentation, ni même particulièrement performant, mais seulement 
dans l'idée de rendre plus concret (plus visuel) la complexité des algortihmes qui nous entourent au quotidien.

C'est pourquoi j'ai fait le choix de conceptualisé mon circuit avec des transistors NMOS et PMOS, l'abstraction est limitée au maximum.
Le seul aspect auquel j'ai porté une certaine attention est le nombre de transistor utilisé par le circuit, j'ai essayé de réduire au minimum 
les dépenses en transistor.
C'est la raison derrière certains choix peut être discutable qui ont été fait, comme l'usage de porte logique OU à transmission de deux transistors.

    note: La porte logique OU à transmission de deux transistors est composée d'un PMOS et d'un NMOS.
    Pour commencer, cette porte est à transmission et renvoie un signal moins puissant en sortie qu'en entré, ce qui nuit aux performances.
    De plus, elle a des performances moyennes catastrophique car elle transmet des `0` depuis le transistor PMOS et des `1` depuis le NMOS, ce qui 
    n'est pas optimal.

J'ai mélangé les circuits de technologie CMOS ainsi qu'à transmission de signal, encore et toujours dans le but d'économisé des transistors.

Par ailleurs, je ne suis pas un professionnel, aussi il est possible qu'en dehors de tout compromis que j'ai pu faire et dont j'ai fait mention 
plus haut, le circuit ne soit pas des plus optimal.

Par exemple, après avoir terminé le schéma du circuit je me suis rendu compte qu'il y avait une architecture qui permettait de se passer 
des démultiplexeurs sur les devants des portes logiques NIMPLY.
Cette architecture fait l'économie d'un pourcentage interéssant de transistor (en restant relatif au faible coût du circuit), d'autant plus que 
les gains augmentent croissant avec la taille des entrées du circuit.
Cette seconde architecture est aussi disponible sous la forme d'un diagramme très simplifié dans le dossier du nom de "REPOSITORY_NAME", avec 
quelques explications sur les gains qui ont été faits en passant d'un circuit à l'autre.

Je conseil à tout lecteur d'avoir des bases solides en informatique pour lire la suite du document, car je ne vais pas prendre le temps de tout expliqué 
dans le détail.


i. Survol du standard IEEE-754

IEEE is a consortium of professionals in the field of electronics. 
Its role is to propose standards that are widely respected by major companies in the small world of electronic engineering.

La norme IEEE-754 vise à décrire un encodage universelle des nombres à virgule, peut importe l'architecture ou la microarchitecture d'un ordinateur.
Le standard définit un encodage reprenant les même éléments que l'écriture scientifique binaire, c'est à dire :
    - un signe (encodé du champs sur un bit)
    - un multiplicande (encodage du champs sur plusieurs bits)
    - une mantisse tronquée (encodage du champs sur plusieurs bits)

Définissons rapidement le rôle de chaque champs.

Le champs du bit de signe remplace le signe de l'écriture scientifique binaire d'un nombre, par un bit à `1` pour un `-` et à `0` pour un `+`.

Tandis que le champs de l'exposant utilise un encodage par biais pour enregistré un nombre N représentant la puissance à laquelle élevé la base `2` 
du multiplicande.
L'encodage par biais utilise une valeur constante `B` (le biais) que l'on ajoute à tout nombre `X` encodable en binaire non signé sur un champs 
de `n` bits, la somme `X+B` forme le nombre représenté.

Quant au champs de la mantisse tronquée, elle est équivalente à la mantisse du nombre à encodé (définit par son écriture scientifique binaire) 
avec le masquage du bit de poids le plus fort étant à `1`.
C'est cette opération de masquage qui est à la base du nom de mantisse "tronquée", de plus, c'est une optimisation permettant de gagner un bit 
de précision sur l'encodage de chaque nombre.

Le format IEEE-754 définit plusieurs taille pour l'encodage de nombre à virgule flottante, les principaux encodages sont le simple précision de `32` bits 
et le double précision de `64` bits.
Il existe néanmoins d'autre taille d'encodage comme le half precision de `16` bits.
A ma connaissance, le half precision est majoritairement utilisé dans l'application de réseau de neuronne au sein du vaste domaine qu'est l'intelligence 
artificielle.

Le circuit électronique que j'ai crée ne gère que des flottants half precision, voici à quoi ressemble l'encodage d'un tel flottant:

    field name   |   sign bit    exponent       truncated mantissa
    bit weight   |      15    14 13 12 11 10    9 8 7 6 5 4 3 2 1 0

Je tient à précisé que la valeur du biais du champs d'exposant est de `-15`.


ii. Contextualisation du problème

Nous venons tout juste de voir l'encodage d'un nombre flottant half precision en fin de chapitre précédent.
Désormais, prenons un nombre à virgule écrit en décimale comme `3.75` et convertissons le en nombre à virgule flottante de 16 bits, voici ce que ça donne:

    0 10000 1110000000

                                        2.1 - représentation d'un nombre positif codé en half precision

Cette écriture est très proche de l'écriture scientifique binaire du nombre.
Ce qui importe c'est que le codage du nombre en lui même ne se sert que du champs d'exposant et de la mantisse tronquée, le bit de signe permet 
simplement de savoir si le nombre est positif ou négatif.
Voici la représentation en half precision du nombre `-3.75` cette fois-ci:

    1 10000 1110000000

                                        2.2 - représentation d'un nombre négatif codé en half precision

Nous voyons bien dans les représentations de `3.75` et `-3.75` que le champs d'exposant et de la mantisse tronquée ne changent pas.

En bref, nous avons la valeur absolu du nombre d'un côté avec le champs d'exposant et la mantisse tronquée, et de l'autre le signe.

Due à cela, les unités de calcul flottantes ne pourraient pas fonctionner correctement si le signe des opérandes était pris en compte dans les calculs.
Le bit de signe (lorsqu'il est à `1`) serait perçu par le circuit de calcul comme n'importe quel autre bit d'un certain poids (une certaine puissance 
de `2`), dès lors, le calcul serait faussé.
C'est à cause de cela que les unités de calcul flottantes ne travaillent qu'avec la valeur absolu de chacun de leurs opérandes, le bit de signe 
des opérandes flottants n'est pas pris en compte.

Plus particulèrement, les unités de calcul dont il sera le sujet dans ce document sont des additionneurs et soustracteurs flottants.
Certain calcul d'addition ou de soustraction posent problème.
Nous allons déjà voir lesquels, puis je résumerai les raisons qui font de ces opérations de vrai petit casse tête.
Voici les opérations mise en cause:

(1)- `(+A)+(-B)`
    - Cette somme requiert un opérande positif et négatif, dans les faits l'opérande négatif pose problème car le bit de signe ne peut pas être pris 
    en compte par l'additionneur flottant.
    Ce calcul est équivalent à `(+A)-B` et l'addition se transforme en soustraction, le plus important c'est qu'il n'y a plus d'opérande flottant négatif.
    Il faudra donc un soustracteur flottant capable d'effectuer la soustraction `A-B`.

(2)- `(-A)+(+B)`
    - Cette somme requiert à nouveau un opérande positif et négatif, dans les faits l'opérande négatif pose problème car le bit de signe ne peut pas 
    être pris en compte par l'additionneur flottant.
    Ce calcul est une différence entre deux opérandes `A` et `B`.
    Une autre manière de procédé à ce calcul de différence serait de faire `A-B`, mais il y a alors un problème qui se pose à nous.
    Si la valeur absolu de l'opérande `A` est strictement plus grande que celle `B` alors le résultat de la somme initial doit être négatif, mais avec 
    le calcul de `A-B` le soustracteur flottant générera l'opposé du résultat attendu par l'opération initial.
    Par conséquent si `|A|>|B|` alors le résutat en sortie du soustracteur doit être négatif et le bit de signe à `1`, sinon à `0` car `|B|>=|A|`.

(3)- `(+A)-(+B)`
    - Cette soustraction est assez basique et ne pose pas de soucis de signe au niveau de ses opérandes flottants, ceci ne l'empêchera pas de nous 
    causé des problèmes de débordement dont je parlerai plus loin dans ce chapitre.
    Nous aurons juste besoin d'un circuit soustracteur pour effectué le calcul de `A-B`.

(4)- `(-A)-(-B)`
    - Avec cette soustraction nous retrouvons le problème des deux premières opérations d'addition, le problème des opérandes négatifs.
    Cette soustraction se simplifie en le calcul `(-A)+B` et la solution est alors la même que celle de l'opération numéro (2).

Nous pouvons remarqué que toutes les solutions permettant d'exécuté ces calculs passent inexorablement par un circuit soustracteur.
Maintenant que nous avons pris connaissance de ces opérations et des raisons pour lesquels elles sont problèmatiques.
Je dois admettre qu'il y a un problème dont je n'ai pas encore parlé, le débordement arithmétique, qui est pourtant à prendre en compte pour chacun 
des calculs qui figurent ci-dessus.

Enfaite, il n'y a pas de grande différence entre un circuit additionneur ou soustracteur pour les nombres entier, et pour les nombres à virgule flottante.

Plus particulèrement, concernant les nombres entiers non signé.
Une soustraction `A-B` de deux opérandes entiers codé en binaire non signé, ne fournit un résultat correct que lorsque `A>=B`.
Sinon le résultat devrait être négatif, "devrait", car dans les faits c'est un débordement arithmétique bas qui se produit et le résultat devient
dès lors complètement faux.
Par exemple, procédons au calcul `5-6` avec des opérandes en binaire non signé, cela donne `0b101-0b110 = 0b111` ou pour le dire autrement `5-6 = 7`.
Cela vous semble incohérent je l'espère, car ça l'est.
Ceci est la parfaite illustration de ce que l'on appelle le débordement arithmétique. 

Ce problème est également partagé par les nombres à virgule flottante, voyons pourquoi (sans rentrer dans le détail).
Les unités de calcul flottants perçoivent les valeurs absolus de leurs opérandes (le bit de signe n'est donc pas pris en compte).
Si nous soustrayons l'opérande `B` à l'opérande `A` et que `|B|>|A|`, alors le résultat devrait être négatif, mais le soustracteur ne peut pas généré 
de résultat négatif vu que les opérandes sur ses entrées sont non signés.
Pour déduire le signe du résultat il faudrait déjà avoir accès aux bits de signe des opérandes.
Par conséquent, le soustracteur ne peut fournir que des résultats eux même non signé, sous peine de généré un résultat incorrect.
De plus, un débordement arithmétique bas se produirait pour les même raisons qu'avec des opérandes entiers non signé.

    Le codage des nombres entiers signés en complément à deux se sert des débordements arithmétique pour engendré des résultats valides 
    en sortie des unités de calcul.
    Un exemple est celui de l'addition 1+(-1) = 0b0001 + 0b1111 = 0b10000.
    Dans les faits, le bit à `1` le plus à gauche du résultat sort de la représentation du codage sur 4 bits des opérandes.
    Il n'est pas pris en compte et le résultat ressemble plutôt à 0b0000, c'est à dire 0.

Le circuit intégré que j'ai conceptualisé se veut être une solution à ce problème, nous verrons plus dans le détails son architecture dans le prochain
chapitre.
Attardons nous cependant sur les fonctions du circuit qui le rendent indispensable à l'exécution des quatres opérations flottantes qui précéde.

Traitons déjà du cas des opérations `(+A)+(-B)` et `(+A)-(+B)`.
Le point commun de ces calculs c'est qu'ils se simplifient tous deux en l'opération `A-B`.
Premièrement, nous savons que nous aurons besoin d'un soustracteur flottant pour ces deux calculs, mais ça ce n'est pas nouveau.
Par contre nous savons désormais que le soustracteur flottant ne pourra généré un résultat correct que si il est nulle ou positif, il faut donc 
s'en assuré en comparant la valeur absolu nos opérandes `A` et `B`.

Dans le cas de ces deux opérations, le circuit effectue la comparaison `|B|>|A|`:

- En cas d'échec de la comparaison, le circuit a la certitude que l'opération `A-B` génère un résultat correct car `|A|>=|B|`.
Le circuit fait alors en sorte de positionné l'opérande `A` sur la première entrée du soustracteur et `B` sur la seconde, afin de formé le calcul `A-B` 
et non l'inverse.
De plus, il doit généré le bit de signe manquant au résultat positionné sur la sortie du soustracteur.
Le bit de signe généré doit être un `0` car le résultat attendu par nos deux opérations est alors positif ou nulle.

- En cas de réussite de la comparaison, le circuit a la certitude que l'opération `A-B` génère un résultat incorrect.
Le circuit fait alors en sorte de positionné l'opérande `B` sur la première entrée du soustracteur et `A` sur la seconde, afin de formé le calcul `B-A` 
et de ne pas produire le calcul inverse (ce qui serait fatal).
De plus, il doit généré le bit de signe manquant au résultat positionné sur la sortie du soustracteur.
Le bit de signe généré doit être un `1` car le résultat attendu par nos deux opérations est assurément négatif.

Ce sont les exemples les plus simple.
Passons désormais au traitement que doit apporté le circuit aux deux opérations suivantes que sont `(-A)+(+B)` et `(-A)-(-B)`.
Ces deux opérations ont aussi un point commun, elles se simplifient toutes deux en l'opération `(-A)+B`.
Le problème ici, c'est que nous faisons face à une addition flottante dont le permier des deux opérandes est négatif, ce qui rend l'addition impossible 
car l'opérande négatif sera perçu en valeur absolu par le circuit de calcul.
L'additionneur flottant se retrouverais alors à produire l'opération `A+B` plutôt que `A-B`.
Cependant, remarquons que `(-A)+B` est un calcul de différence entre les deux opérandes flottants `A` et `B`, il est alors possible de remanié 
ce calcul de différence en `A-B`.

Dans le cas des deux opérations `(-A)+(+B)` et `(-A)-(-B)`, le circuit effectue la comparaison `|A|>|B|`:

- En cas d'échec de la comparaison, le circuit ne sait pas si l'opération `A-B` est correct ou non car `|A|<=|B|`.
Le circuit gère néanmoins cette situation d'égalité ou d'infériorité de l'opérande `A` envers `B` de la même manière, en faisant en sorte de positionné 
l'opérande `B` sur la première entrée du soustracteur et `A` sur la seconde, afin de formé le calcul `B-A`.
De plus, il doit généré le bit de signe manquant au résultat positionné sur la sortie du soustracteur.
Le bit de signe généré doit être un `0` car le résultat attendu par nos deux opérations est alors positif ou nulle.

- En cas de réussite de la comparaison, le circuit a la certitude que l'opération `A-B` génère un résultat correct.
Le circuit fait alors en sorte de positionné l'opérande `A` sur la première entrée du soustracteur et `B` sur la seconde, afin de formé le calcul `A-B` 
et non l'inverse.
De plus, il doit généré le bit de signe manquant au résultat positionné sur la sortie du soustracteur.
Le bit de signe généré doit être un `1` car le résultat attendu par nos deux opérations est assurément négatif.

Voilà les fonctions que remplit le circuit intégré que j'ai pensé. 
Pour résumé, il compare le cas d'une supérioté stricte d'un opérande flottant envers l'autre selon l'opération que le soustracteur doit produire.
Puis achemine les opérandes sur les entrées du soustracteur afin de configuré correctement l'opération, et enfin génère un bit de signe, le tout, 
en fonction du résultat de la comparaison et de l'opération à produire.

Par ailleurs, j'ai décidé d'appellé ce circuit "Unité de Configuration de la FPU" en raison du rôle centrale qu'il joue pour la FPU (plus particulèrement 
le soustracteur flottant) lors de l'exécution d'une des quatres opérations flottantes que l'on a vu plus haut.
Il est possible que je face référence à l'Unité de Configuration de la FPU par le terme Unité de Configuration, par la suite.

Voyons désormais comment le circuit est architecturé.


iii. L'architecture du circuit

Le chapitre précédent permet de contextualisé le rôle de l'Unité de Configuration de la FPU.
Nous avons pu comprendre que les fonctions principales du circuit était:
- L'acheminement des opérandes sur les bonnes entrées du soustracteur flottant.
- La génération d'un bit de signe pour le résultat non signé en sortie du soustracteur.

Ces fonctions dépendent cependant elles même d'une comparaison entre les opérandes des opérations dont il est le sujet dans le chapitre précédent.
Le circuit est donc principalement formé d'un comparateur dont nous allons parlé dans ce qui suit.

Commençons par définir les tables de vérité des circuits qui composent le comparateur, nous retrouvons les portes logiques NIMPLY, XOR, AND et OR en 
plus des circuit de Leading Zeroes et de démultiplexeurs.

       NIMPLY                XOR                 OR                  AND
         
    | L | R | Q |       | L | R | Q |       | L | R | Q |       | L | R | Q |
    | 0 | 0 | 0 |       | 0 | 0 | 0 |       | 0 | 0 | 0 |       | 0 | 0 | 0 |
    | 0 | 1 | 0 |       | 0 | 1 | 1 |       | 0 | 1 | 1 |       | 0 | 1 | 0 |
    | 1 | 0 | 1 |       | 1 | 0 | 1 |       | 1 | 0 | 1 |       | 1 | 0 | 0 |
    | 1 | 1 | 0 |       | 1 | 1 | 0 |       | 1 | 1 | 1 |       | 1 | 1 | 1 |

                                        3.1 - tables de vérité des portes logiques utilisées dans le comparateur

Si vous vous demandez ce qu'est une porte logique, imaginez que c'est une fonction mathématique pour laquelle nous fournissons des données en entrée 
et recevons un résultat en sortie.
Les entrées et sorties sont ici des bits (des tensions électrique présentes ou non au sein d'un fil conducteur).

En l'occurence dans l'encart `3.1` les entrées sont notés `L` et `R` pour Left et Right respectivement, tandis que `Q` représente la sortie.

Une table de vérité est un tableau illustrant chaque résultat `Q` obtensible depuis chacun des ensembles d'entré `L` et `R` d'un circuit donné.

Plus fondamentalement, les portes logiques sont enfaite de simple agencement de transistor permettant d'augmenté la complexité de traitement 
d'un système.
Voici une analogie parlante pour les curieux de physique.
Les transistors sont aux circuits électroniques ce que sont les particules élémentaires à la physique, un transistor serait donc l'équivalent 
d'un électron, d'un quark up, ou autre.
Les portes logiques seraient quant à elles des particules composites, c'est à dire des particules formés de plusieurs autres particules plus élémentaire. 
Pour finir, un circuit comme l'Unité de Configuration de la FPU ressemblerait plutôt à un atome, composé d'un noyau de particule composite (des protons 
et des neutrons), ainsi que de particule élémentaire (des électrons). 

Pour en revenir à l'électronique tout de même.
L'Unité de Configuration doit effectué une comparaison de la supériorité stricte d'un opérande flottant envers un deuxième, donc soit `|A|>|B|`, soit 
l'inverse `|B|>|A|`.
Je précise que cette comparaison ne sollicite que la valeur absolu des opérandes flottants, d'où le fait que le circuit n'ait que 15 entrées (le bit 
de signe n'est pas pris en compte).

Chaque bit de poids i des opérandes `A` et `B` seront notés `A[i]` et `B[i]`, nous pouvons remplacé l'indice i par une constante.
En somme, `A[3]` fait référence au bit de poids `3` de l'opérande `A`.

--

Bref, commençons.
Le comparateur est composé de porte logique NIMPLY dont la table de vérité est accessible dans l'encart `3.1`. 
Chaque paire de bits de même poids provenant des opérandes `A` et `B` sont envoyés en entré des portes logique NIMPLY, mais pas n'importe comment car 
l'opération NIMPLY n'est pas commutative.

    note: Je rappelle qu'en mathématique, le principe de commutativité opératoire intervient lorsque l'ordre des opérandes pour une opération 
    n'impacte pas le résultat de l'opération elle même.
    La multiplication fait partie des opérations commutative, faire 2 * 3 ou l'inverse ne change rien au résultat.
    Les opérations logiques peuvent autant être commutative, que ne pas l'être.

La porte NIMPLY vérifie si le bit sur l'entrée `L` est strictement plus grand que celui sur l'entrée `R`, si c'est le cas la sortie `Q` sera à `1` et 
sinon à `0`.
Nous souhaiterions alors positionné nos bits d'opérande en fonction de la comparaison à produire.
Pour `|A|>|B|` nous voudrions que `B[i]` soit acheminé sur `L` et `A[i]` sur `R` ainsi que l'inverse pour la condition `|B|>|A|`.

D'apparence cela peut paraitre contre intuitif, mais nous verrons plus tard pourquoi ce choix a été fait.

Par conséquent, nous avons besoin de deux circuits démultiplexeurs permettant l'acheminement des bits d'opérande sur chacune des entrées d'une porte 
logique NIMPLY.
Pour faire simple et rapide, un démultiplexeur est un circuit électronique de sélection configurable, permettant de connecté une unique entrée 
sur une seule des deux sorties qu'il posséde.
Voici la table de vérité d'un circuit démultiplexeur:

          DEMUX

    | E | P | Q0 | Q1 |
    | 0 | 0 | 0  | .  |
    | 0 | 1 | .  | 0  |
    | 1 | 0 | 1  | .  |
    | 1 | 1 | .  | 1  |

                                        3.2 - table de vérité d'un démultiplexeur 1 vers 2

Le bit d'entré `E` signifie "Entry" et correspond au fameux bit d'entré que le circuit doit copié sur l'une de ses sorties, puis `P` pour "Parameter" 
configure le circuit convenablement.
Pour finir, `Q0` et `Q1` sont les sorties du circuit.
Lorsque l'entrée de paramètrage `P` est à `0` le circuit copie/connecte l'entrée `E` sur la sortie `Q0`, et inversement quand `P` vaut `1` avec `Q1`.
Par ailleurs, nous remarquerons dans l'illustration de la table de vérité du démultiplexeur, encart `3.2`, que la sortie qui ne transmet par le bit 
d'entrée contient un `.` systèmatiquement.
Ce point symbolise ce que l'on appelle "l'état de haute impédance" d'un circuit, ceci ce produit lorsqu'un fil conducteur (ici la sortie du circuit) 
n'est ni relié à la masse (le 0 volt) ni à l'alimentation.

    Dans les faits, l'état de haute impédance ne se manifeste pas toujours selon l'architecture du démultiplexeur qui peut être à transmission de signal 
    ou basé sur du CMOS.

Pour en finir avec les portes logiques NIMPLY et leurs démultiplexeurs, voyons comment l'Unité de Configuration de la FPU s'en sort pour paramétrer 
correctement ces circuits de sélection.
Si nous reprenons les opérations flottantes qui nécessitent l'intervention de l'Unité de Configuration lors de leurs exécutions, nous remarquerons que 
les calculs `(+A)+(-B)` et `(+A)-(+B)` engendrent la comparaison `|B|>|A|`.
Au contraire, les opérations `(-A)+(+B)` et `(-A)-(-B)` engendrent la comparaison `|A|>|B|`.
En étant attentif, nous observons que le bit de signe de l'opérande `A` est à `0` lorsque la comparaison à effectué est `|B|>|A|`, puis à `1` pour 
la comparaison `|A|>|B|`.
L'Unité de Configuration de la FPU paramètre alors chacun de ces démultiplexeurs avec le bit de signe de l'opérande `A`, de sorte à ce que les bits 
d'opérande (`A[i]` et `B[i]`) soient positionnés convenablement sur les bonne entrées des portes logiques NIMPLY.

--

Il y a quelques chose d'important à noté avec la porte logique NIMPLY, cette dernière ne génère un bit à `1` en sortie que si `L>R`.
En revanche, un bit à `0` sur la sortie `Q` peut être due à deux choses:
- Une égalité entre les entrées `L` et `R` (`L=R`).
- Une supériorité stricte de l'entrée `R` sur `L` (`L<R`).
Nous pouvons donc savoir quand est ce qu'un bit d'opérande sur l'entrée `L` est plus grand que celui sur `R`, mais pas l'inverse.
Lorsqu'une porte NIMPLY génère une sortie `Q` à `0`, le bit sortant est alors une sorte de "zéro anonyme" (ce terme reviendra plus bas) car ont ne connait 
pas vraiment la raison pour laquelle il a été généré.

C'est pourquoi le comparateur utilise aussi des portes logiques XOR.
L'opération logique XOR (eXclusive OR) permet de vérifié si deux bits d'opérande sont égaux, ou l'inverse l'un de l'autre.
L'unité de Configuration de la FPU utilise ces portes logiques pour vérifié si un zéro anonyme généré par une porte NIMPLY est due à une égalité entre 
les deux bits d'entré, ou une supériorité de l'entrée `R` sur `L`.
Dans le premier cas, la porte logique XOR en charge de cette même paire de bits générera un bit à `0` sur sa sortie `Q`, mais dans le second  
la sortie `Q` sera à `1.`
Vous pouvez vérifié cela de vous même avec l'illustration de la table de vérité du XOR dans l'encart `3.1`.
De cette manière, l'Unité de Configuration sait quand un zéro anonyme est issu d'une égalité entre deux bits d'opérandes, et par élimination lorsque 
`L<R`.

Veuillez remarqué que l'opération XOR est commutative.
Comme pour les portes logiques NIMPLY, chaque paire de bits de même poids des opérandes `A` et `B` sont envoyés en entré des portes logiques XOR, 
qu'importe l'ordre.

--

Après les portes logiques NIMPLY, se trouve un circuit que j'ai nommé "Leading Zeroes" en lien avec les fonctions CLZ (Count Leading Zeroes) 
des langages de programmation modernes.

Pour expliqué le fonctionnement du circuit de Leading Zeroes ou LZ, il me faut déjà rentré dans le détail de l'encodage binaire non signé dont nous 
faisons usage pour codé nos nombres à virgule flottante.

Le système binaire (ou base `2`) est exclusivement utilisé dans les ordinateurs et autre machine électronique, principalement pour des raisons 
de fiabilité.
Le sysytème décimale (ou base `10`) est quant à lui utilisé par nous les Hommes, mais autrement rien ne change vraiment.
La base binaire et la base `10` sont très similaire, voyons pourquoi.

Avec la base décimale, nous écrivons un nombre comme la somme des produits dont l'un des facteurs est une puissance de `10` et l'autre un chiffre 
de `0` à `9`.
Prenons par exemple le nombre `103` qui est équivalent à `1 * 10^2 + 0 * 10^1 + 3 * 10^0`.
Nous retrouvons exactement la même chose avec la base binaire, nous remplaçons simplement les puissances de `10` par des puissances de `2` et 
les chiffres sont limités à `0` et `1`.
En base `2`, le nombre `103` est égale à `1 * 2^6 + 1 * 2^5 + 0 * 2^4 + 0 * 2^3 + 1 * 2^2 + 1 * 2^1 + 1 * 2^0` ou autrement dit `0b1100111`.
Le nombre `103` tel que nous venons de l'écrire est codé en binaire non signé.

Pour résumer, chaque bit d'un nombre représenté en binaire non signé est un facteur d'une puissance de `2`.
Les puissances de `2` vont de `0` à `N-1` où `N` est le nombre de bits qui compose le nombre.
Il est fréquent de parlé de MSB ou Most Significant Bit, pour faire référence au bit facteur de la plus grande puissance de `2` dans un nombre 
codé en binaire non signé.
Il est possible de précisé le type de bit que l'on cible.
Le MSB à `1` se veut être dans un nombre le bit de poids le plus significatif parmis les bits à `1`.
Le "poids" d'un bit fait référence à la puissance de `2` dont il est le facteur, plus cette puissance est grande et plus le poids l'est également, 
et inversement.
Réciproquement, il existe aussi le terme de LSB ou Least Significant Bit pour faire référence au bit de poids le plus faible d'un nombre.

Voici quelques chose de très important à comprendre pour la suite.
En binaire non signé, tout bit à `1` et de poids `i` sera strictement supérieur à la somme des bits de poids inférieurs à `i`.
Reprenons comme exemple le nombre `103` codé en binaire, c'est à dire `0b1100111`.
Le bit de poids 5 (le sixième bit en partant de la droite) est à `1`, il est alors certains que la somme des bits de poids inférieur est strictement 
inférieur à `1 * 2^5`.
Vous pourriez vous dire que c'est parce qu'il y a des bits à `0` que cela est vrai, mais non, `1 * 2^5` est égale à `32` et reste strictement supérieur 
à `1 * 2^4 + 1 * 2^3 + 1 * 2^2 + 1 * 2^1 + 1 * 2^0` ce qui vaut `31`.

    note: Ceci est vrai pour le binaire non signé mais aussi pour n'importe quel base numérique, la base décimale y compris.
    Par exemple, `1 * 10^2` est égale à `100` et est strictement supérieur à `9 * 10^1 + 9 * 10^0` ce qui vaut `99`.

Maintenant que nous avons toutes les bases, voyons quel est le rôle du circuit Leading Zeroes avant de nous plongé dans son architecture.

Commençons avec un petit rappel.
L'unité de Configuration de la FPU répartit sur les entrées de chacune des portes logiques NIMPLY, des bits de même poids provenant de deux opérandes 
flottants différents l'un de l'autre (`A` et `B`).
Lorsqu'une porte logique NIMPLY a sur sa sortie `Q` un bit à `1`, cela veut dire que son entrée `L` est strictement supérieur à son autre entrée `R`.
Autrement dit, nous savons que le bit de poids `i` de l'entrée `L` est à `1` et celui de l'entrée `R` à `0`.
En reprenant ce que nous venons de voir ci-dessus.
Nous pouvons en conclure que la somme des bits de poids inférieur à `i` de l'opérande positionné sur les entrées `R`, sera toujours strictement inférieur 
à la valeur du bit de poids `i` à `1` de l'opérande sur les entrées `L`.
D'où le fait que l'opérande sur les entrées `L` soit strictement supérieur à celui sur les entrées `R`, dans l'intervalle du bit de poids `0` 
jusqu'au bit de poids `i`.

Si nous prenons désormais le bit de poids le plus fort à `1` de la sortie des portes logiques NIMPLY.
Nous savons alors que l'opérande sur les entrées `L` est strictement supérieur à celui sur les entrées `R`, dans l'intervalle du bit de poids `0` 
jusqu'au MSB à `1`.
Cependant, cela ne veut pas dire que cet opérande est strictement plus grand que l'autre.
La raison en est que le bit de poids le plus fort à `1` en sortie du NIMPLY, n'est pas forcémment lui même le MSB.
A ce stade, seul les bits de poids supérieur au bit de poids le plus fort à `1` peuvent rendre l'opérande sur les entrées `R` strictement plus grand 
que celui sur les entrées `L`.
Mais ces bits sont des `0`, et sont considéré comme des zéros anonyme vu qu'ils sortent des portes NIMPLY.

Mais nous avons une solution contre ce problème, rappellez vous en, les portes logiques XOR.
Rappellons que ces dernières vérifient si les bits d'opérande de même poids sont égaux ou non, en cas d'égalité la sortie sera à `0` et à `1` sinon.

Nous voudrions donc pouvoir vérifié les sorties des portes logiques XOR, mais pas n'importe lesquels.
Seulement les sorties de XOR ayant traité les même bits d'opérande que ceux ayant généré un bit de poids supérieur au MSB à `1` en sortie des portes 
logiques NIMPLY.

Pour cela, l'Unité de Configuration de la FPU utilise un circuit de masquage qui est en réalité un ensemble de porte logique AND.
En gros, un circuit de masquage a deux entrées `E` et `P` ainsi qu'une sortie `Q`.
Une entrée `E` est copiée sur la sortie `Q` si l'entrée `P` est à `1`, dans le cas où l'entrée `P` vaut `0` alors `Q` est invariablement à `0`, 
l'entrée `E` se retrouve masqué.
Je vous laisse vérifié qu'une porte logique AND remplit bien ce rôle en allant observer sa table de vérité dans l'encart `3.1`.

Le circuit de Leading Zeroes a pour but de généré les bits `P` (de paramètrage) de l'ensemble des circuits de masquage.
Mais maintenant que nous savons quel sont les sorties des portes logiques XOR à prendre en compte, nous pouvons voir comment est-ce que le circuit 
fonctionne.

Chaque porte AND a une entrée, disons `L`, qui est connectée à la sortie d'une porte XOR.
La seconde entrée `R` de chacune des portes logiques sera alors celle de paramètrage, elle est directement relié à la sortie du circuit Leading Zeroes.
Notre circuit LZ utilise la sortie des portes logiques NIMPLY comme entré, et commence par inversé absolument tous les bits de poids supérieur 
au bit de poids le plus fort à `1`.
Ces bits à `1` serviront de bit de paramètrage pour les portes logiques XOR adéquat.
Les portes XOR dont nous parlons sont celles qui permetteront de savoir si un zéro anonyme est due à une égalité entre les deux bits d'opérande ou non.
Avec une entrée de paramètrage à `1` le bit en sortie du XOR sera simplement copié sur la sortie du circuit de masquage (la porte logique AND).

Cependant, lorsque le circuit de Leading Zeroes atteint le MSB à `1` de son entré, il masque le bit et renvoie un `0`.
Il fera de même avec tous les bits de poids inférieur, et comprendre pourquoi est simple.
Seuls les bits à `0` (les zéros anonyme) pourraient avoir besoin d'être désambiguïsé par les sorties des portes logiques XOR, ce qui fait que nous 
oublions directement le MSB à `1` et tous les bits à `1` de poids inférieur.
Deuxièmement, que les bits à `0` de poids plus faible que le MSB à `1` soient due à une supériorité de l'entrée `R` sur `L` ou une égalité.
Cela ne change rien au fait que l'opérande sur les entrées `L` est supérieur à celui sur `R`, dans l'intervalle du bit de poids `0` jusqu'au MSB à `1`.
C'est pourquoi nous n'avons pas besoin des sorties de ces portes XOR, et que nous les masquons en générant des bits de paramètrage à `0` pour 
le bit de poids le plus fort à `1` ainsi que tout ceux de poids inférieur.

Pour terminé sur le sujet du circuit de Leading Zeroes, nous allons parlé de son architecture.

Le circuit est composé de plusieurs démultiplexeurs, portes logiques NOT et portes logiques OR.
Vous pouvez retrouvé la table de vérité d'une porte logique OR dans l'encart `3.1`, celle du démultiplexeur à l'encart `3.2`, et voici la table 
de vérité de la porte logique NOT:

       NOT

    | E | Q |
    | 0 | 1 |
    | 1 | 0 |

                                        3.3 - table de vérité d'une porte logique NOT

Cette porte logique ne possède qu'une entrée (`E`) et l'inverse sur sa sortie, c'est ce qu'on appelle une négation binaire, d'où le nom de ce circuit.

Pour commencer, rappellons que le circuit Leading Zeroes utilise la sortie des portes NIMPLY comme d'une entrée.

La sortie `Q` de la porte NIMPLY qui traite les MSB de nos opérandes flottants (les bits `A[14]` et `B[14]` pour des opérandes `A` et `B`) est simplement 
inversé par une porte logique NOT, voyons pourquoi.
Ce bit est celui qui a le poids le plus fort de la sortie des portes NIMPLY, par conséquent:
- Si le bit est à `0`, le circuit doit l'inversé car il est un zéro anonyme de poids supérieur au bit de poids le plus fort à `1`.
- Si le bit est à `1`, le circuit comprend qu'il a atteint le bit de poids le plus fort à `1` de la sortie des portes NIMPLY.
Etant donné que le bit est à `1`, il n'est pas ambïgue et n'a donc pas besoin de la sortie de la porte XOR qui traite les bits d'opérande eux même 
à l'origine du bit sous-jacent.
Le bit de paramètrage de la porte logique AND (circuit de masquage) est donc mis à `0` par l'inversion du bit initialement à `1`.

La sortie de la porte NOT sera aussi utilisée en entré d'une porte logique OR, j'en parlerai plus bas.

Les sorties `Q` des portes logiques NIMPLY restantes sont chacunes connectées à l'entrée `E` d'un circuit démultiplexeur.
Les démultiplexeurs ont deux sorties dont l'une envoie le bit transmis depuis l'entrée `E` sur une porte logique NOT (la sortie `Q1`), tandis que 
l'autre transmet simplement le bit tel quel (sortie `Q0`).
Retrouvez les explications du fonctionnement d'un démultiplexeur en dessous de l'encart `3.2`, si nécessaire.
Il me faut précisé que la sortie de cette porte logique NOT ainsi que la sortie `Q0`, sont elles même reliées l'une à l'autre pour être transmises 
aux autres circuits, comme nous le verrons juste en dessous.

En outre, chaque entrée de paramètrage `P` des démultiplexeurs reçoit le bit sortant d'une porte logique OR.

    La porte logique OR retourne un bit à `1` sur sa sortie `Q` lorsque au moins l'une de ses entrées `L` ou `R` est aussi à `1`.
    Sinon elle renvoie un bit à `0`.
    Je précise que l'opération OR est commutative.

Parce qu'elle est un peu spéciale, nous allons parlé de la porte logique OR du démultiplexeur dont l'entrée `E` est connectée au bit de poids `13`.
La première entrée `L` de la porte logique OR de ce démultiplexeur est le bit de poids `i` (ici `13`) sur l'entrée `E` du démultiplexeur lui même.
Mais c'est la deuxième entrée `R` de la porte OR qui est spéciale.
La seconde entrée `R` est connectée à la sortie de la porte logique NOT qui a pour rôle d'inversé le MSB (le bit de poids `14` sortant des portes NIMPLY).

Les autres démultiplexeurs, ceux dont les bits sur les entrées `E` sont d'un poids `i` strictement inférieur à `13`, ont un cablage légèrement 
différent pour leurs portes logiques OR. 
La première entrée `L` de la porte OR est toujours le bit de poids `i` de l'entrée `E` du démultiplexeur lui même.
Tandis que la seconde entrée `R` est cette fois-ci connectée à la sortie du démultiplexeur précédent (dont l'entrée `E` traite le bit de poids `i+1`).

En bref, nous comprenons déjà pourquoi une porte logique NOT inverse le MSB du résultat des portes NIMPLY.
Maintenant, après avoir vu l'achitecture ainsi que les connexions entre circuit, voyons comment le tout fonctionne de consort.

Je rappelle que le circuit de Leading Zeroes doit:
- Premièrement, inversé tout bit tant que son poids est supérieur au bit de poids le plus fort à `1`.
- Masqué par un bit à `0`, tout bit de poids inférieur ou égale au bit de poids le plus fort à `1`.

Pour commencer, nous savons que le MSB de l'entré du circuit LZ est inversé par une porte logique NOT.
Si le bit de poids `14` (le MSB) est à `0`, alors il est de poids supérieur au MSB à `1` et il doit être inversé.
Cependant si le bit est à `1`, il est d'un poids inférieur ou égale au MSB à `1` (en l'occurence égale), et doit être masqué par un bit à `0`.
Nous observons que la porte logique NOT fait donc bien son travaille.

Ensuite, nous avons les démultiplexeurs.
Le cablage de la porte logique OR de l'un d'entre eux est un peu différent des autres, comme j'ai pu l'expliqué.
Mais rien ne change fondamentalement et les circuits fonctionnent de la même manière, alors voyons comment.

Les explications vont se basé sur les entrées `L` et `R` des portes logiques OR dont la sortie `Q` paramètre chaque circuit démultiplexeur, ceci va 
permettre de mieux comprendre le fonctionnement global du circuit.

Si le bit sur l'entrée `L` (donc l'entrée `E` du démultiplexeur) de la porte OR est à:

- `0`, alors l'algèbre Booléen nous informe que `0 OR R` est égale à `R`.
Cela veut donc dire que la sortie de la porte OR dépendra uniquement du bit présent sur son entrée `R`.
Par conséquent, si le bit de l'entrée `R` est à:

    - `0`, cela signifie que le circuit a déjà rencontré le bit de poids le plus fort à `1`.
    Donc le bit positionné sur l'entrée `E` du démultiplexeur est de poids inférieur au MSB à `1`, il faut le masqué par un bit à `0`.
    Ca tombe bien car la porte logique OR génère une sortie `Q` à `0` (équivalente à `R` comme mentionné ci-dessus), le bit à `0` sur l'entrée `P` 
    connecte alors l'entrée `E` sur la sortie `Q0` du démultiplexeur.
    Rappellons que la sortie `Q0` transmet tel quel le bit sur l'entrée `E`, le bit sortant du démultiplexeur est donc un `0`.

    - `1`, cela signifie que le circuit n'a pas encore traité le bit de poids le plus fort à `1`.
    Donc le bit positionné sur l'entrée `E` du démultiplexeur est de poids supérieur au MSB à `1`, il faut l'inversé.
    Ca tombe bien car la porte logique OR génère une sortie `Q` à `1` (équivalente à `R` comme mentionné ci-dessus), le bit à `1` sur l'entrée `P` 
    connecte alors l'entrée `E` sur la sortie `Q1` du démultiplexeur.
    Rappellons que la sortie `Q1` transmet le bit sur l'entrée `E` à une porte logique NOT, le bit sortant du démultiplexeur est donc l'inverse de `E`, 
    c'est à dire `1`.

- `1`, alors l'algèbre Booléen nous informe que `1 OR R` est systèmatiquement égale à `1`.
Comme nous le savons le bit de poids le plus fort à `1` de toute l'entrée doit être masqué par un bit à `0`, et tout bit de poids inférieur à ce dernier 
doit aussi l'être.
Par conséquent, que le bit sur l'entrée `E` du démultiplexeur soit le MSB à `1` ou un bit à `1` mais de poids plus faible, il faut le masqué 
par un bit à `0`.
Et encore une fois ça tombe bien car la porte logique OR génère un bit à `1` sur sa sortie `Q`, le bit à `1` sur l'entrée `P` connecte alors 
l'entrée `E` sur la sortie `Q1` du démultiplexeur.
La sortie `Q1` transmet le bit sur l'entrée `E` à une porte logique NOT, le bit sortant du démultiplexeur est donc l'inverse de `E`, c'est à dire `0`.
Précisons que la seconde entrée `R` de la porte logique OR ne changera rien à la sortie de la porte logique, mais nous pouvons tout de même en parlé.
Si le bit sur l'entrée `R` de la porte OR est à `0`, alors cela veut dire que le poids du bit à `1` sur l'entrée `E` est strictement inférieur à celui 
du bit de poids le plus fort à `1`.
Au contraire, lorsque cette entrée `R` est à `1`, cela veut dire que le bit à `1` sur `E` est le bit de poids le plus fort à `1`.

Pour résumer la chose.
Les portes logiques OR acheminent l'entrée `E` sur deux sorties possibles, la sortie `Q0` qui transmet simplement le bit sur `E` et `Q1` qui l'inverse 
à l'aide d'une porte logique NOT.
La porte OR est configuré par deux entrées, chaque entrée est une donnée importante du circuit.
L'entrée `R` permet de savoir si le bit de poids le plus fort à `1` a déjà été traité par le circuit ou non.
Tandis que l'entrée `L` informe le circuit du bit en attente sur l'entrée `E` du démultiplexeur.
En somme, si `R` est à `1` le circuit n'a pas encore traité le MSB à `1` et le bit sur l'entrée `E` est à `0`, il doit être inversé pour devenir 
un bit à `1` et être envoyé sur l'entrée `R` de la prochaine porte logique OR.
Jusqu'à que l'entrée `E` soit elle même à `1` et que la porte logique OR ait deux bits à `1` sur ses entrées, ainsi le bit résultant de l'inversion 
sera `0`.
A partir d'ici, le circuit à traité le MSB à `1` et toutes les entrées devront désormais être masquées par des bit à `0`.
Chaque nouvelle porte logique OR aura un bit à `0` sur son entrée `R`, ainsi la sortie `Q` de ces portes sera tributaire du bit sur l'entrée `L`.
Si `L` transmet un bit à `1` la sortie du OR le sera aussi et l'entrée `E` à `1` sera connecté à la sortie `Q1`, qui inverse le bit en `0`.
En revanche, si l'entrée `L` contient un bit à `0` alors la sortie du OR le sera également et le bit à `0` sur l'entrée `E` sera connecté à la sortie 
`Q0` qui transmet simplement le bit.

    Lors de la conceptualisation du circuit j'ai vite compris que cette implémentation n'était pas idéale pour les performances.
    La raison en est que le traitement des sorties des portes NIMPLY est séquentiel.
    Ce circuit de Leading Zeroes commence par traité les bits de poids les plus fort et fini par ceux de poids les plus faible.
    Ceci est due au fait que les portes logiques OR génèrent le bit de paramètrage de leurs démultiplexeurs à partir de deux bits parmis lesquels 
    figure la sortie du démultiplexeur précédent.
    Une solution serait de traité tous les bits de sortie des portes NIMPLY en même temps, mais ce n'est pas ce que j'ai choisi de faire pour 
    deux raisons.
    Premièrement, j'y ait pensé malheureusement un peu trop tard (-_-').
    Deuxièmement, comme je l'ai mentionné en introduction du document, le circuit que j'ai conçu ne se veut pas être le plus rapide possible mais 
    simplement valide dans sa logique de fonctionnement.
    De plus, je cite "Le seul aspect auquel j'ai porté une certaine attention est le nombre de transistor utilisé par le circuit, j'ai essayé 
    de réduire au minimum les dépenses en transistor." et la solution dont je parle nécessite très probablement de beaucoup plus de transistor 
    que l'implémentation que j'ai choisi.

-- 

Il y a un réseau de porte logique OR qui agrége les sorties du circuit de masquage.
Cet ensemble de porte OR est à plusieurs niveau, et chaque étage prend en entrée les sorties du niveau précédent.
Ces portes logiques OR forment la sortie du comparateur, une sortie de un bit, qui est à `1` lors d'une comparaison réussie et à `0` quand elle 
échoue.

Ce qui il y a d'important à comprendre, ce sont les implications d'avoir un (ou plusieurs) bit à `1` en sortie du circuit de masquage.
Je rappelle que le circuit de masquage prend en entrée chaque sortie des portes logiques XOR, qui traitent elles même en entrée les même 
paires de bits d'opérande que les portes logique NIMPLY.
Une paire de bits d'opérande qui en sortie des portes logiques NIMPLY génère un zéro anonyme de poids supérieur au MSB à `1`, contraint le circuit 
Leading Zeroes à faire une copie de la sortie du XOR qui a traité ses même bits d'opérandes, sur la sortie du circuit de masquage.
Cette copie de la sortie du XOR permettra de savoir si les bits d'opérandes sont égaux ou non, si oui alors le bit copié est `0`, autrement le bit 
sera à `1`.
Un bit à `1` n'est pas anodin, car une porte XOR ne génère une sortie de cette valeur que lorsque ses deux entrées sont l'inverse l'une de l'autre.
Sachant que les même entrées que celle du XOR ont générés un zéro anonyme en sortie d'une porte NIMPLY.
Nous savons par élimination que le bit en question de l'opérande positionné sur les entrées `L` des portes NIMPLY, est strictement inférieur à celui 
de l'opérande sur les entrées `R`.
Sinon, la sortie de la porte logique NIMPLY serait à `1`.
En conclusion, l'opérande sur les entrées `R` est strictement plus grand que celui sur les entrées `L` des portes NIMPLY.

N'oublions pas que les démultiplexeurs en amont des portes logiques NIMPLY, permettent d'acheminé convenablement les bits des opérandes sur les bonnes 
entrées, le tout, en fonction de la comparaison à produire.
Pour une condition `|A|>|B|`, les démultiplexeurs acheminent les bits de l'opérande `B` sur les entrées `L` des portes NIMPLY et ceux de l'opérande `A` 
sur les entrées `R`.
Il est vrai que c'est un peu déroutant.
Etant donné qu'une porte NIMPLY verifie si `L>R` nous nous attendrions à ce que pour la condition `|A|>|B|` ce soit les bits de l'opérande `A` qui se 
retrouvent sur les entrées `L` des portes NIMPLY, et ceux de `B` sur les entrées `R`.

Enfaite si nous faisions cela, le bit de sortie du comparateur serait à `1` lors d'un échec de comparaison et à `0` pour une réussite, nous aurions 
alors besoin d'inversé ce dernier avec une porte logique NOT.
Le besoin de cette inversion est simple à comprendre.
Le bit de signe que génère l'Unité de Configuration de la FPU est ni plus ni moins que la sortie du comparateur.
Mais pour que ce bit de signe ait la bonne valeur (en fonction du calcul à produire), il faut impérativement que la sortie du comprateur soit à `1` 
lorsque le test d'une condition est réussie, et `0` quand il ne l'est pas.

Essayons de comprendre pourquoi le circuit peut se passé de cette porte logique NOT.
Les portes logiques NIMPLY génèrent un résultat partiel d'une comparaison entre les opérandes sur les entrées `L` et `R`.
Si la sortie n'est composée que de bits à `0` alors c'est une égalité partiel entre les deux opérandes, si elle contient un ou plusieurs bits à `1` 
c'est une supériorité partiel de l'opérande sur les entrées `L` vis à vis de celui sur les entrées `R`.
La notion de résultat "partiel" traduit simplement le fait que le résultat reste ambigue à cause des zéros anonyme.
Les portes XOR vérifient si les bits de même poids des opérandes sont égaux ou non.
Le circuit de Leading Zeroes configure le circuit de masquage en fonction du résultat partiel fournit par les portes NIMPLY.
Ainsi, le circuit de masquage permet de ne copié sur sa sortie que les bits sortant des portes XOR qui désambiguïse les zéros anonyme problématique 
du résultat partiel sur la sortie des portes NIMPLY.
Le réseau de portes logiques OR agrége simplement les bits de sortie du circuit de masquage pour formé la sortie du comparateur.

Il faut que les portes NIMPLY produisent les comparaisons inverse à celle devant être testé par l'Unité de Configuration de la FPU.
J'en est parlé lors des explications sur le réseau de démultiplexeur qui configure les entrées des portes NIMPLY, en début de chapitre iii.
Si la condition à vérifié par l'Unité de Configuration de la FPU est `|A|>|B|`.
Alors les portes NIMPLY génèrent le résultat partiel de la condition inverse, et les portes XOR serviront à vérifié si les zéros anonyme qui sont 
ambïgu sont due à des égalités, ou si l'un d'entre eux est due à une infériorité d'une entrée `L` vis à vis d'une entrée `R`.
Dans le cas d'une égalité, les sorties des portes XOR qui désambiguïse les zéros anonyme seront à `0`, l'opérande sur les entrées `L` des portes NIMPLY 
est supérieur ou égale à celui sur les entrées `R`.
Le réseau de porte OR agrége les sorties à `0` des XOR en question, et le comparateur renvoie un bit à `0` pour cause d'échec de la comparaison.
Mais dans le second cas, au moins une sortie des portes XOR qui désambiguïse les zéros anonyme est à `1`.
L'opérande sur les entrées `L` des portes NIMPLY est inférieur à celui sur les entrées `R`.
Le réseau de porte OR agrége les sorties des XOR en question avec un (ou plusieurs) bit à `1`, la sortie du comparateur renvoie un bit à `1` car 
la comparaison a réussie.

-- 

Pour finir, n'oublions pas que parmis les fonctionnalités de l'Unité de Configuration de la FPU se trouve la fonction d'ordonnancement des opérandes 
sur les entrées des unités de calcul (soustracteur flottant).
Petit rappel, les opérations flottantes faisant appel à l'Unité de Configuration sont `(+A)+(-B)`, `(+A)-(+B)`, `(-A)+(+B)` et `(-A)-(-B)`.
Chacun de ces calculs s'effectue sous la forme de `A-B` dans un circuit soustracteur flottant, qui ne peut pas généré de résultat négatif rappellons-le.
Vous pouvez lire à nouveau le chapitre ii, et plus particulièrement la fin de celui-ci si vous avec besoin de vous rafraichir la mémoire.

Concrétement, si `|B|>|A|` le résultat du soustracteur sera complètement faux à cause du débordement arithmétique que ce calcul engendre.
Comme expliqué dans la chapitre ii, c'est le rôle de l'Unité de Configuration de la FPU de modifié l'ordre des opérandes afin de rendre le calcul 
viable.
L'acheminement des opérandes flottants sur les entrées du soustracteur est assuré par des circuits démultiplexeurs qui se situent en amont 
de l'unité de calcul.
Ces démultiplexeurs sont paramétrés par l'Unité de Configuration de la FPU elle même, au travers d'un bit de configuration qu'elle génère.

Lorsque le bit de configuration des démultiplexeurs est à `0`, l'opérande `A` est acheminé sur la première entrée du soustracteur et l'opérande `B` 
sur la seconde, le calcul qui s'en suit est `A-B`.
Avec le bit de configuartion à `1`, c'est l'inverse qui se produit et l'opérande `B` se situe sur la première entrée du soustracteur tandis que 
l'opérande `A` sur la seconde. 
Le calcul devient alors `B-A`.

La génération du bit de configuration se fait au travers d'une porte logique XOR, qui se situe après la sortie du comparateur.
Cette porte logique utilise en entrée:
- Le bit de sortie du comparateur lui même.
- Le bit de signe de l'opérande `A`.

Je rappelle que le bit de signe de l'opérande `A` est déjà utilisé par l'Unité de Configuration de la FPU pour paramétrer les démultiplexeurs 
qui précédent les portes logiques NIMPLY du comparateur.
En bref, pour les calculs nécessitant l'intervention de l'Unité de Configuration (ceux qui figurent ci-dessus), le bit de signe de l'opérande `A` 
est systèmatiquement à `0` lorsque le comparateur procéde au calcul de la condition `|B|>|A|`.
Au contraire, ce dernier est toujours à `1` quand le comparateur doit effectué le test de la condition suivante `|A|>|B|`.
Encore une fois, si vous avez besoin de vous rafraichir la mémoire je vous invite à lire le début du chapitre iii.

La porte logique XOR n'utilise pas au hasard ces deux bits en entrée.
L'acheminement des opérandes flottants sur les entrées d'un soustracteur dépend du calcul à produire (donc de la condition à évalué), ainsi que 
du résultat de la comparaison.
Voyons pourquoi la porte XOR est le circuit parfait pour généré le bit de configuration.

- Si le bit en sortie du comparateur et le bit de signe de l'opérande `A` sont tout deux à `0`.
Cela veut dire dans un premier temps que le comparateur effectue le test de la condition référencé par la valeur du bit de signe de l'opérande `A`, 
c'est à dire `|B|>|A|`.
La sortie du comparateur nous informe que la comparaison a échoué.
Donc en valeur absolu l'opérande `A` est supérieur ou égale à `B`, et la soustraction `A-B` engendre un résultat positif ou nulle, donc correct.
Par conséquent le bit de configuration devrait être à `0`, exactement comme la sortie de la porte logique XOR lorsque ces deux bits d'entré sont à `0`.

- Si le bit en sortie du comparateur est à `0` et le bit de signe de l'opérande `A` est à `1`.
Cela veut dire dans un premier temps que le comparateur effectue le test de la condition référencé par la valeur du bit de signe de l'opérande `A`, 
c'est à dire `|A|>|B|`.
La sortie du comparateur nous informe que la comparaison a échoué.
Donc en valeur absolu l'opérande `B` est supérieur ou égale à l'opérande `A`, et la soustraction `A-B` engendre un résultat négatif ou nulle, donc 
potentiellement incorrect.
Par conséquent le bit de configuration devrait être à `1`, pour changé le calcul en `B-A`.
Ce que génère la sortie de la porte logique XOR lorsque ces deux bits d'entré sont l'inverse l'un de l'autre.

- Si le bit en sortie du comparateur est à `1` et le bit de signe de l'opérande `A` est à `0`.
Cela veut dire dans un premier temps que le comparateur effectue le test de la condition référencé par la valeur du bit de signe de l'opérande `A`, 
c'est à dire `|B|>|A|`.
La sortie du comparateur nous informe que la comparaison est un succès.
Donc en valeur absolu l'opérande `B` est strictement supérieur à l'opérande `A`, et la soustraction `A-B` engendre un résultat négatif, donc incorrect.
Par conséquent le bit de configuration devrait être à `1`.
Ce que génère la sortie de la porte logique XOR lorsque ces deux bits d'entré sont l'inverse l'un de l'autre.

- Si le bit en sortie du comparateur et le bit de signe de l'opérande `A` sont tout deux à `1`.
Cela veut dire dans un premier temps que le comparateur effectue le test de la condition référencé par la valeur du bit de signe de l'opérande `A`, 
c'est à dire `|A|>|B|`.
La sortie du comparateur nous informe que la comparaison est un succès.
Donc en valeur absolu l'opérande `A` est strictement supérieur à `B`, et la soustraction `A-B` engendre un résultat positif, donc correct.
Par conséquent le bit de configuration devrait être à `0`, exactement comme la sortie de la porte logique XOR lorsque ces deux bits d'entré sont à `1`.

En bref, inversé de l'ordre des opérandes `A-B` en `B-A` n'est pas problèmatique.
Mais n'oublions pas que c'est le rôle du bit de signe (la sortie du comparateur comme expliqué précédemment), de contraindre le résultat en sortie 
du soustracteur à avoir le bon signe en fonction l'opération initial.
