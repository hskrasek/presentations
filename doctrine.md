slidenumbers: true
footer: Doctrine 2 — Laravel Austin

# [fit] Doctrine 2

---

# [fit] What is Doctrine?

An **object-relational mapper** (ORM) for PHP, that uses the **Data Mapper** pattern.

^ The Data Mapper pattern aims for a complete separation of your domain/business logic from the persistence in a relational database management system.
^ One of the benefits of Doctrine is that the programmer is able to focus more on the object-oriented business logic first, and the persistence layer second.

---

# [fit] Entities

Objects that have a distinct identity, you also hear these called **reference objects**

^ Entities are objects with identity. A conceptual meaning inside your domain. For example in an application each article has a unique id, of which you can uniquely identity each article by that id

^ Since entities don't extend any base class with all of the database logic, they are just plain old PHP classes with properties and accessor methods. Typically the only way to access the data on this objects is through these accessor methods

^ Due to this separation of domain/business logic and persistence logic, we have to tell Doctrine how it should map the columns from our database tables

---

# [fit] Meta Data

---

[.hide-footer]

```php, [.highlight: 2-8, 12-15, 18-20]
<?php

use Doctrine\ORM\Mapping AS ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="articles")
 */
class Article
{
    /**
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    protected $id;

    /**
     * @ORM\Column(type="string")
     */
    protected $title;

    public function getId()
    {
        return $this->id;
    }

    public function getTitle()
    {
        return $this->title;
    }

    public function setTitle($title)
    {
        $this->title = $title;
    }
}
```

^ There are many ways to map database columns to entities. In this example we are using annotations, but you can also use YAML, XML or PHP arrays.

^ The laravel-doctrine package has another way to map columns to entities, using a more fluent syntax similar to Laravel's schema builder, that can actually be used in non-Laravel packages as well.

---

[.hide-footer]

```yaml
# App.Article.dcm.yml
App\Article:
  type: entity
  table: articles
  id:
    id:
      type: integer
      generator:
        strategy: AUTO
  fields:
    title:
      type: string
```

---

[.hide-footer]

```xml
// App.Article.dcm.xml
<?xml version="1.0" encoding="UTF-8"?>
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                          http://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity name="App\Article" table="articles">

        <id name="id" type="integer" column="id">
            <generator strategy="AUTO"/>
            <sequence-generator sequence-name="tablename_seq" allocation-size="100" initial-value="1" />
        </id>

        <field name="title" column="title" type="string" />
    </entity>

</doctrine-mapping>
```

---

[.hide-footer]

```php
<?php
return [
    'App\Article' => [
        'type'   => 'entity',
        'table'  => 'articles',
        'id'     => [
            'id' => [
                'type'     => 'integer',
                'generator' => [
                    'strategy' => 'auto'
                ]
            ],
        ],
        'fields' => [
            'title' => [
                'type' => 'string'
            ]
        ]
    ]
];
```

---

[.hide-footer]

```php, [.highlight: 9-21]
<?php

class Article
{

    protected $id;
    protected $title;

    public static function loadMetadata(Doctrine\ORM\Mapping\ClassMetadata $metadata)
    {
        $metadata->mapField(array(
           'id'        => true,
           'fieldName' => 'id',
           'type'      => 'integer'
        ));

        $metadata->mapField(array(
           'fieldName' => 'title',
           'type'      => 'string'
        ));
    }
}
```

---

# [fit] Embeddables

An **embeddable** is an object that can be mapped to a set of columns in a database table. This object will not be mapped to a table by itself, but will be used inside an entity, so its columns will be mapped to the entity's table instead.

^ http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/tutorials/embeddables.html

---

[.hide-footer]

```php, [.highlight: 3-5, 8-9, 12, 15-25]
<?php

use Doctrine\ORM\Annotation as ORM;

/** @ORM\Entity */
class User
{
    /** @ORM\Embedded(class = "Address") */
    private $address;
}

/** @ORM\Embeddable */
class Address
{
    /** @ORM\Column(type = "string") */
    private $street;

    /** @ORM\Column(type = "string") */
    private $postalCode;

    /** @ORM\Column(type = "string") */
    private $city;

    /** @ORM\Column(type = "string") */
    private $country;
}
```

