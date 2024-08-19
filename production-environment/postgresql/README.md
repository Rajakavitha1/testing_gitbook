# PostgreSQL

You can install PostgreSQL through managed cloud services or by deploying it directly within a Kubernetes cluster. You can also opt to use an existing PostgreSQL instance outside of the Kubernetes cluster that runs Seldon Enterprise Platform. If you have a database already set up on-premises or in the cloud, you can integrate it with Seldon Enterprise Platform by adding the connection details to the metadata-postgres secret in the namespace where Seldon Enterprise Platform is running. Substitute the values with those from your database:

```
kubectl create secret generic -n seldon-system metadata-postgres \
  --from-literal=user=your_user \
  --from-literal=password=your_password \
  --from-literal=host=your.postgres.host \
  --from-literal=port=5432 \
  --from-literal=dbname=metadata \
  --from-literal=sslmode=require \
  --dry-run=client -o yaml \
  | kubectl apply -n seldon-system -f -
```  
