# Golang

The main reason for this project was to learn more about Go and familiarise how to create API resources so I can implement them for microservices architecture.

## Echo (APIs)

The backend uses the Echo framework to create API resources that execute handler functions when called from the frontend.

Some API resources are protected by putting them in restricted groups, that require a valid JWT token:

```go
apiRestricted := e.Group("/api/v1/restricted")

apiRestricted.Use(echojwt.WithConfig(echojwt.Config{
		SigningKey:  hmacSecret,
		TokenLookup: "header:Authorization",
	}))
```

APIs in the backend are responsible for things such as:

- Updating prayer times to the frontend daily
- Calling databases to register new users and handle logins
- Setting cookies on the browser

## Middleware

The backend uses the following middleware

1. Recover - recovers the server if it panics and terminates.
2. Secure - provides security against cross-site scripting (XSS) attacks
3. CORS - allows cross origin requests from only specific URLs
4. JWT middleware - checks if JWT tokens are legit

## Handler functions

Theres a lot of different types of logic within the handler functions, here are some highlights, but they have their own pages under the **Skills Used In Project** section:

1. Encrypting and decrypting users password using bcrypt.
2. Creating JWT tokens
3. Setting Cookies in the browser
4. Getting/Posting data to Redis database/SQL postgres DB
5. Calling 3rd party APIs
6. Automatically sending emails using the smtp Go package.
7. Type safety through parsing data into structs
8. Error handling and sending correct API status codes/ data in body of API requests to frontend
