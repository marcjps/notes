
# PHP

## Command line 

* Display version = php -v
* Display configuration = php -i 
* Display enabled extensions = php -m

## PHP and Apache on Docker

There are Docker images for PHP, some include Apache.

Use the tags on the images to find the versions that you need..

https://hub.docker.com/_/php/

Run the server like this:

```text
docker run -d -p 80:80  --name apache -v /home/ms/projects/php/pdo1/www:/var/www/html php:8.0.2-apache-buster
```

## Composer on Docker

There are Docker images for Composer.

Call the docker composer like this (or wrap this in a shell script):

```text
docker run --rm --interactive --tty \
  --volume $PWD:/app \
  composer <command>
```


## Composer

PHP's package manager is Composer.

The composer package library is available at https://packagist.org/

The package config is composer.json.

```php
{
    "require": {
        "sabre/xml": "^2.2",
        "twig/twig": "^3.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^9.5"
    }
}
```

Note:

* There are both production and development packages.

To add a package:

```text
composer require (--dev) (package)
```

To restore the packages:

```text
composer install
```

To make a production only restore:

```text
composer install --no-dev
```

You will end up with a "vendor" directory that contains the downloaded packages.

To import the packages from your PHP code:

```php
require_once 'vendor/autoload.php'
```

Note:

* The vendor folder should not be accessible from URLs on the web server.


## String functions

* Comparing strings = $one === "foobar". 
* Case insensitive compare = (strcasecmp("foo", "bar") == 0)
* Length = strlen($str)
* Index of = strpos("where is it", "it")
* Last index of = strpos("where is it it", "it")
* Substring = substr("aahello worldaa", 2, 11)
* Replace = str_replace("the", "a", "this is the string") - see also str_ireplace (case insensitive)
* Format = sprintf("%d %s\n", 1, "ok")
* Tokenise = explode(' ', "split into words") - see also str_split which will split based on position rather than content.
* Join = join(" ", array("one", 2)) - same as implode()
* Trim = trim(" whoops "), ltrim(" whoops\n"), rtrim("whoops ")
* Lower case = strtolower("OK")
* Upper case = strtoupper("OK")
* Count occurances = substr_count("the one one one thing", "one")

Combine strings:

```php
print "one " . "two";
```

Interpolation:

```php
foo = 'ok';
print "this works $foo\n";
```

Heredoc for multi line strings:

```php
print <<<EOF
One two
three four

EOF;
```

## Loops

C style for loops:

```php
for ($i = 1; $i < 11; $i++) {
    $total = $total + $this->database->getProfitForMonth($i);
}
```

Foreach loop:

```php
$months = [1,2,3,4,5,6,7,8,9,10,11,12];
foreach ($months as $month) {
    $total = $total + $this->database->getProfitForMonth($month);
}
```

Note:

* Like C, also supports do..while() and while()...

## Regex

The hash/pound at the begininng and end of the pattern is a delimter to separate the expression from the options.  E.g. adding an "i" after the final hash sets the "case insensitive" option.

```php
$pattern = "#^([a-z]+)/(\d\d\d\d)/(\d+)$#";
$text = "uksi/2020/123";
$count = preg_match($pattern, $text, $groups);
print "found $count matches\n";
print_r($groups);
print "\n";
```

$count is the count of matches and $groups are the matched groups.

## Arrays

Initialisation:

```php
$array = [
    "foo" => "a",
    "bar" => "b" 
];
$array2 = [ "one", "two" ];
```

Add:

```php
$array["baz"] = "barry";
array_push($array2, "three"  );
```

Remove:

```php
array_pop($array2);
```

Iterate:

```php
foreach ($array2 as $key => $value) {
    print "$key => $value\n";
}
```

```php
for ($i = 0; $i < count($array2); $i++) {
        print $array2[$i];
}
```

Map with an anonymous function:

```php
array_map( function ($arg)  {
    print("$arg\n");
}, $array2 );
```

Filter with an anonymous function:

```php
$result = array_filter( $array2, function($p) {
        if ($p === "two") return $p;
} );
print_r($result);
```

PHP 7.4 supports arrow functions syntax too.



## Dates

Get current date:

```php
$dateTime = new DateTime();
```

Parse date from string:

```php
$dateTime = DateTime::createFromFormat('Y-m-d', '2020-01-02');
```

