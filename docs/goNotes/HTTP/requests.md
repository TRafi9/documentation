# Requests

`*http.Request` is a parameter that is a pointer to a struct that holds information about the current request.

When an HTTP server receives a request from a client it creates a `http.Request` object that represents that request. The object is then populated with information extracted from the incoming request, things such as request method, URL, headers & body are extracted.

**Note that the body is represented as an `io.ReadCloser` object.**

After processing the request, the HTTP server can send back a response to the client usually using `http.ResponseWriter`
