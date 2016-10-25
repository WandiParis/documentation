# [Easy Admin Bundle](https://github.com/javiereguiluz/EasyAdminBundle) - Symfony Admin Generator

TL;DR : Un petit `bundle` développé par un dev de SensioLabs qui permet de générer un Bo CRUD en seulement quelques paramétrages `yaml`.

## Installation

#### Download

```bash
> composer require javiereguiluz/easyadmin-bundle
```

#### Enable

```php
# app/AppKernel.php
$bundles = array(
    // ...
    new JavierEguiluz\Bundle\EasyAdminBundle\EasyAdminBundle(),
    // ...
);
```

#### Routing

```yaml
# app/config/routing.yml
easy_admin_bundle:
    resource: "@EasyAdminBundle/Controller/"
    type:     annotation
    prefix:   /admin
```

#### Assets

```bash
> php bin/console assets:install --symlink
```

## Security

`EasyAdminBundle` ne propose pas de services d'authentification, en revanche il affiche les informations de l'utilisateur couramment loggué sous Sf (`Guard`).

Le moyen le plus simple de restreindre l'accès par une `authentification base64` au Bo est d'implémenter le `firewall` directement dans `security.yml`

```yaml
# app/config/security.yml
security:
    providers:
        in_memory:
            memory:
                users:
                    admin:  { password: "%user_pwd%", roles: 'ROLE_ADMIN' }

    encoders:
        Symfony\Component\Security\Core\User\User:
            algorithm: sha512
            encode_as_base64: true
            iterations: 1

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        secured_area:
            pattern:    ^/
            anonymous: ~
            http_basic:
                realm: "Back-Office"

        main:
            anonymous: ~

    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
```

Pour le hash du password, j'utilise `sha512` + encodage en `base64` en 1 seule itération:
> http://cryptage.online-convert.com/fr/generateur-sha512

Ne pas oublier (best practices ;) de délocaliser le `%user_pwd%` dans `parameters.yml`:

```yaml
# app/config/parameters.yml
parameters:
    # ...
    user_pwd: MON_SUPER_ENCODAGE_B64_APRÈS_HASHAGE_512_EN_1_ITÉRATION==
    # ...
```

## Settings

#### Basic usage

```yaml
# app/config/config.yml
easy_admin:
    entities:
        - AppBundle\Entity\Category
        - AppBundle\Entity\Product
        - AppBundle\Entity\Page
        - AppBundle\Entity\User
```

> protocol://vhost.domain.tld/web/app_dev/admin

#### Admin name & color theme

```yaml
# app/config/config.yml
easy_admin:
    site_name: YOUR_PROJECT Back-Office
    design:
        brand_color: '#MYHEXA'
    # ...
```

#### Label &amp; Disable Actions

```yaml
# app/config/config.yml
easy_admin:
    # ...
    entities:
        User:
            class: AppBundle\Entity\User
            label: 'Utilisateurs'
            disabled_actions: ['delete', 'new']
     # ...
```

#### Fields to view / manage

```yaml
# app/config/config.yml
easy_admin:
    # ...
    entities:
        Category:
            class: AppBundle\Entity\Category
            label: 'Catégories'
            list:
                actions: # allowed actions
                    - { name: 'new', label: 'Ajouter', icon: 'add' }
                    - { name: 'show', label: 'Afficher', icon: 'search' }
                    - { name: 'edit', label: 'Modifier', icon: 'edit' }
                    - { name: 'delete', label: 'Supprimer', icon: 'delete' }
                fields: # fields to display
                    - { property: 'id', label: 'ID' }
                    - { property: 'name', label: 'Nom' }
                    - { property: 'parent', label: 'Catégorie Parente' }
                    - { property: 'position', label: 'Position' }
            show: # view
                fields: # fields to display
                    # ...
            form: # add AND edit
                fields: # fields to display
                    # ...
            edit: # ONLY edit
                fields: # fields to display
                    # ...
    # ...
```

#### Doctrine types

`array`, `association`, `bigint`, `blob`, `boolean`, `date`, `datetime`, `datetimetz`, `decimal`, `float`, `guid`, `integer`, `json_array`, `object`, `simple_array`, `smallint`, `string`, `text`, `time`

#### Custom types

`email`, `image`, `raw`, `tel`, `toggle`, `url`, `easyadmin_autocomplete` (sur une **fk uniquement**)

#### Formating dates

```yaml
# app/config/config.yml
easy_admin:
    formats:
        date:     'd/m/Y'
        time:     'H:i'
        datetime: 'd/m/Y H:i:s'
    # ...
```