^ If all of the fields in your embeddable are nullable, it's recommended to initialize the embeddable to avoid getting a null value

^ By default, Doctrine names your columns by prefixing them, using the value object name. In this example we'd have address_street, address_postalCode, etc.

---

#[fit] Mapped Superclasses

**Mapped superclasses** allow us to share common state between our entities, without it being an entity itself.

^ When modeling our entities, we may sometimes find the need to create a hierarchy or family of entities, and do so by using inheritance in our PHP classes. If we do so, Doctrine offers us several ways to map those classes to our database. Mapped superclass is one of them.

^ http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/inheritance-mapping.html#mapped-superclasses

---

# [fit] Single Table Inheritance

An inheritance strategy where all classes are mapped to a single database table

^ In order to distinguish which row represents which type in a the hierarchy a so-called discriminator column is used.

---

[.hide-footer]

```php, [.highlight: 4-10, 15-18]
<?php
namespace MyProject\Model;

/**
 * @Entity
 * @InheritanceType("SINGLE_TABLE")
 * @DiscriminatorColumn(name="discr", type="string")
 * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
 */
class Person
{
    // ...
}

/**
 * @Entity
 */
class Employee extends Person
{
    // ...
}
```

---

# [fit] Class Table Inheritance

An inheritance strategy where each class is mapped to several tables: its own, and the parent tables

^ The table of a child class is linked to the table of a parent class through a foreign key constraint. Doctrine 2 implements this strategy through a discriminator column in the topmost table of the heirarchy, because this is the easiest way to achieve polymorphic queries with Class Table Inheritance

^ This is very similar to Eloquents polymorphic relationships

---

```php, [.highlight: 4-10, 15-16]
<?php
namespace MyProject\Model;

/**
 * @Entity
 * @InheritanceType("JOINED")
 * @DiscriminatorColumn(name="discr", type="string")
 * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
 */
class Person
{
    // ...
}

/** @Entity */
class Employee extends Person
{
    // ...
}
```

---

# [fit] Associations

Instead of working with foreign keys, you work with references to objects

^ Doctrine will convert these references to foreign keys internally. These will either be a reference to a single object, or a collection of objects

---

[.hide-footer]

# Many-To-One (**Unidirectional**)

```php, [.highlight: 2-3, 7-11, 14-15]
<?php
/** @Entity */
class User
{
    // ...

    /**
     * @ManyToOne(targetEntity="Address")
     * @JoinColumn(name="address_id", referencedColumnName="id")
     */
    private $address;
}

/** @Entity */
class Address
{
    // ...
}
```

^ One tip for working with relations is to read from left to the right, where the left word refers to the current entity.

---

[.hide-footer]

# One-To-One (**Unidirectional**)

```php, [.highlight: 2-3, 7-12, 17-18]
<?php
/** @Entity */
class Product
{
    // ...

    /**
     * One Product has One Shipment.
     * @OneToOne(targetEntity="Shipment")
     * @JoinColumn(name="shipment_id", referencedColumnName="id")
     */
    private $shipment;

    // ...
}

/** @Entity */
class Shipment
{
    // ...
}
```

^ Note that the @JoinColumn is not necessary here, as the defaults would be the same

---

[.hide-footer]

# One-To-One (**Bidirectional**)

```php, [.highlight: 2-3, 7-12, 16-17, 21-26]
<?php
/** @Entity */
class Customer
{
    // ...

    /**
     * One Customer has One Cart.
     * @OneToOne(targetEntity="Cart", mappedBy="customer")
     */
    private $cart;

    // ...
}

/** @Entity */
class Cart
{
    // ...

    /**
     * One Cart has One Customer.
     * @OneToOne(targetEntity="Customer", inversedBy="cart")
     * @JoinColumn(name="customer_id", referencedColumnName="id")
     */
    private $customer;

    // ...
}
```