Format date to string:

```php
echo $dateTime->format('Y-m-d H:i:s');
print "\n";
echo $dateTime->format('d/m/Y');
print "\n";
echo $dateTime->format(DATE_ATOM);  // Atom date format, https://tools.ietf.org/html/rfc4287
print "\n";
```

Add 10 days:

```php
$date->add(new DateInterval('P10D'));
```

## JSON

* json_decode = turn a string of JSON into an array.
* json_encode = turn an array into a string of JSON.


## Functions

Functions now support declaring return types.

The scalar types are int, string, bool and float.

```php
function foo() : int {
    $one = 1;
    return $one;
}
```


## Constants

In PHP 5:

```php
define("ONE", 1);
printf("There can be only %d\n", ONE);
```

In PHP 7.4: 

```php
const MIN_VALUE = 0.0; 
```


## Classes

Classes now support a lot of features.  Here's an example class.

```php
class BankAccountController
{
    private Database $database;
    public string $Name;

    public function __construct($database)
    {
        $this->database = $database;
    }

    public function getProfitForYear() : int
    {
        $total = 0;
        $months = [1,2,3,4,5,6,7,8,9,10,11,12];
        foreach ($months as $month) {
            $total = $total + $this->database->getProfitForMonth($month);
        }
        return $total;
    }

    public function GenerateStatement() : string {
        $result = "";
        $result .= "Account name: $this->Name\n";
        $profit = $this->getProfitForYear();
        $result .= "Profit for year: $profit\n";
        return $result;
    }
}
```

```php
$bankAccountController = new BankAccountController($database);
$bankAccountController->Name = "Marc's Account";
echo $bankAccountController->GenerateStatement();
```

Notes:

* The object type can be checked with instanceof
* Supports public, private and protected members
* Supports inheritance and overriding methods
* Supports defining interfaces
* Supports static functions
* Supports namespaces and subnamespaces, they apply to classes, interfaces, constants and functions
* Supports $this keyword - remember the $
* Suppors attributes and reflection
* Constructor is a function called __construct()
* Traits are subsets of classes that adds specific functions to a class.
* You can iterate over the class members with foreach.


### Autoloading classes

You normally need to include all referenced files using include statements.  

But PHP supports this strange hack for loading classes it doesn't know about.  Add this function and it will call it when it encounters an unknown class.

```php
spl_autoload_register(function ($class_name) {
    include $class_name . '.php';
});
```

That only looks in the root folder of the project.  

A guy over shows a way for loading classes from different folders:  https://stackoverflow.com/questions/5280347/autoload-classes-from-different-folders  (something that should work by default, *grumble*)

Put an autoloader in a separate file, called something like bootstrap.php.  Then include that file from all the other files.

e.g Bootstrap.php:

```php
spl_autoload_register(function ($className) {
    # Usually I would just concatenate directly to $file variable below
    # this is just for easy viewing on Stack Overflow)
    $ds = DIRECTORY_SEPARATOR;
    $dir = __DIR__;

    // replace namespace separator with directory separator (prolly not required)
    $className = str_replace('\\', $ds, $className);

    // get full name of file containing the required class
    $file = "{$dir}{$ds}{$className}.php";

    // print "Looking for: $file";

    // get file if it is readable
    if (is_readable($file)) require_once $file;
});
```

Put the classes in Folders with matching namespace names.

e.g Controllers/DogController.php:

```php
namespace Controllers;
class DogController {
```

Then reference them like this..

```php
require_once 'bootstrap.php';

use Controllers\DogController;
...
$dogController = new DogController();
```

and they will be autoloaded.


## Errors and exceptions

* From PHP 7 most errors are thrown as exceptions.  Hurrah!  Before that they need an error handler function set.
* Supports try, catch, finally
* Supports nested exception blocks and rethrowing caught exceptions
* Supports custom exception classes by extending Exception

A simple way to create a custom exception which may work is this:

```php
class MyException extends Exception { }
```

## Throwing exceptions

```php
throw new MyException('Division by zero.');
```
## Catching exceptions

```php
try {
    echo inverse(5) . "\n";
    echo inverse(0) . "\n";
} catch (MyException $e) {
    echo 'Caught exception: ',  $e->getMessage(), "\n";
}
```


## Database using PDO

This is better database API than the others that PHP provides. 

