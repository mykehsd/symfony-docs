Practical Symfony: Day 3
========================

== The Bundle
To start our development we must create a bundle that will contain all of the functionality for a specific feature.  There is some great documentation on the [Symfony2 bundle system](http://symfony.com/doc/current/book/page_creation.html#page-creation-bundles) that will explain more about what bundles are and when to create them.  To create our first bundle enter the following command:

```
php app/console generate:bundle --namespace=Jobeet/JobBundle --format=yml --no-interaction
```


== The Data Model
Those of you itching to open your text editor and lay down some PHP will be happy to know today's tutorial will get us into some development. We will define the Jobeet data model, use an ORM to interact with the database, and build the first module of the application. But as Symfony2 does a lot of the work for us, we will have a fully functional web module without writing too much PHP code.

=== The Relational Model
The user stories we have written yesterday describe the main objects of our project: jobs, affiliates, and categories. Here is the corresponding entity relationship diagram:
![Model](http://www.symfony-project.org/images/jobeet/1_2/03/diagram.png)

In addition to the columns described in the stories, we have also added a created_at field to some tables. Symfony recognizes such fields and sets the value to the current system time when a record is created. That's the same for updated_at fields: Their value is set to the system time whenever the record is updated.

=== The Schema

To store the jobs, affiliates, and categories, we obviously need a relational database.

But as Symfony2 is an Object-Oriented framework, we like to manipulate objects whenever we can. For example, instead of writing SQL statements to retrieve records from the database, we'd rather prefer to use objects.

The relational database information must be mapped to an object model. This can be done with an ORM tool and thankfully, Symfony2 has two of them available: Propel and Doctrine. In this tutorial, we will use Doctrine which is included in the Symfony2 standard edition that we have already installed.

The ORM needs a description of the tables and their relationships to create the related classes. There are two ways to create this description schema: by introspecting an existing database or by creating it by hand.




