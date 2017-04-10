# Ecriture du fichier `gulpfile.js`

## Création du fichier

Le fichier doit s'appeler `gulpfile.js`.

## Importation des tâches

Les tâches sont à importer de la manière suivante :

```js
const gulp = require("gulp");
const styles = require("@wandiparis/gulp-styles");
const javascripts = require("@wandiparis/gulp-javascripts");
const images = require("@wandiparis/gulp-images");
```

## Définition de la tâche `compile`

La tâche `compile` est obligatoire, et a pour rôle de compiler l'ensemble des
ressources front-end du projet. Elle peut potentiellement être différente pour
chaque projet. Voici un exemple :

```js
const gulp = require("gulp");
const styles = require("@wandiparis/gulp-styles");
const javascripts = require("@wandiparis/gulp-javascripts");
const images = require("@wandiparis/gulp-images");

const compile = gulp.parallel(
    styles(),
    javascripts({rootDir: __dirname})(),
    images()
)
```

## Définition de la tâche `watch`

La tâche `watch` est elle-aussi obligatoire. Elle a pour rôle de lancer la ou
les tâches adéquate(s) lorsqu'un fichier est modifié. Elle peut, elle aussi,
potentiellement être différente pour chaque projet. Voici un exemple :

```js
const gulp = require("gulp");
const styles = require("@wandiparis/gulp-styles");
const javascripts = require("@wandiparis/gulp-javascripts");
const images = require("@wandiparis/gulp-images");

const watch = () => {
    gulp.watch("assets/scss/**/*.scss", styles());
    gulp.watch("assets/js/**/*.js", styles());
    gulp.watch("assets/img/**/*.{jpg,png,gif,svg}", images());

    javascripts({rootDir: __dirname}, {watch: true})();
}
```

## Exportation des tâches

Une fois les tâches `compile` et `watch` écrites, il est nécessaire de les
exporter, afin de pouvoir les utiliser via la CLI :

```js
// ... importation des tâches et définition des tâches compile et watch

module.exports = {
    compile,
    watch,
};
```

## Exemple de `gulpfile` complet

```js
const gulp = require("gulp");
const baseStyles = require("@wandiparis/gulp-styles");
const baseJavascripts = require("@wandiparis/gulp-javascripts");
const images = require("@wandiparis/gulp-images");
const fonts = require("@wandiparis/gulp-fonts");
const sprite = require("@wandiparis/gulp-sprite");

const styles = baseStyles({
    src: "assets/scss/*.scss",
    sassOptions: { includePaths: ["node_modules"] },
});

const javascripts = baseJavascripts({
    rootDir: __dirname,
});

const compile = gulp.parallel(
    javascripts,
    fonts(),
    gulp.series(
        sprite(),
        styles,
        images()
    )
);

const watch = () => {
    gulp.watch("assets/scss/**/*.scss", styles);
    gulp.watch("assets/img/**/*.{jpg,png,gif,svg}", images());
    gulp.watch("assets/img/icons/*.png", sprite());

    baseJavascripts({
        rootDir: __dirname,
    }, {
        watch: true,
    })();
};

module.exports = {
    compile,
    watch,
};
```
