
# Angular.js

AngularJS is different from Angular.  Angular JS refers to versions 1.x and Angular to versions 2+.  AngularJS was initially released in 2010.  Angular (2+) was released in 2016.

## Why AngularJS?

I think AngularJS has these strengths:

* A great component architecture.  You can easily grab a third party module and plug it into your website.  
* A mature ecosystem with lots of modules available.
* Effortless 2-way data binding, it is amazing.
* It supports unit testing.
* Amazingly hackable and simple.  I don't know why people say it is difficult.

And these weaknesses:

* Being entirely client side makes it different from the normal and not really suited to the average web site.
* The recommended coding patterns changed significantly over time.  
* Angular (2+) is a completely different thing with no code compatibillity at all.
* Possibly a complex application might eventually become code-hell.  Maybe it doesn't scale to really really high complexity.

## Development tools

Here's some commonly used development tools for AngularJS.  We do not have to use the tools.  Sometimes I find things easier without them.  But they are do useful things.

All the tools are written in nodejs, which is possibly a bit of a curse.  Billions of dependencies.

Also note these tools might be slowly being superseded by different tools.  Bower could be replaced by yarn or npm.  And Grunt could be replaced by Gulp.

* Bower - package management
* Yo - application scaffolding
* Grunt - a task runner, a bit like ant, which can build/run/test for us

Here's some steps to get the tools up and running in Docker.

```text
docker run --net="host" -v ~/projects/angularjs:/projects:z --name ajs -it ubuntu:latest /bin/bash
apt-get -y update
```

Install and update nodejs:

```text
apt-get -y install git npm curl
npm cache clean -f
npm install -g n
n stable
ln -sf /usr/local/n/versions/node/<VERSION>/bin/node /usr/bin/node 
```

Install angularjs tools:

```text
npm install -g yo generator-karma generator-angular generator-component-angularjs grunt-cli bower
```

Now we have the tools, but they get tetchy about running as root.  So we make a user.

```text
passwd root
adduser ms
su ms
```

#### Yo - scaffold an angular app

```text
mkdir /projects/1
cd /projects/1
yo angular [appName]
```

#### Grunt - serve the app

Start grunt to serve it:

```text
grunt serve
```

Go to http://localhost:9000

Grunt will trigger a browser refresh if you edit and save the any of the files in the project.

#### Bower - manage dependencies

If you view source and look at the bottom of the page, and you will see several script tags referencing different modules.  Those are the dependencies added by the bower package manager. 

*bower.json*  specifies our AngularJS module dependencies.  You can hop over to http://ngmodules.org/, pick another module and add it to the list.  To get the module installed, run:

```text
bower install 
grunt serve
```

That will simply download it (and all its dependencies) and inject the script tag reference.


#### Grunt - Run unit tests

*gruntfile.js* - kinda similar to an ant build file, this contains code for grunt's tasks on our project.

I had to fix some broken things in my environment:

```text
(from root account) apt-get install libfontconfig
(from project folder) npm install grunt-karma --save-dev
grunt test
```

That will execute unit tests on our code, e.g. the test in test/spec/controllers/main.js

#### Yo - Scaffolding other things

Yo can of course scaffold other javascript frameworks.

But it can also now scaffold other structures into our angular app.

Create a controller, view: and route:

```text
yo angular:route myroute
```

Create a controller:

```text
yo angular:controller user
```

etc. etc.

Refer to: https://github.com/yeoman/generator-angular

#### An editor?

Well, there's lots of things you can use here:

* WebStorm
* Sublime Text
* Visual Studio 
* Visual Studio Code 
* Probably many others


## A crash course in Angular stuff

### Module creation (app.js)

I'd usually have a file called app.js that creates a "module" for the entire application.  We add our controllers, services, and other stuff into the module later.

The module definition in app.js looks something like this.  The list of things named in the parameters are dependencies that the module will use.  The dependencies can be called upon later.  For each of the dependencies you must have the necessary script tags included in by the HTML file.  (you can do this manually or bower+grunt can do it for you)

