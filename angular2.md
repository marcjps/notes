
# Angular (2+)

Angular (2+) was released in 2016.  It takes some concepts from AngularJS, adds a lot of new ones, and is not code-compatible with AngularJS.

How is it different from AngularJS?

* Routing to components always.  This is better.  (although has been ported backward to AngularJS 1.5+)
* Using ES6 and TypeScript.  I think the learning curve is steeper, and it means we have to compile.  But the language features (e.g. classes) are obviously more powerful.
* HTML5 URIs by default (not hash URIs).  This means it's not so easy to serve from the filesystem.  (but still possible)

Overall it feels less hackable (harder to make quick solutions) and more enterprisy (more stable results).  The thing I loved about AngularJS to begin with was its hackable nature, so this is a big loss for me.   Angular (2+) is more poweful (and feels more refined/evolved) which may in the end be helpful if we wanted to build a complex application.  That makes it hard to say which one is better.  They are different.

## Scaffolding

Scaffolding is done using angular-cli, which is written in the ubiquitous nodejs.

Get a working npm.

	docker run --net="host" -v ~/projects/angularjs:/projects:z --name ajs -it ubuntu:latest /bin/bash
	apt-get -y update
	apt-get -y install git npm curl

We need a nodejs version higher than 6.9.7.  These commands upgrade it.

	npm cache clean -f
	npm install -g n
	n stable
	ln -sf /usr/local/n/versions/node/<VERSION>/bin/node /usr/bin/node 

Install angular-cli

	npm install -g @angular/cli

Generate a new project

	ng new my-project
	cd my-project

Generate a new component/service/class

	ng g component geoff
	ng g service geoff
	ng g class geoff

Serve project

	ng serve

	http://localhost:4200

Build a production release

	ng build -prod

The files will be compiled and minimised.  The release output is stored in the dist/ folder.

## Hosting

### Local filesystem

Looks like Angular 2 is not really designed to support hosting on a local filesystem, but it is still possible.  Some of the steps needed to acheive it are:

* Remove the base-url from the index.html
* Enable HashLocationStrategy.  This will turn the links into the AngularJS style hash links
* Don't perform any Ajax requests

Ajax requests generally don't work because web broswers don't allow them to run from local files.  To read in any static JSON files you can import it directly using a script tag.

To enable the hash location strategy, simply add this parameter for the router in your app.module.

   RouterModule.forRoot(routes, { useHash: true }) 

### IIS

For IIS hosting notes see [iis](iis.md)

### nginx

All we need to do for nginx is add the rewrites into its config.

As an aside, to start up an nginx in docker, use this command:

	docker run --name nginx -v /home/ms/projects/angularjs/meetup/dist:/usr/share/nginx/html:ro -d -p 80:80 nginx

Then you have to shell into this container and edit the config to add the rewrite.  Edit the nginx config file which contains the "server" declaration and have it look something like this.

/etc/nginx/conf.d/default.conf:

	server {
	    listen       80;
	    server_name  localhost;
	    root   /usr/share/nginx/html;
	    index  index.html index.htm;
	    location / {
	        try_files $uri$args /$uri$args /index.html;
	    }
	}

try_files specifies a rewrite rule that redirects to /index.html if attempts to find a matching local file fail.


## Bootstrap and font-awesome

Bootstrap will bring some style to our app.  The bootstrap glyphicons have been removed in favour of font-awesome, so we'll use that too.

We need to include the Bootstrap CSS, which can be done with a link tag in the HTML head.  Helpfully though, npm can bring this in for us, which will make it easier to keep it up to date.

	npm install --save bootstrap@4.0.0-alpha.6 font-awesome

Edit src/styles.css and add these two imports:

	@import "~bootstrap/dist/css/bootstrap.min.css";
	@import "~font-awesome/css/font-awesome.css";

### Using bootstrap

Find the CSS reference at: https://v4-alpha.getbootstrap.com/components/alerts/

### Using font-awesome 

Example:

	<i class="fa fa-long-arrow-right" aria-hidden="true"></i>

Find the icons reference at: http://fontawesome.io/icons/

### Adding bootstrap directives

We'll use Angular UI team's bootstrap module to bring in some Angular directives for Bootstrap.  

	npm install --save @ng-bootstrap/ng-bootstrap

We can add these to the view template to easily create working pagination and similar.

Find the directive reference at: https://ng-bootstrap.github.io/#/components/accordion

