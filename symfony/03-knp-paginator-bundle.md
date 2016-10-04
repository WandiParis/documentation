# [Knp Paginator Bundle](https://github.com/KnpLabs/KnpPaginatorBundle) - Symfony Paginator

TL;DR : Un petit `bundle` qui permet de paginer toute la couche `ORM` ou `ODM`.

## Installation

#### Download

```bash
> composer require knplabs/knp-paginator-bundle
```

#### Enable

```php
# app/AppKernel.php
$bundles = array(
    // ...
    new Knp\Bundle\PaginatorBundle\KnpPaginatorBundle(),
    // ...
);
```

## Settings

```yaml
# app/config/config.yml
knp_paginator:
    page_range: 5                      # default page range used in pagination control
    default_options:
        page_name: page                # page query parameter name
        sort_field_name: sort          # sort field query parameter name
        sort_direction_name: direction # sort direction query parameter name
        distinct: true                 # ensure distinct results, useful when ORM queries are using GROUP BY statements
    template:
        pagination: KnpPaginatorBundle:Pagination:sliding.html.twig     # sliding pagination controls template
        sortable: KnpPaginatorBundle:Pagination:sortable_link.html.twig # sort link template
```

## Usage

#### `Controller` ou `Repository`

Le `bundle` permet de paginer:
* `array`
* `Doctrine\ORM\Query`
* `Doctrine\ORM\QueryBuilder`
* `Doctrine\ODM\MongoDB\Query\Query`
* `Doctrine\ODM\MongoDB\Query\Builder`
* `Doctrine\ODM\PHPCR\Query\Query`
* `Doctrine\ODM\PHPCR\Query\Builder\QueryBuilder`
* `Doctrine\Common\Collection\ArrayCollection`
* `ModelCriteria` - Propel ORM query
* array with `Solarium_Client` and `Solarium_Query_Select` as elements

```php
# src/AppBundle/Controller/BlogController.php

public function indexAction(Request $request)
{
    $qbd = $this->get('doctrine')->getRepository('AppBundle:News')->createQueryBuilder('n');
    $query = $qbd->select(array('n', 't'))
                ->join('n.tags', 't')
                ->orderBy('n.id', 'DESC')
                ->getQuery();
    $paginator  = $this->get('knp_paginator');
    $pagination = $paginator->paginate(
        $query, /* query NOT result */
        $request->query->getInt('page', 1) /*page number*/,
        1 /*limit per page*/
    );
    
    return $this->render('AppBundle:Blog:index.html.twig', array(
        'pagination' => $pagination
    ));
}
```

#### `View`

```twig
{# src/AppBundle/Resources/views/Blog/index.html.twig #}

{# ... #}

{% for news in pagination %}
    {# ... #}
{% endfor %}

{# ... #}

 <div class="navigation">
    {{ knp_pagination_render(pagination) }}
</div>

{# ... #}
```

#### HTML généré

```html
<div class="navigation">
        
	<div class="pagination">
	
		<span class="current">1</span>
		
		<span class="page">
			<a href="/app_dev.php/fr/blog?page=2">2</a>
		</span>
	
		<span class="page">
			<a href="/app_dev.php/fr/blog?page=3">3</a>
		</span>
	
		<span class="page">
			<a href="/app_dev.php/fr/blog?page=4">4</a>
		</span>
	
		<span class="page">
			<a href="/app_dev.php/fr/blog?page=5">5</a>
		</span>

		<span class="next">
			<a href="/app_dev.php/fr/blog?page=2">&gt;</a>
		</span>

		<span class="last">
			<a href="/app_dev.php/fr/blog?page=14">&gt;&gt;</a>
		</span>
	</div>

</div>
```