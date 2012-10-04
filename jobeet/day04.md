Practical Symfony: Day 4
========================

## The Controller and the View
Yesterday, we explored how Symfony2 simplifies database management by abstracting the differences between database engines, and by converting the relational elements to nice object oriented classes. We have also played with Doctrine to describe the database schema, create the tables, and populate the database with some initial data.

Today, we are going to customize the basic job bundle we created yesterday. The job bundle already has all the code we need for Jobeet:

* A page to list all jobs
* A page to create a new job
* A page to update an existing job
* A page to delete a job

Although the code is ready to be used as is, we will refactor the templates to match closer to the Jobeet mockups.

### The MVC Architecture
If you are used to developing PHP websites without a framework, you probably use the one PHP file per HTML page paradigm. These PHP files probably contain the same kind of structure: initialization and global configuration, business logic related to the requested page, database records fetching, and finally HTML code that builds the page.

You may use a templating engine to separate the logic from the HTML. Perhaps you use a database abstraction layer to separate model interaction from business logic. But most of the time, you end up with a lot of code that is a nightmare to maintain. It was fast to build, but over time, it's more and more difficult to make changes, especially because nobody except you understands how it is built and how it works.

As with every problem, there are nice solutions. For web development, the most common solution for organizing your code nowadays is the [MVC design pattern](http://en.wikipedia.org/wiki/Model-view-controller). In short, the MVC design pattern defines a way to organize your code according to its nature. This pattern separates the code into three layers:

* The Model layer defines the business logic (the database belongs to this layer). You already know that Symfony2 stores all the classes and files related to the Model in the Entity directory.

* The View is what the user interacts with (a template engine is part of this layer). In Symfony2, the View layer by default is powered by (Twig)[http://twig.sensiolabs.org], a very powerful and simple templating language.  They are stored in Resources/views directory.

* The Controller is a piece of code that calls the Model to get some data that it passes to the View for rendering to the client. When we installed Symfony2 the first day, we saw that all requests are managed by front controllers (app.php and app_dev.php). These front controllers delegate the real work to actions. As we saw yesterday, these actions are logically grouped into Bundles.

![MVC](http://www.symfony-project.org/images/jobeet/1_2/04/mvc.png)

Today, we will use the mockup defined in day 2 to customize the homepage and the job page. We will also make them dynamic. Along the way, we will tweak a lot of things in many different files to demonstrate the Symfony2 directory structure and the way to separate code between layers.

### The Layout
First, if you have a closer look at the mockups, you will notice that much of each page looks the same. You already know that code duplication is bad, whether we are talking about HTML or PHP code, so we need to find a way to prevent these common view elements from resulting in code duplication.

More often than not, templates in a project share common elements, like the header, footer, sidebar or more. In Symfony2, we like to think about this problem differently: a template can be decorated by another one. This works exactly the same as PHP classes: template inheritance allows you to build a base "layout" template that contains all the common elements of your site defined as blocks (think "PHP class with base methods"). A child template can extend the base layout and override any of its blocks (think "PHP subclass that overrides certain methods of its parent class").

#### twBootstrap
Since one of the core ideas of Symfony2 is reusing code, that also includes HTML and CSS.  The (twBootstrap)[http://twitter.github.com/twbootstrap] project is a great design framework that we can use to improve the design and portability of our website.

Lets create a base template that will wrap all of the pages that we are using.
````
# app/Resources/views/base.html.twig
<!DOCTYPE html>
<html>
    <head>
        {% block head %}
            <link href="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.1.1/css/bootstrap-combined.min.css" rel="stylesheet">
            <title>{% block title %}{% endblock %} - Jobeet </title>
        {% endblock %}
    </head>
    <body>
        <div class="navbar">
            <div class="navbar-inner">
                <a class="brand" href="#">Jobeet</a>
                <ul class="nav">
                    <li><a href="{{ path('job') }}">Show Jobs</a></li>
                    <li><a href="{{ path('job_new') }}">Post a Job</a></li>
                </ul>

                <form class="navbar-search pull-left">
                    <input type="text" class="search-query" placeholder="Enter some keywords (city, country, position, ...)">
                </form>
            </div>
        </div>
        <div class="container">
                {% block body %}{% endblock %}
        </div>

        <br/> 

        <div id="footer">
            <div class="navbar navbar-fixed-bottom">

            {% block footer %}
                <a class="brand" href="#">&copy; Copyright 2012 by Jobeet.org</a>
            {% endblock %}

            </div>
        </div>
    </body>
    <script src="//netdna.bootstrapcdn.com/twitter-bootstrap/2.1.1/js/bootstrap.min.js"></script>
</html>
```

Now we will modify our templates that we created yesterday to include our base template so all of our pages look the same.  Here is one example:
```
# src/Jobeet/JobBundle/Resources/views/Job/index.html.twig
{% extends '::base.html.twig' %}
{% block title %}View all{% endblock %}

{% block body %}
  .. include existing page code here
{% endblock %}
```

### The Job Homepage

As seen in day 3, the job homepage is generated by the index action of the job controller. The index action is the Controller part of the page and the associated template, Resources/views/job/index.html.twig, is the View part:
```
src/
  Jobeet/
    JobBundle/
      Controller/
        JobController.php
      Resources/
        views/
          Job/ 
            edit.html.twig
            index.html.twig
            new.html.twig
            show.html.twig
```

### The Action

Each action is represented by a method of a class. For the job homepage, the class is JobController (the name of the module suffixed by Controller) and the method is indexAction() (the name of the action suffixed by Action). It retrieves all the jobs from the database:
```
    public function indexAction()
    {
        $em = $this->getDoctrine()->getManager();

        $entities = $em->getRepository('JobeetJobBundle:Job')->findAll();

        return $this->render('JobeetJobBundle:Job:index.html.twig', array(
            'entities' => $entities,
        ));
    }
```

### The Template
In the template code, the for iterates through the list of Job objects ($entities), and for each job, each column value is output. Remember, accessing a column value is as simple as calling an accessor method which name begins with a dot (.) and the camelCased column name (for instance the .location method for the location column).

Let's clean this up a bit to only display a sub-set of the available columns:
```
{% extends '::base.html.twig' %}
{% block title %}View all{% endblock %}

{% block body %}
<h1>Job list</h1>

<table class="records_list table table-striped table-bordered table-hover">
    <tbody>
    {% for entity in entities %}
        <tr>
            <td>{{ entity.location }}</td>
            <td>{{ entity.position }}</td>
            <td><a href="{{ path('job_show', { 'id': entity.id }) }}">{{ entity.company }}</a></td>
        </tr>
    {% endfor %}
    </tbody>
</table>

{% endblock %}
```

The {{ path() }}  function call in this template is a Twig helper that we will discuss tomorrow.

=== The Job Page Template
Now let's customize the template of the job page. Open the show.html.twig file and replace its content with the following code:
```
#src/Jobeet/JobBundle/Resources/views/Job/show.html.twig
{% extends '::base.html.twig' %}
{% block title %}{{ entity.company }} - {{ entity.position}} {% endblock %}

{% block body %}

<h1>{{ entity.company }}</h1>
<h2>{{ entity.location }}</h2>
<h3>{{ entity.position }} 
        <small>({{ entity.type }})</small>
</h3>

<div class="description">
        {{ entity.description }}
</div>

<h4>How to apply?</h4>
<p class="how-to-apply">{{ entity.howtoapply }}</p>

<a href="{{ path('job_edit', { 'id': entity.id }) }}">
    Edit
</a>
{% endblock %}
```

=== The Job Page Action
The job page is generated by the show action, defined in the showAction() method of the job controller:
```
    /**
     * Finds and displays a Job entity.
     *
     */
    public function showAction($id)
    {
        $em = $this->getDoctrine()->getManager();

        $entity = $em->getRepository('JobeetJobBundle:Job')->find($id);

        if (!$entity) {
            throw $this->createNotFoundException('Unable to find Job entity.');
        }

        $deleteForm = $this->createDeleteForm($id);

        return $this->render('JobeetJobBundle:Job:show.html.twig', array(
            'entity'      => $entity,
            'delete_form' => $deleteForm->createView(),        ));
    }
```
As in the index action, the JobeetJobBundle:Job entity class is used to retrieve a job, this time by using the find() method. The parameter of this method is the unique identifier of a job, its primary key. The next section will explain why the $id parameter is passed into the showAction() method.

If the job does not exist in the database, we want to forward the user to a 404 page, which is exactly what the createNotFoundException exception does. 

=== See you Tomorrow

Today, we have described some design patterns used by Symfony2. Hopefully the project directory structure now makes more sense. We have played with the templates by manipulating the layout and the template files. We have also made them a bit more dynamic thanks to slots and actions.



