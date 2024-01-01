# Client credential grant type

You’ll use the client credential grant type when an application needs to access an OAuth2 protected resource, but no one is involved in the transaction. With the client credential grant type, the OAuth2 server only authenticates based on the application name and the secret key provided by the resource owner.

The difference between the password grant type and the client credential grant type is that a client credential grant authenticates by only using the registered application name and the secret key.

For example, let’s say that the O-stock application has a data analytics job that runs once an hour. As part of its work, it makes calls out to O-stock services. However, the O-stock developers still want that application to authenticate and authorize itself before it can access the data in those services. This is where the client credential grant type can be used.

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

1. The resource owner registers the O-stock data analytics application with the OAuth2 service. The resource owner provides the application name and receives back a secret key.
2. When the O-stock data analytics job runs, it presents its application name and secret key provided by the resource owner.
3. The O-stock OAuth2 service authenticates the application using the provided application name and the secret key and then returns an OAuth2 access token.
4. Every time the application calls one of the O-stock services, it presents the OAuth2 access token received with the service call.