* It can access different types of SQL servers
* It supports parameterised queries (aka prepared statements)
* It supports transactions 
* It throws exceptions when there are errors
* It will reuse a pool of connections
* Ensure pdo_mysql (or equivalent) is installed and enabled.


```php
$host = '172.17.0.1';
$db   = 'test';
$user = 'php';
$pass = 'donut';
$charset = 'utf8mb4';
$age = 39;

try 
{
    // connect
    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    $pdo = new PDO($dsn, $user, $pass, $options);

    // parameterised query (prepared statement)
    $stmt = $pdo->prepare('SELECT * FROM person WHERE age > :age');
    $stmt->execute(['age' => $age]);
    while ($row = $stmt->fetch())
    {
        $name = $row['name'];
        print  "<p>$name</p>";
    }

} catch (\PDOException $e) {
    print $e->getMessage();
    print $e->getCode();
}
```

## REST API with no framework

To do this they rewrite all API requests to a single endpoint, turning the path into a query string parameter.  The rewrites are done on the web server using something like mod_rewrite.

Then they write API entrypoint code in PHP which figures out which controller is being called (from the path), which (if any) data item is being referenced (from the path) and what type of method is being called (from the request type).  Once that's all been decoded the PHP code calls a controller method which does the work of querying the database.

For example, it'd be something like this:

* POST /api/person -> rewrites to /api.php?path=/person
* GET /api/person/1 -> rewrites to /api.php?path=/person/1
* api.php will read the path and the request type, and have a large set of conditions deciding which controller and method to call.
* POST /api.php?path=/person would call personController::create
* GET /api.php?path=/person/1 would call personController::get(1)
* etc.



## XML

### SimpleXml

SimpleXml is an XML API built into PHP.  It's a very weird API.  For 'weird' you could substitute 'broken'.

* It can't construct the root element, it has to be parsed from a string 
* It doesn't automatically escape ampersands
* It does actually support xpath queries (at least some of them)
* It can create elements in namespaces
* It can be accessed like a PHP objects/array

Creating XML:

```php

// create root
$root = new SimpleXMLElement('<root xmlns="http://foo"/>');

// create elements and attributes
$things = $root->addChild('things');
$thing = $things->addChild('thing', 'i am thing &amp; safe?');  // it doesn't escape & automatically
$thing->addAttribute('id', 1);

// create an element in another namespace
$elementInAnotherNamespace = $root->addChild('another', 'i am another', 'http://bar');

// show xml
print $root->asXml();

```

Parsing XML: 

```php
// parse from a string
$simpleXmlDocument = simplexml_load_string("<root><things><thing id=\"1\">one</thing><thing id=\"2\">two</thing></things><another xmlns=\"http://bar\">i am another</another></root>");

// access array
foreach ($simpleXmlDocument->things->thing as $thing)
{
    print $thing['id'];
    print $thing;
}

// xquery
foreach ($simpleXmlDocument->xpath("/root/things/thing") as $thing)
{
    print $thing['id'];
    print $thing;
}

// xquery using a namespace
$simpleXmlDocument->registerXPathNamespace('bar', 'http://bar'); // allows us to use the namespace prefix in the xpath
print_r($simpleXmlDocument->xpath("bar:another") );
```


### XML DOM

* I like this is DOM as it is similar to the XML DOM API in other languages
* It will automatically escape special characters as long as you call createTextNode.  
* Annoyingly, perhaps, it provides other ways to populate the inner value of nodes, and they don't all do the escaping.
* XPath queries requires a separate DOMXPath object

Creating XML:

```php
$namespace = 'http://foo';
$xml = new DomDocument('1.0', 'UTF-8');
$root = $xml->createElementNS($namespace, 'example');
$xml->appendChild($root);

$node = $xml->createElementNS($namespace, 'node');
$root->appendChild($xml->createTextNode("this is & safe")); // will escape & but only when using createTextNode.

$root->appendChild($node);
$node->setAttribute('something', 'my attribute');

// show xml
print $xml->saveXML();
```

ParsingXML: 

