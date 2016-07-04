# Calling an external IdP (Facebook, Github, etc.) API.

When we authenticate using an external IdP (Facebook, Github, etc.), the IdP usually provides an access token with the apropiate scope() in order to consume their API. To avoid leaking this access token to the browser we decided to removed it from the user profile. If you need to get the user's IDP access token, you will need to call `/api/v2/users/{user-id}` endpoint with `read:user_idp_tokens` scope.

The goal of this document is to show our recommended way to do it from a SPA/Native application. The basic flow the following:

1. Create a non interactive client with `read:user_idp_tokens` scope
2. Create an application that will proxy the requests from the SPA/Native app to the IdP external API. This app will:  
    1. Validate your id_token using the SPA/Native app client secret
    2. Extract the `sub` from the id_token which contains the user_id
    3. Execute the client credentials exchange to get a valid APIv2 access token (using the non interactive client)
    4. Call the api/v2/users/{id} API and send the access_token in the header
    5. Call the IdP external API using the IdP access token (`user.identities[0].access_token`)  
3. From your SPA/native app call your API and send your id_token


## Create a non interactive client to interact with Auth0 Management API

To create a non interactive client you can check the document [How to create a non interactive client](/)

> Make sure that this client can be granted `read:user_idp_tokens` scope.

## Create an application to proxy request from the application to the IdP API

The application will use the id_token to verify the request and if it is a valid token, then it will request an access token to call Auth0 Management API (client credentials flow) and will use the `sub` claim from the id_token (which contains the user id) to call `/api/v2/users/{user-id}` and get the user's IdP access token. Then, with the access token you can call the IdP External API.

> We will show you who you can accomplish this by using Auth0 Webtasks.

Create a new file called `proxy.js` that will be your webtask. It will be a simple expres app that exposes and endpoint.

  ```js
  "use latest";
  ​
  const jwt     = require('jsonwebtoken');  
  const request = require('request');
  const express = require('express');
  const Webtask = require('webtask-tools');
  const app     = express();
  ​
  app.get('/', function (req, res) {
    // TODO: Logic here
  });
  ​
  module.exports = Webtask.fromExpress(app);

  ```

The endpoint will verify the token sent in the authorization header before doing the client credentials flow.

  ```js
  app.get('/', function (req, res) {
    if (!req.headers['authorization']){ return res.status(401).json({ error: 'unauthorized'}); }

    const context = req.webtaskContext;
    const token = req.headers['authorization'].split(' ')[1];

    return jwt.verify(token, new Buffer(context.data.ID_TOKEN_CLIENT_SECRET, 'base64'), function(err, decoded) {
      // TOOD: execute exchange and get the user profile
      ...
    });  
  });
  ```

  > Note: `context.data.ID_TOKEN_CLIENT_SECRET` contains the client secret for the native/SPA app.

Create a helper function called `getAccessToken` to execute the client credentials flow and get an access token to consume Auth0 Management API

  ```js
  function getAccessToken(context, cb){
    const options = {
      url: 'https://' + context.data.ACCOUNT_NAME + '.auth0.com/oauth/token',
      json: {
        audience: 'https://' + context.data.ACCOUNT_NAME + '.auth0.com/api/v2/',
        grant_type: 'client_credentials',
        client_id: context.data.CLIENT_ID,
        client_secret: context.data.CLIENT_SECRET
      }
    };

    return request.post(options, function(err, response, body){
      if (err) return cb(err);
      return cb(null, body.access_token);
    });
  }
  ```

  > Note: `context.data.CLIENT_ID` and `context.data.CLIENT_SECRET` are the keys from the non interactive client.

Create a helper function called `getUserProfile` to call the `/api/v2/users/{user-id}` endpoint from Auth0 Management API and get the user profile with the IdP access token.

  ```js
  function getUserProfile(context, userId, token, cb){
    const options = {
      url: 'https://' + context.data.ACCOUNT_NAME + '.auth0.com/api/v2/users/' + userId,
      json: true,
      headers: {
        authorization: 'Bearer ' + token
      }
    };

    return request.get(options, function(error, response, user){
      return cb(error, user);
    });
  }
  ```

Finally let's put all this together in the endpoint logic. After verifying the id_token, call the `getAccessToken` to execute client credentials flow and then call the `getUserProfile` method with `decoded.sub` as user id to get the user profile with the access token.

  ```js
  app.get('/', function (req, res) {
    ...

    return jwt.verify(token, new Buffer(context.data.ID_TOKEN_CLIENT_SECRET, 'base64'), function(err, decoded) {
      if (err) return res.status(500).json(err);

      return getAccessToken(context, function(err, access_token){
        if (err) return res.status(500).json(err);

        return getUserProfile(context, decoded.sub, access_token, function(err, user){
          if (err) return res.status(500).json(err);

          const access_token = user.identities[0].access_token;

          // TODO : call IdP external API

          return res.status(200);
        });
      });
    });  
  });
  ```

  > Most of the times, there's going to be just one identity in the identities array, but if you've used [account linking feature](/link-accounts) there might be more than one.

The `accessToken` we get here will have access to call all the APIs you've specified in Auth0 dashboard.

To deploy this webtask you will can use wt-cli specifying your own secrets

```bash
wt create proxy.js --secret ACCOUNT_NAME=[your-account] --secret ID_TOKEN_CLIENT_SECRET=[app-secret] --secret CLIENT_ID=[non-interactive-client-id] --secret CLIENT_SECRET=[non-interactive-client-secret]
```

## Call your API with your id_token

Now when you authenticate in your SPA/Native app you just need to use the token to call the proxy backend.

  ```js
  lock.show(function(err, token, profile) {
    // Function to call Google's API with the accessToken
    getGoogleContactsWithToken(token);
  })
  ```
