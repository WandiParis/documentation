# [Insights](https://insight.sensiolabs.com) - Best Practices

TL;DR : Un petit récap des "erreurs" facilement évitables pour passer le test Sensio Insight like a boss!

## Parameters.yml

#### Données sensibles
Ne jamais commit le fichier `parameters.yml` qui contient des valeurs confidentielles (credentials SQL, mot de passe, etc).

Décrire seulement les clés sans valeur (~) ou avec des valeurs par défaut dans `parameters.yml.dist`

Les valeurs seront demandées au moment du `composer install`

```yaml
# app/config/parameters.yml.dist
parameters:
    database_host:     127.0.0.1
    database_port:     ~
    database_name:     ~
    database_user:     root
    database_password: null
    # ...
```    

> https://insight.sensiolabs.com/what-we-analyse/symfony.app.confidential_parameters_file_present_in_repository

#### Secret key (CSRF)
Il faut changer la valeur par défaut du `secret` Symfony

```yaml
# app/config/parameters.yml.dist
parameters:
    # ...
    secret: ThisTokenIsNotSoSecretChangeIt
```    

Pour le générer > http://nux.net/secret

> https://insight.sensiolabs.com/what-we-analyse/symfony.obvious_csrf_key

## Services 

#### Injection de dépendance
Imaginons que vous ayez besoin de la `Request` **ET** de la `Session`.

Ne jamais passer le **container de service** à un service mais les services souhaités.

```yaml
services:
    foo_service:
        class:     AppBundle\FooService
        arguments: ["@service_container"] # BAD WAY
        arguments: ["@Request", "@session"] # GOOD WAY
        scope:     request
```

> https://insight.sensiolabs.com/what-we-analyse/symfony.inject_request_service

## Tests unitaires 
PHP doit être capable d'accéder et d'exécuter l'ensemble des tests unitaires de l'app (lol).