To implement the navbar collapse feature:  https://stackoverflow.com/questions/37438683/is-there-a-way-to-build-the-mobile-nav-bar-in-ng2-bootsrap


## Routing

Add a new component:

	ng g component Geoff

Remember!  The component will be automatically called GeoffComponent, not Geoff.

In the module, we can import the router.

app.module.ts:

	import { RouterModule, Routes } from '@angular/router';

Define the routes:

	const appRoutes: Routes = [
	  { path: '', redirectTo: '/geoff', pathMatch: 'full' },
	  { path: 'geoff', component: GeoffComponent },
	  { path: 'thing', component: AppComponent }
	];

Note that the first route will handle the home page.  In this case I'm sending a redirect but you could call a component.

In the imports section of the component, call the router.

	imports: [
		...
		RouterModule.forRoot(appRoutes)
	]

In the view:

	<router-outlet></router-outlet>

Reference: https://angular.io/docs/ts/latest/tutorial/toh-pt5.html

### Routing to a nested view from the URI

Add child route.  Note the URL for parent route geoff then child route marc becomes geoff/marc.

	const appRoutes: Routes = [
	  { path: '', redirectTo: '/geoff', pathMatch: 'full' },
	  { path: 'geoff', component: GeoffComponent,
	    children: [
	      { path: 'marc',  component: MarcComponent },
	    ]
	  },
	  { path: 'thing', component: AppComponent }
	];

Add router-outlet into the geoff view.

	<router-outlet></router-outlet>

Then link from anywhere.

	<li>Visit <a routerLink="/geoff/marc" routerLinkActive="active">Geoff then Marc</a></li>


### Using route parameters

We add a link.

	<li><a routerLink="/fool/6" routerLinkActive="active">Fool 6</a></li>

We add a new route to the route list.

	{ path: 'fool/:number', component: FoolComponent } 

In the component, we pick up the param from the router and pass it to the view.

	import { Component, OnInit } from '@angular/core';
	import { Router } from '@angular/router';
	import { ActivatedRoute } from '@angular/router';

	@Component({
	  selector: 'app-fool',
	  templateUrl: './fool.component.html',
	  styleUrls: ['./fool.component.css']
	})
	export class FoolComponent implements OnInit, OnDestroy {

	  number: string;
	  sub: any;

	  constructor(private router: Router, private route: ActivatedRoute) {}

	  ngOnInit() {
	    this.sub = this.route.params.subscribe(params => {
	      this.number = params['number'];
	    });
	  }

	  ngOnDestroy() {
		this.sub.unsubscribe();
	  }
	}

## Calling components

This turns out to be pretty easy.  The scaffolded component had a "selector" within the component dectoration.

	@Component({
	  selector: 'app-fool',
	  templateUrl: './fool.component.html',
	  styleUrls: ['./fool.component.css']
	})

To embed the component into another view, all we need to do is use this selector, like this:

	<app-fool></app-fool>

### Passing parameters into a component (Input)

The @Input decorator is used to specify input parameters.

	export class MeetupTableComponent implements OnInit {
	  @Input() hello: string;
	  constructor() { }
	  ngOnInit() {
	  }
	}

We can pass this value in easily.

	<app-meetup-table hello="marc"></app-meetup-table>


### Raising events from a component (Output)

A component can raise events which can be picked up by the calling component.  

The @Output decorator is used to define an event.

	@Output() userUpdated = new EventEmitter();

We can raise the event from our component code when we desire.

	this.userUpdated.emit(this.user);

The calling component can bind to this event in the usual way.  (See Event Binding, below)

	<user-profile (userUpdated)="handleUserUpdated($event)"></user-profile>


### Lifecycle hooks

Angular gives us some hooks into events that happen during the component lifecycle.  One we use frequently is ngOnInit which runs when a component is initialised.  There's also ngOnChanges which is called whenever any @Input parameter value changes.

Reference: https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html



## Binding things in view templates

### Binding properties

Property binding is setting a Element (DOM) property, Component property or Directive property to the result of an expression.  There are 3 different syntaxes we can use to achieve property binding.  Notice the square parenthesis example.

	<input #box (keyup)="0"/>
	<input type="text" [value]="box.value"/>
	<input type="text" value="{{box.value}}"/>
	<input type="text" bind-value="box.value"/>

All three of these have set the value property of the input field to data value called box.value.  The property bindings are one-way.  (output only)

