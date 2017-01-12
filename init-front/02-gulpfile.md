# Installation de Gulp et des tâches

## Création du fichier

Le fichier doit obligatoirement s'appeler `gulpfile.babel.js`. Cela est dû au
fait que nous écrivons ce fichier en ES6+, ce qui oblige Gulp à le lancer en
utilisant babel.

## Importation des tâches

Les tâches sont à importer de la manière suivante :

```js
import gulp from 'gulp'
import styles from 'gulp-styles'
import javascripts from 'gulp-javascripts'
import images from 'gulp-images'
```

## Définition de la tâche `compile`

La tâche `compile` est obligatoire, et a pour rôle de compiler l'ensemble des
ressources front-end du projet. Elle peut potentiellement être différente pour
chaque projet. Voici un exemple :

```js
import gulp from 'gulp'
import styles from 'gulp-styles'
import javascripts from 'gulp-javascripts'
import images from 'gulp-images'

const compile = gulp.parallel(
    styles(),
    javascripts(),
    images()
)
```

## Définition de la tâche `watch`

La tâche `watch` est elle-aussi obligatoire. Elle a pour rôle de lancer la ou
les tâches adéquate(s) lorsqu'un fichier est modifié. Elle peut, elle aussi,
potentiellement être différente pour chaque projet. Voici un exemple :

```js
import gulp from 'gulp'
import styles from 'gulp-styles'
import javascripts from 'gulp-javascripts'
import images from 'gulp-images'

const watch = () => {
    gulp.watch('assets/scss/**/*.scss', styles())
    gulp.watch('assets/js/**/*.js', styles())
    gulp.watch('assets/img/**/*.{jpg,png,gif,svg}', images())
}
```

## Exportation des tâches

Une fois les tâches `compile` et `watch` écrites, il est nécessaire de les
exporter, afin de pouvoir les utiliser via la CLI :

```js
// ... importation des tâches et définition des tâches compile et watch

export {
    compile,
    watch
}
```

## Exemple de `gulpfile` complet

```js
import gulp from 'gulp'
import styles from 'gulp-styles'
import javascripts from 'gulp-javascripts'
import images from 'gulp-images'

const compile = gulp.parallel(
    styles(),
    javascripts(),
    images()
)

const watch = () => {
    gulp.watch('assets/scss/**/*.scss', styles())
    gulp.watch('assets/js/**/*.js', styles())
    gulp.watch('assets/img/**/*.{jpg,png,gif,svg}', images())
}

export {
    compile,
    watch
}
```
