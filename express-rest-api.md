# Express REST API

This is my first attempt at making a REST API using node.js and Express.  I'm using a MySQL database.

## Setting up a project

I used [Express Generator](https://expressjs.com/en/starter/generator.html) to scaffold a project using node.js and Express.  

```text
sudo npm install express-generator -g
express --view=pug expressrestapi
```

This gets us some boilerplate code with a couple of sample API entry points.

To run the project, use:

```text
npm start
```

Then visit http://localhost:3000 to see the result.

The generated project includes code for a couple of sample routes.

* http://localhost:3000/ returns a page created from a Pug (aka Jade) template.  
* http://localhost:3000/users returns a string.

## Connecting to MySQL

We can add a [MySQL client](https://github.com/mysqljs/mysql) to the project using this command:

```text
sudo npm install mysql --save
```

I found tutorials on the web showing how to connect to MySQL by creating a connection and calling connect, for example:

```javascript
var mysql = require("mysql");

connection = mysql.createConnection({
    host     : 'zen',
    database : 'jobsystem',
    user     : 'jobsystem',
    password : 'test'
});

connection.connect();
```

This works but is not very helpful here.  The application has one connection, and if that breaks, then our code has to deal with making a new, working connection.  That's a bit messy.

Luckily the client supports connection pools.  We can define a connection pool by adding this code in app.js:

```javascript
var mysql = require("mysql");

pool = mysql.createPool({
    host     : 'zen',
    user     : 'jobsystem',
    database : 'jobsystem',
    password : 'test'
});
```

The connection pool will automatically deal with making new connections and will deal with discarding and replacing broken ones.

## Implementing an API to query a table

To run a query, I added some code to the users.js file, replacing the original sample that returned a string.  

```javascript
router.get('/', function (req, res, next) {
    query = "SELECT * from jobsystem_service";
    pool.query(query, function(error, results, fields) {
        if (!error)
            res.send(JSON.stringify({"status": 200, "error": null, "response": results}));
        else
            res.send(JSON.stringify({"status": 500, "error": error, "response": null}));
    });
});
```

The MySQL client allows us to run the query on the pool, which will execute it on a connection for us.

Accessing http://localhost:3000/users now returns the results of the query.  I am also able to restart the MySQL server, breaking any existing connections, and new connections are automatically established without raising an error.

When the MySQL service is stopped, the pool returns the "error" value instead of the "results".  I handled that and had the API result become a 500 error.

## Parameterising queries

The [MySQL client documentation](https://github.com/mysqljs/mysql#escaping-query-values) gives us a few ways to parameterise queries.  

We can put "?" placeholders into the SQL statement and then pass an array of corresponding parameters into the query.

```javascript
let params = [139];
query = "SELECT * FROM jobsystem_service WHERE ID = ?";
pool.query(query, params, function(error, results, fields) {
```

Parameterising queries protects them against SQL injection attacks.

## HTTP methods

We can implement APIs for different HTTP methods like this:

```javascript
// HTTP POST
router.post('/', function (req, res, next) {
    res.send("post one");
});

// HTTP GET
router.get('/', function (req, res, next) {
    res.send("get all");
});

// HTTP GET
router.get('/:identifier', function (req, res, next) {
    res.send("get one");
});

// HTTP PUT
router.put('/:identifier', function (req, res, next) {
    res.send("put one");
});

// HTTP DELETE
router.delete('/:identifier', function (req, res, next) {
    res.send("delete one");
});
```

## Reading route parameters from the URI

For a route such as /service/foo, we can read "foo" using req.params:

```javascript
router.get('/:identifier', function (req, res, next) {
    res.send("you posted " + req.params.identifier);
});
```

## Reading query string parameters from the URI

For a route such as /service?thing=foo, we can read "foo" using req.query:

```javascript
router.get('/', function (req, res, next) {
    res.send("you get " + req.query.thing);
});
```

## Reading parameters from the post body

For reading a HTTP POST or PUT with a JSON body we can read a parameter from the body using req.body.

```javascript
router.post('/', function (req, res, next) {
    res.send("you posted " + req.body.thing);
});
```

## Full API implementation 

### GET API - retrieve all rows

```javascript
router.get('/', function (req, res, next) {

    let query = "SELECT * FROM jobsystem_service";
    pool.query(query, function(error, results, fields) {

        if (!error)
            res.send(JSON.stringify({"status": 200, "error": null, "response": results}));
        else
            res.send(JSON.stringify({"status": 500, "error": error, "response": null}));
    });

});
```

### GET API - retrieve one row

```javascript
router.get('/:identifier', function (req, res, next) {

    let params = [req.params.identifier]
    let query = "SELECT * FROM jobsystem_service WHERE Identifier = ? ";
    pool.query(query, params, function(error, results, fields) {

        if (!error)
            res.send(JSON.stringify({"status": 200, "error": null, "response": results}));
        else
            res.send(JSON.stringify({"status": 500, "error": error, "response": null}));
    });
});
```

### POST API - create a row

Here's a sample post body:

```json
{
    "Name": "marc",
    "Description": "test",
    "TimeoutInSecs": "30",
    "IsEnabled": "1",
    "RetryAttempt" : "0",
    "Identifier": "test-test"
}
```

Here's the API which recieves the body and inserts a new row using these parameters.

I'm using a feature of the MySQL client which lets us pass an array of named parameters in to the query.  Potentially we could simply pass the req.params array in, but we should probably make sure the API handles only the parameters we are expecting.

```javascript
router.post('/', function (req, res, next) {

    let params = {
        Name: req.body.Name,
        Description: req.body.Description,
        TimeoutInSecs: req.body.TimeoutInSecs,
        IsEnabled: req.body.IsEnabled,
        RetryAttempt: req.body.RetryAttempt,
        Identifier: req.body.Identifier
    };
    let query = "INSERT INTO jobsystem_Service SET ?";
    pool.query(query, params, function(error, results, fields) {

        if (!error)
            res.send(JSON.stringify({"status": 200, "error": null, "response": results}));
        else
            res.send(JSON.stringify({"status": 500, "error": error, "response": null}));
    });

});
```

### PUT API - update an existing row

For this statement we read the "identifier" param from the URI, and the other params from the body.

```javascript
router.put('/:identifier', function (req, res, next) {

    let params = [
        req.body.Name,
        req.body.Description,
        req.body.TimeoutInSecs,
        req.body.IsEnabled,
        req.body.RetryAttempt,
        req.params.identifier
    ];
    let query = "UPDATE jobsystem_Service SET Name=?, Description=?, TimeoutInSecs=?, IsEnabled=?, RetryAttempt=? WHERE Identifier = ?";
    pool.query(query, params, function(error, results, fields) {

        if (!error)
            res.send(JSON.stringify({"status": 200, "error": null, "response": results}));
        else
            res.send(JSON.stringify({"status": 500, "error": error, "response": null}));
    });

});
```

### DELETE API - delete a row

```javascript
router.delete('/:identifier', function (req, res, next) {

    let params = [
        req.params.identifier
    ];
    let query = "DELETE FROM jobsystem_Service WHERE Identifier = ?";
    pool.query(query, params, function(error, results, fields) {

        if (!error)
            res.send(JSON.stringify({"status": 200, "error": null, "response": results}));
        else
            res.send(JSON.stringify({"status": 500, "error": error, "response": null}));
    });
});

```