We really are talking to DOM properties, e.g.

	<p [style.color]="'orange'">style using property syntax, this text is orange</p>
	<p [className]="'marcsBlue'">CSS class using property syntax, this text is blue</p>


### Binding events 

Event bindings let us run code when an event occurs.  Notice the rounded parenthesis example.

	<input #box (keyup)="somethingelse = box.value"/>

Above I'm trapping the keyup event and setting a variable.

### Doing a two-way binding

A two-way binding is something we can both see as an output, and make changes to which will then be reflected elsewhere.  In AngularJS we did it using ng-model attribute on form items.

To do a two-way binding, we just do both a both property and event binding at the same time.  Here's the example.

	<input [value]="username" (input)="username = $event.target.value" />
	<p>Hello {{username}}!</p>

The NgModel directive helps us do this with less writing.  It implements the ability to consume events and write them to an output variable.  Here's what the code looks like if we explicitly call NgModel.

	<input [ngModel]="username" (ngModelChange)="username = $event" />
	<p>Hello {{username}}!</p>

And here's the sugary sweet syntax which does the same thing with less code.  Here we have both the property binding and the event binding the ngModel directive, which is why we have both square and rounded parenthesis.

	<input [(ngModel)]="username" />
	<p>Hello {{username}}!</p>

This guy in the reference explains how to implement the same two-way binding that NgModel is doing, on a member variable in our own component: https://blog.thoughtram.io/angular/2016/10/13/two-way-data-binding-in-angular-2.html

### Template and Model driven forms

Angular provides two ways to build forms.

Template driven forms are most similar to the AngularJS approach, where lots of form logic (e.g validation) can be achieved in the template only.  By peppering the view template with simple hints we can get a working form with impressive features.

Model driven forms work the opposite way, the form logic is described mostly programatically inside the component.  This is a better approach from a unit testing perspective, because it makes the form more testable.



## A bunch of simple examples

### Outputting something

	<p>Something {{somethingelse}}</p>

### A text-field updating member variable

In the component:
	
	  somethingelse: string;

In the view: 

	<input type="text" [(ngModel)]="somethingelse" />

How can we detect when this changes?  There's a lot of ways.  You could change it to bind the event explicitly, or change it from a member variable to a getter and setter, or trap it in the component using OnChanges.

### A button calling a function

In the component:

	clearPeople() {
		this.persons = [];
	}

In the view:

	<button (click)="clearPeople()">Clear the people</button>

### Template reference variables

This instantiates a variable in the template from a DOM object.  It uses a hash syntax.  

	<input #box (keyup)="0"/>
	<p>{{box.value}}</p>

Template reference variables aren't updated unless there's a binding for any event.  So we have keyup bound to nothing much.


## Pipes 

Just like AngularJS filters, we can use pipes to process data.

### Using pipes

Here's some examples:

	<p>A string {{somethingelse | uppercase}}</p>
	<p>A string {{somethingelse | lowercase}}</p>
	<p>A string  {{somethingelse | titlecase}}</p>
	<p>A number {{thenumber}}</p>
	<p>A number {{thenumber | number:'1.0-0'}}</p>
	<p>A number {{thenumber | currency: "GBP": true }}</p>
	<p>A date {{thedate | date : "dd/MM/yyyy HH:mm:ss"}}</p>

### Implementing a new pipe

Generate the pipe using:

	ng g pipe personFilter

app.module should import the pipe.

	import { PersonFilterPipe } from './person-filter.pipe';

app.module should reference the pipe in declarations.

	declarations: [
		...
		PersonFilterPipe
	],

The pipe should implement some code to filter items.

	import { Pipe, PipeTransform } from '@angular/core';

	@Pipe({
	  name: 'personFilter'
	})
	export class PersonFilterPipe implements PipeTransform {

	  transform(value: any, args?: any): any {
	    let [name] = args;
	    return value.filter(person => {
	      return (name == undefined || person.name.indexOf(name) !== -1);
	    });
	  }

	}

