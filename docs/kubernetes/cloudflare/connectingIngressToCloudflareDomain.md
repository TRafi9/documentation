# Connecting Ingress To Cloudflare Domain

Once our application is up and running, accessible through a service with a load balancer type, and reachable via a public IP, we can integrate our Cloudflare domain. This means users will be redirected to our application upon visiting the domain.

To do this, we need to perform the following steps:

1. Install an Ingress controller on our Kubernetes cluster.

    ```shell
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
    ```

2. Create and get secrets needed from Cloudflare and apply to your Kubernetes cluster. This allows Kubernetes to connect to your Cloudflare domain. [See this guide](https://documentation.breadnet.co.uk/kubernetes/nginx-ingress/nginx-ingress-with-cloudflare-origin-server-ssl-tls/#configure-nginx-ingress) (TODO: rewrite this page later).

3. Create a YAML manifest for Ingress and point it to your Cloudflare domain and service. Make sure to add the annotations.

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: tpm-frontend-ingress
      annotations:
        nginx.ingress.kubernetes.io/auth-tls-pass-certificate-to-upstream: "true"
        nginx.ingress.kubernetes.io/auth-tls-secret: default/cf-orig
        nginx.ingress.kubernetes.io/auth-tls-verify-client: "on"
        nginx.ingress.kubernetes.io/auth-tls-verify-depth: "1"
    spec:
      ingressClassName: nginx
      tls:
        - hosts:
            - tpm.talhaprojects.com
          secretName: cf
      rules:
        - host: tpm.talhaprojects.com
          http:
            paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: tpm-frontend
                    port:
                      number: 80
    ```

4. Deploy the YAML manifest and inspect it using `kubectl get ingress`. You should see an IP address available.

5. Go to Cloudflare and into your project.

6. Navigate to DNS -> Records and create a new Type A record.

7. Set the IPv4 address to the IP address you got from step 4, set proxied to ON and TTL Auto.

8. Save DNS record and allow it to propagate.
