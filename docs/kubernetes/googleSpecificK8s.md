# section on how to get google stuff running by adding cloudsql credentials as secret

kubectl create secret generic cloudsql-credentials --from-file=service_account.json=application_default_credentials.json

- mount it to pod
- then mount it to container in pod
- point secret reference to that volume at container level
