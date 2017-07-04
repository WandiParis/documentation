# Création des scripts npm

Afin de pouvoir lancer les tâches exportées par le `gulpfile`, il faut créer des
scripts npm. Ces scripts sont à ajouter à la propriété `scripts` du
fichier `package.json`. Lorsqu'on exécute un script npm, le dossier
`./node_modules/.bin` est temporairement ajouté à la variable d'environnement
`PATH`. Cela nous permet d'utiliser le binaire `gulp` comme si nous l'avions
installé globalement.

Les scripts obligatoires sont les suivants :

```
"scripts": {
    "compile": "gulp compile",
    "compile:prod": "cross-env NODE_ENV=production npm run compile",
    "prestart": "npm run compile",
    "start": "gulp watch"
}
```

Note : le script `compile:prod` utilise la dépendence
[`cross-env`](https://www.npmjs.com/package/cross-env) pour affecter la variable
d'environnement `NODE_ENV` de manière unifiée peu importe l'OS de la machine. Il
faut donc l'installer : `npm install --save-dev cross-env`

* `compile` permet de compiler l'ensemble des ressources du projet
* `compile:prod` permet de compiler l'ensemble des ressources du projet et de
minifier celles qui doivent l'être pour être déployées en production (CSS et JS)
* `prestart` est automatiquement lancé lorsqu'on lance le script `start`
* `start` permet d'observer les modifications apportées aux fichiers et de
lancer des tâches en fonction de celles-ci

Pour lancer un script, il suffit de lancer la commande suivante :

```
npm run NOM_DU_SCRIPT
```

---

[Retour au sommaire CSS](/init-front/00-index.md) /
[Retour au sommaire global](/README.md)
