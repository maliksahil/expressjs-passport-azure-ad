This example demonstrates how to use [Express](http://expressjs.com/) 4.x and
[Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad/) to create a protected Web API using the v2 endpoints.

# Instructions

## Register in Azure AD

1. Go to portal.azure.com and go to active directory, in there go to app registrations
2. Create a new app registration, ensure that the redirect URI includes `http://localhost:3000`
3. Under the **Expose an API** section, add an application ID URI, (example: https://mytestapp.sahilplayground.onmicrosoft.com)
4. That's it! Copy the application ID (clientID), AppID Uri, and Tenant ID, you'll need them for the next step.

## How to run this example

1. Git clone this repo, and run npm install.
2. Update server.js options object (see line 6) as an example,

```
var options =  {
  identityMetadata: "https://login.microsoftonline.com/<tenantidguid>/v2.0/.well-known/openid-configuration",
  clientID: "<clientidguid>",
  issuer: "https://sts.windows.net/<tenantidguid>/",
  audience: "<youraudienceurifortheapi>",
  loggingLevel: "info",
  passReqToCallback: false
};
```
3. Run `node server.js` to start your WebAPI on localhost:3000

## How to call this API

So to call this API, we will use Auth Code flow v2.
Here is how, 

1. Perform login and request a code by visiting the following URL, line breaks added for clarity

```
https://login.microsoftonline.com/<tenant>.onmicrosoft.com/oauth2/v2.0/authorize?
response_type=code&
client_id=<clientid>&
redirect_uri=http%3A%2F%2Flocalhost%3A3000%2Fcallback&
scope=openid
```

Note that the client ID is of an app that has access to call the above API, or the client ID of the API itself (since API can call itself)

2. This will redirect your browser to a URL that looks like this, copy paste the "Code" part,

```
http://localhost:3000/callback?
code=..longuglycode..&
session_state=<someguid>
```
3. Create a POST request as follows with this code to request an access token,
```
curl -X POST \
  https://login.microsoftonline.com/sahilplayground.onmicrosoft.com/oauth2/v2.0/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'cache-control: no-cache' \
  -d 'redirect_uri=http%3A%2F%2Flocalhost%3A3000%2Fcallback&client_id=<clientid>&grant_type=authorization_code&code=<codefromabove>&client_secret=<clientsecretgeneratedintheapi>&scope=appiduri/.default'
  ```
4. This should return you an access token and an id_token

5. Now call the API like this,
```
curl -X GET \
  http://localhost:3000/api \
  -H 'Authorization: Bearer accesstokenfromabove' \
  -H 'cache-control: no-cache'
  ```

This should return a JSON object with the name of the current user