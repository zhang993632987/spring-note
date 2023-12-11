# Built-in Predicate Factories

Built-in predicates are objects that allow us to check if the requests fulfill a set of conditions before executing or processing the requests. For each route, we can set multiple Predicate Factories, which are used and combined via the logical AND. Table 8.1 lists all the built-in Predicate Factories in Spring Cloud Gateway.

| Predicate  | Description                                                                                                                                                                          | Example                                             |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| Before     | Takes a date-time parameter and matches all the requests that happen before it.                                                                                                      | Before=2020-03-11T...                               |
| After      | Takes a date-time parameter and matches all the requests that happen after it.                                                                                                       | After=2020-03-11T...                                |
| Between    | Takes two date-time parameters and matches all the requests between them. The first date-time is inclusive, and the second one is exclusive.                                         | <p>Between=2020-03-11T...,</p><p>2020-04-11T...</p> |
| Header     | Receives two parameters, the name of the header, and a regular expression and then matches its value with the provided regular expression.                                           | Header=X-Request-Id, \d+                            |
| Host       | Receives an Ant-style pattern separated with the “.” hostname pattern as a parameter. Then it matches the `Host` header with the given pattern.                                      | Host=\*\*.example.com                               |
| Method     | Receives the HTTP method to match.                                                                                                                                                   | Method=GET                                          |
| Path       | Receives a Spring PathMatcher.                                                                                                                                                       | Path=/organization/{id}                             |
| Query      | Receives two parameters, a required parameter and an optional regular expression, then matches these with the query parameters.                                                      | Query=id, 1                                         |
| Cookie     | Takes two parameters, a name for the cookie and a regular expression, and finds the cookies in the HTTP request header, then matches its value with the provided regular expression. | Cookie=SessionID, abc                               |
| RemoteAddr | Receives a list of IP addresses and matches these with the remote address of a request.                                                                                              | RemoteAddr=192.168.3.5/24                           |

These predicates can be applied in the code programmatically or via configurations.

