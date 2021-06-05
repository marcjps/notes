# REST API design

### The difference between REST and RPC design philosophy

Remote Procedure Call (RPC) is a design philosophy where we use an API to invoke an operation on a remote server.  RPC in different forms has been used extensively in computing for many years.  An example of an RPC operations on URIs would be along the lines of:

* api/drink_add = Add a new drink
* api/drink_view = View a drink
* api/drink_search = Search for drinks
* api/drink_reorder = Trigger reordering a drink from warehouse
* api/drink_rating_view = view drink rating
* api/drink_rating_set = set drink rating

REST philosophy is about resources rather than operations, borrowing from the approach that HTTP uses to access web pages.  URIs are only used to describe resources, and operations are performed on them using HTTP Methods.  This limits the operations that can be performed on resources to a small number (e.g. Create, Read, Update, Delete).  An example of REST URIs and operations are:

* POST api/drinks = Add a new drink
* GET api/drinks = List drinks
* PUT api/drinks/123 = Update drink 123
* DELETE api/drinks/123 = Delete a drink

So with the REST philosophy, we cannot tell the server to do something, such as reordering a drink like we could with the RPC call.  We'd need to design our system so that the server did the drink ordering based on the resource data rather than using an explicit command from a client.  

Since this is the case, the appropriate API design to use really depends on the scenario.

Note: also of course, all REST calls are really just a limited set of RPC calls that interact with resources, but let's not go there.  REST vs RPC are design philosophies not particularly technical qualities. 

### English lesson

People like to refer to verbs and nouns when describing APIs.  For no particular reason, lets go all Cambridge English Dictionary for a moment.

* Verb = a word or phrase that describes an action, condition, or experience
* Noun = a word that refers to a person, place, thing, event, substance, or quality

In REST our HTTP Methods are verbs (GET, POST, PUT, DELETE) and our URIs are nouns. (The name of drink 123: /api/drinks/123).

### Versioning

I think a big thing to remember in REST API development is that the API isn't an exposure of the applications internal models to the outside world.  It is Just Another Interface for the external consumers.  We may need to change our application code and keep the API working the same way in order to support the existing external consumers.

This is a reason why Versioning APIs is super-important.  Our application code will change, we need the old APIs to still work, and we may want to add new APIs.  We cannot do this properly if we did not put version numbers on our API URIs.

E.g. API version 1:

	/api/v1/drinks
	/api/v1/drinks/124

E.g. later, different, API:

	/api/v2/beverages/drinks
	/api/v2/beverages/drinks/124

### Should our UI call our API?

It is certainly possible to build an API for everyone and then build our own UI on top.  This is perhaps an example of Eating Your Own Dog Food, i.e. you're gonna be damn sure your API is good if you have to use it yourself.

However, my personal opinion right now, purely based on development experiences, I think no, our UI shouldn't be built on the same API that we present to external consumers.

We're developing a UI experience (for certain consumers) and an API experience (for certain other consumers).  The two things don't necessarily operate the same way.  We can make our life much more difficult than necessary by setting a requirement that our UI runs on our API when no customer gave us this requirement.  Since our food is for dogs not humans, I think we can make better dog food if we keep thinking about what the dogs need from it, not what we need from it.


### Pluralise the name or not?

We have to stick to one naming convention, either:

	/api/drinks
	/api/drink

Pluralising (/api/drinks) seem to be the most common way to go.

### JSON or XML

For APIs that are consumed by JavaScript on web pages, JSON is clearly the better option due to its smaller size and faster parsing.  

### HATEOAS or not?

HATEOAS principle is to include links to contextually available API URIs in the responses, so theoretically the client programmer (or the code itself) knows what URI to use next.

Providing the URIs can save the client code the effort of building the URIs, although then the server code takes on the same burden.  Which one is best perhaps depends on your scenario.

## Authentication

If your API allows a client to update data, then you need authentication.  You may also need this for rate-limiting purposes. (see below)

