---
layout: single
title: Symfony 4 Sortable Doctrine Extension
excerpt: "Symfony 4: Doctrine sortable extension"
date: 2019-11-20
classes: wide
header:
  teaser: /assets/images/sortableDoctrine.jpg
  teaser_home_page: true
categories:
  - Symfony
tags:
  - Symfony
---

This is how to configure this extension on Symfony 4. First you need to install the bundle following the instruccions here: https://symfony.com/doc/master/bundles/StofDoctrineExtensionsBundle/installation.html#installation-using-symfony-flex

Configure the bundle in `config/packages/stof_doctrine_extensions.yaml` (create the file if it doesn’t exist) like this:

```yaml
stof_doctrine_extensions:
    default_locale: eu_ES
    orm:
        default:
            sortable: true
```

Edit your entity and add a new column

```php
/**
 * @Gedmo\SortablePosition
 * @ORM\Column(name="position", type="integer")
 */
private $position;
```

Don’t forget to add the `use Gedmo\Mapping\Annotation as Gedmo;`

And here comes the main change you have to do in symfony4, you need to use `SortableRepository`. If you aren’t using a repository for your entity you can add this line on your entity:

```php
/**
 * @ORM\Entity(repositoryClass="Gedmo\Sortable\Entity\Repository\SortableRepository")
 */
```

But if you are using a repository with your custom querys, you need to make 2 changes:

Change `ServiceEntityRepository` to `SortableRepository` and import `use Gedmo\Sortable\Entity\Repository\SortableRepository;` class:

  ```php
  class EmployeeRepository extends SortableRepository
  ```

Change the construct function to:

  ```php
  public function __construct(EntityManagerInterface $em)
  {
      parent::__construct($em, $em->getClassMetadata(Employee::class));
  }
  ```