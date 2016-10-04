# [EWZ Recaptcha Bundle](https://github.com/excelwebzone/EWZRecaptchaBundle) - Symfony Recaptcha Bundle

TL;DR : Un petit `bundle` qui permet d'intégrer [`Google Recaptcha`](https://www.google.com/recaptcha/intro/index.html)

## Installation

#### Download

```bash
> composer require excelwebzone/recaptcha-bundle
```

#### Enable

```php
# app/AppKernel.php
$bundles = array(
    // ...
    new EWZ\Bundle\RecaptchaBundle\EWZRecaptchaBundle(),
    // ...
);
```

## Settings

```yaml
# app/config/config.yml
ewz_recaptcha:
    public_key:  XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    private_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    locale_key:  '%kernel.default_locale%'        sortable: KnpPaginatorBundle:Pagination:sortable_link.html.twig # sort link template
```

Penser à activer le translator pour avoir messages d'erreur/succès de Google dans la bonne locale ;)

```yaml
# app/config/config.yml
framework:
    translator:      { fallbacks: ["%locale%"] }
```

## Usage

#### `Form`

```php
# src/AppBundle/Form/Type/ContactType.php

# ...
use EWZ\Bundle\RecaptchaBundle\Form\Type\EWZRecaptchaType;

class ContactType extends AbstractType
{

    /**
     * @param FormBuilderInterface $builder
     * @param array $options
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            # ...
            ->add('recaptcha', EWZRecaptchaType::class)
        ;
    }
    
    /**
     * @param OptionsResolver $resolver
     */
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Contact'
        ));
    }
}
```

#### `Entity`

```php
# src/AppBundle/Entity/Contac.php

# ...
use EWZ\Bundle\RecaptchaBundle\Validator\Constraints as Recaptcha;

/**
 * Contact
 *
 * @ORM\Table(name="contact")
 * @ORM\Entity(repositoryClass="AppBundle\Repository\ContactRepository")
 */
class Contact
{
    # ...
    
    /**
     * @Recaptcha\IsTrue
     */
    public $recaptcha;
    
    # ...
}
```

#### `View`

```twig
{# src/AppBundle/Resources/views/Home/contact.html.twig #}

{# ... #}

{{ form_start(form, ... }}

{# ... #}

<div class="captcha">
    {% form_theme form 'EWZRecaptchaBundle:Form:ewz_recaptcha_widget.html.twig' %}
    {{ form_widget(form.recaptcha, { 'attr': {
        'options' : {
            'theme': 'light',
            'type': 'image'
        },
    } }) }}
</div>

{# ... #}

```