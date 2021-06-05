
# Misc. JavaScript Language features

This page lists some useful features of ES5, ES6 and beyond.

### Let

Let is used to declare variables.  It is a slight improvement to var, as it is block scoped.  (i.e. it is only in scope between the curly braces that it is declared within).  Var was function scoped (would be in scope for an entire function regardless)

	let something = 3;

	for (let day of this.days) {
		console.log("day: " + day.toDateString());
	}

### Classes (and inheritance)

ES6 brings classes to JavaScript for the first time.

	class Bird {
	  constructor(weight, height) {
	    this.weight = weight;
	    this.height = height;
	  }

	  walk() {
	    console.log('walk!');
	  }
	}

	class Penguin extends Bird {
	  constructor(weight, height) {
	    super(weight, height);
	  }

	  swim() {
	    console.log('swim!');
	  }
	}

	let penguin = new Penguin(...);
	penguin.walk(); //walk!
	penguin.swim(); //swim!


### Arrow functions

The arrow operator provides a quick way of defining a function.

Instead of:
	
	var multiply = function(x, y) {
		return x * y;
	}

We have:

	var multiply = (x, y) => { 
		return x * y; 
	}

### String interpolation and template strings 

Strings declared using backticks support string interpolation.

	console.log(`hello my name is ${name}, I am ${age} years old`);

It also works when accessing properties, no problem with:

	console.log(`The value is: ${fee.fi.fo.fum}`);

They also allow linebreaks within the string (similar to heredoc in PHP).

	console.log(`hello my name is ${name}, 
	I am ${age} years old`);


### Modules 

A module is a file that contains JavaScript code.  In ES6 you can't access the functions or variables defined in the file unless you explicitly export them from the module and then import them in the calling code.

To export things we use the export keyword:

	export class MeetupDataService implements OnInit { ... }

	export const the_number = 42;

You can also add a "default" export which becomes the default thing you get when referencing the module.

	export default class { ... };

To import it we use the import keyword.

	import {MeetupDataService} from './meetup-data.service';
	import {foo, bar} from 'my-module';
	import * as myModule from 'my-module';

Note: webpack combines all our JavaScript files into a single file (main.js).  Somehow it fixes the imports so they still work.  I think that's what the big chunk of boilerplate code that webpack adds to the application does.

## Spread and rest

Spread: passes an array into a function as a set of separate arguments.  

	let args = [3, 5];
	add(...args); // same as add(3,5)

	let cde = ['c', 'd', 'e'];
	let scale = ['a', 'b', ...cde, 'f', 'g'];  // same as scale = ['a', 'b', 'c', 'd', 'e', 'f', 'g']

Rest: allows infinite optional arguments to a function.

	function print(a, b, c, ...more) {
	  console.log(more[0]);
	  console.log(arguments[0]);
	}
	print(1, 2, 3, 4, 5);
	// 4
	// 1


### Strict comparison operators

Strict comparison operators that will stop items evaluating to equal when they equal (but are different data types).

* === must be equal (and same type)
* !== not equals (or not same type)

### forEach

Just loop through a sequence of items.

	that.state.servicestatus.forEach( (tempServiceStatus) => {
	    if (tempServiceStatus.Service.ID === item.ID) {
	        serviceStatus = tempServiceStatus;
	    }
	});

### find

Find an item in a sequence of items.

	serviceStatus =
	    that.state.servicestatus.find(
	        tempServiceStatus => tempServiceStatus.Service.ID === item.ID
	    );

### map

Transforms a sequence of Bars into other Foos.

	let Foos = this.state.Bars.map(function (item, i) {
	    return (
	    	<Foo />
	    )
	};

### Implementing a promise

Write a function that does an async operation and wrap the code in "return new promise".  When the async operation completes, call "resolve".  If there's an error, call "reject".

	function getUsername() {

		var rl = readline.createInterface(process.stdin, process.stdout);
		rl.setPrompt('username> ');
		rl.prompt();

		return new Promise(function (resolve, reject) { 
			rl.on('line', function(line) {
				line = line.trim();
				if (line.length > 0) {
					rl.close();
					resolve(line);
				}
				else {
					rl.prompt();            
				}
			}); 
		});
	}

Call the function and retrieve the promise.  Then wait for it to call back with "then".

	var userNamePromise = getUsername();
	userNamePromise.then( userName => {
		console.log("The username is " + userName);
	});



## References

* Learn ES6 = https://ponyfoo.com/articles/es6

