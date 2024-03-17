# Getting ENV vars from pods to expose to containers

You can find the variables by doing the following.

1. Find the pod you want with `kubectl get pods`
2. Exec into the pod using `kubectl exec -it <pod name> -- /bin/sh`
3. Type in `env` into terminal to bring up env vars

From here you can see the env vars in the pods and can call them in your code on the server side.
e.g. in python you could do:

```python
import os
os.getenv(ENV_VARIABLE_NAME)
```

and in javascript you can do:
`const VAR = process.env.<ENV VAR NAME>`

e.g.

```javascript
const TPM_BACKEND_SERVICE = process.env.TPM_BACKEND_SERVICE;
const TPM_BACKEND_SERVICE_PORT = process.env.TPM_BACKEND_SERVICE_PORT;
```
