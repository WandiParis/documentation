# Bonnes pratiques Wandi

Ce document a pour but de recenser l'ensemble des processus et bonnes pratiques
propres à Wandi.

## Général

### Gulp


### SVN



## CSS

TL;DR : Le code CSS utilise le préprocesseur [Sass](http://sass-lang.com/),
l'architecture [7-1 Pattern](https://sass-guidelin.es/fr/#architecture), la
méthodologie [BEM](http://putaindecode.io/fr/articles/css/bem/) et est analysé
par [StyleLint](http://stylelint.io/).

### Sass

[Sass](http://sass-lang.com/) est un préprocesseur CSS. Il apporte des éléments
de langages qui n'existent pas dans CSS, tels que des variables, des structures
conditionnelles, des boucles, des fonctions...

La syntaxe utilisée est le [SCSS](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#syntax),
qui n'est pas interprété par les navigateurs. Le code est donc compilé vers du
CSS pur pour pouvoir être interprété par les navigateurs.

### 7-1 Pattern

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


### BEM

[BEM](http://putaindecode.io/fr/articles/css/bem/) (pour Block Element Modifier)
est une méthode de nommage des classes CSS. Le principal but de cette méthode
est d'obtenir des sélecteurs léger, ne mettant en jeu qu'une ou deux classes.

Cette méthode part du principe qu'un élément du DOM est soit un `Block`, soit un
`Element`, et que chacun de ces deux types d'éléments peuvent recevoir un ou
plusieurs `Modifier`s.

Un `Block` est le noeud racine d'un composant d'interface. Un `Element` est un
noeud au sein d'un `Block`. Enfin, un `Modifier` est une classe qui vient
surcharger les styles de base d'un `Block` ou d'un `Element` afin d'y apporter
une subtilité.

Une syntaxe bien précise doit être utilisée afin de distinguer facilement un
`Block`, un `Element` ou un `Modifier`. Chez Wandi, nous avons choisit d'établir
les règles suivantes :

* un `Block` s'écrit en **UpperCamelCase**
* un `Element` s'écrit en **camelCase**
* un `Modifier` s'écrit en **camelCase**
* le `Block` et l'`Element` sont séparés par un `_` (underscore)
* le `Block`/`Element` et le `Modifier` sont séparés par un `-` (tiret)

Exemple : un composant de Slider

```html
<!-- Block Slider -->
<div class="Slider">
    <!-- Element Slider_slide -->
    <div class="Slider_slide">
        <p>Slide 1 content</p>
    </div>
    <div class="Slider_slide">
        <p>Slide 2 content</p>
    </div>
    <!-- Element Slider_controls -->
    <div class="Slider_controls">
        <!-- Element Slider_control avec le modifier Slider_control-previous -->
        <button class="Slider_control Slider_control-previous" type="button">
            Previous
        </button>
        <!-- Element Slider_control avec le modifier Slider_control-next -->
        <button class="Slider_control Slider_control-next" type="button">
            Next
        </button>
    </div>
</div>
```

```css
.Slider {
    /* Styles block Slider */
}

.Slider_slide {
    /* Styles element Slider_slide */
}

.Slider_controls {
    /* Styles element Slider_controls */
}

.Slider_control {
    /* Styles element Slider_control */
}

.Slider_control-previous {
    /* Styles modifier Slider_control-previous */
}

.Slider_control-next {
    /* Styles modifier Slider_control-next */
}
```



### StyleLint



## JavaScript

TL;DR : Le code JavaScript utilise la syntaxe moderne ES2015 (et supérieur), est
transpilé vers ES5 par Babel et analysé par ESLint. Le code est découpé selon
l'architecture suivante : le fichier `global.js` est le point d'entrée de
l'application, il se charge d'importer l'ensemble des modules dont l'application
a besoin; les dossiers `modules`, `utils` et `vendors` contiennent les
classes et fonctions composant l'application. Enfin, Webpack se charge de
compiler l'ensemble du code en un seul fichier.

### ECMAScript 2015+


### Transpilation (Babel)


### ESLint


### Architecture


### Module bundle (Webpack)
