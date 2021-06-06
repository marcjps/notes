
# Symfony


Its a server side web application framework.  It provides routing, view templating, ORM, security, plus a bunch of surrounding utilities like dev environment, package mangement, console, etc. 

I don't think I like it much, like many frameworks, it disconnects us from what's happening underneath.  It's the kind of framework Microsoft would write.  The forms stuff is actually strongly reminiscent of Drupal 7 forms.


## Environment

* Twig is view templates
* Doctrine is ORM
* Flex is a package manager for Symfony recepies.  Find them at https://flex.symfony.com/


# Set up

## Create a project

composer create-project symfony/skeleton MyProject
composer create-project symfony/website-skeleton MyProject

* skeleton is a blank project and you have to install any features you want
* website-skeleton includes view templating, annotations, etc


## Folder structure

* config = configuration - bundles = packages installed by flex, routes, etc. 
* src = controllers, services
* templates = views
* Entity = doctrine entity classes

## Run debug server (if not using apache etc)

```text
composer require server
php bin/console server:run
```

Or run it in the background:

```text
php bin/console server:start
php bin/console server:stop
```


## List symfony console commands

```text
php bin/console
```

## Install "make" scaffolder/generator

```text
composer require make
composer require annotations
```

## Dump function 

This prints the contents of an object nicely in the HTML page.

```text
composer require var-dumper
```

Then in the code:

```php
dump($thing)
```

## Profiler toolbar

The profiler toolbar is a bar that shows up on all pages of the website and includes useful debug information. (such as which route you are accessing, and whether you are logged in)

To enable the profiler bar run:

```text
composer require profiler --dev
```

That will install the package and add configuration that causes it to be displayed during development.

# Controllers and views

## Create a controller

```text
php bin/console make:controller MainController
```

* A PHP class in src/Controller folder
* Extends AbstractController
* @Route annotation specifies URI.  You could alternatively define the routes in config/routes.yaml.
* You can put a route on the class (as a prefix) and on the methods.
* Actions in controllers ought to return Symfony\Component\HttpFoundation\Response.
* On Apache the URL becomes site/public/index.php/main.  We could fix that with an URL rewrite.
* If views are installed the make:controller command also creates a view.

There's a lot of dependency injection tricks.

Retrieve the parameter from the request object.

```php
/**
 * @Route("/show/{id}", name="show")
 */
 declare function show(Request $request, PostRepository $postRepository) {
    $id =  $request->get("id"); 
    $post = $postRepository->find($id);
    return $this->render( 'show.html.twig', [ 'post' => $post ] );
 }
```

Have the parameter passed in.

```php
/**
 * @Route("/show/{id}", name="show")
 */
 declare function show($id, PostRepository $postRepository) {
    $post = $postRepository->find($id);
    return $this->render( 'show.html.twig', [ 'post' => $post ] );
 }
```

Have the post found and passed in automatically.  Yikes.

```php
/**
 * @Route("/show/{id}", name="show")
 */
 declare function show(Post $post) {
    return $this->render( 'show.html.twig', [ 'post' => $post ] );
 }
```

To return a redirect:

```php
return this->redirect($this->generateUrl(route: 'post.index'));
return new RedirectResponse($this->urlGenerator->generate('post.index'));
```

Note: $request->files contains details of files uploaded in file upload fields.


To list the routes:

```text
php bin/console debug:router
```

## Call a view

```text
composer require template
```

* To render a view from the controller

```php
return $this->render( 'hello.html.twig', [ 'foo' => 'bar' ] );
```

* First parameter is the twig template name
* Second parameter is a key-value array of the data that will be available to the template.
* render() returns a Response.
* There's an alternate renderView which returns the rendered content (probably as a string)

## Create a view template

Create the hello.html.twig file in the templates folder.  
Output a variable:

```text
{{ name }}
```

The variables have special characters escaped by default (but you can turn it off with the raw filter)

Use the path helper to create the URL based on the function name:

```html
<a href="{{ path('controller_function_name', {function_parameter_name: 'parameter_value' }) }}">Click</a>
```

e.g.

```html
<a href="{{ path('post.show', {id: post.id }) }}">Click</a>
```

Create a loop:

```html
{% for post in posts %}
    <p><a href="{{ path('post.show', {id: post.id }) }}">{{ post.title }}</a></p>
{% endfor %}
```

Note the annoying different usages of {% %} and {{ }}

## Layouts

* From inside the template you can reference a parent layout 

```html
{% extends('base.html.twig') %}
```

* The child template needs to define the block where it fits in the parent using 

```html
{% block 'body' %} 
<p>my content</p>
{% endblock %}
```

## Show a banner 

In the controller:

```php
$this->addFlash(type:"success", message:"Hello");
```

I guess this is a default feature of the controller parent class.