```javascript
angular.module('myApp', [
	'ui.router'
]);
```

### Routes

Routing is done using an additional module called AngularUI Router.

Call in the ui-router dependency by adding a reference to the ui-router module using bower (or by adding a script tag).  Add ui.router to the module dependency list. (already shown above)

In the main index page for the site, call in a view using the ui-view directive.  E.g.

```html
<div class="ui-view"></div>
```

Add routes to the module.  I usually do this in app.js too, here's how.  The .config method on a module runs when the module starts up.  Notice we just grab a module and then add stuff to it.  This lets us be lazy not worry about passing objects around (or adding more global objects).

app.js

```javascript
angular.module('myApp').config(function ($stateProvider, $urlRouterProvider) {
	$urlRouterProvider.otherwise('/');
	$stateProvider
		.state('home', {
		url: '/',
		templateUrl: 'views/main.html',
		controller: 'MainCtrl'
		} )
		.state('about', {
		url: '/about',
		templateUrl: 'views/about.html',
		controller: 'AboutCtrl'
		} )
		.state('geoff', {
		url: '/geoff/{something}',
		templateUrl: 'views/geoff.html',
		controller: 'GeoffCtrl'
		} )
		.state('geoff-with-param', {
		url: '/geoff?something',
		templateUrl: 'views/geoff.html',
		controller: 'GeoffCtrl'
		} );
});
```

Okay, so we can see what is happening there.  This will matching the hash-bang URLs (using the url parameter) to pick which state is active.  Then we're giving the state a controller and a view template.  We can add parameters as shown by my "somethings".

### Controller

We should define our controllers in separate javascript files.  Once again we can just grab the module and add a controller to it.  We can get hold of the state parameters by looking for them in $stateParams.

geoff.js

```javascript
angular.module('myApp').controller('GeoffCtrl', function ($scope, $stateParams) {
	$scope.something = $stateParams.something;
});
```

### View

Finally we need our view, which we said lives at /views/geoff.html.

The view does bugger all but print out the "something".  Remember that the view will replace the ui-view in the main page of the app.

geoff.html

```html
<p>This is the geoff view.  The something is {{something}}</p>
```

### Links

We can simply link to these different pages using #! URIs, e.g.

```html
<a ng-href="#!/about">About</a>
<a ng-href="#!/geoff/marc">Geoff</a>
<a ng-href="#!/geoff?something=fred">Geoff with param</a>
```

A normal href would work, but ng-href is used because it stops anyone clicking the link before angular has executed the client side scripts on the page (which is necessary if the URI itself contains angular expressions).

Angular UI router also gives us a directive for linking using the name of the state.

```html
<li><a ui-sref="about">About</a></li>
<li><a ui-sref="geoff({something: 'marc'})">Geoff</a></li> 
<li><a ui-sref="geoff-with-param({something: 'fred'})">Geoff with param</a></li>
```

Visit Angular UI Router's tutorial to see how to route to components (instead of directly to controllers and templates) and how to use nested views (a view within a view).  https://ui-router.github.io/ng1/tutorial/

#### Okay, what is $scope?

Scope is the object which passes from a controller to the view.  In the controller we can add variables and functions to the scope, and then we can access them from the view.

It is possible to have multiple scopes on a page using features like directives (see below) or by calling different controllers from the view.  Scopes on the page are hierarchical based on the nesting of the tags in the view.  The child scopes inherit the functions and variables from the parent ones.

Extra ref: https://docs.angularjs.org/guide/scope



### Filters

Filters are used to format data.

There are some built-in filters, and we can also define our own filter in a module.

#### Call a filter from a view template

```html
<p>Short string: {{some.shortString}}</p>
<p>Short string: {{some.shortString | uppercase}}</p>
<p>Short string: {{some.shortString | lowercase}}</p>
<p>Short string as JSON: {{some.shortString | json}}</p>
<p>Today's date: {{some.todaysDate}}</p>
<p>Today's date formatted: {{some.todaysDate | date : 'dd/MM/yy HH:mm:ss'}}</p>
<p>Number: {{some.number}}</p>
<p>Number as currency: {{some.number | currency : '£'}}</p>
```

