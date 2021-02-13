# Authentication & Authorization

## Terminology

*Authentication*: prove that you're who you're claiming to be

*Authorization*: allow or deny access to a particular resource

*OAuth 2.0*: de-facto standard for authorizationOpenId connect: authentication system based on OAuth 2.0: authorization does an implicit authentication, and then in a second step your authorized to request user info. OAuth was created to remove the need for users to share their passwords with third-party applications.

*OpenID*: OpenID was created for federated authentication, that is, letting a third-party authenticate your users for you, by using accounts they already have.

*OpenID Connect*: OpenID on top of OAuth

## OAuth 2.0 Flows

OAuth has multiple 'flows':

### Implicit flow <response_type=token>

1. User clicks 'login with Xxx' button on website
2. Browser is taken to authorization provider Xxx who possibly askes for username/password
3. Auth provider Xxx sends back an `id_token` or `access_token` as query param or hash fragment in the URL
4. Javascript grabs this token from the URL and either:
   * Decodes the JWT `id_token` to get user ID
   * Uses the access_token to fetch more user info. 

### Authorization Code Grant flow <response_type=code>

1. Application registers with Xxx provider and obtains a client ID and secret
2. User clicks 'login with Xxx' button on website
3. Browser is taken to authorization provider Xxx who possibly askes for username/password
4. Auth provider Xxx sends back a code as query param to the redirect URL
5. On server side, the code is used to request a real `access_token` via `POST` call
6. The answer will get the `access_token`
7. On server side, this access_token can then get exchanged for more user info

### Implementing the Authorization Code Grant flow with Google as Xxx provider

0. Register application at https://console.developers.google.com/apis/credentials?authuser=1&folder=&organizationId=&project=hinolugi-localdev
1. User clicks login -> go to server at http://localhost:8080/oauth-login-with-google
2. Server sends following 302 redirect to the browser: 
   ```
   GET https://accounts.google.com/o/oauth2/v2/auth?
       client_id=${GOOGLE_CLIENT_ID}&state=${secretKey}&
       response_type=code&scope=openid%20email%20profile&
      redirect_uri=http%3A%2F%2Flocalhost%3A8080%2Foauth
   ````
3. Once browser has redirected, google will do authentication and authorization and eventually send a response to the redirect URI (so http://localhost:8080/oauth):
That response will contain a `code` parameter
4. Then do a
   ```
   POST https://accounts.google.com/o/oauth2/token?
        code=${requestToken}&client_id=${GOOGLE_CLIENT_ID}&
        client_secret=${GOOGLE_CLIENT_SECRET}&
        grant_type=authorization_code&
        redirect_uri=http%3A//localhost%3A8080/oauth
   ```
   with the just received code to get an actual `access_token`
5. Finally do a
   ```
   GET https://www.googleapis.com/oauth2/v3/userinfo
   ``` 
   with header `Authorization: Bearer {access-token}` to get user info
   
## Questions

### How to not 'leave the current SPA page' and hereby disturbing the client-side state?
{TODO}


## Practical

### Application Registrations:

With Github:
With Google: https://console.developers.google.com/apis/credentials?authuser=1&project=hinolugi-localdev&folder=&organizationId=
With Microsoft: https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationMenuBlade/Overview/appId/3c8c4905-824b-4626-b409-c1d9291d98a5/objectId/59faa595-8c29-4e2f-ab88-3bc4270de493/isMSAApp/true/defaultBlade/Overview/appSignInAudience/AzureADandPersonalMicrosoftAccount 
With Facebook: https://developers.facebook.com/docs/facebook-login/manually-build-a-login-flow/

## Links

- Decode JWT `id_token`s: https://jwt.io/
- Java Auth Server impl: https://github.com/authlete/java-oauth-server
- Good explanantion of authentication of OAuth: https://medium.com/@darutk/full-scratch-implementor-of-oauth-and-openid-connect-talks-about-findings-55015f36d1c3 
- Good explanation of how to impl.: https://aaronparecki.com/oauth-2-simplified
- Good node/express intro: https://medium.com/@evangow/server-authentication-basics-express-sessions-passport-and-curl-359b7456003d
- https://www.smashingmagazine.com/2017/05/oauth2-logging-in-facebook/
- https://oauth.net/articles/authentication/#common-pitfalls
- https://medium.com/better-programming/building-secure-login-flow-with-oauth-2-openid-in-react-apps-ce6e8e29630a 
- https://stackoverflow.com/questions/1087031/whats-the-difference-between-openid-and-oauth
- https://stackoverflow.com/questions/13387698/why-is-there-an-authorization-code-flow-in-oauth2-when-implicit-flow-works-s
- https://stackoverflow.com/questions/6666267/architecture-for-merging-multiple-user-accounts-together
- https://medium.com/@darutk/the-simplest-guide-to-oauth-2-0-8c71bd9a15bb
- https://www.youtube.com/watch?v=jeRALmfyoqg

