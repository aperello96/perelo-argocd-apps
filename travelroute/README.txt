kubectl create secret generic mongo-secret \
  --namespace=travelroute \
  --from-literal=MONGO_INITDB_ROOT_USERNAME=<YOUR_USERNAME> \
  --from-literal=MONGO_INITDB_ROOT_PASSWORD=<YOUR_PASSWORD>