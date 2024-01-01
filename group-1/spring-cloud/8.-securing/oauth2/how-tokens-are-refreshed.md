# How tokens are refreshed

A client can present the refresh token to the OAuth2 authentication service, and the service will validate the refresh token and then issue a new OAuth2 access token.

<figure><img src="../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

1. The user logs in to O-stock but is already authenticated with the OAuth2 service. The user is happily working, but unfortunately, their token expires.
2. The next time the user tries to call a service (say, the organization service), the O-stock application passes the expired token to the organization service.
3. The organization service tries to validate the token with the OAuth2 service, which returns an HTTP status code 401 (unauthorized) and a JSON payload, indicating that the token is no longer valid. The organization service returns an HTTP 401 status code to the calling service.
4. The O-stock application gets the 401 HTTP status code and the JSON payload back from the organization service. The O-stock application then calls the OAuth2 authentication service with a refresh token. The OAuth2 authentication service validates the refresh token and then sends back a new access token.
