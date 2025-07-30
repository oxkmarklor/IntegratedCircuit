# Démonstration mathématique

Deux opérandes __Half Precision__ $\alpha$ et $\beta$ de $16$ bits, pour une comparaison de stricte supériorité de l'un envers l'autre.

Encodage d'un nombre _Half Precision_:

$$\left[\left(S_{15}\right), \ \left(E_{14}, \ E_{13}, \ E_{12}, \ E_{11}, \ E_{10}\right), \ \left(T_9, \ T_8, \ T_7, \ T_6, \ T_5, \ T_4, \ T_3, \ T_2, \ T_1, \ T_0\right)\right]$$

$S =$ ___Sign___,
$E =$ ___Exponent___,
$T =$ ___Truncated Mantissa___

Voici une opération logique dont nous allons avoir besoin $Nimply\left(x, \ y\right) \rightarrow x \wedge Not(y)$

__(1)__
