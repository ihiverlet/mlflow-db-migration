#  MLFLOW database Migration

1. You have started Mlflow with a postgres database you want to upgrade
2. Open a terminal 
3. Check that your database is running correctly - its pod name is usually `mlflow-db-0` and we'll use this name is the next steps.
```bash
kubectl get pods | grep mlflow
```
4. Retrieve the database name, username and password for connection.  
One way to do this is by checking the description of the Mlflow pod and retrieving the backend store URI. It looks like this : `postgresql://[username]:[password]@[host]:[port]/[database]`  
```bash
kubectl describe pod mlflow-xxxx-xxx-xx | grep -i backend-store-uri   
```
You should get the following result :  `--backend-store-uri=postgresql://username:mypassword@mlflow-db:5432/mlflow`  
5. Let's save your data  
```bash
kubectl exec -it mlflow-db-0 -- pg_dump mlflow -U [username] -O -F c -v -f tmp/dump.backup
``` 
6. Retrieve the dump
```bash
kubectl cp [namespace]/mlflow-db-0:tmp/dump.backup dump.backup
``` 
7. ⚠️ Check that you have the file with expected content in your workspace - be careful as the next step is irreversible
8. Uninstall mlflow and delete the volume associated with your database 
```bash
helm ls  
helm uninstall mlflow-xxxxx  
kubectl get pvc    
kubectl delete pvc data-mlflow-db-0  
```  
9. Start a new mlflow instance with the upgraded version of postgres
10. Restore the data - refer to step 4 if you do not know your connection credentials
``` bash
kubectl cp dump.backup [namespace]/mlflow-db-0:/tmp/dump.backup  
kubectl exec -it mlflow-db-0 -- pg_restore -h mlflow-db -p 5432 -d mlflow -U [username] -v tmp/dump.backup
```
Do not forget to clean 🧹
```bash
kubectl exec -it mlflow-db-0 -- rm /tmp/dump.backup
```
11. Enjoy 🙂
