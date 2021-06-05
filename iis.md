# IIS

## Hosting a Single Page Application

### Rewriting all URIs to /

We need all URIs to redirect to the Single Page so that the JavaScript router can process the URI.

* Install the URL Rewriter Module on the web server (can be done with the Web Platform Installer)
* Add Rewrite rules which rewrite all routes to / so the JavaScript single page will run.
* If you are also running Web API, negate its paths in the rewrite otherwise they won't work.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<system.webServer>
		<rewrite>
			<rules>
				<rule name="Routes" stopProcessing="true">
					<match url=".*" />
					<conditions logicalGrouping="MatchAll">
						<add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
						<add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
						<add input="{REQUEST_URI}" pattern="^/(api)" negate="true" />
					</conditions>
					<action type="Rewrite" url="/" />   - or whatever your base-url is
				</rule>
			</rules>
		</rewrite>
	</system.webServer>
</configuration>
```

### Adding MIME types

IIS will refuse to serve files if the file extensions aren't included in its MIME type list.

Luckily you can define new MIME types in web.config.  

```xml
<configuration>
	<staticContent>
		<mimeMap fileExtension=".woff" mimeType="application/font-woff" />
		<mimeMap fileExtension=".woff2" mimeType="application/font-woff2" />
		<mimeMap fileExtension=".json" mimeType="application/json" />
	</staticContent>
</configuration>
```

### Caching static files

In web.config we can take control of how long static files (e.g. JavaScript) are cached for.  If they are cached too long, then it is difficult for us to release updates.  

The code below sets a one-hour cache.

```xml
<system.webServer>
    <staticContent>
        <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="0.01:00:00"/>
    </staticContent>
</system.webServer>
```


## Dynamic types in Web API

The C# "dynamic" type means we can create objects without strong typing.  

I rather love dynamic as I feel it's often not necessary to strongly type items.  

In serious production projects doing strong typing feels like the better thing to do.  I feel that dynamic is a bit of a cheating shortcut!

### Posting a dynamic into a Web API

The first amazing way to use a dynamic is to accept it as the posted data into an API method, e.g.

```
[HttpPost]
[System.Web.Http.Route("Search")]
public dynamic Search(dynamic options)
{
	string query = options.query;
	DateTime date = options.date;
	...
}
```

I find this is a benefit when my JavaScript has posted in something to a method but I'm not quite sure how C# is going to interpret it.  Using the dynamic I can put a breakpoint in the function and then examine the incoming object in the debugger.

If I'm trying to access a parameter which doesn't exist in the post, I will get an exception thrown from the exact line of code with the error.  That is useful for resolving the problem.  When using strong typing I might get a more ambiguous 400 Bad Request.

Of course this also saves me defining a type which matches the incoming form post.  (which could be a type I may never need elsewhere)

### Creating a dynamic directly from a database result set and returning them as an API response

The second amazing way to use a dynamic is in the responses from the API.  

Using Strong Typing you would read a database result set, fill in a strongly typed object matching all the fields, and return the object from the API.  Maintaining objects when you want to work on different queries is a bit of a drag.

Using the dynamic we can simply build the dynamic object on the fly from the result set.  When we change the query, the dynamic and the API response changes to match.  That saves some effort.

The code to create a dynamic from a database reader is along these lines.

```
using System.Dynamic;
...
private dynamic SqlDataReaderToExpando(SqlDataReader reader)
{
	var expandoObject = new ExpandoObject() as IDictionary<string, object>;
	for (var i = 0; i < reader.FieldCount; i++)
		expandoObject.Add(reader.GetName(i), reader[i]);
	return expandoObject;
}
```

You can then return the dynamic as the Web API response, and you will get a JSON/XML response from the API which matches the response from the SQL query.  That's a neat trick.

Possibly an ORM like Hibernate or Entity Framework might also reduce our object maintenance drag, as it will create the classes matching the database for us.  But for me the dynamic is a simple, fast and flexible approach.  


## Some specialised workarounds for things that don't work in Web API by default

### Dots in URIs

By default .Net won't process an URI that contains a dot.  For Web API this means the API method won't be called even if the pattern matches.

If you need that to work, adding RunAllManagedModulesForAllRequests is one way to achieve it.  

```xml
<system.webserver>  
	<modules RunAllManagedModulesForAllRequests="true">
	...
	</modules>
</system.webserver> 
```

Note: This may have security implications and is perhaps not recommended for external facing services.

### Accept connections from other machines in IIS Express

When using multiple machines for development, sometimes I need IIS Express to accept incoming connections.

Run this command to allow access to the appropriate port:

```
netsh http add urlacl url=http://*:6426/ user=everyone
```

Ensure the port is open on Windows Firewall.

Look at IIS Express and note the name of the running application.  

In your Solution Folder there will be a hidden file at .vs/config/applicationhost.config.  

Edit the file and look for the name of the running application we noted earlier.  It should have a definition that looks like this:

```xml
<site name="JobSystemAdministration(4)" id="100">
	<application path="/" applicationPool="Clr4IntegratedAppPool">
		<virtualDirectory path="/" physicalPath="C:\Source\jsc\MySQL\JobSystemAdministration" />
	</application>
	<bindings>
		<binding protocol="http" bindingInformation="*:6426:localhost" />
	</bindings>
</site>
```

Change the binding and replace the "localhost" restriction with an asterisk.

```xml
<binding protocol="http" bindingInformation="*:6426:*" />
```

Then shut down IIS Express and restart it.

Note: if you add serverAutoStart="true" to a site, it will start that site when you run IIS Express directly from the project folder.  (but then you can't debug it without a debugger attached)

Reference: https://stackoverflow.com/questions/3313616/iis-express-enable-external-request


### Cross Origin Resource Sharing (CORS)

When using multiple machines for development, sometimes I need the Web API to accept incoming requests from a JavaScript SPA running on a different machine.  

Web Browsers normally prevent JavaScript from opening network connections to remote servers (except the server that the JavaScript is running from)  

CORS is new feature which enables the connections.  I think this works by placing new HTTP Headers on the target server responses.  The header specifies what servers running JavaScript are allowed to connect.  The Web Browser checks the headers on the target server and will allow matching JavaScripts to call the server.  (where normally the Web Browser would deny it)

Here are the steps to enable CORS on Web API.

* Install the CORS module using Nuget

```
Install-Package Microsoft.AspNet.WebApi.Cors
```

* In WebApiConfig.cs, add the line:

```
config.EnableCors();
```

* On the Web API class, add the following decoration:

```
[EnableCors(origins: "http://localhost:3000,http://localhost:6246", 
	headers: "*", 
	methods: "*",
	SupportsCredentials = true)]
public class TestController : ApiController
{
}
```

Note: The origins are the list of URIs which should be allowed to make requests to the API.  

Note 2: SupportsCredentials enables use of incoming Basic Authentication against the API server.

Reference: https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api






