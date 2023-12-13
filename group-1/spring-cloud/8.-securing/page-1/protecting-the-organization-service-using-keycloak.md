# Protecting the organization service using Keycloak

Once we’ve registered a client in our Keycloak server and set up individual useraccounts with roles, we can begin exploring how to protect a resource using SpringSecurity and the Keycloak Spring Boot Adapter. While the creation and managementof access tokens is the Keycloak server’s responsibility, in Spring, the definition ofwhich user roles have permissions to do what actions occurs at the individual servicelevel. To set up a protected resource, we need to take the following actions:
