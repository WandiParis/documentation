# [Easy Admin Bundle](https://github.com/javiereguiluz/EasyAdminBundle) - Mise en place d'ACL

TL;DR : `EasyAdmin` est kick-ass mais n'implémentera pas d'ACL avant sa version 2. 

Entendons par là qu'il est pour l'instant **impossible nativement** d'appliquer une quelconque gestion de droits sur le Bo, soit on y accède, soit on y accède pas, end of story.

L'idée est donc d'essayer de plugger `EasyAdmin` sur la `sécurité` de Symfony (guard, roles, security, firewall, etc.)

## Authentification
On se base sur le système de Symfo, donc n'importe quel système qui implémente le guard, les firewalls et la security sont compatibles.

Pour mes tests, j'ai installé [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle) que tout le monde connait et qui fait gagner un temps fou.

Je ne reviendrai pas sur l'install de ce bundle (JB doit nous le présenter ;), je l'ai branché sur une entity `User` qui extends `BaseUser` et j'ai crée des users aux privilèges distincts à partir des fixtures:
```php
# src/AppBundle/DataFixtures/ORM/LoadUser.php

public function load(ObjectManager $em)
{
	$userManager = $this->container->get('fos_user.user_manager');

	$users = array(
		array('lastname' => 'Admin',
			'firstname' => 'Super',
			'email' => 'root@jeu-coca-cola.fr',
			'username' => 'root',
			'password' => '82JLe8HunA',
			'roles' => ['ROLE_USER', 'ROLE_ADMIN', 'ROLE_SUPER_ADMIN'],
		),
		array('lastname' => 'Admin',
			'firstname' => 'Basic',
			'email' => 'admin@jeu-coca-cola.fr',
			'username' => 'admin',
			'password' => 'TcnV1Nltf4',
			'roles' => ['ROLE_USER', 'ROLE_ADMIN'],
		),
		array('lastname' => 'Bientz',
			'firstname' => 'Laurent',
			'email' => 'laurent.bientz@wandi.fr',
			'password' => 'P@ssw0rd',
			'roles' => ['ROLE_USER'],
		),
	);

	foreach($users as $userInfos){

		// fos specific
		$user = $userManager->createUser();
		$user->setUsername((array_key_exists('username', $userInfos) && !empty($userInfos['username'])) ? $userInfos['username'] : $userInfos['email']);
		$user->setEnabled(true);
		foreach($userInfos['roles'] as $role) {
			$user->addRole($role);
		}
		#$user->setSalt(base_convert(sha1(uniqid(mt_rand(), true)), 16, 36));
		$password = (array_key_exists('password', $userInfos) && !empty($userInfos['password'])) ? $userInfos['password'] : $this->generatePassword(mt_rand(8,14));
		$user->setPlainPassword($password);

		$user->setLastname($userInfos['lastname']);
		$user->setFirstname($userInfos['firstname']);
		$user->setEmail($userInfos['email']);

		// fos specific
		$userManager->updateUser($user);

		$em->persist($user);
	}

	$em->flush();
}
```

Nos 3 super profils sont donc:
* `root` qui est god
* `admin` qui est un "simple" admin
* `user` qui correspond à un user sur le front

# Security

J'ai crée les 3 rôles décrit plus haut et j'ai restreint l'accès au bo (/admin) au rôle `ROLE_ADMIN` minimum.

```yaml
# app/config/security.yml

security:
    encoders:
        FOS\UserBundle\Model\UserInterface: sha512

    role_hierarchy:
        ROLE_USER:        ROLE_USER # user front-end
        ROLE_ADMIN:       ROLE_USER # admin (minimum level to access back-office)
        ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN] # super-admin

    providers:
        fos_userbundle:
            id: fos_user.user_provider.username_email

    firewalls:
        main:
            pattern: ^/
            form_login:
                provider: fos_userbundle
                csrf_token_generator: security.csrf.token_manager
            logout:       true
            anonymous:    true

    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/account, role: ROLE_USER }
        - { path: ^/admin/, role: ROLE_ADMIN }
```

Selon la hierarchie des rôles, sans ACL, les users `root` et `admin` peuvent accéder au back-office après login, le user `laurent` non.

# Config

L'idée est d'ajouter un nouveau noeud `role` au niveau d'EasyAdmin pour décrire le rôle nécessaire à l'accès d'une ressource.

