# JavaScript

TL;DR : Le code JavaScript utilise la syntaxe moderne ES2015 (et supérieur), est
transpilé vers ES5 par Babel et analysé par ESLint. Le code est découpé selon
l'architecture suivante : le fichier `global.js` est le point d'entrée de
l'application, il se charge d'importer l'ensemble des modules dont l'application
a besoin; les dossiers `modules`, `utils` et `vendors` contiennent les
classes et fonctions composant l'application. Enfin, Webpack se charge de
compiler l'ensemble du code en un seul fichier.

* [ECMAScript 2015+](/js/01-es2015-and-beyond.md)
* [Transpilation (Babel)](/js/02-transilation-babel.md)
* [ESLint](/js/03-eslint.md)
* [Architecture](/js/04-architecture.md)
* [Module bundle (Webpack)](/js/05-module-bundling-webpack.md)