Créer le fichier `phpunit.xml.dist` à la racine du projet:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- https://phpunit.de/manual/current/en/appendixes.configuration.html -->
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://schema.phpunit.de/4.8/phpunit.xsd"
         backupGlobals="false"
         colors="true"
         bootstrap="app/autoload.php">
    <php>
        <ini name="error_reporting" value="-1" />
        <server name="KERNEL_DIR" value="app/" />
    </php> 

    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>tests</directory>
            <directory>src/*Bundle/Tests</directory>
            <directory>src/*/*Bundle/Tests</directory>
            <directory>src/*/Bundle/Tests</directory>
        </testsuite>
    </testsuites>

    <filter>
        <whitelist>
            <directory>src</directory>
            <exclude>
                <directory>src/*Bundle/Resources</directory>
                <directory>src/*/*Bundle/Resources</directory>
                <directory>src/*/Bundle/*Bundle/Resources</directory>
            </exclude>
        </whitelist>
    </filter>
</phpunit>

```

> non accessible sur sensio :(

## Favicon
Il faut écraser les 2 `favicons` livrées par défaut avec Sf, elle peuvent indiquer des infos critiques pour un vilain h@ck3r.

Écraser les 2 fichiers suivants:
```bash
> web/apple-touch-icon.png
> web/favicon.ico
```

> non accessible sur sensio :(

## Error pages
Il faut customiser les **pages d'erreur** de Sf.

Créer le dossier:
```bash
> app/Resources/TwigBundle/views/Exception
```

Et les fichiers relatifs aux erreurs HTTP:
```bash
> error302.html.twig
> error400.html.twig
> error401.html.twig
> error403.html.twig
> error404.html.twig
> error500.html.twig
> #...
> errorXXX.html.twig
```

Ou à minima le fichier générique:
```bash
> error.html.twig
```

> https://insight.sensiolabs.com/what-we-analyse/symfony.configuration.error_pages_should_be_customised

## Settings

#### Config
Supprimer le **fichier de config** `config.php` déployé par défaut par Sf, il peut indiquer des infos critiques pour un vilain h@ck3r:

```bash
> rm web/config.php
```

> https://insight.sensiolabs.com/what-we-analyse/symfony.web_config_should_not_be_present

#### Git ignore
Les fichiers spécifiques à l'utilisateur (.idea, .buildpath, .project, etc.) ne doivent pas être placés dans le `.gitignore` mais dans le ignore global de la machine:

```ini
# Windows | ~/.gitconfig
[user]
	name = Your Name
	email = your.email@domain.tld
[core]
	excludesfile = your\path\to\your\global\.gitignore

# Unix
git config --global core.excludesfile 'your/path/to/your/global/.gitignore'
```

> non accessible sur sensio :(

#### EOL
Tous les fichiers doivent se terminer par un saut de ligne vide `\r`

Pour les `anciens`, entendons par là un simple `retour chariot`:

```yaml
# app/config/routing.yml
app:
    resource: "@AppBundle/Resources/config/routing.yml"
# ceci est une ligne vide (obligé SyntaxHighlighter supprime la ligne vide)
```

> non accessible sur sensio :(

#### Bootable App
Une app Sf doit être **bootable**. 

Il faut indiquer quelle version de `Doctrine` utiliser pour pouvoir faire sa tambouille:

```yaml
# app/config/config.yml
# ...
doctrine:
    dbal:
        server_version: 5.6 # MAGIC
# ...
```

> https://insight.sensiolabs.com/what-we-analyse/symfony.application_not_bootable

#### Session
Il faut **changer l'identifiant de session** utilisé par le cookie par défaut pour des raisons de sécurité.

Une pratique simple consiste à donner le nom du projet en guise de clé de session :

```yaml
# app/config/config.yml
# ...
framework:
    session:
        name: project-name # MAGIC
# ...
```

> https://insight.sensiolabs.com/what-we-analyse/symfony.request.session_cookie_default_name

## Composer

##### composer.json

Le `composer.json` doit être valide.

Sf s'installe avec un name foireux (tout du moins sous PHPStorm) et ne possède pas de propriété `description`.

Corriger le **name** et ajouter la **description**:

```json
{
    "name": "WandiParis/project-name",
    "description": "project-description",
    // ...
}
```

> non accessible sur sensio :(

##### composer.lock

Le `composer.lock` doit être up to date:
```bash
> composer update
```

> non accessible sur sensio :(

## Code obsolète

#### Use
Ne pas laisser des `use` inutiles.

Checker votre IDE, si elle est décente, sinon good luck!

> non accessible sur sensio :(

#### Méthodes, propriétés, variables, paramétres
Idem, ne pas laisser de `method`, `property`, `variable`, `parameter` inutiles.

> non accessible sur sensio :(

#### Todo / fixme
Ne pas laisser de `todo` ou `fixme` dans le code.

> non accessible sur sensio :(

#### Code commenté
Ne pas laisser de **code commenté** dans le code.

Attention, il y en a par défaut dans `web/app.php`, le supprimer tout bonnement:

```php
# ...
// REMOVE THESE LINES
/*
$apcLoader = new Symfony\Component\ClassLoader\ApcClassLoader(sha1(__FILE__), $loader);
$loader->unregister();
$apcLoader->register(true);
*/
# ...
// THIS ONE
//$kernel = new AppCache($kernel);
# ...
// AND THIS ONE
//Request::enableHttpMethodParameterOverride();
# ...
```

> non accessible sur sensio :(

## HTTP methods

#### POST
Une action accessible en `POST` doit toujours rediriger en `GET` après soumission.

```php
public function createAction(Request $request)
{
    $form = $this->createForm(...);

    $form->bind($request);
    if ($form->isValid()) {
        // do some sort of processing

        return $this->redirect($this->generateUrl(...));
    }

    return $this->render(...);
}
```

> https://insight.sensiolabs.com/what-we-analyse/symfony.controller.missing_redirect_after_post

#### Routing & methods
Toujours préciser dans `routing.yml` **les méthodes HTTP** disponibles pour accéder à une route.

```yaml
# src/AppBundle/Resources/config/routing.yml
key:
    path:     /url/rewrte
    defaults: { _controller: AppBundle:Controller:Action }
    methods: ["GET", "POST"] # for example
```

> https://insight.sensiolabs.com/what-we-analyse/symfony.routing.action_not_restricted_by_method

## Length

#### Controller

Les actions des `Controller` ne doivent pas excéder **20 lignes** (sic)
> HF & GL

Insights nous permet de dépasser ce cap pour max **10%** des actions de l'ensemble de l'app.

Il faut délocaliser toute les gros traitements dans des `Services`, la logique SQL dans les `Repositories`, etc.

> https://insight.sensiolabs.com/what-we-analyse/symfony.controller.action_method_too_long

#### Views

Même logique que côté `Controller`, cette fois côté `Twig`. 

Cette fois-ci le max est de **5%** des actions et la longueur maximale tolérée est de **200 lignes**.

> non accessible sur sensio :(

## Booléens

Les opérations sur les `bool` doivent être opérées **strictement** (`===`):

```php
if ($foo->isValid() === true){
    # do some stuff
}
if ($foo->isValid() !== false){
    # do some stuff
}
```

> non accessible sur sensio :(
