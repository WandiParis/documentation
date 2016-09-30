# Transpilation (Babel)

Les navigateurs implémentent la spécification après sa finalisation, mais
chacun à leur rythme. Il n'est donc pas toujours possible d'utiliser les
nouveautés en production car certains navigateurs ne les ont pas encore
implémentées.

Toutefois, la majorité des nouveautés d'ES2015 n'est que du sucre syntaxique. Il
est donc possible de transformer du code écrit en ES2015 en code ES5 que la
totalité des navigateurs récents sait interpréter. C'est ce pour quoi
[Babel](https://babeljs.io/) est fait.

Babel est basé sur un écosystème de plugins qui proposent chacun une
fonctionnalité. Pour embarquer un ensemble de fonctionnalités, des plugins
spéciaux, appelés "presets" existent. Pour être toujours synchronisés avec la
dernière spécification, nous utilisons
[babel-preset-latest](https://babeljs.io/docs/plugins/preset-latest/). En
fonction des besoins du projet, d'autres presets et/ou plugins peuvent être
utilisés.

TODO : faire le lien avec la tâche gulp javascripts

---

[Précédent : ECMAScript 2015+](/js/01-es2015-and-beyond.md) /
[Suivant : ESLint](/js/03-eslint.md)
