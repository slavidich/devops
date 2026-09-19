`kubectl create namespace postgres` 
`kubectl create secret generic postgres-secret --from-literal=POSTGRES_PASSWORD=SuperSecretPassword -n postgres `