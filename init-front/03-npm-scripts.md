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
    "prestart": "npm run compile",
    "start": "gulp watch"
}
```

* `compile` permet de compiler l'ensemble des ressources du projet
* `prestart` est automatiquement lancé lorsqu'on lance le script `start`
* `start` permet d'observer les modifications apportées aux fichiers et de
lancer des tâches en fonction de celles-ci

Pour lancer un script, il suffit de lancer la commande suivante :

```
npm run NOM_DU_SCRIPT
```
