# section on exposing frontend to public

check frontend-service yaml, no ingress needed for external IP as Loadbalancer provisioning will do this
through the following command:
apply frontend service
then use external Ip and port exposed by service to see frontend: 159.65.212.242:3000

# configuring cloudflare domain to link up to kubernetes

- have the loadbalancer public IP address and expose a port for main traffic, e.g. 443 for standard https traffic
- allow traffic from loadbalancer IP address through creating a dns record in cloudflare
- create and get origin server ssl/tls certs from cloudflare (will need these to marry up domain access to http)
- save cert and private key file locally e.g. tls.crt & tls.key
- pass those values into a kubernetes secret of type tls, which expects a --cert and --key value:
  `kubectl create secret tls cloudflare-tls-secret --cert=/path/to/file/tls.crt --key=/path/to/file/tls.key`
- add the secret to the frontend ingress yaml manifest & make sure to add it under tls heading too.
- map the ports using an ingress file to point to your port exposed by loadbalancer e.g. 443

# finish off things learned


- storing secret credentials outside of code and calling through env vars
- setting up user login experience which includes parsing form data coming from frontend, uploading to DB, verifying emails and authenticating user during login
- sometimes setting a var to type any in tsx is useful, e.g. allowing users to sending strings instead of ints for verification, this allows the backend to first check
  if user string can be converted to an int before wasting more resources trying to query the db with a string, this can be further improved by checking length of int etc
- blocking user login if user has not verified email
- how to send error responses from the backend and display those responses on the frontend for user

- kubernetes - done
- JWT tokens - done
- login system - done
- crud calls in Go - done
- password encryption - done

- CURRENTLY - learning implementing JWT Auth and storing JWT securely in cookies - passing back and forth through authorization header
  what this project demonstrates

- understanding of writing sql statements for postgres

- FIND OUT - why api calls needed to be moved onto server side rather than client side in nextjs to access it by kubernetes? also move all calls to serverside
