# What is Octet-stream and where does it come from?

When sending a response in a HTTP request, Go will automatically set the following three headers
`Date`, `Content-Length` & `Content-Type`.

The `Content-Type` header is set by Go sniffing the data in the response body using its underlying `http.DetectContentType()` function.
This means that it will automatically set the content type if it is able to detect it. However, if it fails to detect it, it will
set it to `Content-Type: application/octet-stream`.

**NOTE**
Go sometimes has trouble distinguishing between JSON and plaintext so you can get around this by setting the Content type yourself as follows:

```go
w.Header().Set("Content-Type", "application/json")
w.Write([]byte(`{"name":"Alex"}`))
```