Ce `role` peut se positionner à plusieurs niveau:
* directement dans les entrées du menu pour les rubriques parentes englobant plusieurs entités
```yaml
# app/config/config_easyadmin.yml

easy_admin:
    design:
        menu:
            - { label: 'Utilisateurs', role: ROLE_SUPER_ADMIN, icon: 'user-circle-o', children: [ { label: 'Utilisateurs', entity: 'User', icon: 'user' } ] }
```
* au niveau d'une entité de base
```yaml
# app/config/config_easyadmin.yml

easy_admin:
    entities:
        User:
            class: AppBundle\Entity\User
            role: ROLE_SUPER_ADMIN
```
* au niveau d'une sous-action d'une entité
```yaml
# app/config/config_easyadmin.yml

easy_admin:
    entities:
        Article:
            class: AppBundle\Entity\Article
            role: ROLE_ADMIN
            list:
                # ...
            show:
                # ...
            new:
                # ...
                role: ROLE_SUPER_ADMIN
            edit:
                # ...
            delete:
                # ...
                role: ROLE_SUPER_ADMIN
```

Cette granularité permet de masquer complétement une rubrique parente (et donc tous les entités filles), masquer une entité ou masquer une action d'une entité.

Exemple complet pour la suite:
```yaml
# app/config/config_easyadmin.yml

easy_admin:
    site_name: CCEP Back-Office
    formats:
        datetime:   'd/m/Y à H\hi e'
        date:       'd/m/Y'
        time:       'H\hi e'
    design:
        brand_color: '#E01A22'
        form_theme:
            - 'vertical'
        assets:
            js:
                - '/bundles/cksourceckfinder/ckfinder/ckfinder.js'
                - '/js/setup-ckfinder.js'
            css:
                - '/css/easyadmin-tuning.css'
        menu:
            - { label: 'Utilisateurs', role: ROLE_SUPER_ADMIN, icon: 'user-circle-o', children: [ { label: 'Utilisateurs', entity: 'User', icon: 'user' } ] }
            - { label: 'Blog', role: ROLE_ADMIN, icon: 'comment', children: [ { label: 'Articles', entity: 'Article', icon: 'file-text-o' } ] }
    entities:
        User:
            class: AppBundle\Entity\User
            role: ROLE_SUPER_ADMIN
            list:
                title: 'Utilisateurs'
                actions:
                    - { name: 'new', label: 'Ajouter', icon: 'add' }
                    - { name: 'show', label: '', icon: 'search' }
                    - { name: 'edit', label: '', icon: 'edit' }
                    - { name: 'delete', label: '', icon: 'trash' }
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'lastname', label: 'Nom' }
                    - { property: 'firstname', label: 'Prénom' }
                    - { property: 'username', label: 'Login' }
                    - { property: 'email', label: 'Email' }
                    - { property: 'roles', label: 'Rôles' }
                    - { property: 'enabled', label: 'Activé ?', type: 'toggle' }
            show:
                title: 'Fiche de l''utilisateur'
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'lastname', label: 'Nom' }
                    - { property: 'firstname', label: 'Prénom' }
                    - { property: 'username', label: 'Login' }
                    - { property: 'email', label: 'Email' }
                    - { property: 'roles', label: 'Rôles' }
                    - { property: 'enabled', label: 'Activé ?' }
            new:
                title: 'Ajouter un utilisateur'
            edit:
                title: 'Modifier un utilisateur'
            form:
                fields:
                    - { type: 'group', label: 'Infos', icon: 'info-circle' }
                    - { property: 'lastname', css_class: 'col-sm-6', label: 'Nom' }
                    - { property: 'firstname', css_class: 'col-sm-6', label: 'Prénom' }
                    - { type: 'group', css_class: 'col-sm-6', label: 'Identifiants', icon: 'key' }
                    - { property: 'username', label: 'Login' }
                    - { property: 'email', label: 'Email' }
                    - { type: 'group', css_class: 'col-sm-6', label: 'Mot de passe', icon: 'lock' }
                    - { property: 'plainPassword', type: 'repeated', type_options: { type: 'Symfony\Component\Form\Extension\Core\Type\PasswordType', first_options:  {label: 'Mot de passe' }, second_options: { label: 'Confirmer le mot de passe' }, invalid_message: 'Les deux mots de passe ne concordent pas.' }}
                    - { type: 'group', label: 'Permissions', icon: 'shield' }
                    - { property: 'roles', label: 'Rôles' }
                    - { type: 'group', label: 'Etat', icon: 'unlock' }
                    - { property: 'enabled', label: 'Activé ?' }
        Article:
            class: AppBundle\Entity\Article
            role: ROLE_ADMIN
            list:
                title: 'Articles'
                actions:
                    - { name: 'new', label: 'Ajouter', icon: 'add' }
                    - { name: 'show', label: '', icon: 'search' }
                    - { name: 'edit', label: '', icon: 'edit' }
                    - { name: 'delete', label: '', icon: 'trash' }
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'title', label: 'Titre' }
                    - { property: 'image', label: 'Photo', type: 'image', base_path: "%article_path%" }
                    - { property: 'excerpt', label: 'Accroche' }
            show:
                title: 'Fiche de l''article'
                fields:
                    - { property: 'id', label: 'ID' }
                    - { property: 'title', label: 'Titre' }
                    - { property: 'image', label: 'Photo', type: 'image', base_path: "%article_path%" }
                    - { property: 'excerpt', label: 'Accroche' }
                    - { property: 'content', label: 'Contenu', type: 'raw' }
            new:
                title: 'Ajouter un article'
                role: ROLE_SUPER_ADMIN
            edit:
                title: 'Modifier un article'
            delete:
                role: ROLE_SUPER_ADMIN
            form:
                fields:
                    - { type: 'group', label: 'Infos', icon: 'info-circle' }
                    - { property: 'title', label: 'Titre' }
                    - { type: 'group', label: 'Résumé', icon: 'pencil' }
                    - { property: 'excerpt', label: 'Accroche' }
                    - { type: 'group', label: 'Contenu riche', icon: 'file-text-o' }
                    - { property: 'content', label: 'Contenu', type: 'ckeditor' }
```

