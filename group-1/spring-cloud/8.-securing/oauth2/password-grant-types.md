# Password grant types

You’ll use this grant type when both the application and the services explicitly trust one another.

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

1. The owner of the application registers with the OAuth2 application service and provides a unique name for their application. The OAuth2 service then provides a secret key back to register the application. The name of the application and the secret key provided by the OAuth2 service uniquely identify the application trying to access any protected resources.
2. The user logs into O-stock and provides their login credentials to the application. O-stock passes the user credentials and the application name/application secret key directly to the OAuth2 service.
3. The O-stock OAuth2 service authenticates the application and the user and then provides an OAuth2 access token back to the user.
4. Every time the O-stock application calls a service on behalf of the user, it passes along the access token provided by the OAuth2 server.
5. When a protected service is called (in this case, one of the licensing or organization services), the service calls back to the O-stock OAuth2 service to validate the token.&#x20;
   * If the token is good, the invoked service allows the user to proceed.&#x20;
   * If the token is invalid, the OAuth2 service returns an HTTP status code 403, indicating that the token is invalid.