^ The following relationship is bidirectional due to the customer having a reference to the cart, and vice-versa

^ In this example we see the `mappedBy` and `inversedBy` annotations for the first time. They tell Doctrine which property on the other side referes to the object.

^ We had a choice of sides on which to place the `inversedBy` attribute. Because it is on the Cart, that is the owning side of the relation, and thus holds the foreign key

---

[.hide-footer]

# One-To-One (**Self-referencing**)

```php, [.highlight: 2-3, 7-12]
<?php
/** @Entity */
class Student
{
    // ...

    /**
     * One Student has One Student.
     * @OneToOne(targetEntity="Student")
     * @JoinColumn(name="mentor_id", referencedColumnName="id")
     */
    private $mentor;

    // ...
}
```

---

[.hide-footer]

# One-To-Many (**Bidirectional**)

```php, [.highlight: 2-5, 7-12, 14-15, 17-22]
<?php
use Doctrine\Common\Collections\ArrayCollection;

/** @Entity */
class Product
{
    /**
     * One Product has Many Features.
     * @OneToMany(targetEntity="Feature", mappedBy="product")
     */
    private $features;
}

/** @Entity */
class Feature
{
    /**
     * Many Features have One Product.
     * @ManyToOne(targetEntity="Product", inversedBy="features")
     * @JoinColumn(name="product_id", referencedColumnName="id")
     */
    private $product;
}
```

^ One to Many associations have to be bidirectional, unless you are using a join table. This is because the many side in a one-to-many association holds the foreign key, making it the owning side.

^ Bidirectional requires the `mappedBy` attribute on the "one" side and the `inversedBy` attribute on the "many" side.

^ This means there is no difference between a bidirectional one-to-many and a bidirectional many-to-one

---

[.hide-footer]

# One-To-Many (**Unidirectional with Join Table**)

```php, [.highlight: 2-3, 5-13, 16-17]
<?php
/** @Entity */
class User
{
    /**
     * Many User have Many Phonenumbers.
     * @ManyToMany(targetEntity="Phonenumber")
     * @JoinTable(name="users_phonenumbers",
     *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
     *      inverseJoinColumns={@JoinColumn(name="phonenumber_id", referencedColumnName="id", unique=true)}
     *      )
     */
    private $phonenumbers;
}

/** @Entity */
class Phonenumber
{
    // ...
}
```

^ A unidirectional one-to-many can be mapped through a join table. Peeking under the hood, Doctrine actually considers this as a unidirectional many-to-many whereby a unique constraint on one of the join columns enforces the one-to-many cardinality

---

[.hide-footer]

# One-To-Many (**Self-referencing**)

```php, [.highlight: 2-3, 5-16]
<?php
/** @Entity */
class Category
{
    /**
     * One Category has Many Categories.
     * @OneToMany(targetEntity="Category", mappedBy="parent")
     */
    private $children;

    /**
     * Many Categories have One Category.
     * @ManyToOne(targetEntity="Category", inversedBy="children")
     * @JoinColumn(name="parent_id", referencedColumnName="id")
     */
    private $parent;
}
```

^ In this example we setup a hierarchy of Category objects with self referencing relationships. From a database perspective is known as an adjacency list approach

---

[.hide-footer]

# Many-To-Many (**Unidirectional**)

```php, [.highlight: 2-3, 5-13, 16-17]
<?php
/** @Entity */
class User
{
    /**
     * Many Users have Many Groups.
     * @ManyToMany(targetEntity="Group")
     * @JoinTable(name="users_groups",
     *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
     *      inverseJoinColumns={@JoinColumn(name="group_id", referencedColumnName="id")}
     *      )
     */
    private $groups;
}

/** @Entity */
class Group
{
    // ...
}
```

^ Real many-to-many associations are less common. Here we see a unidirectional association between users and groups

^ Why are they less common? Typically you want to association additional attributes with an association, which is when you introduce an association class. Because of this the direct many-to-many association disappears and is replaced by one-to-many/many-to-one associations between the 3 participating classes.

---

[.hide-footer]

