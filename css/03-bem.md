# BEM

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

---

[Suivant : StyleLint](/css/04-stylelint.md)
