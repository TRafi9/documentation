# Kubernetes

Kubernetes is an open source system used to deploy, scale and manage containerised applications anywhere.

It is great for the following reasons:

1. It abstracts away infrastruture, as it handles compute/network/storage on behalf of your workloads.
2. It has built in service health monitoring, restarting containers if they are unhealthy or fail, making sure you have high availability.
3. It has built in commands to do a lot of heavy lifting that goes into application management, using `kubectl`.

# Components of Kubernetes

![Cluster diagram](images/cluster_diagram.png)

## Cluster

This is what all your kubernetes parts run in.

## Namespaces

Namespaces conceptually compartmentalise your resources into virtual clusters, that way you can allocate a set amount of resources to each namespace. For example, if you create a namespace for each department in your company, you can limit the amount of CPU/RAM that namespace has to distribute amongst the components inside it.

Once you have a namespace, you can go ahead and add resources to it, such as variables using configMaps and secrets, or RAM/CPU set up from the resource part of the deployment manifest (the deployment.yaml).

### Creating a namespace

`kubectl create namespace <namespace name>`

### Get all namespaces

`kubectl get ns`

## ConfigMaps

ConfigMaps are objects used to store non-confidential variables as key : value pairs, pods can consume ConfigMaps as env variables/ comand line args or as config files in a volume. The advantage of ConfigMaps is that it allows you to decouple environment specific configurations from your application, making them easily portable and adjustable.

### Creating ConfigMaps from kubectl

`kubectl -n <namespace name> create configmap <configmap name> --from-literal ENV_VAR_NAME=value`

### list out configmaps:

`kubectl -n <namespace name> get cm`

### output configmaps as yaml

`kubectl -n <namespace name> get cm <configmap name> -o yaml

## Secrets

Secrets are similar to configMaps except they are encrypted at rest. It is important to keep secrets separate from configMaps as you can apply rbac to give access to secrets to some people but not others.

### Creating a secret from kubectl

```
kubectl -n <namespace name> create secret generic <secret name> \
--from-literal SECRET_KEY=secretValue \
--from-literal SECRET_KEY_2=secret_value_2
```

### Setting secrets in deployment

Once you have created the secret, you can then pass the secret to your deployment yaml manifest like so:

```
    spec:
      containers:
        - name: tpm-backend
          image: tpm-backend:0.0.2
          imagePullPolicy: Never
          ports:
          - containerPort: 80
          env:
          - name: SECRET_NAME_IN_CONTAINER
            valueFrom:
                secretKeyRef:
                    name: SECRET_NAME
                    key: SECRET_KEY
```