Now we can call the pipe (using it's name) from a template.  We do not need to reference it in the component.

	<input #box (keyup)="0"/>
	<p>You typed {{box.value}}</p>
	<li *ngFor="let person of persons | personFilter:box.value ">
		{{ person.name }} is feeling {{person.feeling}}
	</li>



## Directives

### ngIf

Optionally show or hide an element.

	<p *ngIf="meetups == undefined || meetups.length == 0">No matching results found.</p>

### ngFor

Allows you to loop through data.

    <tr *ngFor="let meetup of meetups">
      <td>{{meetup.Date | date : "EEE" }}</td>
    </tr>

### NgSwitch

Select one case out of several choices.

      <td [ngSwitch]="meetup.URI.length != 0">
        <a href="{{meetup.URI}}" *ngSwitchCase="true">
        	...
        </a>
        <span *ngSwitchCase="false">
        	...
        </span>
      </td>

### NgClass

Sets a class on an element based on an expression.
	
	<p [ngClass]="{'first': true, 'second': true, 'third': false}"></p>


### NgStyle

Sets CSS styles on an element based on an expression.

	<div [ngStyle]="{'color': 'blue', 'font-size': '24px', 'font-weight': 'bold'}">
	  style using ngStyle
	</div>

## Services

Services are usually used for fetching data.

### Create a service

We'll implement a service that makes a HTTP request to get some data.

In the module, import the HTTP helper, and add it to the imports section.

module.ts

	import { HttpModule }      from '@angular/http';
	... snip...
	imports: [
		BrowserModule,
		FormsModule,
		HttpModule,
		RouterModule.forRoot(appRoutes)
	],

Generate a service using

	ng g service MeetupData

In the service, import the HttpModule.   Add a method to retrieve data from a data folder.

meetup-data.service.ts:

	import { Injectable } from '@angular/core';
	import { Headers, Http, Response } from '@angular/http';

	import 'rxjs/add/operator/map';

	export class Meetup {
	  URI: string;
	  Name: string;
	  Date: Date;
	  Attendees: string;
	  GroupURI: string;
	  Score: number;
	  Matches: string;
	  IsNew: boolean;
	}

	@Injectable()
	export class MeetupDataService {

	  constructor (private http: Http) {}

	  getMeetupsRaw() {
	    return this.http.get("data/meetups.json").map((res:Response) => res.json());
	  }
	  
	}

Note: To get "ng serve" to retain the "data" folder in the build, I added it to .angular-cli.json's "assets" array.  Maybe it's smarter to put the data file in the "assets" folder which is already included there.  :-)

### Consume the service

In the component that consumes the service, import the service and retrieve the data into an array.  Remeber to import the service and add it as a provider.

weekends.component.ts:

	import { Component, OnInit } from '@angular/core';
	import {MeetupDataService, Meetup} from '../meetup-data.service';

	@Component({
	  selector: 'app-weekends',
	  templateUrl: './weekends.component.html',
	  styleUrls: ['./weekends.component.css'],
	  providers: [MeetupDataService]
	})
	export class WeekendsComponent implements OnInit {

	  meetupDataService : MeetupDataService;
	  meetups : Meetup[];

	  constructor(meetupDataService : MeetupDataService) {
	    this.meetupDataService = meetupDataService;
	  }

	  ngOnInit() {
	    this.meetupDataService.getMeetupsRaw().subscribe( m => { this.meetups = m; } );
	  }
	}

When we interact with the Meetups array, Typescript will check at compile-time that we're using the types correctly.  However, no checking is performed at runtime.  If the JSON service returns different types than we expect, no error about the type differences will be thrown for us.  Instead we'll just have different types than we thought (and might run into some other error).

### Caching http response data

Often we want a piece of data retrieved by a service available multiple times without requesting it again.  

Here's someone's code to make a cache.  It does work.

Ref: https://stackoverflow.com/questions/36271899/what-is-the-correct-way-to-share-the-result-of-an-angular-2-http-network-call-in

I figured out something interesting whilst working on this.

I had injected the service into each component, and listed it in the Providers [] array for the component.  What that actually did was create a different service for each component.  Since there was no singleton, and no shared member variables, it didn't cache.

I found the solution.  We instead specify the provider in the root component (which is called app.component) for me.  And then in all the lower level components, we use an "import" to get the service but we *do not* specify it in the providers array.  The provider is inherited from the root component.  Using this method, the service is a singleton for all the components, and the caching worked. 

Another interesting point, is the caching would not work if this wasn't a Single Page Application, as each page-reload would clear everything.  Luckily, it is working correctly as a SPA already.  


## TypeScript

TypeScript brings some type checking to JavaScript.  It adds new features to the languauge which are checked by the TypeScript "transpiler" during compilation of the code.  It compiles the code down to normal JavaScript.  The type-checking features are not applied at runtime.

