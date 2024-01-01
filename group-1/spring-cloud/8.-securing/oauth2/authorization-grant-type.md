# Authorization grant type

The authorization code grant allows different applications from different vendors to share data and services without having to expose a user’s credentials across multiple applications.

Let’s say you have an O-stock user who also uses Salesforce.com. The O-stock customer’s IT department has built a Salesforce application that needs data from an O-stock service (the organization service).&#x20;

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

1. The user logs in to the O-stock application and generates an application name and application secret key for the Salesforce application. As part of the registration process, they also provide a callback URL for the Salesforce application. This Salesforce callback URL is called after the OAuth2 server authenticates the O-stock credentials.
2.  The user configures their Salesforce application with the following information:

    * The application name they created for Salesforce
    * The secret key they generated for Salesforce
    * A URL that points to the O-stock OAuth2 login page

    Now when the user tries to use the Salesforce application and access O-stock data via the organization service, they are redirected to the O-stock login page. The user provides their O-stock credentials.&#x20;

    * If they are valid credentials, the O-stock OAuth2 server generates an authorization code and redirects the user to Salesforce via the URL provided in step 1.
    * The OAuth2 server sends the authorization code as a query parameter on the callback URL.
3. The custom Salesforce application persists the authorization code. Note that this authorization code isn’t an OAuth2 access token.
4. Once the authorization code is stored, the Salesforce application presents the secret key generated during the registration process and the authorization code back to the O-stock OAuth2 server. The O-stock OAuth2 server validates the authorization code and then returns an OAuth2 token to the custom Salesforce application
5. The Salesforce application calls the O-stock organization service, passing an OAuth2 token in the header.
6. The organization service validates the OAuth2 access token passed into the O-stock service call. If the token is valid, the organization service processes the user’s request.
