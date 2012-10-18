Practical Symfony: Day 7
########################

Yesterday you expanded your knowledge of Symfony2 in a lot of different areas: querying with Doctrine, fixtures, debugging, services and custom configuration. And we finished with a little challenge for today.

I hope you worked on the Jobeet category page as today's tutorial will then be much more valuable for you.

Ready? Let's talk about a possible implementation

### The Category Route
First, we need to add a route to define a pretty URL for the category page. Previously Symfony2 split our url's associated to jobs into it's own routing file (src/Jobeet/JobBundle/Resources/config/routing/job.yml), so following that idea of keeping our routing files clean, lets create a new category.yml routing file

```
# src/Jobeet/JobBundle/Resources/config/routing.yml
JobeetJobBundle_category:
    resource: "@JobeetJobBundle/Resources/config/routing/category.yml"
    prefix:   /category
```

And in our new category routing file, lets create a route for a specific category.
```
# src/Jobeet/JobBundle/Resources/config/routing/category.yml
category:
    pattern:  /{slug}
    defaults: { _controller: "JobeetJobBundle:Category:show" }
```

The pattern defined by the category route acts like /category/* where the wildcard is given the name slug. For the URL /category/programming, the slug variable gets a value of programming, which is available for you to use in your controller (keep reading).  The _controller parameter is a special key that tells Symfony which controller should be executed when a URL matches this route. The _controller string is called the logical name. It follows a pattern that points to a specific PHP class and method, which we need to create:

```
// src/Jobeet/JobBundle/Controller/CategoryController.php
namespace Jobeet\JobBundle\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class CategoryController extends Controller
{
    public function showAction($slug)
    {
    	$category = $this->container->get('jobeet_job.category_manager')->findBySlug($slug);
        
        if (!$category) {
            throw $this->createNotFoundException('Unable to find Category.');
        }

        return $this->render('JobeetJobBundle:Category:show.html.twig', array(
			'category' 	=> $category
		));    	
	}
}
```

And of course here's our view template
```
// src/Jobeet/JobBundle/Resources/views/Category/show.html.twig
{% extends '::base.html.twig' %}
{% block title %}{{category.name}} Jobs{% endblock %}

{% block body %}
<h1>Jobs in the {{ category.name }} category</h1>

<table class="records_list table table-striped table-bordered table-hover">
    <tbody>
    {% for entity in jobs %}
        <tr>
            <td>{{ entity.location }}</td>
            <td>{{ entity.position }}</td>
            <td><a href="{{ path('job_show', { 'company': entity.company, 'position': entity.position, 'location': entity.location, 'id': entity.id }) }}">{{ entity.company }}</a></td>
        </tr>
    {% endfor %}
    </tbody>
</table>
{% endblock %}
```

Now we can link to the new category pages from our homepage
```
// src/Jobeet/JobBundle/Resources/views/Job/index.html.twig
<h2><a href="{{ path('category', { 'slug': category.name }) }}">{{ category.name }}</a></h2>

### Category Service
Now that we have our new Category routes, controller, and template, the only thing missing is our jobeet_job.category_manager service.  We will eventually create methods to handle all of our category services, but for now we just create our findBySlug method.

Lets define our service
```
# src/Jobeet/JobBundle/Resources/config/services.yml
parameters:
  ...
  jobeet_job.CategoryManager.class: Jobeet\JobBundle\Model\CategoryManager

services:
  ...

  jobeet_job.category_manager:
    class: %jobeet_job.CategoryManager.class%
    arguments: [ @doctrine.orm.entity_manager ]
```

Lets create our service class
```
// src/Jobeet/JobBundle/Model/CategoryManager.php
namespace Jobeet\JobBundle\Model;

class CategoryManager
{
	// Our entity manager
	private $em;

	/** 
	 * __construct
	 * @param (object) entityManager 
	 * @return null
	 */
	public function __construct( $entityManager )
	{
		$this->em = $entityManager;
	}

	/** 
	 * findBySlug
	 * @param (string) Slug value
	 */
	public function findBySlug ($slug)
	{
		$repository = $this->em->getRepository('JobeetJobBundle:Category');
		$query = $repository->createQueryBuilder('c')
			->where('c.name = :name')
			->setParameter('name', $slug )
			->getQuery();
		try {			
			return $query->getSingleResult();
    	} catch (\Doctrine\Orm\NoResultException $e) {
    		return false;
    	}			

	}