There's another filter, simply called "filter", that filters items from an array using a string match.

#### Call a filter from code

You can use $filter in a controller to run any filter.

```javascript
this.filteredDate = $filter('date')(this.todaysDate, "yyyy-MM-dd");
this.lowerString = $filter('lowercase')(this.shortString);
```


#### Define a filter

Creating a new filter is childs play.  Add a filter to your module.

```javascript
angular.module('myApp').filter('addWow', function () {
	return function(x) {
	return x + " wow.";
	}
});
```

Then call the filter.

```html
<p>Short string wow: {{some.shortString | addWow }}</p>
```

### Directives

Directives are functions that you can call from the HTML view, by referencing their name in an element, attribute, or class.

Below is a list of some useful built-in directives.  Plugins frequently provide new directives that you can use.  (e.g. ng-table)  We can also define our own directive in a module.

#### ng-app

ng-app is a directive that we have to have in our page to start up the Angular application.  If it is not there, Angular doesn't run any code.  We usually put this reference close to the top of the page. (e.g. on the html, or body elements)  The module name is usually passed in.

```html
<body ng-app="myApp">
```

#### ng-controller, "controller as"

The ng-controller directive gives you a way to explicitly call a particular controller from a view.  

```html
<div ng-controller="someCtrl">
	<p>Some controller says {{something}}</p>
</div>
```

This can be used if you need multiple controllers in a page.  I think you shouldn't really do this because in MVC models, views are not supposed to be tied to a specific controller.  

ng-controller has an additional syntax that eliminates using $scope.  We set our member variables and functions directly on the controller instead of on scope.  

```javascript
angular.module('myApp').controller('someCtrl', function () {
	this.something = "good morning very politely";
});
```

Then in the view we use the controller object instead of $scope.

```html
<div ng-controller="someCtrl as some">
	<p>Some controller says {{some.something}}</p>
</div>
```

I think this is probably better than using $scope.  But in practise I mostly used $scope so far.   It is a pity that this requires referencing the controller name from the view.

Also ref: https://toddmotto.com/digging-into-angulars-controller-as-syntax/


#### ng-repeat, sorting and filtering lists

ng-repeat iterates over a list and repeats the element that it is placed on.

Here's a simple example.  In the controller we return a list.

```javascript
angular.module('myApp').controller('someCtrl', function () {
	this.names = [ 'Marc', 'Catherine', 'Jenny', 'Adam' ]
});
```

Now in the view template we print each list item using the li tag.

```html
<ul>
	<li ng-repeat="name in some.names">{{name}}</li>
</ul>
```

We can sort it by calling the orderBy filter:

```html
<li ng-repeat="name in some.names | orderBy : name">{{name}}</li>
```

or in reverse using either of these:

```html
<li ng-repeat="name in some.names | orderBy : -name">{{name}}</li>
<li ng-repeat="name in some.names | orderBy : name : true">{{name}}</li>
```

and we can limit the items returned using the limitTo filter.

```html
<li ng-repeat="name in some.names | orderBy : name | limitTo: 2">{{name}}</li>
```

We can also filter items from the list using the "filter" filter which does a string match.  Below I'm using an input field to match on.

```html
<input type="text" ng-model="filterNames" />
<ul>
	<li ng-repeat="name in some.names | filter : filterNames">{{name}}</li>
</ul>
```

Check https://docs.angularjs.org/api/ng/filter/filter to see a couple more ways the filter can be used.

#### ng-if and ng-show

ng-if removes an element from the page based on a condition.  I quite often use it in situations where a list I am iterating through is empty, and I want to display a message describing that.  An example is below.

```html
<input type="text" ng-model="filterNames" />
<ul>
<li ng-repeat="name in some.names | filter : filterNames as filteredNames">{{name}}</li>
</ul>
<p ng-if="filteredNames.length == 0">No names match this filter.</p>
```

