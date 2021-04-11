# MSFT SQL SERVER ON K8s

- The following instruction can be used to run a Microsoft SQL Server on an existing Kubernetes cluster.

# SQL SERVER on VMware TKGI running on vSphere

- We're going to use an existing K8s cluster named `small` running on TKGI / vSphere. 
- We start by invoking the TKGI API to get the Kubectl context, we then clone this repo, and apply [`sql-server-complete-deployment.yml`](./sql-server-complete-deployment.yml):

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

- Using the following commands, open a shell session to access your newly created SQL Server `pod`:
- _Note: please ignore the error message `groups: cannot find name for group ID 10001` should you see it._
```
kubectl exec $(kubectl get pods -n sqlserver | tail -n 1 | awk '{ print $1 }') -n sqlserver -it -- /bin/bash
```
- Wait for the new `mssql@mssqlinst:/$` Linux prompt and execute the following commands to take a look inside the SQL Server `pod`: 

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
- Let's create a database:

```
1> CREATE DATABASE TestDB
2> SELECT Name from sys.Databases
3> GO
```
- Let's create a table and insert some data:

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

# Quick recap

- The [`sql-server-complete-deployment.yml`](./sql-server-complete-deployment.yml) script created a `namespace` called `sqlserver`, placing in it several K8s objects necessary for SQL Server: `deployment`,`replicaset`, `pod`, `service`, `secret`, and `persistent volume claim`. 
- The script also create a `storage class` named `standard` that is based on the vSphere environment that I'm using to create and test this repo.
- You accessed the `sqlcmd` CLI that exists inside the SQL Server container to execute various SQL commands.
- You finished by deleting all the K8s objects involved with your SQL Server container environment.

# Notes on the environment I used:

- I'm using a MacBook with kubectl, tkgi and other tools that a typical K8s user is expected to have & use.
- The `small` K8s cluster had 1 Master and 3 Worker nodes all configured as `4 vCPU 16GB RAM 32GB HDD` virtual machines.

```
kubectl get nodes -o wide
NAME                                   STATUS   ROLES    AGE   VERSION            INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
61e328a4-d709-4669-a804-6856c0ee8a54   Ready    <none>   9h    v1.19.6+vmware.1   10.0.7.14     10.0.7.14     Ubuntu 16.04.7 LTS   4.15.0-133-generic   docker://19.3.14
6e2ac16b-3261-45ae-9854-2847c0504f42   Ready    <none>   8h    v1.19.6+vmware.1   10.0.7.13     10.0.7.13     Ubuntu 16.04.7 LTS   4.15.0-133-generic   docker://19.3.14
d4586d00-435a-4b79-aa23-c48d45e0034e   Ready    <none>   17h   v1.19.6+vmware.1   10.0.7.11     10.0.7.11     Ubuntu 16.04.7 LTS   4.15.0-133-generic   docker://19.3.14
```