#### Formating numbers

```yaml
# app/config/config.yml
easy_admin:
    formats:
        number: '%.2f'
    # ...
```

#### Full configuration reference

> https://github.com/javiereguiluz/EasyAdminBundle/blob/master/Resources/doc/book/configuration-reference.md

- - -

## Extend / Plugins

#### [VichUploaderBundle](https://github.com/dustin10/VichUploaderBundle)


###### Download

```bash
> composer require vich/uploader-bundle
```

###### Enable

```php
# app/AppKernel.php
$bundles = array(
    // ...
    new Vich\UploaderBundle\VichUploaderBundle(),
    // ...
);
```

###### Settings

```yaml
# app/config/config.yml
vich_uploader:
    db_driver: orm
    mappings:
        product_image:
            uri_prefix:         "%product_images_path%"
            upload_destination: '%kernel.root_dir%/../web/uploads/products'
            inject_on_load:     false
            delete_on_update:   true
            delete_on_remove:   true
```

```yaml
# app/config/parameters.yml
parameters:
    # ...
    product_images_path: /uploads/products
    # ...
```

###### Entities

```php
# src/AppBundle/Entity/Product.php

use Symfony\Component\HttpFoundation\File\File;
use Vich\UploaderBundle\Mapping\Annotation as Vich;

/**
 * @ORM\Entity
 * @Vich\Uploadable
 */
class Product
{
    /**
     * @ORM\Column(type="string", length=255)
     * @var string
     */
    private $image;

    /**
     * @Vich\UploadableField(mapping="product_images", fileNameProperty="image")
     * @var File
     */
    private $imageFile;

    /**
     * @ORM\Column(type="datetime")
     * @var \DateTime
     */
    private $updatedAt;

    // ...

    public function setImageFile(File $image = null)
    {
        $this->imageFile = $image;

        // VERY IMPORTANT:
        // It is required that at least one field changes if you are using Doctrine,
        // otherwise the event listeners won't be called and the file is lost
        if ($image) {
            // if 'updatedAt' is not defined in your entity, use another property
            $this->updatedAt = new \DateTime('now');
        }
    }

    public function getImageFile()
    {
        return $this->imageFile;
    }

    public function setImage($image)
    {
        $this->image = $image;
    }

    public function getImage()
    {
        return $this->image;
    }
}
```

###### EasyAdminBundle

List & Show:

```yaml
# app/config/config.yml
easy_admin:
    entities:
        Product:
            # ...
            list: 
                fields:
                    - { property: 'image', type: 'image', base_path: %app.path.product_images% }
            show: 
                fields:
                    - { property: 'image', type: 'image', base_path: %app.path.product_images% }
    # ...
```

Add & Edit:

```yaml
# app/config/config.yml
easy_admin:
    entities:
        Product:
            # ...
            form:
                fields:
                    - { property: 'imageFile', type: 'file' }
    # ...
```

###### Full configuration reference

> https://github.com/javiereguiluz/EasyAdminBundle/blob/master/Resources/doc/tutorials/upload-files-and-images.md

- - -

#### [IvoryCKEditorBundle](https://github.com/egeloen/IvoryCKEditorBundle)

###### Download

```bash
> composer require egeloen/ckeditor-bundle
```

###### Enable

```php
# app/AppKernel.php
$bundles = array(
    // ...
    new Ivory\CKEditorBundle\IvoryCKEditorBundle(),
    // ...
);
```

###### Assets

```bash
> php bin/console assets:install --symlink
```

###### Settings

```yaml
# app/config/config.yml
ivory_ck_editor:
    input_sync: true
    default_config: base_config
    configs:
        base_config:
            entities_latin: false
            toolbar:
                - { name: "document", items: [ "Source" ] }
                - { name: "cut", items: [ "Cut", "Copy", "Paste" ] }
                - { name: "basicstyles", items: [ "Bold", "Italic", "Underline" ] }
                - { name: "insert", items: [ "Image" ] }
                - { name: "links", items: [ "Link", "Unlink" ] }
                - { name: "justify", items: [ "JustifyLeft", "JustifyCenter", "JustifyRight", "JustifyBlock" ] }
                - { name: "paragraph", items: [ "NumberedList", "HorizontalRule", "TextColor" ] }
                - { name: "styles", items: [ "Format" ] }
```

> http://docs.cksource.com/ckeditor_api/symbols/CKEDITOR.config.html

