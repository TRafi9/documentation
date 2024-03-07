While developing my application, I needed a way to make sure that users could only access specific backend api endpoints, depending on if the user was logged in or not.
I stumbled upon cookies, which was great because I needed something where I could store the JWT token.

Cookies are a great option for storing JWT tokens for the following reasons:

1. They automatically get sent with every request from your client to the backend, so you don't have to manually send them with every request.
2. Cookies are set by the server, so this allowed me a way to set the jwt token value from the backend, rather than exposing this code to the client side, making it easier to tamper with.
3. JWT's are designed to be stateless, storing a JWT in a cookie allows the server to remain stateless.
4. You can set expiry times on cookies, which you can choose how your server will respond to, most of the time the server will continue with the request and ignore the cookie if it is expired.

## Setting a JWT cookie in Golang

Here is an example of how I set a cookie in golang.

```
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
                SameSite: http.SameSiteNoneMode,
                Secure:   true, // set true if using HTTPS

                Expires: time.Now().Add(24 * time.Hour),
            }

c.SetCookie(cookie)
```

The code above sends a cookie from the golang server, using `SetCookie` this sets the jwt cookie in the `set-cookie` response headers when the API call has been made:

```
const jwtCookie = response.headers.get("set-cookie");
```

## Receiving Cookie and setting it in browser

One thing that threw me off was I was trying to make the api call on the client side, calling the backend directly on the clients webpage. This was not good as kubernetes required the calls to my backend to be server side. When I switched it to serverside (by creating an API call under the /api/ directory), I struggled to get the jwt to be set in my browser. This is because I was not doing the following things.

### Calling the cookie from the set-cookie response header.

The response header was sent by the golang server with `c.SetCookie(cookieName)`. The code below allowed me to get the jwt cookie, which was held in set-cookie.

```
const jwtCookie = response.headers.get("set-cookie");
```

### Propogating Cookie value to client side from server side

Sending the cookie value in the header response of my NextJs serverside API call. The code below

```
res.setHeader("Set-Cookie", jwtCookie);
```

When the client receives the response from the NextJs API route, it automatically recognises the Set-Cookie header and stores the cookie sent.

### Adding credentials:"include"

Adding `credentials:"include"` flag in the API request. You need to set `credentials: "include"` in the fetch options when making cross-origin requests that require sending and receiving cookies.

## Receiving and setting cookie from serverside in NextJs Example

```

      const response = await fetch("http://localhost:8080/api/v1/login", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          userEmail,
          userPassword,
        }),
        credentials: "include",
      });

      if (response.status == 200) {
        const dataRes = await response.json();
        const jwtCookie = response.headers.get("set-cookie");
        // Set the "jwt" cookie in the API route's response headers
        if (jwtCookie) {
          res.setHeader("Set-Cookie", jwtCookie);
        }

        res.status(200).json(dataRes);
      } else {
        // If response is not successful, parse JSON response to get error message and status code
        const errorResponse = await response.json();
        res.status(500).json(errorResponse);
      }

```
