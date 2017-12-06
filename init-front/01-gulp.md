# Installation de Gulp et des tâches

## Création du fichier `package.json`

La première chose à faire avant d'installer Gulp est de créer un fichier
`package.json` :

```
npm init -y
```

## Installation des tâches

Nous disposons de plusieurs tâches génériques pré-packagées installables
directement via npm. Elles sont regroupées dans le repository
[gulp-tasks](https://github.com/WandiParis/gulp-tasks).

Chaque tâche peut être installée indépendamment. Il est toutefois possible
d'installer l'ensemble des tâches souhaitées en une seule commande. Par exemple,
si je souhaite installer `gulp-styles`, `gulp-javascripts` et `gulp-images`, je
peux faire ceci :

```
npm install --save-dev @wandiparis/gulp-styles @wandiparis/gulp-javascripts @wandiparis/gulp-images
```

Ainsi, npm installera l'ensemble des dépendances de ces 3 tâches. Ces tâches
embarquent déjà la version 4.0 de Gulp dans leurs dépendances. Il n'y a donc
pas besoin d'installer Gulp à part.

Attention, certaines tâches (`gulp-javascripts`, par exemple) nécessitent de
créer des fichiers de configuration. Lorsqu'une tâche a besoin d'un ou plusieurs
fichiers de configuration, sa documentation l'indique clairement.

[Suivant : Création du fichier `gulpfile.js`](/init-front/02-gulpfile.md)
