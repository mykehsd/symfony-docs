Practical Symfony: Day 3
========================

## The Bundle
To start our development we must create a bundle that will contain all of the functionality for a specific feature.  There is some great documentation on the [Symfony2 bundle system](http://symfony.com/doc/current/book/page_creation.html#page-creation-bundles) that will explain more about what bundles are and when to create them.  To create our first bundle enter the following command:

```
php app/console generate:bundle --namespace=Jobeet/JobBundle --dir=src/ --format=yml --no-interaction
```


## The Data Model
Those of you itching to open your text editor and lay down some PHP will be happy to know today's tutorial will get us into some development. We will define the Jobeet data model, use an ORM to interact with the database, and build the first module of the application. But as Symfony2 does a lot of the work for us, we will have a fully functional web module without writing too much PHP code.

### The Relational Model
The user stories we have written yesterday describe the main objects of our project: jobs, affiliates, and categories. Here is the corresponding entity relationship diagram:
![Model](http://www.symfony-project.org/images/jobeet/1_2/03/diagram.png)

In addition to the columns described in the stories, we have also added a created_at field to some tables. Symfony recognizes such fields and sets the value to the current system time when a record is created. That's the same for updated_at fields: Their value is set to the system time whenever the record is updated.

### Setting up our database
Before we really begin, we need to configure our database connection information. By convention, this information is usually configured in an app/config/parameters.yml file:
```
# app/config/parameters.yml
parameters:
    database_driver:    pdo_mysql
    database_host:      localhost
    database_name:      jobeet
    database_user:      root
    database_password:  password
```

Now that Symfony2 knows about our database, we can have it create the database for us:
```
php app/console doctrine:database:create
```

### The Schema

To store the jobs, affiliates, and categories, we obviously need a relational database.

But as Symfony2 is an Object-Oriented framework, we like to manipulate objects whenever we can. For example, instead of writing SQL statements to retrieve records from the database, we'd rather prefer to use objects.

The relational database information must be mapped to an object model. This can be done with an ORM tool and thankfully, Symfony2 has two of them available: Propel and Doctrine. In this tutorial, we will use Doctrine which is included in the Symfony2 standard edition that we have already installed.

The ORM needs a description of the tables and their relationships to create the related classes. There are two ways to create this description schema: by introspecting an existing database or by creating it by hand.

As the database does not exist yet and as we want to keep Jobeet database agnostic, let's create the schema file by hand with the following commands:

```
Entity files go here
```

Now that your entities are created, lets create our table structure in our database.
```
php app/console doctrine:schema:update --force
```

### The initial data
The tables have been created in the database but there is no data in them. For any web application, there are three types of data:

*Initial data*: Initial data are needed for the application to work. For example, Jobeet needs some initial categories. If not, nobody will be able to submit a job. We also need an admin user to be able to login to the backend.

*Test data*: Test Data are needed for the application to be tested. As a developer, you will write tests to ensure that Jobeet behaves as described in the user stories, and the best way is to write automated tests. So, each time you run your tests, you need a clean database with some fresh data to test on.

*User data*: User data are created by the users during the normal life of the application.

Each time symfony creates the tables in the database, all the data is lost. To populate the database with some initial data, we could create a PHP script, or execute some SQL statements with the mysql program. But as the need is quite common, there is a better way with Symfony2: use a 3rd party library to help us automatically load the data.  In this case we will use a the bundle *DoctrineFixturesBundle*.  To automatically include an external library in Symfony2, we just need to add it into our composer.json file and composer will download the source, setup autoloading and prepare the codebase to be used in our project.  

First add the requirement in composer.json
```
# composer.json
   "require": {
      ....
      "doctrine/doctrine-fixtures-bundle": "dev-master"
   },
```

Next tell composer to update our project
```
php composer.phar update
```

Our new library is now installed into our project, but we need to tell Symfony2 to make it available for use:
```
# app/AppKernel.php
// ...
public function registerBundles()
{
    $bundles = array(
        // ...
        new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle(),
        // ...
    );
    // ...
}
```

Now that we have our new bundle ready, lets create our classes to import the data.  Create a new directory called 'Datafixtures/ORM' in our bundle, and create 2 new bundles
```
# src/Jobeet/JobBundle/DataFixtures/ORM/LoadCategoryData.php
<?php

namespace Jobeet\JobBundle\DataFixtures\ORM;

use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;
use Jobeet\JobBundle\Entity\Category;

class LoadCategoryData extends AbstractFixture implements OrderedFixtureInterface
{
    /**
     * {@inheritDoc}
     */
    public function load(ObjectManager $manager)
    {
        // Our array of categories to create
        $categories = array('Design', 'Programming', 'Manager', 'Administrator');

        foreach ($categories as $category_name)
        {
            $category = new Category();
            $category->setName($category_name);
            $manager->persist($category);

            // Create a reference so that we can use this category to link to other entities
            $this->addReference('category-' . strtolower($category_name), $category);

        }

        // Save all new entities to the database
        $manager->flush();
    }

    /**
     * {@inheritDoc}
     */
    public function getOrder()
    {
        return 100; // the order in which fixtures will be loaded
    }    
}

# src/Jobeet/JobBundle/DataFixtures/ORM/LoadJobData.php
<?php

namespace Jobeet\JobBundle\DataFixtures\ORM;

use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;
use Jobeet\JobBundle\Entity\Job;

class LoadJobData extends AbstractFixture implements OrderedFixtureInterface
{
    /**
     * {@inheritDoc}
     */
    public function load(ObjectManager $manager)
    {
        $job_sensio_labs = new Job();
        $job_sensio_labs->setCategory( $manager->merge($this->getReference('category-programming')));
        $job_sensio_labs
            ->setType('full-time')
            ->setCompany('Sensio Labs')
            ->setLogo('sensio-labs.gif')
            ->setUrl('http://www.sensiolabs.com/')
            ->setPosition('Web Developer')
            ->setLocation('Paris, France')
            ->setDescription('You\'ve already developed websites with symfony and you want to work with Open-Source technologies. You have a minimum of 3 years experience in web development with PHP or Java and you wish to participate to development of Web 2.0 sites using the best frameworks available.')
            ->setHowToApply('Send your resume to fabien.potencier [at] sensio.com')
            ->setIsPublic(true)
            ->setIsActivated(true)
            ->setToken('job_sensio_labs')
            ->setEmail('job@example.com')
            ->setExpiresAt( new \DateTime('2010-10-10'));
        $manager->persist($job_sensio_labs);
        $this->addReference('job_sensio_labs', $job_sensio_labs);

        $job_extreme_sensio = new Job();
        $job_extreme_sensio->setCategory( $manager->merge($this->getReference('category-design')));
        $job_extreme_sensio
            ->setType('part-time')
            ->setCompany('Extreme Sensio')
            ->setLogo('extreme-sensio.gif')
            ->setUrl('http://www.extreme-sensio.com/')
            ->setPosition('Web Designer')
            ->setLocation('Paris, France')
            ->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolorin reprehenderit in')
            ->setHowToApply('Send your resume to fabien.potencier [at] sensio.com')
            ->setIsPublic(true)
            ->setIsActivated(true)
            ->setToken('job_extreme_sensio')
            ->setEmail('job@example.com')
            ->setExpiresAt( new \DateTime('2010-10-10'));
        $manager->persist($job_extreme_sensio);
        $this->addReference('job_extreme_sensio', $job_extreme_sensio);

        // Save all new entities to the database
        $manager->flush();
    }

    /**
     * {@inheritDoc}
     */
    public function getOrder()
    {
        return 200; // the order in which fixtures will be loaded
    }    
}
```

Each time we want to re-import our data fixtures, we can run the following command:
```
php app/console doctrine:fixtures:load
```


### See it in Action in the Browser

Symfony is able to automatically generate a crud (Create Read Update and Delete) for a given model that provides basic manipulation features:
```
php app/console doctrine:generate:crud  --entity=JobeetJobBundle:Job --route-prefix=job --with-write=yes --format=yml
php app/console server:run
```

Now open your browser to http://localhost:8000/job/ to see your CRUD application running.  
The doctrine:generate:crud generates a job controller in our JobeetJob bundle. As with most symfony tasks, some files and directories have been created for you under the src/Jobeet/JobBundle directory.  Specifically checkout the Resources/views/Job/ and Controller/ directories.



## See you Tomorrow

That's all for today. I have warned you in the introduction. Today, we have barely written PHP code but we have a working web module for the job model, ready to be tweaked and customized. Remember, no PHP code also means no bugs!

If you still have some energy left, feel free to read the generated code for the module and the model and try to understand how it works. If not, don't worry and sleep well, as tomorrow, we will talk about one of the most used paradigm in web frameworks, the [http://en.wikipedia.org/wiki/Model-view-controller](MVC design pattern).