# Many-To-Many (**Bidirectional**)

```php, [.highlight: 2-3, 5-10, 13-14, 16-20]
<?php
/** @Entity */
class User
{
    /**
     * Many Users have Many Groups.
     * @ManyToMany(targetEntity="Group", inversedBy="users")
     * @JoinTable(name="users_groups")
     */
    private $groups;
}

/** @Entity */
class Group
{
    /**
     * Many Groups have Many Users.
     * @ManyToMany(targetEntity="User", mappedBy="groups")
     */
    private $users;
}
```

^ Same example as above, but this one is bidirectional

^ For many-to-many associations you can chose which entity is the owning and which is the inverse side.

^ There is a simple semantic rule to decide which side is more suitable to be the owning side. Just ask yourself which entity is responsible for the connection management. For example if you have an Article and a Tag, whenever you want to connect an Article to a Tag, the Article is responsible for the relation

---

[.hide-footer]

# Many-To-Many (**Self-referencing**)

```php, [.highlight: 2-3, 5-19]
<?php
/** @Entity */
class User
{
    /**
     * Many Users have Many Users.
     * @ManyToMany(targetEntity="User", mappedBy="myFriends")
     */
    private $friendsWithMe;

    /**
     * Many Users have many Users.
     * @ManyToMany(targetEntity="User", inversedBy="friendsWithMe")
     * @JoinTable(name="friends",
     *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
     *      inverseJoinColumns={@JoinColumn(name="friend_user_id", referencedColumnName="id")}
     *      )
     */
    private $myFriends;
}
```

^ In this example of users having friends, it is bidirectional so has a field named `$friendsWithMe` and `$myFriends`

---

# [fit] Entity Manager

The central access point to ORM functionality

^ The Entity Manager can be used to find, persist, flush and remove entities.

^ Ideally you would form units of work that operator on your objects and call flush when you're done. It is suggested that while serving a single HTTP request, you not call flush more than 0-2 times.

---

# [fit] Finding Entities

```php
$article = EntityManager::find('App\Article', 1);
$article->setTitle('Different title');

$article2 = EntityManager::find('App\Article', 1);

if ($article === $article2) {
    echo "Yes we are the same!";
}
```

^ Due to the Entity Managers implementation of the Identity Map Pattern, you can retrieve a model once, change something on it, find it again and they'll be equal.

---

# [fit] Persisting

```php
$article = new Article;
$article->setTitle('Let\'s learn about persisting');

EntityManager::persist($article);
EntityManager::flush();
```

^ When passing entities through the persist method, the entity becomes managed. Persist does not cause any INSERT queries.

^ The persistence isn't synced with the database until the flush method is called. This is because Doctrine utilizes a strategy called "transactional write-behind".

^ Due to this strategy, once flush is called all the necessary SQL statements will be executed in a single, short transaction

^ Ideally you would form units of work that operator on your objects and call flush when you're done. It is suggested that while serving a single HTTP request, you not call flush more than 0-2 times.

---

# [fit] Flushing

```php
// Flush all changes
EntityManager::flush();

// Only flush changes of given entity
EntityManager::flush($article);
```

^ Any change you've made to an entity that you want persisted to the database, will have to be flushed by the EntityManager

^ Ideally you would form units of work that operator on your objects and call flush when you're done. It is suggested that while serving a single HTTP request, you not call flush more than 0-2 times.

---

# [fit] Removing

```php
EntityManager::remove($article);
EntityManager::flush();
```

^ As mentioned before, marking an entity as REMOVED does not actually sync this change to the database until the flush method has been called.

^ Flushing after every change to an entity or every use of persist/remove/refresh is considered an anti-pattern and reduces the performance of your application

^ Ideally you would form units of work that operator on your objects and call flush when you're done. It is suggested that while serving a single HTTP request, you not call flush more than 0-2 times.

---

# [fit] Repositories

Design pattern abstraction for your persistence layer

^ Repositories are modeled as collections when abstracting away persistence lingo. Common methods you'll find are find, findByName

