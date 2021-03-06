### Ref
```
https://www.kasten.io/kubernetes-lab
https://www.youtube.com/watch?v=JdI_zJA7vrY

```

1. Add the K10 Helm repository
```
We are setting up VMs, Kubernetes 1.18, and storage infrastructure for you.

Please be patient as this might take a couple of minutes.

If you run into any issues with setup, please contact support@kasten.io!
helm install ...

```

2. Install MySQL
```
kubectl create namespace mysql
helm install mysql bitnami/mysql --namespace=mysql

watch -n 2 "kubectl -n mysql get pods"

MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') \
  -- mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "CREATE DATABASE k10demo"
```

3. Install K10 and config storage
```
Install K10 and Configure Storage
Install Kasten K10
In this step, we will actually install K10 by running the following commands:

kubectl create namespace kasten-io; \
  helm install k10 kasten/k10 --namespace=kasten-io
To ensure that Kasten K10 is running, check the pod status to make sure they are all in the Running state:

watch -n 2 "kubectl -n kasten-io get pods"
Once all pods have a Running status, hit CTRL + C to exit watch.

Configure the Local Storage System
Once K10 is running, use the following commands to configure the local storage system.

kubectl annotate volumesnapshotclass csi-hostpath-snapclass k10.kasten.io/is-snapshot-class=true
```

4. View k10 Dashborad
```
View K10 Dashboard
Expose the K10 dashboard
While not recommended for production environments, let's set up access to the K10 dashboard by creating a NodePort. Let's first create the configuration file for this:

cat > k10-nodeport-svc.yaml << EOF
apiVersion: v1
kind: Service
metadata:
  name: gateway-nodeport
  namespace: kasten-io
spec:
  selector:
    service: gateway
  ports:
  - name: http
    port: 8000
    nodePort: 32000
  type: NodePort
EOF

Now, let's create the actual NodePort Service

kubectl apply -f k10-nodeport-svc.yaml

View the K10 Dashboard
Once completed, you should be able to view the K10 dashboard in the other tab on the left. We recommend taking the K10 dashboard tour at this point.
```

5. Backup Policy Creation
```
Backup Policy Creation
We will now create a policy to backup MySQL to the previously configured object storage location.

Creating a Policy
From the main K10 dashbboard, click on the Policies card. There should be no policies visible at this point.

Click Create New Policy and:

Give the policy a name (e.g., `backup-mysql)
Select Snapshot for the Action
Select Hourly for the Action Frequency
Leave the Snapshot Retention selection as-is
Select By Name for Select Applications and then, from the dropdown, select mysql
Leave all other settings as-is and select Create Policy
Running the Policy
The above policy will only run at the scheduled time (by default, at the top of the hour). To run the policy manually for the first time, click on Run Once on the Policies page. Confirm by clicking Run Policy and then go back to the main dashboard to view the job in action. Verify that the job has successfully completed.
```

6. Restore from Backup
```
Restore from Backup
Now that we have a MySQL backup, let's go simulate accidental data loss and then recover the system from that loss.

Causing Data Loss
For the purposes of this test drive, we will simply drop the k10demo database we had created earlier.

MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "DROP DATABASE k10demo"

Verify that the database has been deleted by running:

kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SHOW DATABASES LIKE 'k10demo'"

Recovering Data
To recover MySQL, go to the K10 dashboard, click Applications, and then select Restore on the MySQL card.

Click on a recent restore point and then select the Exported restore point (this is stored in the object storage system instead of as a non-durable snapshot on the storage system). In this case, we will select the default Application Name option to restore in place (Restore as "mysql"). Leave all other selections as-is, click on Restore, and confirm the action.

Return to the main dashboard to view the Restore job and verify that it completes successfully.

Verify Restore
To verify that our data was recovered, run the following command in the terminal to view the restored database:

Copy to clipboard
MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SHOW DATABASES LIKE 'k10demo'"
```
7. Confirm email for Quick Start k8s Book
```

```