ng-show is very similar.  ng-show toggles the visibility of the item using CSS.  Possibly ng-show will toggle items more quickly than ng-if. (as ng-if is altering the DOM and ng-show is only altering the CSS)


#### ng-model

ng-model allows us to bind form input controls to variables.  The binding is 2-directional: If we change the input control, the variable changes.  If we change the variable, the value shown in the input control changes.

Here's a bunch of examples of binding different types of inputs.

```html
<hr />
<input type="text" ng-model="inputString" />
<p>The string you typed is: {{inputString}}</p>

<hr />
<input type="date" ng-model="inputDate" />
<p>The date you picked is: {{inputDate | date : 'dd/MM/yy'}}</p>

<hr />
<select ng-model="inputSelect">
<option value="a">a</option>
<option value="b">b</option>
<option value="c">c</option>
</select>
<p>The option you picked is: {{inputSelect}}</p>

<hr />
<input type="checkbox" ng-model="inputCheck"/>
<p>The check is: {{inputCheck == true}}</p>

<hr />
<input type="radio" ng-model="inputRadio" name="optionA" value="a"/><label for="optionA">A</label>
<input type="radio" ng-model="inputRadio" name="optionB" value="b"/><label for="optionB">B</label>
<input type="radio" ng-model="inputRadio" name="optionC" value="c"/><label for="optionC">C</label>
<p>The radio is on: {{inputRadio}}</p>
```

Angular has built in support for form field validation.  It has a raft of validation features so go out and find a tutorial on that if you need to do it.

#### ng-click

ng-click can call some code (or a function on a controller) when an element is clicked.  It is quite useful on buttons.

Here's a function being defined in a controller.

```javascript
angular.module('myApp').controller('someCtrl', function () {
	this.shortString = 'Hello world';
	this.changeSomething = function() {
	this.shortString = 'Short string has been changed'
	console.log("hello");
	};
});
```

And here's ng-click calling the function.

```html
<button ng-click="some.changeSomething()">Click me</button>
<p>Short string: {{some.shortString}}</p>
```

#### ng-class, ng-class-even, ng-class-odd and ng-style

These directives can be used to alter the class attribute on an element based on an expression.

Here is my crap example, which sets a class to "green" when the filteredNames array contains Marc.

```html
	<style type="text/css">
	.green {
	  color: green;
	}
	</style>
	<input type="text" ng-model="filterNames" />
	<ul>
		<li ng-repeat="name in some.names | filter : filterNames as filteredNames">{{name}}</li>
	</ul>
	<p ng-class= "{ 'green' : filteredNames.indexOf('Marc') != -1}">Marc is in the list</p>
```

ng-style is similar but can set the style attribute (which contains CSS rather than class names).

ng-class-even and ng-class-odd place classes on even or odd occurances of elements in the page.  This is useful in tables if you feel like having a stripy table with every other row in a different style.

#### Define a directive

Here's some simple code to define a directive.  

```javascript
angular.module('myApp').directive('hackPlanet', function() {
	return {
	restrict: 'AEC',
	template: 'Hack the planet!'
	};
});
```

Here's some ways we can call it.

```html
<hack-planet/>
<div hack-planet/>
<div class="hack-planet"/>
```

The "restrict" option says whether the directive can be used in a attribute, element or class respectively.  

Note: We should create components instead of directives.  See below.


## Services

A service is a chunk of code you can get hold of using dependency injection.

There are some built-in services, and we can also define our own service in a module.

#### $filter

We covered this above, the filter service can call a filter.

```javascript
	this.filteredDate = $filter('date')(this.todaysDate, "yyyy-MM-dd");
	this.lowerString = $filter('lowercase')(this.shortString);
```

#### $http 

I use the http service a lot, usually to fetch JSON data from a REST API.  Here's what my pattern looks like.


