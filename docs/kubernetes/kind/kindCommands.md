# Kind Commands

Kind is a tool used to run Kubernetes clusters inside Docker containers for local development and testing purposes.

### Get cluster names in kind

`kind get clusters`

### Load docker images into kind

`kind load docker-image <Image name> -n <cluster name>`

### Loading docker images on kind clusters

1. Retag the image to something that isn't the tag `latest` as this causes issues in picking up the tag in kind. You need to make sure the deployment yaml points to this image in the image tag section too.

`docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`

2. Add the docker image into the cluster using the `kind` cli tool.
   `kind load docker-image SOURCE_IMAGE[:TAG] -n <CLUSTER NAME>`

3. Make sure to update the deployment yaml to point to new image

Example

1.

`docker tag tpm-backend:latest tpm-backend:0.0.1`

2.  `kind load docker-image tpm-frontend:0.0.1 -n kind`

3.

```diff
containers:
        - name: tpm-backend
-            image: tpm-backend:latest
+            image: tpm-backend:0.0.2
          imagePullPolicy: Never
```