^ Doctrine comes with a generic ObjectRepository interface that easily lets you find one, many or all entities by ID, an array of filters, or by complex Criteria. Along with an implementation in EntityRepository

---

# [fit] Getting a Repository

```php
$repository = EntityManager::getRepository(Scientist::class);
```

^ Getting a repository from the entity manager will return an instance of a generic EntityRepository, ready to be queried for the requested instance.

---

[.hide-footer]

# [fit] Reusing through **Inheritance**

```php
<?php

use Doctrine\ORM\EntityRepository;

class DoctrineScientistRepository extends EntityRepository implements ScientistRepository
{
    // public function find($id) already implemented in parent class!

    public function findByName($name)
    {
        return $this->findBy(['name' => $name]);
    }
}

// Then, in one of your ServiceProviders
use App\Research\Scientist;

class AppServiceProvider
{
    public function register()
    {
        $this->app->bind(ScientistRepository::class, function($app) {
            // This is what Doctrine's EntityRepository needs in its constructor.
            return new DoctrineScientistRepository(
                $app['em'],
                $app['em']->getClassMetaData(Scientist::class)
            );
        });
    }
}
```

---

[.hide-footer]

# [fit] Reusing through **Composition**

```php
<?php

use Doctrine\Common\Persistence\ObjectRepository;

class DoctrineScientistRepository implements ScientistRepository
{
    private $genericRepository;

        public function __construct(ObjectRepository $genericRepository)
        {
            $this->genericRepository = $genericRepository;
        }

        public function find($id)
        {
            return $this->genericRepository->find($id);
        }

        public function findByName($name)
        {
            return $this->genericRepository->findBy(['name' => $name]);
        }
}

// Then, in one of your ServiceProviders
use App\Research\Scientist;

class AppServiceProvider
{
    public function register()
    {
        $this->app->bind(ScientistRepository::class, function(){
            return new DoctrineScientistRepository(
                EntityManager::getRepository(Scientist::class)
            );
        });
    }
}
```

^ This approach gives you total control of your repositories API. For example if you don't want to allow fetching all of your Scientist, you simply omit that method from your interface/implementation.

^ Inheriting the Doctrine repository would force the findAll method onto your custom repository API

---

# [fit] Using your **custom repository**

```php, [.highlight: 3-9, 15-25]
<?php
namespace MyDomain\Model;

use Doctrine\ORM\EntityRepository;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="MyDomain\Model\UserRepository")
 */
class User
{

}

class UserRepository extends EntityRepository
{
    public function getAllAdminUsers()
    {
        return $this->em->createQuery('SELECT u FROM MyDomain\Model\User u WHERE u.status = "admin"')
                         ->getResult();
    }
}

$admins = $em->getRepository('MyDomain\Model\User')->getAllAdminUsers();
```

^ You can always inject your repositories directly, but you can also tell Doctrine to use your custom repository instead.

---

[.footer: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html]

# [fit] DQL

# [fit] **Doctrine Query Language**, an **Object Query Language**

^ DQL provides powerful querying capabilities over your object model.

^ A common mistake for beginners is to mistake DQL for being a form of SQL and using table names, column names and joining arbitrary tables together.

^ With DQL you need to this of it as a query language for you object model not your relational schema

---

[.footer: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html]

# [fit] DQL **SELECT** clause

```php
<?php
$query = $em->createQuery('SELECT u FROM MyProject\Model\User u WHERE u.age > :age');
$query->setParameter('age', 20);
$users = $query->getResult();
```

^ `u` is whats called the identification variable or alias, that refers to the User class

^ Following the FROM keyword is a FQCN, which is followed by the alias

^ The expression `u.age` in the WHERE is a path expression. The path expression refers to the `age` field on the User class

---

[.footer: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html]

# [fit] Joins

```php, [.highlight: 3-4]
<?php
$query = $em->createQuery("SELECT u FROM User u JOIN u.address a WHERE a.city = 'Berlin'");
$query = $em->createQuery("SELECT u, a FROM User u JOIN u.address a WHERE a.city = 'Berlin'");
$users = $query->getResult();
```

