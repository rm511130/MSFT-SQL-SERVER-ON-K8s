# MSFT SQL SERVER ON K8s

- Microsoft SQL Server 2017 and 2019 can be deployed as containers on Kubernetes.
- Following the instructions in this repo, I'll show you how to get an environment up and running.

# SQL SERVER 2019 on VMware TKGI running on vSphere

- Log into an existing K8s cluster. In this example I'm using TKGI on vSphere, so I'm using the TKGI API to get my Kubectl context for a K8s cluster named "small":

```
tkgi login -a https://api.pks.pcf4u.com:9021 -u pks_admin -p password -k
tkgi get-credentials small
```

- 
