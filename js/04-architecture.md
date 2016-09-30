# Architecture

Le JavaScript est organisé selon 3 dossiers et un fichier :

* `modules/` : contient l'ensemble des composants d'interface réutilisables
* `utils/` : contient des utilitaires qui peuvent être utilisés dans plusieurs
modules
* `vendors/` : contient d'éventuelles librairies externes qui ne sont pas
disponibles sur le registry npm
* `global.js` : importe et instancie les modules dont le site a besoin

Le dossier `vendors` a la particularité de n'accueillir que les librairies qui
ne sont pas disponibles sur npm. En effet, pour simplifier la gestion des
dépendances externes, nous privilégions l'installation des librairies par npm.

---

[Précédent : Module bundle (Webpack)](/js/03-module-bundling-webpack.md)
[Suivant : ESLint](/js/05-eslint.md) /
