# Exposing apps on kind through ingress

See: https://kind.sigs.k8s.io/docs/user/ingress/

## Why it needs a special setup

Normally you can access your application through the external IP address provided by your ingress controller, which will forward data to the service attached.

Because Kind runs locally, it does not have access to the external IP, as that is normally provisioned by the cloud provider, therefore you have to setup a specific nginx controller for your kind cluster, and also make sure extraPortMappings and node-labels are available on your cluster.

In the example below, we create a cluster with `extraPortMappings` the `extraPortMappings` are used to allow the localhost to make requests to the ingress controller over ports 80 and 443.
You can also use `node-labels` to allow the ingress controller to run on specific node(s) matching the label selector.

## Deploying the cluster

```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

## Deploying the nginx ingress controller

We then need to apply the specific NGINX manifest for the kind cluster using:
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml`

The manifests contains kind specific patches to forward the hostPorts to the ingress controller, set taint tolerations and schedule it to the custom labelled node.

## Example ingress deployment

Once we have this setup, we can provision our frontend application to be exposed through localhost like so:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tpm-frontend-ingress
spec:
  rules:
    - host: localhost
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: tpm-frontend
                port:
                  number: 3000
```
