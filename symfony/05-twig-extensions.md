# [Twig Extensions](https://github.com/twigphp/Twig-extensions) - Common Twig Extensions

TL;DR : Des extensions Twig utiles qui ont été sorties du core.

## Installation

#### Download

```bash
> composer require twig/extensions
```

#### Enable

```yaml
# app/config/services.yml
services:
    twig.extension.text:
       class: Twig_Extensions_Extension_Text
       tags:
            - { name: twig.extension } 
    twig.extension.i18n:
        class: Twig_Extensions_Extension_I18n
        tags:
            - { name: twig.extension }
    twig.extension.intl:
        class: Twig_Extensions_Extension_Intl
        tags:
            - { name: twig.extension }
    twig.extension.array:
        class: Twig_Extensions_Extension_Array
        tags:
            - { name: twig.extension }
    twig.extension.date:
        class: Twig_Extensions_Extension_Date
        tags:
            - { name: twig.extension }
    
```

## Text

#### Wrap

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{{ "Lorem ipsum dolor sit amet, consectetur adipiscing" | wordwrap(10) }}

{# ... #}

```

> Lorem ipsu

>  m dolor si

>  t amet, co

>  nsectetur

>  adipiscing

#### Truncate

###### Cut chars

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{{ "Hello World!" | truncate(5) }}

{# ... #}

```

> Hello...

###### Cut words

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{{ "Hello World!" | truncate(4, true) }}

{# ... #}
```

> Hello...

###### Cut chars width another ending

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{{ "Hello World!" | truncate(7, false, "??") }}

{# ... #}
```

> Hello W??

## i18n

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{% trans "Hello World!" %}

{% trans string_var %}

{% trans %}
    Hello World!
{% endtrans %}

{# ... #}

```

## intl

Penser à activer `php_intl` dans le `php.ini`.

Pour voir la liste des paramètres des différentes extensions:
> http://twig.sensiolabs.org/doc/extensions/intl.html

#### Localized Date

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{{ object.date|localizeddate('none', 'none', 'fr', null, "d MMMM Y à HH:mm:ss") }}

{# ... #}

```

#### Localized Number

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{{ product.quantity|localizednumber('scientific') }}

{# ... #}

```

#### Localized Currency

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{{ product.price|localizedcurrency('EUR') }}

{# ... #}

```

## Array

```twig
{# src/AppBundle/Resources/views/Controller/action.html.twig #}

{{ array|shuffle() }}

{# ... #}

```