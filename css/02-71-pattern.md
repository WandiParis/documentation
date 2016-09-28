# 7-1 Pattern

Sass permet de découper le code CSS, tout en ne générant qu'un seul fichier lors
de la compilation. Le [7-1 Pattern](https://sass-guidelin.es/fr/#architecture)
est une architecture générique qui découpe le code Sass en 7 dossiers et un
fichier.

Les dossiers représentent des éléments d'interface, allant du plus générique au
plus spécifique :

* `base` : contient les règles les plus génériques, qui s'appliquent à
l'ensemble du site : styles standard pour les éléments HTML, règles générales de
typographie...
* `layout` : contient les styles définissant les grandes parties du gabarit du
site : grille, header, footer, sidebar, formulaires, menu...
* `components` : contient les styles de composants d'interface : bouton, slider,
modale, tooltip...
* `pages` : contient les styles spécifiques de pages le nécessitant
* `themes` : contient les styles des différents thèmes du site
* `utils` : contient les variables, mixins, fonctions, helpers...
* `vendors` : contient le code CSS d'éventuelles librairies tierces

Le fichier, nommé `global.scss`, se contente uniquement d'importer l'ensemble
des dossiers, dans l'ordre suivant :

* `utils`
* `vendors`
* `base`
* `layout`
* `components`
* `pages`
* `themes`

De cette manière, tous les projets utilisent la même architecture et il est
simple de passer de l'un à l'autre, ou de travailler sur un projet entamé par
quelqu'un d'autre sans être perdu.

---

[Suivant : BEM](/css/03-bem.md)