```javascript
var canceller;

if (canceller)
	canceller.resolve();
canceller = $q.defer();
$http.get('/data.json', {timeout: canceller.promise}).then(
function success(response) {
	$scope.names = response.data.names;
	console.log("names loaded OK");
	console.log(this.names);
},
function error(response) {
	console.log("names load failed");
	console.log(response.status);
	console.log(response.statusText);
	console.log(response.data);
});
```

The canceller feature isn't necessary, but it is useful.  It cancels the previous HTTP request if we call this code again before the previous one completed.  This is useful in situations where we may make many calls rapidly (e.g. an autocomplete/search)

When writing this I noticed that $scope updated with my new data from the request but the Controller As syntax did not.  That's interesting.

The success and error handlers changed in Angular 1.6.  The above code is the new version.

*Caching HTTP responses*

We can cache HTTP responses on the client side if we like.  The simplest, permanent cache is enabled like this:

```javascript
$http.get('/data.json', {cache: true} )..
```

With this code, each subsequent call will return the same response without making another request.

A slightly more powerful (but not powerful enough..) version built into AngularJS is CacheFactory.  The CacheFactory object provides caching functions.  You can give data a Cache ID, then set it into the cache and retrieve it later.  (or remove it) Unfortunately CacheFactory doesn't provide automatic expiry of items after a period of time.

To get full, powerful caching behaviour with auto-expiry, there's a module called angular-cache.  View it's usage here: https://github.com/jmdobry/angular-cache

#### $log 

This is a safe logger.  I should probably use it instead of console.log (which is not meant to be supported in all browsers)

```javascript
$log.info('Hello world');
```

#### $timeout

The timeout service is used to generate a one-off delay.  Here's an example that prints a message in the console after a second.

```javascript
$timeout( function() {
	console.log("timeout elapsed");
}, 1000);
```

#### $interval

The interval service is used to fire an event repeatedly.  Here's an example that repeats a message 10 times then cancels.
   	
```javascript
var count = 0;
$scope.heartbeat = function() {
	console.log("interval elapsed");
	count++;
	if (count == 10)
		$interval.cancel(timer);
};
var timer = $interval($scope.heartbeat, 1000);
```

#### $watch

$watch can be used to watch a variable and trigger a function when the variable changes.	

```javascript
$scope.$watch('inputString', function() {
	console.log("input string changed.");
});
```

#### $q

Q allows us to implement a "promise", a promise to complete an aynchronous function and provide a return value from it.

We can run some code and do something in response to the code completing, but without blocking the thread in the meantime.

```javascript
function addWow(input) {
	return $q(function(resolve, reject) {
		setTimeout(function() {
			resolve(input + ".. wow!!");
			//reject(input + ".. wow!!");
		}, 5000);
	});
}

var promise = addWow('Something is happening');

promise.then(function(the_result) {
	$log.info('Success: ' + the_result);
}, function(the_result) {
	$log.error('Failed: ' + the_result);
});

$log.info("I have gone past here without blocking");
```

The above code will run and complete, printing the message "I have gone past here without blocking".  Later on it will print the success message in the then() function after the slow function completes.  The promise will call either the success or failed functions if we respond with the corresponding resolve() or reject() functions.

### Defining our own service

We can simply define a service on our module.

```javascript
angular.module('myApp').service('wow', function() {
	this.addWow = function (x) {
	return x + "..ohwow";
	}
});
```

Then we can inject it into a controller and call it.

```javascript
angular.module('myApp').controller('someCtrl', function ($scope, wow) {
	$scope.someWow = wow.addWow('Marc waz here');
}
```

Note that we don't prefix our service name with dollar, unlike the other services.  This is because Angular want to keep all the dollars.  (really!)  Dollars are reserved for naming Angular's core code to stop clashes.


## Functions

#### angular.forEach

Using forEach you can iterate over an array.

```javascript
var things =  {
	name: "marc",
	age: "38",
	feeling: "fine"
};

angular.forEach(things, function(value, key) {
	$log.info("key: " + key + " value: " + value);
});
```

## Components

Components were added in AngularJS 1.5 and bring together the concepts we had with routes and directives into a slightly more sensible solution.

