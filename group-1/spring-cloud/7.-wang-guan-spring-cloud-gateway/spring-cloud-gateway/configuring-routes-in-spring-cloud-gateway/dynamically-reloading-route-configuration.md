# ❌ Dynamically reloading route configuration

The ability to dynamically reload routes is useful because it allows us to change the mapping of routes without having to restart the Gateway server(s). Existing routes can be modified quickly, and new routes will have to go through the act of recycling each Gateway server in our environment.

Spring Actuator exposes a POST-based endpoint route, actuator/gateway/refresh, that will cause it to reload its route configuration.
