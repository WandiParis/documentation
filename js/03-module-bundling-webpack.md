# Module bundle (Webpack)

Il existe principalement 2 méthodes pour écrire une application JavaScript sous
forme de modules : AMD (Asynchronous Module Definition, méthode implémentée par
[RequireJS](http://www.requirejs.org/)), CommonJS (méthode implémentée par
[NodeJS](https://nodejs.org/en/)). Toutefois, aucune de ces 2 méthode ne fait
partie de la spécification d'ECMAScript. ES2015 apporte une réponse à ce
problème en introduisant un nouveau système de module accompagné de sa syntaxe.
Ce système de module peut être transpilé vers une syntaxe CommonJS, ce qui nous
permet de l'utiliser dès aujourd'hui.

Voici quelques ressources sur les modules :

* [ExploringJS - modules](http://exploringjs.com/es6/ch_modules.html)
* [JavaScript Modules the ES6 Way](https://24ways.org/2014/javascript-modules-the-es6-way/)
* [ECMAScript 6 modules: the final syntax](http://www.2ality.com/2014/09/es6-modules-final.html)

Nous avons fait le choix d'utiliser cette syntaxe, puisqu'elle fait partie du
standard.

Toutefois, nous avons besoin de générer un unique fichier qui sera importé dans
le HTML final. Pour cela, nous utilisons [Webpack](https://webpack.github.io/).
Cet outil permet de résoudre les dépendances entre les modules pour les
assembler dans un seul fichier.

TODO : faire le lien avec la tâche gulp javascripts

---

[Précédent : Transpilation (Babel)](/js/02-transilation-babel.md) /
[Suivant : Architecture](/js/04-architecture.md)
