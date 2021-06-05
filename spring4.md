# Spring

#### Spring Boot MVC project

I'm following this guide to get started with a Spring Boot MVC application: https://spring.io/guides/gs/serving-web-content/  

The Maven archetype we need to start from is as follows:

* Group id: org.springframework.boot
* Artifact id: spring-boot-starter-parent
* Version id: 1.3.2.RELEASE
* Repo: http://central.maven.org/maven2/

To get started with this, we can create a pom.xml and a couple of extra files to make a working web site.

A basic pom.xml for Spring Boot and MVC looks like this:

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	    <groupId>foo</groupId>
	    <artifactId>bar</artifactId>
	    <version>0.0.1-SNAPSHOT</version>
	    <parent>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-parent</artifactId>
	        <version>1.3.2.RELEASE</version>
	    </parent>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-thymeleaf</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-devtools</artifactId>
	        </dependency>
	    </dependencies>
	    <build>
	        <plugins>
	            <plugin>
	                <groupId>org.springframework.boot</groupId>
	                <artifactId>spring-boot-maven-plugin</artifactId>
	            </plugin>
	        </plugins>
	    </build>
	</project>

That's most of our configuration file editing done with.

/src/main/java/Application.java is our main application class.  If we run this main class directly, SpringApplication.run will instantiate a self contained Tomcat to serve the site.

	@SpringBootApplication
	public class Application {

	    public static void main(String[] args) {
	        SpringApplication.run(Application.class, args);
	    }

	}

#### Static pages, controller and view

/src/main/resources/static/index.html can be a static HTML page that it will serve as the root page of the website.  It seems to pick up this location by default.

We can create an MVC controller.  It posts some attributes into a model and then returns the name of the view to use.

	@Controller
	public class GreetingController {

	    @RequestMapping("/greeting")
	    public String greeting(@RequestParam(value="name", required=false, defaultValue="World") String name, Model model) {
	        model.addAttribute("name", name);
	        return "greeting";
	    }
	}

/src/main/resources/templates/greeting.html contains the view template.  It will display a page and include the data we passed in.  The template engine is called Thymeleaf.  It is rather glorious.  We'll look at it more later.

	<!DOCTYPE HTML>
	<html xmlns:th="http://www.thymeleaf.org">
	<head>
	    <title>Getting Started: Serving Web Content</title>
	    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	</head>
	<body>
	    <p th:text="'Hello, ' + ${name} + '!'" />
	</body>
	</html>

We can launch the application at the command line using this command:

	mvn spring-boot:run

It will start Tomcat and host the pages on http://localhost:8080


#### Getting that going in Netbeans

I'm running Netbeans and my whole environment in Docker.   The command to start it is:

	docker run -d --net="host" -it -e DISPLAY=localhost:1.0 -e "TZ=America/Chicago" --name netbeans2 -v ~/projects/netbeans:/root/NetBeansProjects:z psharkey/netbeans-8.1

The project will run in Netbeans if you  open it.  You may need to ensure that Netbeans runs the mvn spring-boot:run command when it starts debugging the project.

To create the project from scratch in Netbeans, we need to use a plugin called NB Spring Boot.  The steps are:

* Install the NB Spring Boot Plugin
* File > New Project > Maven > Spring Boot basic project
* Give it a name
* Edit the Project Files > pom.xml and insert the Thymeleaf dependency.  When we add this it causes Spring Boot to run the project in Tomcat.


	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-thymeleaf</artifactId>
	</dependency>


* Add /src/main/resources/static/index.html so there's a page for it to serve
* Click debug.  It should run.

#### Running on an existing Tomcat

To get this to work in Tomcat, I had to change the Application.java file a bit.  The new file looks like this:

	@SpringBootApplication
	public class BasicApplication extends SpringBootServletInitializer {
	  public static void main(String[] args) {
	      SpringApplication.run(BasicApplication.class, args);
	  }

	  @Override
	  protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
	      return builder.sources(BasicApplication.class);
	  }
	}

Additionally, in pom.xml we change the packaging tag to build either a jar or a war.  "jar" (java application archive) makes it run the site in a self contained server, and "war" will build a war (web application archive) which works on a standalone server.

    <packaging>war</packaging>