# Faire des chocapics

## Injection de dépendances

Pour pouvoir faire des traitements en amont sur le fichier de config utilisé par `EasyAdmin` (cf juste au dessus), nous devons modifier notre bundle pour lui ajouter une `compiler pass`.

Ici, on travaille directement sur le classique `AppBundle`, nous modifions donc `AppBundle.php` pour lui dire qu'au moment où Symfony va charger ce bundle, il doit également exécuter une passe de compilation manuelle.

```php
# src/AppBundle/AppBundle.php

<?php

namespace AppBundle;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\Bundle\Bundle;
use AppBundle\DependencyInjection\Compiler\ConfigPass;

class AppBundle extends Bundle
{
    public function build(ContainerBuilder $container)
    {
        parent::build($container);
        $container->addCompilerPass(new ConfigPass());
    }
}
```

Nous créons dorénavant notre tache de compilation:

```php
# src/AppBundle/DependencyInjection/Compiler/ConfigPass.php
<?php

namespace AppBundle\DependencyInjection\Compiler;

use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class ConfigPass implements CompilerPassInterface
{
    public function process( ContainerBuilder $container ) {

        $config = $container->getParameter('easyadmin.config');

        // use menu to use ROLE_ADMIN role by default if not set
        foreach($config['design']['menu'] as $k => $v) {
            if (!isset($v['role'])) {
                $config['design']['menu'][$k]['role'] = 'ROLE_ADMIN';
            }
        }

        // update entities to use ROLE_ADMIN role by default if not set
        foreach ($config['entities'] as $k => $v) {
            if (!isset($v['role'])) {
                $config['entities'][$k]['role'] = 'ROLE_ADMIN';
            }
        }

        // update views to use entities role by default if not set
        foreach ($config['entities'] as $k => $v) {
            $views = ['new', 'edit', 'show', 'list'];
            foreach ($views as $view) {
                if (!isset($v[$view]['role'])) {
                    $config['entities'][$k][$view]['role'] = $v['role'];
                }
            }
        }

        $container->setParameter('easyadmin.config', $config);
    }
}
```

Ici l'idée est de fixer la configuration d'`EasyAdmin` si aucun `role` n'est défini à plusieurs niveaux:
* Dans le menu, on fixe le niveau d'accès minimum au `ROLE_ADMIN` (si on se trompe dans security.yml, on sera bien content qu'un user du Front ne puisse pas accéder au Bo...)
* Même logique au niveau d'une `Entity`, `ROLE_ADMIN` minimum
* On réplique le rôle de l'`Entity` au niveau de chaque action de cette dernière (list, new, edit, delete)