In the view template you need to paste in some code that iterates through app.flashes and renders each banner in a div.  

# Doctrine ORM

Install:

```text
composer require orm
```

* Config is in config/doctrine.yaml
* But it references the DATABASE_URL which is configured in the .env file in the project root.

Running this command will try to create the database on the configured database server.

```text
php bin/console doctrine:database:create
```

Make an entity class:

```text
php bin/console make:entity 
```

* Then input class name and fields when prompted
* It will create the entity class in the Entity folder with the names and fields you set
* Each property has an annotation that specifies the database field type
* For each property getters and setters are generated

To update the database to match the entities:

```text
php bin/console doctrine:schema:update (--dump-sql) (--force)
```

* dump-sql shows the sql that will be executed
* force executes the command

If you want you can add other fields to the entity class, then regenerate the getters and setters with:

```text
php bin/console make:entity --regenerate
```

It's possible to add relationships (foriegn keys) to entities.  It looks something like this:

```php
/**
 * @ORM|OneToMany(targetEntity="App|Entity|Post", mappedBy="category")
 */
 private $post;
```

where:

* Post is the foreign entity
* Category might be a corresponding field name on the other side of the relationship (I guess this is helpful on the API at least)
* When one side is Many like that, the autogenerated Getter and Setter will contain array code (to hold many items)


## Save an object

```php
$post = new Post()
post->setTitle('This is the title');
$entityManager = $this->getDoctrine()->getManager();
$entityManager->persist($post);
$entityManager->flush();
```

* Without flush() it will not save.

## Delete an object

```php
$entityManager = $this->getDoctrine()->getManager();
$entityManager->remove($post);
$entityManager->flush();
```

## Retrieve all 

```php
public function index(PostRepository $postRepository) {
    $posts = $postRepository->findAll();
    dump($posts);
    // then also return a Response
}
```

* That shows all the posts in the debug banner.  You could send them to the view to be rendered.
* PostRepository is provided by dependency injection.

You can customise the PostRepository and add extra methods that do other queries.  (e.g. joining then filtering)

# Forms

Forms can be related to a class (but don't have to be).

```text
composer require form validator
php bin/console make:form 
```

Specify the name of the entity.  It generates a class that sets up the form builder based on the fields in the entity.

Rather like Drupal 7, In the form builder class you can:

* Add other fields (like a submit button)
* Add custom attributes to the existing fields (like changing the HTML "class")

## Rendering the form

In the controller:

```php
Post $post = new Post();
$form = $this->createForm(PostType::class, $post);
return $this->render( 'create.html.twig', [ 'form' => $form->createView() ] );
```

Add this to your view:

```php
{{ form(form) }}
```

* PostType is the class that was generated by make:form
* Post is an instance of the entity.

To change the default form rendering to bootstrap add form_theme: bootstrap_4_layout_html.twig to twig.yaml.

## Submitting the form

To add a submit button, change the PostType class to add the save button to the formbuilder after the fields.

Handling the form submit is done by adding code to the same action that rendered the form.  The $post changed by reference even though we didn't retrieve a modified one back from the $form.

```php
Post $post = new Post();
$form = $this->createForm(PostType::class, $post);
if ($form->isSubmitted() && $form->isValid()) {
    $entityManager = $this->getDoctrine()->getManager();
    $entityManager->persist($post);
    $entityManager->flush();
    return this->redirect($this->generateUrl(route: 'post.index'));
}
return $this->render( 'create.html.twig', [ 'form' => $form->createView() ] );
```

To just dump all the data in the form:

```php
dump($form->getData());
```

## Validating the form

In the entity class, add validators by adding Assert decorators to the fields.

```php
/**
 * @Assert|NotBlank() 
 */
private $title;
```

In the controller you can get the errors from the form with $form->getErrors();  

The tutorial doesn't say how we render the errors.  

# Security

Set up some auth.

```php
composer require security
php bin/console make:user
php bin/console make:auth
```

* It creates a user entity
* It includes password hashing when the user is saved on the database
* It creates security.yaml which has the secuity config in it
* Security.yaml specifies which URIs require authentication to visit.
* Security.yaml specifies which route will cause the session to be logged out.  (i.e. a route triggered by a log-out button)
* In the CustomAuthenticator there is a callback where you add a redirect for the user after they log in.

Update the database to create the user table:

```text
php bin/console doctrine:schema:update (--dump-sql) (--force)
```

You then need to put a user in the table.  The password in the table needs to be hashed.   You could try to generate it manually, implement a registration form, or implement a custom console command that will add the user.

In the view you can test if the user is currently logged in with:

```php
{% if is_granted('IS_AUTHENTICATED_FULLY') %}
    ...
{% endif %}
```



# References 

* Video Tutorial https://www.youtube.com/watch?v=Bo0guUbL5uo
* Docs https://symfony.com/doc/current/index.html