Once the Application.java and packaging tag are changed, the application can build and deploy to Tomcat.


#### Exploring the Thymeleaf web templating engine

We saw in the previous example that we can pass data into the template by calling model.addAttribute in the controller.  This will work for various data types, including our own objects.

Here's a couple of different ways of outputting a variable in the template:

    <p th:text="'Hello, ' + ${name} + '!'" />
    <p th:inline='text'>Hello [[${name}]]!</p>

If we pass in an object, we can call the public get methods that it exposes.  In this case anEntity is a Plain Old Java Object with no special annotations, which was passed into model.addAttribute.

    <p th:inline='text'>A field from the entity: [[${anEntity.theString}]]</p>
    <p th:inline='text'>Another field from the entity: [[${anEntity.getTheNumber()}]]</p>

We can call some formatting functions to render data types.  Here's how to do some formatting on a number:

    <p th:inline="text">The really big number is: [[${#numbers.formatDecimal(reallybignumber,  0, 'COMMA', 0, 'POINT')}]] </p>

Here's how to do formatting on a date:

    <p th:inline='text'>The date is [[${#dates.format(thedate, 'dd/MM/yyyy')}]]</p>

Here's are some conditional inclusions of an element:

	<p th:if="${theboolean}">By the way, the boolean is true</p>
	<li th:if="${number} &gt; 50">The number is greater than 50</li>

Here's some code to loop through a list and output each item.  The "items" passed from the model is a Java.util.ArrayList<String>

    <ul>
        <li th:each="item : ${items}" th:inline='text'>This list items says: [[${item}]]</li>
    </ul>

The Thymeleaf namespace declaration on the root element is required.  If we miss it out, it doesn't work.  It also doesn't like any unclosed meta tags in the HTML fragment.  Due to these factors I suspect it might be processing the templates as XML files.

The Thymeleaf rabbit hole goes really deep.  It's pretty cool though.

#### Includes

Thymeleaf implements support for including shared html fragments defined in separate HTML files.

There are various different ways they can be used, but here's a simple example.

/src/main/resources/templates/fragments/header.html is the fragment we will include.  It looks like this:

	<html>
	    <head>
	        <title>Nothing</title>
	    </head>
	    <body>
	        <div th:fragment='header'>
	            <h1>Marc's Spring Site</h1>
	            <p th:inline='text'>I am the fragment saying something from the model: [[${name}]]!</p>
	        </div>
	    </body>
	</html>

The template which includes the fragment simply looks like this:

    <div th:replace="fragments/header :: header"></div>

In this scenario, the "header" div from the "fragments/header" page is placed into the template, replacing the div with the th:replace attribute.

If we wish we can call in the entire HTML fragment instead of just one element, by removing the ":: header" selector from the template.

The included fragment does not have any controller logic of its own, which is kind of a shame, as this does not allow for reusable components along the lines of the reusable directives used in AngularJS.

However, the included HTML fragment can access data in the model passed into the template.  So at least we could pass in a fairly complex data model into the template, which includes some data needed used by the fragments and some data used by the template.





#### Capturing a form submission

Here's a simple form submission using an entity, based on this guide: https://spring.io/guides/gs/handling-form-submission/

We create a Plain Old Java Object to hold the data posted by the form.  I called it FormPostModel.

In the get method in the controller, we're sending in an empty FormPostModel object as an attribute.  In the post method we're picking up the posted object using the model and then adding it back as an attribute for the outgoing page.  

    @RequestMapping(path = "userform", method = RequestMethod.GET)
    public String userFormGet(Model model) {
        model.addAttribute("FormPostModel", new FormPostModel());
        return "userform";
    }
    
    @RequestMapping(path = "userform", method = RequestMethod.POST)
    public String userFormPost(FormPostModel FormPostModel, Model model) {
        
        model.addAttribute("FormPostModel", FormPostModel);
        return "userform_result";
    }

The form view page is binding the fields to the blank object that we sent in.

	<form action="#" th:action="@{/userform}" th:object="${FormPostModel}" method="post">
	    <ul>
	        <dt>Username</dt>
	        <dd><input type='text' name='username' th:field='*{username}'/></dd>
	        <dt>Password</dt>
	        <dd><input type='password' name='password' th:field='*{password}'/></dd>
	    </ul>
	    <input type='submit' value='Submit'/>
	</form>


The result view just outputs the values.

        <p th:inline='text'>Username: [[${FormPostModel.Username}]]</p>
        <p th:inline='text'>Password: [[${FormPostModel.Password}]]</p>


#### Form validation

It took quite a lot of hacking around to get this to work correctly, but eventually everything came into alignment.  I loosely followed the guide here: https://spring.io/guides/gs/validating-form-input/

We can add simple validation annotations into the model class, on the member variables.

	public class FormPostModel {
	 
	    @NotNull
	    @Size(min=2, max=30)
	    String username;
	    
	    @NotNull
	    @Min(10)
	    String password;
	    ...
	}

I learned we could change the controllers' Get method slightly, and pass in the form POJO instead of passing in the Model object (then adding the POJO as an attribute)  The annotation @ModelAttribute says that we want this object added to the model as an attribute.  The name passed in gives the object a name.  This name is referenced as the incoming data object for the form tag in the view.  This is not much different, but interesting.

    @RequestMapping(path = "userform", method = RequestMethod.GET)
    public String userFormGet(@ModelAttribute("marcsForm") FormPostModel formPostModel) {
        return "userform";
    }

Things are a lot different on the controllers' post method.  We're passing in the POJO as above, with the extra annotation @Valid, which evidently tells it to perform the validation.  A BindingResult must be the next parameter after the @Valid POJO object.  The bindingResult contains the validation errors for the preceding object.

Inside the controller we can test for errors, and return the original form view if the errors are there.  The POJO is going to go back into the view because of the @ModelAttribute.  The bindingResult doesn't make its way into the view, although Thymeleaf provides some functions we can call to get access to the validation errors.

	@RequestMapping(path = "userform", method = RequestMethod.POST)
    public String userFormPost(@Valid @ModelAttribute("marcsForm") FormPostModel formPostModel, BindingResult bindingResult) {
        System.out.println("** Error count is: " + bindingResult.getErrorCount());
        if (bindingResult.hasErrors()) {
            return "userform";
        }
        return "userform_result";
    }

The view template now contains some extra code to print out the validation errors.  The hasErrors code prints out the error message against the field, and the UL block below prints out all the errors found in the form.

I had some problems where the field name passed into hasErrors didn't match up with the field name in my POJO (which was camelCase).  For some reason passing in neither camelCase nor lowercase worked.  With camelCase it said the field didn't exist and with lower case it returned false when it should have been true.  So I removed the camelCase from the fieldname in the POJO, and then it worked properly.  That is pretty weird.

Another gotcha is that the fields.detailedErrors report has to be inside the form tag for it to find any errors.  I don't really see why as there's nothing there to show that code is contextually linked to the form.  It just is.

	<form action="#" th:action="@{/forms/userform}" th:object="${marcsForm}" method="post">
	    <ul>
	        <dt>Username</dt>
	        <dd>
	            <input type='text' name='username' th:field='*{username}'></input>
	            <span th:if="${#fields.hasErrors('username')}" th:errors="*{username}">This username is invalid</span>
	        </dd>
	        <dt>Password</dt>
	        <dd>
	            <input type='password' name='password' th:field='*{password}'/>
	            <span th:if="${#fields.hasErrors('password')}" th:errors="*{password}">This password is invalid</span>
	        </dd>
	    </ul>
	    <input type='submit' value='Submit'/>
	    <ul>
	        <li th:each="e : ${#fields.detailedErrors()}" th:class="${e.global}? globalerr : fielderr">
	            <span th:text="${e.global}? '*' : ${e.fieldName}">The field name</span>: 
	            <span th:text="${e.message}">The error message</span>
	        </li>
	    </ul>
	</form>



## A REST API

A JSON API would be useful, as it would enable us to add JavaScript to pull data from the application.  I followed this guide for creating an API: http://www.bogotobogo.com/Java/tutorials/Spring-Boot/Spring-Boot-HelloWorld-with-Maven.php

We don't need to change anything to make this work with our existing code, we just need to add a controller and an entity to return.

The REST API controller looks like this:

	@RestController("api")
	public class GeoffController {
	    
	    @RequestMapping(name="geoff", produces = {MediaType.APPLICATION_JSON_VALUE})
	    Geoff home() {
	        
	        Geoff geoff = new Geoff();
	        return geoff;
	    }
	}

The @RestController annotation is really just a shorthand for the @Controller annotation on the class plus @ResponseBody annotation the return values for every method.  Specifying @ResponseBody on the return values makes Spring return the serialized objects, instead of attempting to run a view.  (like it did in our first example)

The corresponding entity like this:

	@XmlRootElement
	public class Geoff {
	    
	    @XmlElement
	    public String name = "Geoffrey";
	    
	    @XmlElement
	    public int number = 42;
	}

With the @XmlRootElement and similar annotations ommited from the entity, the REST API returns a JSON response.  With the XML annotations included, it will return an XML response.  The "produces" annotation can be used to vary the response type.  We could probably discover further ways to vary the response type based on the HTTP Accept header.


## Querying a MySQL database

I installed MySQL in a separate docker container from my Netbeans/Java/Tomcat environmnet.  Both containers were launched with --net=host, which exposes all ports between the containers and the host so everything can talk to each other.  A more sensible way would be to use docker's link command, but I'm too lazy.

Now the application is serving content, I wanted to try querying a database.  Using Hibernate for Java seemed like a fairly natural way to go.

I followed this guide for using Spring and Hibernate together: https://spring.io/guides/gs/accessing-data-mysql/

The pom.xml needed a couple of extra dependencies added.

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
     <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

The reference to JPA is the Java Persistence API, which is a common Java specification for Object Relational Mapping (ORM).  Hibernate is one implementation of this.  Our Spring Boot application will use the Hibernate implementation when we reference JPA.  I am not sure how we might choose a differnet implementation of JPA.

At this point we need to create a configuration file to specify where the database is.

/src/main/resources/application.properties contains:

	spring.jpa.hibernate.ddl-auto=create
	spring.datasource.url=jdbc:mysql://localhost:3306/test
	spring.datasource.username=marc
	spring.datasource.password=password

The first setting here is perhaps the most interesting one.  It tells Hibernate whether it can create the tables on the database.  The options are:

* create: create the schema
* create-drop: drop the schema when the application stops
* validate: validate the schema but make no changes
* update: update the schema as required
* none: neither validate nor change

Now we need an entity object, a repository, and a controller.

#### An entity

The entity object requires the annotation @Entity, and as is common with many ORM, the entity must have an ID field.  The ID field can be an int or a long, or perhaps even some other types.

	@Entity 
	public class User {

	    @Id
	    @GeneratedValue(strategy=GenerationType.AUTO)
	    private Integer id;
	    private String name;
	    private String email;

		public Integer getId() {
			return id;
		}

		public void setId(Integer id) {
			this.id = id;
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public String getEmail() {
			return email;
		}

		public void setEmail(String email) {
			this.email = email;
		}
	}

This is going to cause Hibernate to create a database table called user, with the specified fields.  Hibernate changes the format of the table and field names:

* "User" class becomes "user" table
* SomethingSomethingSomething would become a something_something_something table

The names can be modified by implementing your own naming strategy (ref: http://stackoverflow.com/questions/25526783/hibernate-wont-let-me-use-entity-class-name-for-table-name) although from my current experience it seems best to leave it well alone!

#### A repository

We will come to a repository object to query the database.  We barely need to do anything to create it:

	public interface UserRepository extends CrudRepository<User, Integer> {
	}

Extending CrudRepository will give us some methods for interacting with the database.

The UserRepository specifies that it will handle User entity.  It also specifies the type of the ID field.   (Integer, again)

Perhaps it's quite strange that we're declaring an interface, not an implementation.  However I suppose we probably don't really want to alter Hibernate's implementation of the repository.

#### A controller

Now we can create another controller which calls some methods on the repository.

	@Controller
	@RequestMapping(path="/demo") 
	public class MainController {

	    @Autowired 	           
		private UserRepository userRepository;

		@RequestMapping(path="/create") 
		public @ResponseBody String addNewUser (@RequestParam String name, @RequestParam String email) {
	            User n = new User();
	            n.setName(name);
	            n.setEmail(email);
	            userRepository.save(n);
	            return "Saved";
		}
	        
		@RequestMapping(path="/count")
		public @ResponseBody long countUsers() {
			return userRepository.count();
		}

		@RequestMapping(path="/all")
		public @ResponseBody Iterable<User> getAllUsers() {
			return userRepository.findAll();
	    }
	        
	    @RequestMapping(path="/get")
	    public @ResponseBody User getOne(@RequestParam int id) {
	        return userRepository.findOne(id);
	    }
	    
	    @RequestMapping(path="/update")
	    public @ResponseBody User updateOne(@RequestParam int id) {
	        User user = getOne(id);
	        user.setName( "Modified by update");
	        userRepository.save(user);
	        return getOne(id);
	    }
	    
	    @RequestMapping(path="/delete")
	    public @ResponseBody String deleteOne(@RequestParam int id) {
	        userRepository.delete(id);
	        return "I have deleted it";
	    }    
	}

Note: here we are using @Controller and @ResponseBody instead of @RestController.  They are equivalent.


#### Querying on a different field

Okay, the next part is pretty neat.  We can extend the CrudRepository with additional methods that let us write our own queries.  We'll add a method to find users by name.

We change the repository and add a new method to the interface.  The @Query annotation specifies the query.  Note that we must have an appropriate type of object to return which matches the response from the query.

	public interface UserRepository extends CrudRepository<User, Integer> {
	    
	    @Query("from User where name=:userName")
	    public Iterable<User> findByName(@Param("userName") String userName);
	}

Now we can simply call that from the controller.

    @RequestMapping(path="/search")
    public @ResponseBody Iterable<User> findByName(@RequestParam String name) {
        return userRepository.findByName(name);
    }


#### Querying a database view

Applications often need to present views of data that combine more than one field.

Hibernate doesn't strongly support views, but we can still achieve it.   We can create a view on the database, and then make a matching entity in Java, more or less treating the view like it is a table.

A view can be created in MySQL as follows:

	CREATE VIEW user_address_view AS SELECT u.id, u.name, u.email, ua.address from user u Join user_address ua on ua.user_id = u.id;

As usual we must need to return a unique id field (even though we probably don't need it now as we're not doing any updates).  

Tnen we can create a new entity which matches the columns in the view.  The extra annotation @Immutable tells Hibernate not to try and make any changes to the view.

	@Entity
	@Immutable
	public class UserAddressView {       

	    @Id
	    @GeneratedValue(strategy=GenerationType.AUTO)
	    private Integer id; 
	    
	    private String name;
	    private String email;
	    private String address;    
	    
	    public Integer getId() {
	            return id;
	    }

	    public String getName() {
	            return name;
	    }

	    public String getEmail() {
	            return email;
	    }

	    public String getAddress() {
	            return address;
	    }        
	}

We can make a new repository for this view and expose some query methods.  We could probably use the CrudRepository but I'm using it's base  instead.  Curiously it comes with no public methods, but we can add findAll() and it works.  I'm not sure where it finds the implemnentation.

	interface UserAddressViewRepository extends Repository<UserAddressView, Integer> {
	    
	    Iterable<UserAddressView> findAll();
	    
	}

Querying the view now works from the controller.

        @RequestMapping(path="/view")
        public @ResponseBody Iterable<UserAddressView> view() {
            
            return userAddressRepository.findAll();
        }

#### A shortcut

Spring provides a neat shortcut (RepositoryRestResource) which exposes a Hibernate repository as a working REST API without writing any code.  It is detailed here: https://spring.io/guides/gs/accessing-data-rest/


#### Logging queries

If something goes wrong, we can add a few config lines to get Hibernate to trace the SQL queries.

	spring.jpa.properties.hibernate.show_sql=true
	spring.jpa.properties.hibernate.use_sql_comments=true
	spring.jpa.properties.hibernate.format_sql=true


## Accessing Mongodb

Following this guide: https://spring.io/guides/gs/accessing-data-mongodb/, we need very few simple pieces to access Mongodb.

Edit pom.xml and add a new dependency as follows:

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>                

Create a POJO to represent a data object, implementing the usual getters and setters.  

Create a Repository to access the Mongo database.

	public interface CustomerRepository extends MongoRepository<Customer, String> {

	    public Customer findByFirstName(String firstName);
	    public List<Customer> findByLastName(String lastName);

	} 

Finally, call the repository from your application code.

	repository.deleteAll();
    repository.save(new Customer("Geoff", "Smith"));
    repository.save(new Customer("John", "Smith"));    
	repository.findAll();
	...

## Dependency injection and unit testing

This guide provides an example of dependency injection: https://springframework.guru/dependency-injection-example-using-spring/

In my controller I added the Autowired annotation to inject a ProductService.

    @Autowired
    private ProductService productService;

The injectable class needs to be annotated with @Component, @Service, @Controller or @Repository.  

	@Service
	public class ProductServiceImpl implements ProductService {
		...
	}

OK, wait.  What we really want to do is use one implementation in real life, and a different one during a unit test.  

To do this we create another ProductService and also annotate it with @Service.  To make this one show up during unit testing only, we add @Profile("test").  Finally, to make sure it overrides the original service, which will appear in both scenarios, we add @Primary.

	@Profile("test")
	@Primary
	@Service
	public class ProductService2 implements ProductService {
		...
	}

Our pom.xml requires this dependency to pull in jUnit and a bunch of other useful stuff for testing.

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

We create a jUnit test in /src/tests/java/(package)/BasicApplicationTests.java.   This is annotated with @ActiveProfiles(profiles="test") so that it triggers the mock ProductService.

I added an assert which tests which ProductService was returned by having each one return a different string.

	@RunWith(SpringRunner.class)
	@SpringBootTest
	@ActiveProfiles(profiles = "test")
	public class BasicApplicationTests {
	    
	    @Autowired
	    private ApplicationContext ctx;

		@Test
		public void contextLoads() {
	            MyController controller = (MyController) ctx.getBean("myController");
	            assertEquals("mock product service", controller.getProductService().whichOne());
		}
	}

OK, that works.

#### Mockito

The dependency also pulled in a framework called Mockito, which lets us create the mock right inside the unit test.  A guide for this is here: http://www.baeldung.com/injecting-mocks-in-spring

We can delete the ProductService2 mock class, and replace it with this configuration class which says essentially says "when injecting a product service, return a Mockito mock implementing the ProductService interface"

	@Profile("test")
	@Configuration
	public class ProductServiceConfiguration {
	    @Bean
	    @Primary
	    public ProductService productService() {
	        return Mockito.mock(ProductService.class);
	    }
	}

The Mockito mock isn't a real Product Service because it hasn't got any implementation now.  In the unit test code itself, we can tell the mock ProductService what it should do.  

In the test below I:

* Autowired a product service into the unit test.  This is so I can easily get hold of it to do things to it (instead of getting it from the controller, which perhaps might not be possible depending on our app code)
* Told Mockito to make the ProductService return "Mockito mocked product service"
* Ran the unit test on the controller and discovered that the separate Mockito mock product service injected in the controller behaves how we instructed it to.

	@RunWith(SpringRunner.class)
	@SpringBootTest
	@ActiveProfiles(profiles = "test")
	public class BasicApplicationTests {
	    
	    @Autowired
	    private ApplicationContext ctx;

	    @Autowired
	    private ProductService productService;

	    @Test
	    public void contextLoads() {
	        
	        Mockito.when(productService.whichOne()).thenReturn("Mockito mocked product service");
	        
	        MyController controller = (MyController) ctx.getBean("myController");
	        assertEquals("Mockito mocked product service", controller.getProductService().whichOne());
	    }
	}

I'm rather feeling that I should have two difference instances of the ProductService here and this shouldn't have worked.  I think it works because Spring makes all autowired objects singletons by default.  



## References

* The Spring Guides are amazing: https://spring.io/guides/
* This guy also covers a lot of ground: https://www.mkyong.com/