```

Since we now have a CategoryManager, we should place our getCategoriesWithJobs method that was previously in JobManager into our CategoryManager.  

### Including other templates
Notice that we have copied and pasted the <table> tag that create a list of jobs from the Job/index.html.twig template. That's bad. Time to learn a new trick. When you need to reuse some portion of a template, you need to isolate the piece in it's own template and include those.  This way we can share our HTML code across many pages but only have 1 place to manage code and changes in the future.

Lets create our shared template:
```
# src/Jobeet/JobBundle/Resources/views/Job/jobs_list.html.twig
<table class="records_list table table-striped table-bordered table-hover">
    <tbody>
    {% for entity in jobs %}
        <tr>
            <td>{{ entity.location }}</td>
            <td>{{ entity.position }}</td>
            <td><a href="{{ path('job_show', { 'company': entity.company, 'position': entity.position, 'location': entity.location, 'id': entity.id }) }}">{{ entity.company }}</a></td>
        </tr>
    {% endfor %}
    </tbody>
</table>
```

In both our Category/show.html.twig, we should replace this table snippit with our include tag:
```
{% include 'JobeetJobBundle:Job:jobs_list.html.twig' with {'jobs': jobs} %}
```

And since our jobs parameter is different on our Job:index.html.twig
```
{% include 'JobeetJobBundle:Job:jobs_list.html.twig' with {'jobs': category.active_jobs} %}
```

Great! We are reusing HTML templates across many pages.

### List Pagination

From day 2 requirements:

"The list is paginated with 20 jobs per page."

We've used Composer on our first day to download and setup Symfony2, but we haven't included any other libraries.  Here is our chance!  Using the (packagist)[https://packagist.org] site, we found that there is a bundle to help with our pagination problem, KnpPaginatorBundle.  Let's use this library bundle to add pagination to our site. 

Include in our composer.js file: 
```
    require: {
        "knplabs/knp-paginator-bundle": "dev-master"
    }
```

Now lets actually download and setup our new bundle:
```
php ../composer.phar update
```

Now we just include the bundle into our AppKernel
```
<?php
    // File: app/AppKernel.php
    public function registerBundles()
    {
        return array(
            // ...
            new Knp\Bundle\PaginatorBundle\KnpPaginatorBundle(),
            // ...
        );
    }
```

Now we are ready to use the new paginator service.  First lets add the paginator as an injected service to our job_manager
```
# src/Jobeet/JobBundle/Resources/config/services.yml
    arguments: [ @doctrine.orm.entity_manager, %jobeet_job.default_active_days%, @knp_paginator ]
```

And allow the parameter in our JobManager service as well as modify our getActiveJobs method to accept page # and limit.
```
    private $paginator;

    public function __construct( $entityManager, $active_days, $paginator )
    {
        ...
        $this->paginator = $paginator;
    }

    public function getActiveJobs( Category $category = null, $page = null, $limit = null )
    {
        $repository = $this->em->getRepository('JobeetJobBundle:Job');
        $qb =$repository->createQueryBuilder('j');
        $qb->where('j.expires_at > :expires')
                ->setParameter('expires', new \DateTime() );

        if (null !== $category) {
            $qb->andWhere('j.category = :id')
                ->setParameter('id', $category->getId() );
        }

        $jobs = $this->paginator->paginate($qb, $page, $limit); 

        return $jobs;
        
    }    
```

We don't have any pagination parameters in our routing, so we need to add an optional value.  If we include our new page parameter in our defaults object, then it becomes optional in the URL and by default the value is set to 1.
```
# src/Jobeet/JobBundle/Resources/config/routing/category.yml
category:
    pattern:  /{slug}/{page}
    defaults: { _controller: "JobeetJobBundle:Category:show", page: 1 }
```

Now just use a KnpPaginatorBundle Twig helper to render our pagination in html
```
<div class="navigation">
    {{ jobs.render()|raw }}
</div>
```

We can pass our page value into our job_manager service via the controller by adding our new parameter into our showAction method.
```
    public function showAction($slug, $page)
    {
        $category = $this->container->get('jobeet_job.category_manager')->findBySlug($slug);
        
        if (!$category) {
            throw $this->createNotFoundException('Unable to find Category.');
        }

        $jobs = $this->container->get('jobeet_job.job_manager')->getActiveJobs( $category, $page, 5 );
``` 

### See you Tomorrow

If you worked on your own implementation yesterday and feel that you didn't learn much today, it means that you are getting used to the Symfony2 philosophy. The process to add a new feature to a symfony website is always the same: think about the URLs, create some actions, update the service, and write some templates. And, if you can apply some good development practices to the mix, you will become a Symfony2 master very fast.

Tomorrow will be the start of a new week for Jobeet. To celebrate, we will talk about a brand new topic: tests.