When checking object types, TypeScript doesn't really check that they are references to the same type.  It only checks that they are the same shape.  (i.e. they contain the same member variables etc.)

### Running tsc directly

Our angular-cli is running tsc when it compiles.  We can run it directly on an input .ts file, and it will compile and produce an output .js file.

	npm install typescript
	node_modules/typescript/bin/tsc input.ts

The output JavaScript file is equivalent to the input one, but with the TypeScript code replaced by normal JavaScript code.

Note: to get the decorator to work, we need to add the flags: --target ES5 --emitDecoratorMetadata --experimentalDecorators.  The decorator is replaced with alternative JavaScript code.

#### Types

Here are some of the primitive types supported by TypeScript

* boolean
* number
* string
* Array
* enum 
* any - opt out of type checking
* void - no type at all
* classes and interfaces

We can type objects in let statements.

	let color: string = "blue";
	let some_number: number = 6;

#### Interfaces

Interfaces aren't an ES6 feature, they are a TypeScript feature.

They can be used to define structures for the arugments to a function, e.g. 

	interface Callback {
	  (error: Error, data: any): void;
	}

	function callServer(callback: Callback) {
	  callback(null, 'hi');
	}

	callServer((error, data) => console.log(data));  // 'hi'
	callServer('hi');   


#### Decorators

Decorators are functions.  Classes, parameters, methods and properties can be decorated.  The decorator can run additional code or alter the behaviour of the decorated code.

Here's a go at writing a method decorator.  It can read the input and output parameters of the function that is being called.  It could run other code or even call a different function than was originally intended.  (that's probably very undesirable!)

	function marcs(target: Object, propertyKey: string, descriptor: TypedPropertyDescriptor<any>) {
	  const originalMethod = descriptor.value;

	  descriptor.value = function(...args: any[]) {

	    // do something before the function runs.
	    console.log("Decorator: The input parameters are: " + JSON.stringify(args));
	    args[0] = "Hello Marc";  // change the input value
	    console.log("Decorator: replaced input value parameters with: " + args[0]);

	    // run the function.  We could run something entirely different..
	    let result = originalMethod.apply(this, args);

	    // do something after the function runs
	    console.log("Decorator: the function is returning: " + result);
	    result = 42; // change the result
	    console.log("Decorator: Replaced return value with: " + result);

	    // return the result of the function
	    return result;
	  };

	  return descriptor;
	}

I'm using the decorator below: 

	ngOnInit() {
		let input : string = "Hello world";
		console.log("Input value sent: " + input);
		let result : number = this.saySomething(input);
		console.log("Return value received: " + result);
	}

	@marcs
	saySomething(somestring: string) : number {
		let returnValue : number = 1;
		console.log("Input value received: " + somestring);
		console.log("Return value sent: " + returnValue);
		return returnValue;
	}

The output is:

	Input value sent: Hello world
	Decorator: The input parameters are: ["Hello world"]
	Decorator: replaced input value parameters with: Hello Marc
	Input value received: Hello Marc
	Return value sent: 1
	Decorator: the function is returning: 1
	Decorator: Replaced return value with: 42
	Return value received: 42



## References

### Angular

* A nice simple tutorial = http://learnangular2.com/
* An awesome detailed tutorial = https://angular-2-training-book.rangle.io/
* Quick Start tutorial = https://angular.io/docs/ts/latest/quickstart.html
* Guide = https://angular.io/docs/ts/latest/guide/
* API reference = https://angular.io/docs/ts/latest/api/
* Angular 2 tutorial = https://thinkster.io/tutorials/learn-angular-2
* Angular 1 to 2 quick reference = https://angular.io/docs/ts/latest/cookbook/a1-a2-quick-reference.html
* Angular architecture overview = https://angular.io/docs/ts/latest/guide/architecture.html#!#component
* Quite an interesting tutorial on routes and bootstrap: http://blog.angular-university.io/angular-2-router-nested-routes-and-nested-auxiliary-routes-build-a-menu-navigation-system/

### Bootstrap references

* Bootstrap 4 reference: https://v4-alpha.getbootstrap.com/components/alerts/
* Angular UI Bootstrap directive reference: https://ng-bootstrap.github.io/#/components/accordion

### TypeScript

* TypeScript tutorial: https://www.tutorialspoint.com/typescript/index.htm
