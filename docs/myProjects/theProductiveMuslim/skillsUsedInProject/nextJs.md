# Next JS

I used the Next JS framework for frontend, specifically the pages routing type, storing my api calls serverside under the `/api` folder that called the backend.

Heres an example of some serverside code in Next JS I wrote that sets the cookie in the browser after making an API POST request to the Go server to let the user log in.

```javascript
import { NextApiRequest, NextApiResponse } from "next";

export default async function PostLoginUser(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === "POST") {
    const { userEmail, userPassword } = req.body;

    try {
      const response = await fetch("http://tpm-backend:80/api/v1/login", {
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
    } catch (error) {
      console.error("Error creating user:", error);
      res.status(500).json({ error: "Internal server error" });
    }
  } else {
    res.status(405).json({ error: "Method Not Allowed" });
  }
}
```