```php
$smallXmlString = "<root><things><thing id=\"1\">one</thing><thing id=\"2\">two</thing></things><another xmlns=\"http://bar\">i am another</another></root>";

$doc = new DomDocument('1.0', 'UTF-8');
$doc->loadXml($smallXmlString);
$xpath = new DOMXPath($doc);

// xpath query
foreach ( $xpath->query("//thing[1]") as $node)
{
    print $node->getAttribute('id');
    print $node->nodeValue;
}

// xpath query with namespace
$xpath->registerNamespace('bar', 'http://bar');
foreach ( $xpath->query("//bar:another") as $node)
{
    print $node->nodeValue;
}

```

### Sabre/xml

Need to work on this more.



## Twig

Twig is a template system for PHP.  The design of it appears to have been somewhat inspired by AngularJS.

Install twig with:

```text
composer require "twig/twig:^3.0"
```

### Rendering a template

```php
require_once 'vendor/autoload.php';


$loader = new \Twig\Loader\FilesystemLoader('/var/www/html/templates');
$twig = new \Twig\Environment($loader, [
    'cache' => '/var/www/html/cache',
    'auto_reload' => true
]);

$template = $twig->load('index.html');

$variables = [
    'name' => 'WARLORD',
    'cpu' => '386SX 33MHz'
];

echo $template->render($variables);
```

Notes:

* Templates are "compiled" to the cache folder, so the folder needs to be writable by the web server.
* auto_reload means Twig will notice template source code chages and recreate the cache.  If not set, then maybe it never notices.
* I can pass in objects and simple XML arrays, and it works.
* Templates don't have to be on the filesystem, they can be stored on a database.

### Writing a template

I passed in the objects returned by simple_xml_load into this template.

```html
<div id="main">
    {% if machines|length == 0 %}
        <p>Sorry I haven't got any data today.</p>
    {% endif %}

    {% for machine in machines %}
        <div>
            <h4>{{machine.Name}}</h4>
            <dl>
                <dt>CPU</dt>
                <dd>{{machine.Processor}}</dd>
                <dt>Motherboard</dt>
                <dd>{{machine.Motherboard}}</dd>
                <dt>Memory</dt>
                <dd>{{machine.Memory}}</dd>
                {% for video in machine.VideoCards.VideoCard %}
                    <dt>Video Card {{ loop.index }}</dt>
                    <dd>{{video}}</dd>
                {% endfor %}
                {% for sound in machine.SoundCards.SoundCard %}
                    <dt>Sound Card {{ loop.index }}</dt>
                    <dd>{{sound}}</dd>
                {% endfor %}
                <dt>HDD</dt>
                <dd>{{machine.HardDrive}}</dd>
                <dt>CD-ROM</dt>
                <dd>{{machine.CDROMDrive}}</dd>
                <dt>Power Supply</dt>
                <dd>{{machine.PowerSupply}}</dd>
                <dt>Monitor</dt>
                <dd>{{machine.Monitor}}</dd>
                {% for os in machine.OperatingSystems.OperatingSystem %}
                    <dt>Operating System {{ loop.index }}</dt>
                    <dd>{{os}}</dd>
                {% endfor %}
            </dl>
        </div>
    {% endfor %}
</div>
```

Notes:

