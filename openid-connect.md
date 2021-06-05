# OpenID Connect

This is the first document to read before anything else, it covers a lot:

https://connect2id.com/learn/openid-connect

This list of certified software is quite useful if you want to find tools or libraries:

https://openid.net/developers/certified/

There's a free book called The OpenID Handbook:

https://assets.ctfassets.net/2ntc334xpx65/2yRtkzYHiiBLLSguFsnQs9/419405cee8bd0a7b8f70e20cef22c190/The-openid-connect-handbook-v1.pdf


## Authentication Flows

The different flows look like one of the key things to understand.  

When data is passed to the web client using a parameter in the URL, it's not very secure.  Someone could grab this token and attempt to use it later.

* Implicit Flow is used when passing data on the redirect URL is the only option available.  The client isn't able to make a further request "in secret" to the provider.  So we just redirect to the provider, get authenticated, and then redirect back to the client with a ID token on the URL.  This token is at risk from being stolen so it has a short lifetime.  The only way to refresh it is to perform the redirect again.
* Authorisation Code flow is used when we can keep a longer life token more secret.  We redirect to the provider, get authenticated, and then redirect back with a one-time-use Authorisation Code.  The client then makes a separate request to a token endpoint, sending the Authorisation Code, and it recieves a Access Token (and perhaps an ID Token) in return.  
* Hybrid Flow is a combination of the two.  We make the redirect and get an ID Token we can use immediately without calling the token endpoint.  But we also get the authorisation code so we can get a Access Token for further usage too.

## Types of tokens

So the different types of data being passed around are:

* ID Token - ID tokens contain information about the user (e.g. user profile data) for the client to use.  The ID token format is a JSON Web Token.  (JWT)   The ID token can't be used to request data from the provider.
* Access Token - The access token is a token you can send to the provider to retrieve further information.  The token proves that you have been authenticated as the user.  It's ususally send on the Authorization header.
* Refresh Token - The refresh token is used to get a new access token (after the access token expires).  Refresh tokens have a long lifetime but can be revoked.
* Authorisation Code - The code is just a string which you can send to the token endpoint to retrieve a token.  It usually works only once.

The tokens usually have an expiry time which is set by the identity provider.  Tokens which are being used insecurely on the client have a shorter lifetime than ones that are passed more securely.

## Response Types

When redirecting to the Identity Provider, the client requests one or more response types.

* id_token = I'm asking for an ID Token 
* token = I'm asking for an Access Token
* code = I'm asking for an Authorisation code

## Identity provider

The identity provider I used is Keycloak.  It is open source, can be self-hosted, and can run in a docker container.  It's written in Java and has a SQL database behind it.  It can run on various databases including MySQL and SQL Server.  The docker image uses an embedded H2 database.

To run keycloak in docker use this command:

```text
docker run -p 8080:8080 -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin quay.io/keycloak/keycloak:10.0.1
```

To do some basic setup, go to http://localhost:8080 and log in as admin.

* Hover over the realm dropdown and click Add Realm.  A realm is a top level group of users and clients (applications)
* If you want uses to be able to self-register (sign up) you can enable it in the realm settings.  Users are then able to create new user accounts on the identity provider directly.
* Go to Manage->Users and add a user, to be used on an application.
* If you want to test the user login you can go to: http://localhost:8080/auth/realms/(realm-name)/account and log in to the users profile.
* Then you need to add a client (application).  Getting all the config settings matching between the identity provider and the client is the main challenge of basic setup.

## Client libraries

Here's some info about libraries and examples I looked at.

### React:

* https://github.com/IdentityModel/oidc-client-js - this is a generic JavaScript openidc client.  The other samples all use this.
* https://www.npmjs.com/package/react-openidconnect - this is a simple wrapper around the above.  It's actually a bit buggy (it can't find the id_token in the URL) but managed to fix it.  It is set up to do only implicit flow.
* https://github.com/AxaGuilDEv/react-oidc - in here you can find two examples, one using react context and the other using redux.  I think these both use the authorisation code flow.

For the https://www.npmjs.com/package/react-openidconnect clint, the client settings are in OidcSettings.js.  

```javascript
var OidcSettings = {
    authority: 'http://localhost:8080/auth/realms/marcsrealm/',
    client_id: 'react-test-openid',
    redirect_uri: 'http://localhost:3000',
    response_type: 'id_token token',
    scope: 'openid profile roles',
    post_logout_redirect_uri: 'http://localhost:3000'
};

export default OidcSettings;
```

* The Authority points to the URL of the Identity Provider.  I noticed the client tries to retrieve an URL from the provider that looks like this: http://localhost:8080/auth/realms/marcsrealm/.well-known/openid-configuration so you need to make sure that the setting is pointing to the right place to find it.
* The ClientId is the name you registered this client with in the identity provider.
* The Redirect URI and post_logout_redirect_uri are the URIs where the Idp should redirect back to the application.  
* The Response Type identifies the required flow type.   "id_token token" means return an ID Token and an Access Token.  As we have retrieved these directly from the redirect we're using Implicit Flow.
* The scopes are what data we are requesting access to.  Each scope contains attributes called claims.


### Dotnet Core:

* https://github.com/onelogin/openid-connect-dotnet-core-sample - a good, working dotnet core sample that uses authorisation code flow.

The client settings are in startup.cs (although you're meant to save them to a secret store):

```c#
options.ClientId = "dnc1"; 
options.ClientSecret = "b3748a50-d120-4ff8-b7f4-7ca9adb436ea"; 
options.Authority = "http://localhost:8080/auth/realms/marcsrealm/"; 
options.RequireHttpsMetadata = false;
options.ResponseType = "code";
options.GetClaimsFromUserInfoEndpoint = true;
```

* The ClientId is the name you registered this client with in the identity provider.
* In KeyCloak you need to change the Access Type to "confidential", and then KeyCloak will provide the ClientSecret.
* The Authority points to the URL of the Identity Provider.  I noticed the client tries to retrieve an URL from the provider that looks like this: http://localhost:8080/auth/realms/marcsrealm/.well-known/openid-configuration so you need to make sure that the setting is pointing to the right place to find it.
* RequireHttpsMetadata had to be set to false in my testing because my KeyCloak in docker isn't running on HTTPS.
* The Response Type identifies the required flow type.   Code means return an Authorisatoin Code, we're using Authorisation Code flow.

### Dotnet Framework:

Haven't tried this but this may be a pointer to the code we'd need to implement a client in .net framework.

* https://stackoverflow.com/questions/50839058/openid-connect-with-asp-net-mvc


### Drupal:

There's a Drupal module:

* https://www.drupal.org/project/openid_connect

It says it creates a corresponding local Drupal user after authentication.











