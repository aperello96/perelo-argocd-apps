Before deploy de components:
1. Create ns "kubectl create ns mariadb-pro"
2. Create a secret with your root credentials:
```
kubectl create secret generic mariadb-secret \
  --namespace=mariadb-pro \
  --from-literal=MYSQL_ROOT_PASSWORD=<YOUR_SUPER_SECRET_PASSWORD>
```