### Scaffolding

There's an extra yo generator called yo component-angularjs which can generate components.  For some reason or other, I ended up writing mine from scratch during my testing.

### Routing to components

We can route directly to a component.  The component has a self contained controller and view.  

We use stateProvider and point an URI to the component.

app.js:

```javascript
$stateProvider.state('geoff', {
	url: '/geoff',
	component: 'geoff'
})
```

The component contains the controller logic and a reference to either "templateUri" (a link to a view) or "template" a string variable containing the view content.

geoff.js

```javascript
angular.module('jsc2App').component('geoff', {
	templateUrl: 'components/geoff/geoff.html',
	controller: function($http) {
	var ctrl = this;
	ctrl.something = 'Hello world';
	ctrl.doSomething = function() {
		ctrl.something = 'Something changed';
	};
	}
})
```

In the view we no longer use $scope if we can avoid it.  The controller is bound to $ctrl by default.

geoff.html

```html
	<div>
	  <p>This is the geoff component.</p>
	  <p>The bound variable is {{$ctrl.something}}</p>
	  <button ng-click="$ctrl.doSomething()">Do something</button>
	</div>
```

#### Route parameters

We can still add route parameters using $stateParams.

app.js

```javascript
$stateProvider.state('geoff', {
	url: '/geoff/{thing}',
	component: 'geoff'
})
```

geoff.js

```javascript
angular.module('jsc2App').component('geoff', {
	templateUrl: 'components/geoff/geoff.html',
	controller: function($http, $stateParams) {
	var ctrl = this;
	ctrl.something = 'Hello world - the thing is ' + $stateParams.thing;
	ctrl.doSomething = function() {
		ctrl.something = 'Something changed';
	};
	}
})
```

#### Routing to nested components

Okay, we can use routing to call a nested component.  Oddly I didn't find a tutorial that covers how.  Here's how.

We add the route for a child state.  The nesting seems to be determined by the format of the state name.  When it is something.something then it becomes a nested route.  The URL contains only the part below the parent.  So here when the route is put together I end up with /geoff/{thing}/{subthing}.

app.js

```javascript
    $stateProvider.state('geoff', {
      url: '/geoff/{thing}',
      component: 'geoff'
    })
    .state('geoff.subitem', {
      url: '/{subthing}',
      component: 'geoffsubitem'
    })
```

The rest is straightforward.  We create the subcomponent exactly the same way as we created the previous one.  All that remains then is to add the router outlet into the view for the parent component, the same as usual.

geoff.html

```html
	<ui-view></ui-view>
```

### Calling components 

As Components replace Directives, we can call them explicitly.

We can simply create a component like this:

bird.js

```javascript
angular.module('jsc2App').component('bird', {
	templateUrl: 'components/bird/bird.html'
})
```


bird.html
	
```html
<p>Bird</p>
```

And then call it from another view template.

index.html

```html
<bird />
```

Note: the component can only be called as an element, like this.  This is unlike directives which we could call using attributes, classes and comments.  

### Input parameters

We can pass parameters into the component.  We specify the bindings section in the component to define the parameters that go in and out.  Below I'm adding a parameter called name.  It is a text parameter, which is indicated by the @.

bird.js

```javascript
angular.module('jsc2App').component('bird', {
	bindings: { name: '@' },
	templateUrl: 'components/bird/bird.html'
})
```

bird.html

```html
<p>{{$ctrl.name}} bird.</p>
```

Now when we call it we can pass the parameter in and it will be recieved and displayed.

index.html

```html
<bird name="Big" />
```

The binding types are ugly and unintuitive.  Here are the values.

* @ = text binding.  Pass in text directly, not a variable.
* < = One way binding.  If component changes the variable, it won't reflect to the caller.
* = = Two way binding.  If component changes the variable, it will change for the calling code too.
* & = method binding.  This allows a function callback from the component to the caller.  

To change our text binding to an actual variable we can do something like this:

bird.js