Authentication is usually done by giving each client a token (an API key), which they send with all of their API requests.  REST APIs should always be stateless so do not use a session.  Send the token with every request.

You will need a method of granting tokens to each specific user/client.  This is usually done via some other forum, (not by the API itself) e.g. by showing them in the User Profile page of your website.

Tokens are often used in two parts to make them more secure - a key and a value.  Quite similar to a username and password.

Methods for sending the token with the requests are:

* Basic auth 
* OAuth 2.0 (which also uses the Authorization header)
* Query string parameters

## Rate limiting

We can implement a rate limit to slow down agressive clients who are hurting the server.

In order to do this, it is better if all clients are identified with a different key, as it is difficult to identify clients by other means.

The API server needs to count the request-rate that is being used by each client/key.

In the API responses it is customary to include extra HTTP headers specifying how close to the rate limit the client is.  e.g.

* X-Rate-Limit-Limit - The number of allowed requests in the current period
* X-Rate-Limit-Remaining - The number of remaining requests in the current period
* X-Rate-Limit-Reset - The number of seconds left in the current period

If the client hits the limits, requests should fail with the following response:

	HTTP 429 Too Many Requests


## Operations

### Create

	POST /api/drinks

Request body:

	{
		name: "Pepsi"
	}

Response:

	HTTP 201 Created
	Location: /api/drinks/123

It may be debatable whether we use a HTTP header or the body to redirect to the new location of the object.  Using the location may be better as it saves us creating an extra type for the redirection reponse object.

Return no response body.

### Read all

	GET /api/drinks

Response:

	HTTP 200 OK

Response body:

	[
		{
			id: "123",
			name: "Pepsi",
			href: "/api/drinks/123"
		},
		{
			id: "124",
			name: "Coca-Cola",
			href: "/api/drinks/124"
		}
	]

Note.  Including the hrefs in the responses here is optional.  Depends if you HATEOAS or not.

You can add filtering and sorting as URI parameters, e.g.

	GET /api/drinks?name=Co&sort=name&order=asc


### Read a specific one

	GET /api/drinks/123

Response:

	HTTP 200 OK

Response body:

	{
		id: "123",
		name: "Pepsi"
	}


### Update

	PUT /api/drinks/123

Request body:

	{
		name: "Pepsi Cola"
	}

Response:

	HTTP 200 OK

Return no response body.

### Delete

	DELETE /api/drinks/123

Response:

	HTTP 200 OK

Return no response body.

## Errors

We should utilise HTTP error codes when possible.  We could return a response body giving more information about the error.  This could be a compltely different data type (e.g. an Exception object) from the APIs normal object type.

Not found:

	HTTP 404 Not Found

Unauthorised:

	HTTP 401 Unauthorised

Internal server error:

	HTTP 500 Internal Server Error

Response body:

	{
		type: "Exception",
		message: "Object reference not set to an instance of an object.",
		stack: [
			...
		]
	}


## Dependent (child) resources

The child resource API will operate the same way, but will live on an URI below the main resource, e.g.

	POST /api/drinks/123/ingredient
	GET /api/drinks/123/ingredient
	GET /api/drinks/123/ingredient/100
	PUT /api/drinks/123/ingredient/100
	DELETE /api/drinks/123/ingredient/100

## Long-running jobs

Implementing an API for a background job fits the pattern well.

This is how you would post the initial job to the API.

	POST /api/job

Request body:

	{
		type: "Validation"
	}

Response: 

	HTTP 201 Created
	Location: /api/job/666

No response body.

Now the client can poll the job status.

	GET /api/job/666

Response:
	
	HTTP 200 OK

Response body:

	{
		id: "123",
		type: "Validation",
		status: "Processing"
	}

When the job completes you can include the job results (or just links to them) in the response.

	{
		id: "123",
		type: "Validation",
		status: "Complete",
		output: "/api/job/666/output.pdf"
	}

## Interesting references

* http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api
* https://blog.mwaysolutions.com/2014/06/05/10-best-practices-for-better-restful-api/
* https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/