###### EasyAdminBundle

List & Show:

```yaml
# app/config/config.yml
easy_admin:
    entities:
        Product:
            # ...
            form: 
                fields:
                    - { property: 'description', type: 'ckeditor' }
    # ...
```

###### Full configuration reference

> https://github.com/javiereguiluz/EasyAdminBundle/blob/master/Resources/doc/tutorials/wysiwyg-editor.md

- - -

#### [CKFinderBundle](https://github.com/ckfinder/ckfinder-symfony3-bundle)

Actuellement, rien d'officiel n'est supporté par `EasyAdminBundle`, faut pimper les choses.

###### Hack psr/cache

CKFinder a une mauvaise déclaration de dépendance de `psr/cache`, lui même utilisé par Sf.

Forcer la version 1.0.0 dans le `composer.json` pour éviter tout conflit:

```json
# composer.json
{
    "require": {
        // ...
        "psr/cache": "1.0.0",
        // ...
    }
}
```

###### Download

```bash
> composer require ckfinder/ckfinder-symfony3-bundle
```

###### Enable

```php
# app/AppKernel.php
$bundles = array(
    // ...
            new CKSource\Bundle\CKFinderBundle\CKSourceCKFinderBundle(),
    // ...
);
```

###### Download again (pb de license)

```bash
> php bin/console ckfinder:download
```

###### Assets

```bash
> php bin/console assets:install web
```

###### Routing

```yaml
# app/config/routing.yml
ckfinder_connector:
    resource: "@CKSourceCKFinderBundle/Resources/config/routing.yml"
    prefix:   /
```

###### Directory

```bash
> mkdir -m 777 web/userfiles
```

###### Settings

```yaml
# app/config/config.yml
ckfinder:
    connector:
        authenticationClass: AppBundle\Services\CustomCKFinderAuth
```

###### Authenticator

Dans le meilleur des mondes, faudrait implémenter le `firewall` sur les routes du `CkFinder`.
En attendant, quick & dirty...

```php
<?php
# src/AppBundle/Services/CustomCKFinderAuth.php

namespace AppBundle\Services;

use CKSource\Bundle\CKFinderBundle\Authentication\Authentication as AuthenticationBase;

class CustomCKFinderAuth extends AuthenticationBase
{
    public function authenticate()
    {
        return true;
    }
}
```

###### Plug CkEditor &amp; CkFinder

Il n'y a pas de possibilité proposée par `EasyAdminBundle`, donc la technique la moins *sale* que j'ai trouvé est de créer un `JS` de setup:

```js
// web/js/setup-ckfinder.js
window.onload = function () {
	if (window.CKEDITOR){
		CKFinder.config( { connectorPath: '/app.php/ckfinder/connector' } );
		for (var ckInstance in CKEDITOR.instances){
			CKFinder.setupCKEditor(CKEDITOR.instances[ckInstance]);
		}
	}
}
```

et de le charger (ainsi que `ckfinder.js` bien entendu) dans le `config.yml`:

```yaml
# app/config/config.yml
easy_admin:
    design:
        assets:
            js:
                - '/bundles/cksourceckfinder/ckfinder/ckfinder.js'
                - '/js/setup-ckfinder.js'
```

**ATTENTION**, l'URL du connector étant absolute sur `/app.php`, il se peut que vous ayez besoin de vider la cache de prod:

```bash
> php bin/console cache:clear --env=prod
```

- - -

## Full example