```javascript
angular.module('jsc2App').component('bird', {
bindings: { name: '=' },
templateUrl: 'components/bird/bird.html'
})
```

index.html

```html
<bird name="'Big'" />
```

Now we are passing in a string variable (which I have done using quotes, but could be a variable from the parent component's controller).  As we have 2-way bound it, we could let the child component change it and we would see the changes in the parent.

### Output events

Okay, the output event lets us make a callback to the controller from inside our component. 

In my home component I call the bird component and assign a function in my controller to the callback.  The function just logs the event.

home.html

```html
<!-- note the name mapping - does-something = doesSomething -->
<bird name="'Big bird'" does-something="$ctrl.handleBirdEvent(name, something)" />
```

home.js.

```javascript
angular.module('jsc2App').component('home', {
	templateUrl: 'components/home/home.html',
	controller: function() {
	var ctrl = this;
	ctrl.handleBirdEvent = function(name, something) {
		console.log("Controller heard " + name + " " + something);
	}
	}
})
```

Now in my bird component, I use a button to call a method in the bird controller, and raise the event.

bird.html

```html
<p>
	<p>{{$ctrl.name}}</p>
	<button ng-click="$ctrl.chirp()">Chirp!</button>
	<button ng-click="$ctrl.warble()">Warble!</button>
</p>
```

bird.js

```javascript
angular.module('jsc2App').component('bird', {
	bindings: {
	name: '=',
	doesSomething: '&'
	},
	templateUrl: 'components/bird/bird.html',
	controller: function() {
	var ctrl = this;
	ctrl.chirp = function() {
		/* we have to explictly name the parameters in the event */
		ctrl.doesSomething({name: ctrl.name, something: "chirp"});
	};
	ctrl.warble = function() {
		ctrl.doesSomething({name: ctrl.name, something: "warble"});
	};
	}
})
```

I marked the gotcha's I found with comments:

* The event I called "doesSomething" in the child controller comes through as an attribute named "does-something" in the parent view.  If the name doesn't match properly then the callback just never gets called.
* When I call the event, I have to specify the parameters explictly using json like {} syntax, otherwise get a strange console error.  ("Cannot use 'in' operator to search for '...' in ...")


### Lifecycle hooks

Lifecycle hooks allow us to run code in the component at a series of points during the component lifecycle.  (e.g. during start up or shut down).  

Here's an example of using $onInit: 

bird.js

```javascript
angular.module('jsc2App').component('bird', {
	templateUrl: 'components/bird/bird.html',
	controller: function() {
	var ctrl = this;
	ctrl.$onInit = function() {
		console.log("Bird onInit has been called");
	};
	}
})
```


You can find the full list of lifecycle hooks on this page: https://docs.angularjs.org/guide/component

## Useful modules

There's lots of useful modules.  The AngularUI ones (ui-...) are particularly good.

* ui-router = UI Router, the router we are using above  See the tutorials: https://ui-router.github.io/ng1/tutorial/
* ui-bootstrap = UI Bootstrap makes bootstrap function.  These are amazing.  See the examples:  https://angular-ui.github.io/bootstrap/.  
	* uib-pagination = easy paging controls 
	* uib-tabset = easy tabs
	* uib-datepicker = date picking done in style
* ng-table = lightweight tables with sorting and filtering http://ng-table.com/
* ui-grid = nice formatting for tables, similar to ng-table but maybe heavier
* angular-tree-control = super simple lightweight tree.  http://wix.github.io/angular-tree-control/
* angular-ui-tree = a tree http://angular-ui-tree.github.io/angular-ui-tree/
* angular-xeditable = cool in-place editable things http://vitalets.github.io/angular-xeditable/


## References

* Rather techy intro guide: https://docs.angularjs.org/guide/introduction
* API docs: https://docs.angularjs.org/api
* Bootstrap info: http://getbootstrap.com/components/
* UI Bootstrap info: https://angular-ui.github.io/bootstrap/
* A directory of modules: http://ngmodules.org/
* List of modules by the AngularUI team: https://angular-ui.github.io/