^ There are two types of JOINs, "regular" joins and "fetch" joins

^ Regular joins: Used to limit the results and/or compute aggregate values

^ Fetch joins: In addition to regular joins, used to fetch related entities and included them in the hydrated results.

---

[.footer: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html]

# [fit] Query Builder

An API for conditionally constructing a **DQL** query

^ The API provides a set of classes and methods that are able to programmatically build queries, and provides a fluent API.

---

# [fit] High level API methods

```php
<?php
// $qb instanceof QueryBuilder

$qb->select('u, a') // string 'u' is converted to array internally
   ->from('User', 'u')
   ->join('u.address', 'a')
   ->where($qb->expr()->eq('a.city', '?1'))
   ->orderBy('u.surname', 'ASC')
   ->setParameter(1, 'Berlin');
```

^ Here is the fetch join example from earlier, written using the query builder

---

[.footer: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/native-sql.html]

# [fit] Native **SQL**

Execute native **SELECT SQL**, mapping the results to **Doctrine entities**

^ With NativeQuery you can execute native SELECT SQL statements and map the results to Doctrine entities or any other result format supported by Doctrine

^ When using this feature you need to help Doctrine out, by describing what columns in the result map to which entity property. This is called a ResultSetMapping

---

[.footer: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/native-sql.html]

# [fit] **ResultSetMapping** Example

```php
<?php
// Equivalent DQL query: "select u from User u join u.address a WHERE u.name = ?1"
// User owns association to an Address and the Address is loaded in the query.
$rsm = new ResultSetMapping;
$rsm->addEntityResult('User', 'u');
$rsm->addFieldResult('u', 'id', 'id');
$rsm->addFieldResult('u', 'name', 'name');
$rsm->addJoinedEntityResult('Address' , 'a', 'u', 'address');
$rsm->addFieldResult('a', 'address_id', 'id');
$rsm->addFieldResult('a', 'street', 'street');
$rsm->addFieldResult('a', 'city', 'city');

$sql = 'SELECT u.id, u.name, a.id AS address_id, a.street, a.city FROM users u ' .
       'INNER JOIN address a ON u.address_id = a.id WHERE u.name = ?';
$query = $this->em->createNativeQuery($sql, $rsm);
$query->setParameter(1, 'romanb');

$users = $query->getResult();
```

---

[.build-lists: true]

# [fit] Other Topics

* Proxy objects
* Reference proxies
* Association proxies
* Read-Only entities
* Extra-Lazy collections

^ Proxy object is an object put in place or used instead of the "real" object. Doctrine 2 uses these to realize many features, but primarily for transparent lazy loading.
^ Reference proxies allows you to obtain a reference to an entity where the identifier is known, without loading other properties from the database.
^ Second most important situation where Doctrine uses proxy objects, is when you query for an object that has an association marked LAZY, or isn't added to the DQL.
^ Newer version of Doctrine you can mark entities as read only. You can insert and delete these, but cannot update them through the entity manager.
^ Entities that hold references to large collections will get performance and memory issues. To solve this you can use the EXTRA_LAZY fetch-mode feature for collections.

---

[.build-lists: true]

# [fit] Best Practices

* Constrain relationships
* Use cascades judiciously
* Initialize collections in the constructor
* Don't map foreign keys to fields
* Explicit transaction demarcation

^ It's important to constrain relationships as much as possible. Avoid bidirectional associations if possible, eliminate nonessential associations. Reduces coupling in domain model, simpler code, and less work for Doctrine
^ Automatic cascades may seem handy, but should be used wisely. Do not add cascades to everything
^ It's a best practice to initialize any collections in the constructor of your entity.
^ Foreign keys have no meaning in an object model. Foreign keys are how relational databases establish relationships, meanwhile your object model does so through object references. Mapping these foreign keys to object fields heavily leaks details of the relational model
^ Doctrine automatically wraps all DML operations in a transaction on flush(), but it's considered best practice to set transaction boundaries yourself.

---

# [fit] DEMO

---

# [fit] Questions?
