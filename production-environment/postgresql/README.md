# PostgreSQL

You can install PostgreSQL through managed cloud services or by deploying it directly within a Kubernetes cluster. You can also opt to use an existing PostgreSQL instance outside of the Kubernetes cluster that runs Seldon Enterprise Platform. To configure a PostgreSQL database with Seldon Enterprise Platform, execute the following after substituting the variables `<your_user>`, `<your_password>`,`<metadata>`, and `<your.postgres.host>` with those from your database:

```
kubectl create secret generic -n seldon-system metadata-postgres \
  --from-literal=user=<your_user> \
  --from-literal=password=<your_password> \
  --from-literal=host=<your.postgres.host> \
  --from-literal=port=5432 \
  --from-literal=dbname=<metadata >\
  --from-literal=sslmode=require \
  --dry-run=client -o yaml \
  | kubectl apply -n seldon-system -f -
```  