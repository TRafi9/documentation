# useful commands

### Describe deployments

Gets deployment information

`kubectl describe deployment <deployment name>  -n <namespace>`

e.g.

`kubectl describe deployment tpm-backend  -n tpm`

### Retagging existing docker image

`docker image tag tpm-backend:latest tpm-backend:0.0.1`

### Pushing updated code to kind pod

1. rebuild your image with a new tag e.g.

```diff
-    docker build -t tpm-backend:0.0.2
+    docker build -t tpm-backend:0.0.2
```

2. change deployment file to point image section to new tag

```diff
containers:
        - name: tpm-backend
-          image: tpm-backend:0.0.1
+          image: tpm-backend:0.0.2
```

3. redeploy the deployment file using the following:

`kubectl apply -f <path to deployment.yaml>`

### View pod status

`kubectl get pods`

`kubectl get pods/<pod name>`

### View container logs from the pod its running in

`kubectl logs -f pod/tpm-backend-c5587bf9b-89pds -c cloud-sql-proxy`

-c stands for container
`kubectl logs -f pod/<pod name> -c <container name that is set in deployment.yaml>`

### Get continuous logs of a running container in a pod

`kubectl logs -f <pod name> -c <container name>`
