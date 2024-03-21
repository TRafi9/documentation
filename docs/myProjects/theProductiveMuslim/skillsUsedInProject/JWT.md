# JWTs

When creating the application, I had to figure out a way to make sure only users of the application were able to access certain TPM's API resources.

Enter learning about JWT tokens, I created the following system.

1. User logs in
2. Server checks for valid credentials
3. If credentials are valid, server sets a JWT token in the browser's cookies.
4. Frontend then sends requests with JWT token bound in Header, which is then parsed by server to see if access is allowed to specific API routes.

Here's some of TPM's code where it creates a JWT token and sets a 24 hour expiry time along with the userType.

```go
claims := &Claims{
				UserType: "user",
				User:     loginCredentials.UserEmail,
				RegisteredClaims: jwt.RegisteredClaims{
					Issuer:    "tpm",
					ExpiresAt: jwt.NewNumericDate((time.Now().Add(24 * time.Hour))),
				},
			}

			token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

			// Sign and get the complete encoded token as a string using the secret

			tokenString, err := token.SignedString(hmacSecret)
			if err != nil {
				logger.Errorf("token.SignedString failed, err: ", err.Error())
			}
			cookie := &http.Cookie{
				Name:     "jwt",
				Value:    tokenString,
				Path:     "/",
				HttpOnly: true,
				SameSite: http.SameSiteStrictMode,
				Secure:   true,
				Expires: time.Now().Add(24 * time.Hour),
			}

			c.SetCookie(cookie)

```

4. Once the JWT token is set in the browser, I used it to protect specific backend API endpoints. I did this by adding middleware to specific backend API routes, which checks for a JWT token in any incoming API request, and checks to see if it is valid (signed by same hmac secret key, not expired, not tampered with etc). Here is the code snippet for that, it expects the token from the frontend to come through the header part of the request.

```go
apiRestricted := e.Group("/api/v1/restricted")

// middleware to stop invalid or unauthorized jwt's from accessing these functions
apiRestricted.Use(echojwt.WithConfig(echojwt.Config{
	SigningKey:  hmacSecret,
	TokenLookup: "header:Authorization",
}))
                
apiRestricted.GET("/getPrayerTimes/:dateValue", func(c echo.Context) error {
	return todayPrayerHandler(c, Pt, logger, hmacSecret)
})

```

## Why I did this?

1. Reduce likelyhood of external attacks on TPM api's e.g. DDOS
2. Ability to restrict access further by parsing claims inside handler function, and checking if they are authorized to be making this call.