Cette passe de compilation permet de modifier la configuration avant son chargement par `EasyAdmin`

## Back-End

Pour restreindre chaque tentative d'accès à une action d'`EasyAdmin` en back, on va créer un service qui subscribe les events soulevés par `EasyAdmin`:

```yaml
# app/config/services.yml

services:
    app.subscriber:
        class: AppBundle\EventListener\AppSubscriber
        arguments: ["@security.authorization_checker", "@request_stack", "@easyadmin.config.manager"]
        tags:
            - { name: kernel.event_subscriber }

```
 
Heureusement pour nous, `EasyAdmin` soulève des events en amont de tout ce qui nous intéresse (new, list, edit, show & delete).

Pour chacun de ces événements, on va vérifier si on a bien l'autorisation d'y accéder:

```php
# src/AppBundle/EventListener/AppSubscriber.php

<?php

namespace AppBundle\EventListener;

use JavierEguiluz\Bundle\EasyAdminBundle\Configuration\ConfigManager;
use JavierEguiluz\Bundle\EasyAdminBundle\Event\EasyAdminEvents;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\EventDispatcher\GenericEvent;
use Symfony\Component\HttpFoundation\RequestStack;
use Symfony\Component\Security\Core\Authorization\AuthorizationChecker;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;

class AppSubscriber implements EventSubscriberInterface
{
    private $authorization;
    private $requestStack;
    private $config;

    /**
     * AppSubscriber constructor.
     *
     * @param AuthorizationChecker $authorization
     * @param RequestStack $requestStack
     * @param ConfigManager $config
     */
    public function __construct(AuthorizationChecker $authorization, RequestStack $requestStack, ConfigManager $config)
    {
        $this->authorization = $authorization;
        $this->requestStack = $requestStack;
        $this->config = $config;
    }

    /**
     * @return array
     */
    public static function getSubscribedEvents()
    {
        // return the subscribed events, their methods and priorities
        return array(
            EasyAdminEvents::PRE_NEW => 'checkUserRights',
            EasyAdminEvents::PRE_LIST => 'checkUserRights',
            EasyAdminEvents::PRE_EDIT => 'checkUserRights',
            EasyAdminEvents::PRE_SHOW => 'checkUserRights',
            EasyAdminEvents::PRE_DELETE => 'checkUserRights',
        );
     }

    /**
     * show an error if user is not superadmin and tries to manage restricted stuff
     *
     * @param GenericEvent $event event
     * @return null
     * @throws AccessDeniedException
     */
    public function checkUserRights(GenericEvent $event)
    {
        // if super admin, allow all
        $request = $this->requestStack->getCurrentRequest()->query;
        if ($this->authorization->isGranted('ROLE_SUPER_ADMIN')) {
            return;
        }

        $entity = $request->get('entity');
        $action = $request->get('action');

        $config = $this->config->getBackendConfig();
        // check for permission for each action
        foreach ($config['entities'] as $k => $v) {
            if ($entity == $k && !$this->authorization->isGranted($v[$action]['role'])) {
                throw new AccessDeniedException();
            }
        }
    }
}
```

Ici, je me suis même permis le luxe de dire que c'est open-bar pour le rôle `ROLE_SUPER_ADMIN`.

Une fois que cette partie est en place, en fonction du paramétrage du noeud `role` pour chaque menu, entité, action, l'internaute malicieux ne pourra plus accéder qu'au scope de son rôle ou il se prendra une belle `403 Forbidden`

## Front-End

Bon le back c'est le plus important d'un point de vue sécurité mais d'afficher des liens ou des actions sur lesquels on a pas accès, on est tous d'accord que c'est complétement pourri !

De ce côté, pas de miracle, il va falloir surcharger `EasyAdmin` côté `app` d'un point de vue Twig.

Pour se faire, il faut créer le dossier:

* app/Resources/views/easy_admin 

