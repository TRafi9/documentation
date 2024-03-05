## Adding application default credentials to kubernetes

You will need to add your service account / application default credentials to kubernetes if you want to connect and run things such as cloud sql proxy.

1. Download your `application_default_credentials.json` file from GCP.

2. create a secret that holds the contents of your credentials file.
   `kubectl create secret -n <namespace name> generic <secret name> --fron-file=<key name>=<path to application_default_credentials.json>`
   `kubectl create secret generic cloudsql-credentials --from-file=service_account.json=application_default_credentials.json`

3. In the deployment yaml, add the secret as a volume to the pod running the containers. The secret needs to be added at pod level first, and then can be mounted to a container.

4. Mount the volume to the container within the deployment yaml file, do this using `volumeMounts` and define a path where you want it mounted within the container using `mountPath` (this path will be created if it doesn't already exist). Define the secret you are mounting using `name` which should be the same name as the secret that you set under `volumes`.

5. For cloudsql you can pass the instance name and mounted credentials file under `args`. When secrets are mounted to containers, the key:value pairs are mounted, so you can directly pass them as the following:
   `--credentials-file=/<volume mount path>/<some key in secret>`

#### Example

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tpm-backend
  labels:
    app: tpm-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tpm-backend
  template:
    metadata:
      labels:
        app: tpm-backend
    spec:
      volumes:
        - name: cloudsql-credentials
          secret:
            secretName: cloudsql-credentials
      containers:
        - name: tpm-backend
          image: tpm-backend:0.0.2
          imagePullPolicy: Never
          env:
            - name: USER
              valueFrom:
                secretKeyRef:
                  name: tpmsecrets
                  key: USER
        ....

          ports:
            - containerPort: 8080
              protocol: TCP
        - name: cloud-sql-proxy
          image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.8.2
          volumeMounts:
            - mountPath: /secrets/
              name: cloudsql-credentials
              readOnly: true
          env:
            - name: USER
              valueFrom:
                secretKeyRef:
                  name: tpmsecrets
                  key: USER
          args:
            - starlit-booster-408007:europe-west2:the-productive-muslim
            - --credentials-file=/secrets/service_account.json
```
