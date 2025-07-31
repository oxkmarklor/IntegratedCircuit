# Démonstration mathématique

Il y a une documentation dédié au circuit pour comprendre sa raison d'être ainsi que son architecture, je conseil de la lire pour plus d'information.

La tâche principale du circuit est de généré deux bits de sortie, l'un pour configuré un circuit soustracteur de nombre flottant, et l'autre pour la sortie même de cette unité de calcul.
A cette fin, le circuit nommé __FPU Configuration Unit__ reçoit deux opérandes flottants _Half Precision_ $\alpha$ et $\beta$, et doit procédé à une comparaison de la supériorité stricte de l'un de ces opérandes envers l'autre (en valeur absolu).

$$\left(\vert \alpha \vert \gt \vert \beta \vert\right)$$

Nous avons vus plus haut que les champs d'exposant $E$ étaient traités avant les champs de mantisse tronquée $T$, nous allons donc commencer par ceux-ci.

-- -

Voici quelques définitions d'opération importante pour la démonstration.

$$Nimply \ x, \ y \rightarrow \ x \ \wedge \ \overline{y}$$

__Nimply__ est une opération logique qui n'est pas connu en tant qu'opération en soit, mais plus en tant que porte logique dans les circuits intégré.

$$Xor \ x, \ y \rightarrow \ \left(x \ \wedge \ \overline{y}\right) \ \vee \ \left(\overline{x} \ \wedge \ y\right)$$

__Xor__ est une opération logique très connu en tant qu'opération et qui a sa propre porte logique.

Par ailleurs, $\overline{x}$ est l'opération logique __Not__ qui inverse un bit en son opposé $\left(1 = \overline{0}\right) \vee \left(0 = \overline{1}\right)$.
Elle possède également sa propre porte logique, qui est d'ailleurs assez élémentaire à la conception de circuit en tout genre.

-- -

$$\left(1\right) \quad \sum_{\sigma=14}^0 \ \tau_{\sigma} = \left(Nimply \ \beta_{\sigma}, \ \alpha_{\sigma}\right)$$