* The template can be any kind of file (usually .html)
* It doesn't use PHP language, it has its own language.
* Supports filters/pipes, like {{ foo | upper }}
* Supports loops and conditions
* Supports comments {# comment #}
* Supports "includes"
* Supports inheritance - a parent with defined blocks which can be overridden by a child.  Hmm.  Kind of the opposite to components.
* I'm not sure, but I think Twig provides no particular support for forms or input fields.  Symfony and Drupal might be building their own twig templates or extensions to help them integrate forms.

## Unit testing

### PHPUnit

PHPUnit is a unit testing framework.  It provides a way to write and run tests.  It also provides an API for mocking objects.

Installation: 

```text
composer require --dev phpunit/phpunit ^9
```

To run all tests in a folder:

```text
./vendor/bin/phpunit tests
```

To run tests in a particular file:

```text
./vendor/bin/phpunit Controllers/DogControllerTest.php
```

You can write a configuration file that specifies the tests to run:

phpunit.xml:

```xml
<phpunit bootstrap="bootstrap.php">
    <testsuites>
        <testsuite name="geoff">
            <directory>Controllers</directory>
            <directory>Models</directory>
        </testsuite>
    </testsuites>
</phpunit>
```


Writing tests:

```php
<?php
declare(strict_types=1);
require_once 'bootstrap.php';

use PHPUnit\Framework\TestCase;
use Models\Dog;

class DogTest extends TestCase {

    public function testCanBark() {
        $dog = new Dog();
        $this->assertEquals(
            'Woof!',
            $dog->Bark()
        );
    }

    public function testCanChew() {
        $dog = new Dog();
        $good_number = 5;
        $this->assertLessThan(
            $good_number,
            $dog->Chew($good_number)
        );
    }
}
```


Note:

* It only calls test function names that are prefixed with the word "test".  Alternatively we can add @test in a comment above the method.
* The class has to extend TestCase.
* You can add setUp() and tearDown() methods if you want code to run before and after the tests.
* PHPUnit provides a way to create mocks.


### Mockery

Mockery provides a nicer human readable API for creating and testing mock objects.  It allows us to create objects with preprogrammed responses, which are to be injected into the system under test, and then test how the system used the objects.  The "how the system used" tests are called expectations.  We expect the system to use the object in a particular way.

I think we shouldn't really test expectations.  When unit testing I care if the component works, but I don't want to test how it's implemented.  I think the developer is free to write a completely different implementation, if it still passes the tests, then it is OK.  That's part of the whole point of the unit test really, we can change the implementation and test that it will still work.  The expectations might break this by demanding that it is implemented in a particular way.

Installation:

```text
composer require --dev mockery/mockery
```

We must add a call Mockery::close in our tearDown.

```php
public function tearDown() : void
{
    Mockery::close();
}
```

A mock is meant to be set up with an expectation and preprogrammed to return certain values.  

The mock would usually be used by the class you're testing.  You would verify that the class under test interacted with the mock appropriately.

To create a mock with a method that expects to be called:

```php
$database = Mockery::mock('Database');
$database->shouldReceive('getProfitForMonth')->with(1)->times(1)->andReturn(1);
$this->assertEquals(
    1,
    $database->getProfitForMonth(1),
);
```

Note:

* The created mock is actually inherited from from your named class Database, so it has the correct object type.  But you can't call the real method implementations on your class, unless make a partial mock.
* You can only set the shouldReceive expectation for each named method once.  If you do it again, the second one will just replace the original definition.
* If we call a method on a mock that we didn't preconfigure, the test will fail with "method does not exist on this mock object"
* allows() and expects() are slight variations on shouldReceive()
* times() is used to state how many times we expect this call.  If we call it a different number of times, the test will fail.  You can also use atLeast(), atMost() and between()
* with() is used to state the arguments we expect.  If we call it with different arguments that we specified, the test will fail.  The with() argument can be a list of values.
* It's possible to add closure to the expectation, that runs when the method is called, recieves the argument, and returns true to pass the test, or false to fail it.
* andReturn says the method should return a value when called.  We can also use a closure there.


An alternative is a Spy.

A spy doesn't need to be preprogrammed, but can be interrogated later.  A spy returns null for unknown methods, but doesn't throw an error.

```php
$database2 = Mockery::spy('Database2');
$this->assertEquals(
    null,
    $database2->getProfitForMonth(1),
);
$database2->shouldHaveReceived('getProfitForMonth')->with(1);
```

Note:

* I can still set shouldRecieve expectations on the spy and return preprogrammed values.  The mock throws errors if they are never called, they spy doesn't.
* You can still use shouldHaveReceived after you called the real implementation of your class (enabled with makePartial).
* shouldNotHaveReceieved() provides the opposite of shouldHaveReceived()

A partial mock allows you to mock some methods while calling the original implementation of others.  To make a partial mock:

* Call makePartial() on a mock, which will make any method without an expectation call the original implementation, or
* When creating the mock, define which methods you will mock, like this: $mock = \Mockery::mock('Dog[bark,growl]'), then add expectations for those methods only.





## REST API (frameworks)

Wow, there's a lot of frameworks.   Where do we begin?

* Laravel
* Symfony
* Slim
* Lumen
* Leaf
* Swoft
* Silex
* Wave


# References

* PHP language reference: https://www.php.net/manual/en/langref.php
* PDO tutorial: https://phpdelusions.net/pdo
* PDO examples: https://phpdelusions.net/pdo_examples
* Rest API without any help from framework - https://developer.okta.com/blog/2019/03/08/simple-rest-api-php
* Twig documentation: https://twig.symfony.com/doc/3.x/
* Mockery reference: http://docs.mockery.io/en/latest/reference/