dans lequel on va mettre tous nos `extends`. L'idée n'est pas de reprendre tout son code (#usine) mais de seulement surcharger les `blocks` qui nous intéresse.

**ATTENTION** à bien respecter cette arborescence et non pas le fonctionnement habituel qui consiste à reproduire la même hierarchie côté `app` que côté `vendors`. La même arbo permet d'overrider le fichier mais pas de l'extends ([Pour votre culture](https://github.com/javiereguiluz/EasyAdminBundle/issues/1095#issuecomment-210828725))

### Menu

Le plus basique est de surcharger la sidebar de navigation pour ne faire apparaitre que les liens sur lesquels on a les droits:

```twig
# app/Resources/views/easy_admin/menu.html.twig

{% extends '@EasyAdmin/default/menu.html.twig' %}

{% block main_menu %}
    {% for item in easyadmin_config('design.menu') %}
        {% set currentEntity = item.entity is defined ? easyadmin_config('entities.' ~ item.entity) : '' %}
        {% if (item.type == 'entity' and is_granted(currentEntity.role)) or (item.type == 'empty' and item.role is defined and is_granted(item.role)) %}
            <li class="{{ item.type == 'divider' ? 'header' }} {{ item.children is not empty ? 'treeview' }} {{ app.request.query.get('menuIndex')|default(-1) == loop.index0 ? 'active' }} {{ app.request.query.get('submenuIndex')|default(-1) != -1 ? 'submenu-active' }}">
                {{ helper.render_menu_item(item, _entity_config.translation_domain|default('messages')) }}

                {% if item.children|default([]) is not empty %}
                    <ul class="treeview-menu">
                        {% for subitem in item.children %}
                            {% set currentSubEntity = subitem.entity is defined ? easyadmin_config('entities.' ~ subitem.entity) : '' %}
                            {% if (subitem.type == 'entity' and is_granted(currentSubEntity.role)) or (subitem.type == 'empty' and subitem.role is defined and is_granted(subitem.role)) %}
                                <li class="{{ subitem.type == 'divider' ? 'header' }} {{ app.request.query.get('menuIndex')|default(-1) == loop.parent.loop.index0 and app.request.query.get('submenuIndex')|default(-1) == loop.index0 ? 'active' }}">
                                    {{ helper.render_menu_item(subitem, _entity_config.translation_domain|default('messages')) }}
                                </li>
                            {% endif %}
                        {% endfor %}
                    </ul>
                {% endif %}
            </li>
        {% endif %}
    {% endfor %}
{% endblock main_menu %}
```

Ici on check:
* au 1er niveau si on doit afficher une entrée de menu (qui regroupe N `Entity` children)
* au 2ème niveau si on doit afficher une `Entity`

Si on reprend mon exemple de toute à l'heure:
* le user `god` voit les 2 rubriques + les 2 `Entity`
* le user `admin` voit 1 seule rubrique et 1 seule `Entity` (`Article` en l'occurrence car `User` nécessite le rôle `ROLE_SUPER_ADMIN`)

### Actions (List) 

Là ca se complique, l'idée est de pas faire apparaitre les liens de `Show`, `Add`, `Edit`, `Delete` en fonction du `role`.

Il faut surcharger 3 `blocks`:
 
```twig
# src/Resources/views/easy_admin/list.html.twig

{% extends '@EasyAdmin/default/list.html.twig' %}

{% block new_action %}

    {% set currentEntity = _entity_config.name %}
    {% set currentEntityConfig = easyadmin_config('entities.' ~ currentEntity) %}

    {% if currentEntityConfig.new is defined and currentEntityConfig.new.role is defined and not is_granted(currentEntityConfig.new.role) %}
        {#{{ dump('new to remove') }}#}
    {% else %}
        <div class="button-action">
            <a class="{{ _action.css_class|default('') }}" href="{{ path('easyadmin', _request_parameters|merge({ action: _action.name })) }}" target="{{ _action.target }}">
                {% if _action.icon %}<i class="fa fa-{{ _action.icon }}"></i>{% endif %}
                {{ _action.label is defined and not _action.label is empty ? _action.label|trans(_trans_parameters) }}
            </a>
        </div>
    {% endif %}
{% endblock new_action %}

{% block table_head %}

    {% set currentEntity = _entity_config.name %}
    {% set currentEntityConfig = easyadmin_config('entities.' ~ currentEntity) %}
    {% set allowedActions = [] %}

    {% for action in _list_item_actions %}
        {% set actionName = action.name %}

        {# config for the current action of the current entity in yml and custom role and not allowed#}
        {% if currentEntityConfig[actionName] is defined and currentEntityConfig[actionName].role is defined and not is_granted(currentEntityConfig[actionName].role) %}
            {#{{ dump(actionName ~ ' to remove') }}#}
        {% else %}
            {% set allowedActions = allowedActions|merge({(''~actionName): _list_item_actions[actionName]}) %}
        {% endif %}

    {% endfor %}

    <tr>
        {% for field, metadata in fields %}
            {% set isSortingField = metadata.property == app.request.get('sortField')|split('.')|first %}
            {% set nextSortDirection = isSortingField ? (app.request.get('sortDirection') == 'DESC' ? 'ASC' : 'DESC') : 'DESC' %}
            {% set _column_label = (metadata.label ?: field|humanize)|trans(_trans_parameters) %}
            {% set _column_icon = isSortingField ? (nextSortDirection == 'DESC' ? 'fa-caret-up' : 'fa-caret-down') : 'fa-sort' %}

            <th data-property-name="{{ metadata.property }}" class="{{ isSortingField ? 'sorted' }} {{ metadata.virtual ? 'virtual' }} {{ metadata.dataType|lower }} {{ metadata.css_class }}">
                {% if metadata.sortable %}
                    <a href="{{ path('easyadmin', _request_parameters|merge({ sortField: metadata.property, sortDirection: nextSortDirection })) }}">
                        <i class="fa {{ _column_icon }}"></i>
                        {{ _column_label|raw }}
                    </a>
                {% else %}
                    <span>{{ _column_label|raw }}</span>
                {% endif %}
            </th>
        {% endfor %}

        {% if allowedActions|length > 0 %}
            <th>
                <span>{{ 'list.row_actions'|trans(_trans_parameters, 'EasyAdminBundle') }}</span>
            </th>
        {% endif %}
    </tr>

{% endblock table_head %}

{% block table_body %}

    {% set currentEntity = _entity_config.name %}
    {% set currentEntityConfig = easyadmin_config('entities.' ~ currentEntity) %}
    {% set allowedActions = [] %}

    {% for action in _list_item_actions %}
        {% set actionName = action.name %}

        {# config for the current action of the current entity in yml and custom role and not allowed#}
        {% if currentEntityConfig[actionName] is defined and currentEntityConfig[actionName].role is defined and not is_granted(currentEntityConfig[actionName].role) %}
            {#{{ dump(actionName ~ ' to remove') }}#}
        {% else %}
            {% set allowedActions = allowedActions|merge({(''~actionName): _list_item_actions[actionName]}) %}
        {% endif %}

    {% endfor %}

    {% for item in paginator.currentPageResults %}
        {% set _item_id = attribute(item, _entity_config.primary_key_field_name) %}
        <tr data-id="{{ _item_id }}">
            {% for field, metadata in fields %}
                {% set isSortingField = metadata.property == app.request.get('sortField') %}
                {% set _column_label =  (metadata.label ?: field|humanize)|trans(_trans_parameters)  %}

                <td data-label="{{ _column_label }}" class="{{ isSortingField ? 'sorted' }} {{ metadata.dataType|lower }} {{ metadata.css_class }}">
                    {{ easyadmin_render_field_for_list_view(_entity_config.name, item, metadata) }}
                </td>
            {% endfor %}

            {% if allowedActions|length > 0 %}
                {% set _column_label =  'list.row_actions'|trans(_trans_parameters, 'EasyAdminBundle') %}
                <td data-label="{{ _column_label }}" class="actions">
                    {% block item_actions %}
                        {{ include('@EasyAdmin/default/includes/_actions.html.twig', {
                            actions: allowedActions,
                            request_parameters: _request_parameters,
                            translation_domain: _entity_config.translation_domain,
                            trans_parameters: _trans_parameters,
                            item_id: _item_id
                        }, with_context = false) }}
                    {% endblock item_actions %}
                </td>
            {% endif %}
        </tr>
    {% else %}
        <tr>
            <td class="no-results" colspan="{{ _list_item_actions|length > 0 ? fields|length + 1 : fields|length }}">
                {{ 'search.no_results'|trans(_trans_parameters, 'EasyAdminBundle') }}
            </td>
        </tr>
    {% endfor %}
{% endblock table_body %}
```

Ici on check:
* dans le `block new_action` si on doit afficher le bouton d'ajout
* dans le `block table_head` si on doit afficher l'en tête de la colonne (dans le cas où on a ni le droit de `New`, ni de `Show` et ni de `Delete`)
* dans le `table_body` si on doit afficher le bouton `New`, et/ou `Show` et/ou `Delete` (dans le cas où on aucun droit, ne pas afficher la colonne du tout)
 
### Actions (Show)

Nous devons ne pas faire apparaitre les liens de `Edit` et/ou `Delete` en fonction du `role`.

```twig
# src/Resources/views/easy_admin/show.html.twig

{% extends 'EasyAdminBundle:default:show.html.twig' %}

{% block item_actions %}
    {% set _show_actions = easyadmin_get_actions_for_show_item(_entity_config.name) %}
    {% set _request_parameters = { entity: _entity_config.name, referer: app.request.query.get('referer') } %}

    {% set currentEntity = _entity_config.name %}
    {% set currentEntityConfig = easyadmin_config('entities.' ~ currentEntity) %}
    {% set allowedActions = [] %}

    {% for action in _show_actions %}
        {% set actionName = action.name %}

        {# config for the current action of the current entity in yml and custom role and not allowed#}
        {% if currentEntityConfig[actionName] is defined and currentEntityConfig[actionName].role is defined and not is_granted(currentEntityConfig[actionName].role) %}
            {#{{ dump(actionName ~ ' to remove') }}#}
        {% else %}
            {% set allowedActions = allowedActions|merge({(''~actionName): _show_actions[actionName]}) %}
        {% endif %}

    {% endfor %}

    {{ include('@EasyAdmin/default/includes/_actions.html.twig', {
        actions: allowedActions,
        request_parameters: _request_parameters,
        translation_domain: _entity_config.translation_domain,
        trans_parameters: _trans_parameters,
        item_id: _entity_id
    }, with_context = false) }}
{% endblock item_actions %}
```

Ici on check:
* si on doit afficher le bouton `Show` et/ou `Delete`
 
### Actions (Add/Edit)

C'est exactement la même logique que pour l'action `Show`, il faut ne pas faire appaitre seulement le bouton `Delete` en fonction du `role`.

En revanche, je ne sais pas bien pourquoi mais le `block` a surcharger (`item_actions`) se trouve dans le form theme (`bootstrap_3_layout.html.twig`) et non pas dans le twig d'édition.

L'approche est donc un peu différente, on va créer notre faux form theme côté `app` et le faire hériter du vrai côté `vendor` afin de ne pas devoir reprendre l'ensemble du code du thème et seulement extends le `block` concerné.

Avant toute chose, il faut préciser à `EasyAdmin` notre propre form theme:

```yaml
# app/config/config_easyadmin.yml

easy_admin:
    design:
        form_theme:
            - 'easy_admin/form/bootstrap_3_layout.html.twig'
```

Puis de le créer:

```twig
# src/Resources/views/easy_admin/form/bootstrap_3_layout.html.twig

{% extends 'EasyAdminBundle:form:bootstrap_3_layout.html.twig' %}

{% block item_actions %}
    {# the 'save' action is hardcoded for the 'edit' and 'new' views #}
    <button type="submit" class="btn btn-primary action-save">
        <i class="fa fa-save"></i> {{ 'action.save'|trans(_trans_parameters, _translation_domain) }}
    </button>

    {% set _entity_actions = (easyadmin.view == 'new')
    ? easyadmin_get_actions_for_new_item(easyadmin.entity.name)
    : easyadmin_get_actions_for_edit_item(easyadmin.entity.name) %}

    {% set _entity_id = (easyadmin.view == 'new')
    ? null
    : attribute(easyadmin.item, easyadmin.entity.primary_key_field_name) %}

    {% set _request_parameters = { entity: easyadmin.entity.name, referer: app.request.query.get('referer') } %}

    {% set currentEntity = easyadmin.entity.name %}
    {% set currentEntityConfig = easyadmin_config('entities.' ~ currentEntity) %}
    {% set allowedActions = [] %}

    {% for action in _entity_actions %}
        {% set actionName = action.name %}

        {# config for the current action of the current entity in yml and custom role and not allowed#}
        {% if currentEntityConfig[actionName] is defined and currentEntityConfig[actionName].role is defined and not is_granted(currentEntityConfig[actionName].role) %}
            {#{{ dump(actionName ~ ' to remove') }}#}
        {% else %}
            {% set allowedActions = allowedActions|merge({(''~actionName): _entity_actions[actionName]}) %}
        {% endif %}

    {% endfor %}

    {{ include('@EasyAdmin/default/includes/_actions.html.twig', {
        actions: allowedActions,
        request_parameters: _request_parameters,
        translation_domain: _translation_domain,
        trans_parameters: _trans_parameters,
        item_id: _entity_id
    }, with_context = false) }}
{% endblock item_actions %}
```

### Actions (QuickEdit)

`EasyAdmin` offre la particularité d'offrir un quick-edit directement dans le listing et ce, uniquement pour les champs `boolean` overridé dans la config par un type `toggle` (sic):

Afin de s'assurer que le quick-edit n'est pas accessible sur le listing en fonction du `role`, le seul moyen que j'ai trouvé est de surcharger le `field_toggle` pour faire la vérification et charger le `type_boolean` dans le cas où  les privilèges sont insuffisants:

```twig
# src/Resources/views/easy_admin/field_toggle.html.twig

{% trans_default_domain 'EasyAdminBundle' %}

{% if view == 'show' %}
    {% if value == true %}
        <span class="label label-success">{{ 'label.true'|trans }}</span>
    {% else %}
        <span class="label label-danger">{{ 'label.false'|trans }}</span>
    {% endif %}
{% else %}

    {% if entity_config.edit is defined and entity_config.edit.role is defined and not is_granted(entity_config.edit.role) %}
        {% include '@EasyAdmin/default/field_boolean.html.twig' %}
    {% else %}
        <input type="checkbox" {{ value == true ? 'checked' : '' }}
               data-toggle="toggle" data-size="mini"
               data-onstyle="success" data-offstyle="danger"
               data-on="{{ 'label.true'|trans }}" data-off="{{ 'label.false'|trans }}">
    {% endif %}

{% endif %}
```

### Cosmétique et ergo (Layout)

`EasyAdmin` a enfin implémenté ça nativement, cocorico !
> https://github.com/javiereguiluz/EasyAdminBundle/releases/tag/v1.16.8

~~Au dela des droits custom, ca ne semble pas du luxe de permettre un petit bouton `Logout` sur ce Bo, jamais compris pourquoi il n'y en avait pas de base.~~

```twig
# src/Resources/views/easy_admin/layout.twig

{% extends '@EasyAdmin/default/layout.html.twig' %}

{% block user_menu %}
    <span class="sr-only">{{ 'user.logged_in_as'|trans(domain = 'EasyAdminBundle') }}</span>
    <i class="hidden-xs fa fa-user"></i>
    {% if app.user|default %}
        {{ app.user.username|default('user.unnamed'|trans(domain = 'EasyAdminBundle')) }}
    {% else %}
        {{ 'user.anonymous'|trans(domain = 'EasyAdminBundle') }}
    {% endif %}
     | <i class="fa fa-sign-out"><a href="{{ path('fos_user_security_logout') }}">Déconnexion</a></i>
{% endblock user_menu %}
```

~~La best practice aurait été de surcharger le dictionnaire `EasyAdminBundle` (json ou xlf) côté `app` pour ne pas mettre votre traduction "Déconnexion" en dur.~~

## Résumé

Ci-dessous l'ensemble des fichiers modifiés/ajoutés pour mettre en place toute cette gestion de droits:
* *app*
    * *config*
        * `config_easyadmin.yml` > la conf easyadmin
        * `security.yml` > les firewalls et roles
        * `services.yml` > la déclaration du service AppSubscriber
    * *resources*
        * *views*
            * *easy_admin*
                * *form*
                    * `bootstrap_3_layout.html.twig` > le form theme surchargé
                * `field_toggle.html.twig` > la surchage du toogle pour inclure le boolean
                * `layout.html.twig` > le tuning du layout pour ajouter un bouton 'déconnexion'
                * `list.html.twig` > la surcharge du listing
                * `menu.html.twig` > la surcharge du menu pour la navigation
                * `show.html.twig` > la surchage de la page view
* *src*
    * *AppBundle*
        * *DependencyInjection*
            * *Compiler*
                * `ConfigPass.php` > la pass de compil avant init du bundle
        * *EventListener*
            * `AppSubscriber.php` > le service qui restreint l'accès à une action en fonction des rôles
        * `AppBundle.php` > l'appel à la pass de compil

A terme, j'essayerai d'en faire notre propre `Bundle` qui dépend de `EasyAdmin` pour gagner du temps.