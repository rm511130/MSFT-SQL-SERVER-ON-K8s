# MSFT SQL SERVER ON K8s

- Microsoft SQL Server 2017 and 2019 can be deployed as containers on Kubernetes.
- The instructions below show how to get a SQL Server environment up and running in an existing K8s cluster.
- The [`sql-server-complete-deployment.yml`](./sql-server-complete-deployment.yml) script will place the `deployment`,`replicaset`, `pod`, `service`, `secret`, and `persistent volume claim` in a `namespace` named `sqlserver` which is also created by the same script. The script will also create a `storage class` named `standard` that is based on the vSphere environment I'm using to create and test this repo.
- I'm using a MacBook with kubectl, tkgi and other tools that a typical K8s user is expected to have & use.

# SQL SERVER 2019 on VMware TKGI running on vSphere

- We're going to use a K8s cluster named `small` running on TKGI / vSphere. 
- We start by invoking the TKGI API to get the Kubectl context, we `git clone` this repo, and we `kubectl apply` the contents of the [`sql-server-complete-deployment.yml`](./sql-server-complete-deployment.yml):

```
tkgi login -a https://api.pks.pcf4u.com:9021 -u pks_admin -p password -k
tkgi get-credentials small
git clone https://github.com/rm511130/MSFT-SQL-SERVER-ON-K8s
cd MSFT-SQL-SERVER-ON-K8s
kubectl apply -f ./sql-server-complete-deployment.yml
```

- Wait until the `mssql` pod is running:

```
watch kubectl get pods -n sqlserver
```

- Using the following commands, open a shell session to access your newly created `pod`:
- _Note: please ignore the error message `groups: cannot find name for group ID 10001` should you see it._
```
kubectl exec $(kubectl get pods -n sqlserver | tail -n 1 | awk '{ print $1 }') -n sqlserver -it -- /bin/bash
```
- Wait for the new `mssql@mssqlinst:/$` Linux prompt and execute the following commands to take a look inside the `pod` running SQL Server:

```
cat /etc/*release | head -n 4
whoami
hostname
hostname -I
id
```
- Now let's use `sqlcmd` to execute SQL commands:

```
alias sqlcmd="/opt/mssql-tools/bin/sqlcmd"
sqlcmd -S $(hostname -I) -U sa -P Password1
```

- Once you see the `1>` prompt, you can proceed with the following examples:

```
1> SELECT 1
2> GO
```
- Checking the SQL Server version:
```
1> SELECT @@VERSION
2> GO
```
- Let's create a database, a table and insert some data:

```
1> CREATE DATABASE TestDB
2> SELECT Name from sys.Databases
3> GO
```
```
1> USE TestDB
2> CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT)
3> INSERT INTO Inventory VALUES (1, 'banana', 150); 
4> INSERT INTO Inventory VALUES (2, 'orange', 154);
5> SELECT * FROM Inventory WHERE quantity > 152;
6> GO
```

# Quit, Exit and Clean-up

- Please execute the following commands:

```
1> quit
mssql@mssqlinst:/$ exit
kubectl delete -f sql-server-complete-deployment.yml
```
- You will be left with the cloned contents of this repo.









