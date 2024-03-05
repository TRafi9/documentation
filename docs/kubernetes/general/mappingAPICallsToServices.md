# How to forward API calls from your application to your kubernetes containers.

In this scenario I have a frontend exposed to the public via an nginx ingress.

Since my TPM backend is now sitting in a cluster, I need to reconfigure the API backend calls (made by the user clicking on stuff in the frontend) so that it points to the backend service that sits within my cluster.

e.g. normally I call my backend from `localhost:8080/api/v1/something, but my backend now sits in the cluster and is exposed via a service.

## How to reconfigure the backend calls

1. Exec into the pod running our container.
   As long as we have a service setup properly we can find the ports and name of our service that we need to replace the api call with.
   `kubectl exec -it <pod name>  -- /bin/sh`
   `kubectl exec -it tpm-frontend-8dfb5484b-z5v22 -- /bin/sh`

2. Find the env vars within the pod.
   type `env` into the terminal after execing into the pod.

3. From here you can see the env variables within the pod, we want to pull both the `SERVICE` and the `SERVICE PORT`.
   In TPM the following variables are:
   `TPM_BACKEND_SERVICE` and `TPM_BACKEND_SERVICE_PORT`

4. Call the environment variables into your code, as they should be available from the environment
   JavaScript:
   `ENV_VARIABLE = process.env.ENV_VARIABLE`

5. Rename the API calls to match the new service:

```diff
- http://localhost:8080/api/v1/createUser
+ http://$TPM_BACKEND_SERVICE:$TPM_BACKEND_SERVICE_PORT/api/v1/createUser

```

Note that calls need to be on server side for this to work, aka the call done by the frontend needs to execute on serverside.
