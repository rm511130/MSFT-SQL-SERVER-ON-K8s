# MSFT SQL SERVER ON K8s

- Microsoft SQL Server 2017 and 2019 can be deployed as containers on Kubernetes.
- The instructions below show how to get a SQL Server environment up and running in an existing K8s cluster.
- I'm using a MacBook with kubectl, tkgi and other tools any K8s user is expected to have & use.

# SQL SERVER 2019 on VMware TKGI running on vSphere

- We're going to use a K8s cluster named `small` running on TKGI / vSphere. The TKGI API is invoked to get the Kubectl context:

```
tkgi login -a https://api.pks.pcf4u.com:9021 -u pks_admin -p password -k
tkgi get-credentials small
git clone https://github.com/rm511130/MSFT-SQL-SERVER-ON-K8s
cd MSFT-SQL-SERVER-ON-K8s
```

- Next we execute the `sql-server-complete.yml` script and then watch for the `mssql pod` to come be ready:

```
kubectl apply -f ./sql-server-complete.yml
watch kubectl get pods -n sqlserver
```