```yaml
# app/config/config.yml
ckfinder:
    connector:
        authenticationClass: AppBundle\Services\CustomCKFinderAuth

vich_uploader:
    db_driver: orm
    mappings:
        product_image:
            uri_prefix:         "%product_images_path%"
            upload_destination: '%kernel.root_dir%/../web/uploads/products'
            inject_on_load:     false
            delete_on_update:   true
            delete_on_remove:   true

ivory_ck_editor:
    input_sync: true
    default_config: base_config
    configs:
        base_config:
            entities_latin: false
            toolbar:
                - { name: "document", items: [ "Source" ] }
                - { name: "cut", items: [ "Cut", "Copy", "Paste" ] }
                - { name: "basicstyles", items: [ "Bold", "Italic", "Underline" ] }
                - { name: "insert", items: [ "Image" ] }
                - { name: "links", items: [ "Link", "Unlink" ] }
                - { name: "justify", items: [ "JustifyLeft", "JustifyCenter", "JustifyRight", "JustifyBlock" ] }
                - { name: "paragraph", items: [ "NumberedList", "HorizontalRule", "TextColor" ] }
                - { name: "styles", items: [ "Format" ] }

easy_admin:
    site_name: Orkyn Back-Office
    entities:
        Category:
            class: AppBundle\Entity\Category
            label: 'Catégories'
            list:
                actions:
                    - { name: 'new', label: 'Ajouter', icon: 'add' }
                    - { name: 'show', label: 'Afficher', icon: 'search' }
                    - { name: 'edit', label: 'Modifier', icon: 'edit' }
                    - { name: 'delete', label: 'Supprimer', icon: 'delete' }
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'name', label: 'Nom' }
                    - { property: 'parent', label: 'Catégorie Parente' }
                    - { property: 'position', label: 'Position' }
            show:
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'name', label: 'Nom' }
                    - { property: 'parent', label: 'Catégorie Parente' }
                    - { property: 'position', label: 'Position' }
            form:
                fields:
                    - { property: 'name', label: 'Nom' }
                    - { property: 'parent', label: 'Catégorie Parente' }
            edit:
                fields:
                    - { property: 'name', label: 'Nom' }
                    - { property: 'parent', label: 'Catégorie Parente' }
                    - { property: 'position', label: 'Position' }
        Product:
            class: AppBundle\Entity\Product
            label: 'Produits'
            list:
                actions:
                    - { name: 'new', label: 'Ajouter', icon: 'add' }
                    - { name: 'show', label: 'Afficher', icon: 'search' }
                    - { name: 'edit', label: 'Modifier', icon: 'edit' }
                    - { name: 'delete', label: 'Supprimer', icon: 'delete' }
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'image', type: 'image', base_path: "%product_images_path%" }
                    - { property: 'name', label: 'Nom' }
                    - { property: 'excerpt', label: 'Accroche' }
                    - { property: 'category', label: 'Catégorie' }
            show:
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'image', type: 'image', base_path: "%product_images_path%" }
                    - { property: 'name', label: 'Nom' }
                    - { property: 'excerpt', label: 'Accroche' }
                    - { property: 'category', label: 'Catégorie' }
            form:
                fields:
                    - { property: 'imageFile', type: 'vich_image' }
                    - { property: 'name', label: 'Nom' }
                    - { property: 'excerpt', label: 'Accroche' }
                    - { property: 'description', type: 'ckeditor', label: 'Description' }
                    - { property: 'category', label: 'Catégorie' }
        Page:
            class: AppBundle\Entity\Page
            label: 'Pages'
            list:
                actions:
                    - { name: 'new', label: 'Ajouter', icon: 'add' }
                    - { name: 'show', label: 'Afficher', icon: 'search' }
                    - { name: 'edit', label: 'Modifier', icon: 'edit' }
                    - { name: 'delete', label: 'Supprimer', icon: 'delete' }
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'name', label: 'Nom' }
                    - { property: 'position', label: 'Position' }
            show:
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'name', label: 'Nom' }
                    - { property: 'content', type: 'raw', label: 'Corps' }
                    - { property: 'position', label: 'Position' }
            form:
                fields:
                    - { property: 'name', label: 'Nom' }
                    - { property: 'content', type: 'ckeditor', label: 'Corps' }
            edit:
                fields:
                    - { property: 'name', label: 'Nom' }
                    - { property: 'content', type: 'ckeditor', label: 'Corps' }
                    - { property: 'position', label: 'Position' }
        Ref:
                class: AppBundle\Entity\Ref
                label: 'Références'
                list:
                    actions:
                        - { name: 'new', label: 'Ajouter', icon: 'add' }
                        - { name: 'show', label: 'Afficher', icon: 'search' }
                        - { name: 'edit', label: 'Modifier', icon: 'edit' }
                        - { name: 'delete', label: 'Supprimer', icon: 'delete' }
                    fields:
                        - { property: 'id', label: 'ID' }
                        - { property: 'ref', label: 'Référence' }
                        - { property: 'product1', label: 'Produit 1' }
                        - { property: 'product2', label: 'Produit 2' }
                show:
                    fields:
                        - { property: 'id', label: 'ID' }
                        - { property: 'ref', label: 'Référence' }
                        - { property: 'product1', label: 'Produit 1' }
                        - { property: 'product2', label: 'Produit 2' }
                form:
                    fields:
                        - { property: 'ref', label: 'Référence' }
                        - { property: 'product1', label: 'Produit 1' }
                        - { property: 'product2', label: 'Produit 2' }

```