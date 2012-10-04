Practical Symfony: Day 5
########################

## The Routing
If you've completed day 4, you should now be familiar with the MVC pattern and it should be feeling like a more and more natural way of coding. Spend a bit more time with it and you won't look back. To practice a bit yesterday, we customized the Jobeet pages and in the process, also reviewed several Symfony2 concepts, like the layout, twig, and entties.

Today we will dive into the wonderful world of the Symfony2 routing framework.

### URLs

If you click on a job on the Jobeet homepage, the URL looks like this: /job/1/show. If you have already developed PHP websites, you are probably more accustomed to URLs like /job.php?id=1. How does Symfony2 make it work? How does Symfony2 determine the action to call based on this URL? Why is the $id parameter passed into the action method contain the primary key? Today, we will answer all these questions.

But first, let's talk about URLs and what exactly they are. In a web context, a URL is the unique identifier of a web resource. When you go to a URL, you ask the browser to fetch a resource identified by that URL. So, as the URL is the interface between the website and the user, it must convey some meaningful information about the resource it references. But "traditional" URLs do not really describe the resource, they expose the internal structure of the application. The user does not care that your website is developed with the PHP language or that the job has a certain identifier in the database. Exposing the internal workings of your application is also quite bad as far as security is concerned: What if the user tries to guess the URL for resources he does not have access to? Sure, the developer must secure them the proper way, but you'd better hide sensitive information.

URLs are so important in Symfony2 that it has an entire framework dedicated to their management: the routing framework. The routing manages internal URIs and external URLs. When a request comes in, the routing parses the URL and converts it to an internal URI.

You have already seen the internal URI of the job page in the show.html.twig template:
```
'job_show', { 'id': entity.id}
```
The path helper converts this internal URI to a proper URL
```
/job/2/show
```

The internal URI is made of several parts: job is the module, show is the action and the query string adds parameters to pass to the action. The generic pattern for internal URIs is:
```
MODULE/ACTION?key=value&key_1=value_1&...
```
As the Symfony2 routing is a two-way process, you can change the URLs without changing the technical implementation. This is one of the main advantages of the front-controller design pattern.

### Routing Configuration
The mapping between internal URIs and external URLs is done in the application routing.yml configuration file:
```
# app/config/routing.yml
jobeet_job:
    resource: "@JobeetJobBundle/Resources/config/routing.yml"
    prefix:   /
```
You can see this includes a resource file of our bundle
```
# src/Jobeet/JobBundle/Resources/config/routing.yml
JobeetJobBundle_job:
    resource: "@JobeetJobBundle/Resources/config/routing/job.yml"
    prefix:   /job
```
Which also includes a resource file for our jobs actions
```
# src/Jobeet/JobBundle/Resources/config/routing/job.yml
job:
    pattern:  /
    defaults: { _controller: "JobeetJobBundle:Job:index" }

job_show:
    pattern:  /{id}/show
    defaults: { _controller: "JobeetJobBundle:Job:show" }

job_new:
    pattern:  /new
    defaults: { _controller: "JobeetJobBundle:Job:new" }

job_create:
    pattern:  /create
    defaults: { _controller: "JobeetJobBundle:Job:create" }
    requirements: { _method: post }

job_edit:
    pattern:  /{id}/edit
    defaults: { _controller: "JobeetJobBundle:Job:edit" }

job_update:
    pattern:  /{id}/update
    defaults: { _controller: "JobeetJobBundle:Job:update" }
    requirements: { _method: post }

job_delete:
    pattern:  /{id}/delete
    defaults: { _controller: "JobeetJobBundle:Job:delete" }
    requirements: { _method: post }
```

Why so many different files? This gives us the ability to manage our application routing at different levels and yet provides a logical seperation of areas to define urls.

The routing files describe routes. A route has a prefix (indicated by it's parent routing file), a name (job_show), a pattern (/{id}/show), and some parameters (identified by surrounding { }).

When a request comes in, the routing tries to match a pattern for the given URL. The first route that matches wins, so the order in routing.yml is important. Let's take a look at some examples to better understand how this works.

When you request the url /job/1/show, the first route that matches is the "job_show" route.  Since our bundle routing file specifies that routing/job.yml routes be prefixed with "/job", we look for a route pattern of "{id}/show".  Once we have matched our route, we can see what action should be executed by the defaults entry for that route.  In our case of the "job_show" route, we will execute the showAction method inside Controller/JobController.php.

=== Route Customizations
We can endlessly customize the URL for pages with little effort.  Instead of this url:
```
/job/1/show
```
We can customize it to be more friendly like:
```
/job/sensio-labs/paris-france/1/web-developer/show
```

Edit our routing/job.yml file:
```
# src/Jobeet/JobBundle/Resources/config/routing/job.yml
job_show:
    pattern:  /{company}/{location}/{id}/{position}/show
    defaults: { _controller: "JobeetJobBundle:Job:show" }
```

Then we can tell our controller to accept the parameters
```
# src/Jobeet/JobBundle/Controller/JobController.php
    public function showAction($company, $location, $id, $position)
```

And in our URLs pass in the parameters
```
# src/Jobeet/JobBundle/Resources/views/Job/index.html.twig
{{ path('job_show', { 'company': entity.company, 'position': entity.position, 'location': entity.location, 'id': entity.id }) }}">{{ entity.company }}
```

### Route Debugging
Sometimes all of the different routing files and pattern matching can become confusing.  When this happens we can use the debug command to dump all of the routes for our application.
```
php app/console routing:debug
```

### See you Tomorrow

Today was packed with a lot of new information. You have learned how to use the routing framework of Symfony2 and how to decouple your URLs from the technical implementation.

Tomorrow, we won't introduce any new concepts, but rather spend time going deeper into what we've covered so far.


