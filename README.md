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
1> SELECT GETDATE()
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

```
kubectl top nodes
NAME                                   CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
61e328a4-d709-4669-a804-6856c0ee8a54   84m          2%     1948Mi          12%       
6e2ac16b-3261-45ae-9854-2847c0504f42   75m          1%     661Mi           4%        
d4586d00-435a-4b79-aa23-c48d45e0034e   133m         3%     1124Mi          7% 
```

```
kubectl get pods -n sqlserver
NAME                                READY   STATUS    RESTARTS   AGE
mssql-deployment-64dc97dfbb-gzbrh   1/1     Running   0          4m40s
```

```
kubectl logs -n sqlserver mssql-deployment-64dc97dfbb-gzbrh 
SQL Server 2019 will run as non-root by default.
This container is running as user mssql.
To learn more visit https://go.microsoft.com/fwlink/?linkid=2099216.
2021-04-11 22:12:12.30 Server      Setup step is copying system data file 'C:\templatedata\master.mdf' to '/var/opt/mssql/data/master.mdf'.
2021-04-11 22:12:12.38 Server      Did not find an existing master data file /var/opt/mssql/data/master.mdf, copying the missing default master and other system database files. If you have moved the database location, but not moved the database files, startup may fail. To repair: shutdown SQL Server, move the master database to configured location, and restart.
2021-04-11 22:12:12.40 Server      Setup step is copying system data file 'C:\templatedata\mastlog.ldf' to '/var/opt/mssql/data/mastlog.ldf'.
2021-04-11 22:12:12.41 Server      Setup step is copying system data file 'C:\templatedata\model.mdf' to '/var/opt/mssql/data/model.mdf'.
2021-04-11 22:12:12.43 Server      Setup step is copying system data file 'C:\templatedata\modellog.ldf' to '/var/opt/mssql/data/modellog.ldf'.
2021-04-11 22:12:12.45 Server      Setup step is copying system data file 'C:\templatedata\msdbdata.mdf' to '/var/opt/mssql/data/msdbdata.mdf'.
2021-04-11 22:12:12.47 Server      Setup step is copying system data file 'C:\templatedata\msdblog.ldf' to '/var/opt/mssql/data/msdblog.ldf'.
2021-04-11 22:12:12.48 Server      Setup step is FORCE copying system data file 'C:\templatedata\model_replicatedmaster.mdf' to '/var/opt/mssql/data/model_replicatedmaster.mdf'.
2021-04-11 22:12:12.49 Server      Setup step is FORCE copying system data file 'C:\templatedata\model_replicatedmaster.ldf' to '/var/opt/mssql/data/model_replicatedmaster.ldf'.
2021-04-11 22:12:12.50 Server      Setup step is FORCE copying system data file 'C:\templatedata\model_msdbdata.mdf' to '/var/opt/mssql/data/model_msdbdata.mdf'.
2021-04-11 22:12:12.51 Server      Setup step is FORCE copying system data file 'C:\templatedata\model_msdblog.ldf' to '/var/opt/mssql/data/model_msdblog.ldf'.
2021-04-11 22:12:12.61 Server      Microsoft SQL Server 2019 (RTM-CU10) (KB5001090) - 15.0.4123.1 (X64) 
	Mar 22 2021 18:10:24 
	Copyright (C) 2019 Microsoft Corporation
	Developer Edition (64-bit) on Linux (Ubuntu 20.04.2 LTS) <X64>
2021-04-11 22:12:12.62 Server      UTC adjustment: 0:00
2021-04-11 22:12:12.62 Server      (c) Microsoft Corporation.
2021-04-11 22:12:12.63 Server      All rights reserved.
2021-04-11 22:12:12.63 Server      Server process ID is 388.
2021-04-11 22:12:12.63 Server      Logging SQL Server messages in file '/var/opt/mssql/log/errorlog'.
2021-04-11 22:12:12.64 Server      Registry startup parameters: 
	 -d /var/opt/mssql/data/master.mdf
	 -l /var/opt/mssql/data/mastlog.ldf
	 -e /var/opt/mssql/log/errorlog
2021-04-11 22:12:12.65 Server      SQL Server detected 4 sockets with 1 cores per socket and 1 logical processors per socket, 4 total logical processors; using 4 logical processors based on SQL Server licensing. This is an informational message; no user action is required.
2021-04-11 22:12:12.66 Server      SQL Server is starting at normal priority base (=7). This is an informational message only. No user action is required.
2021-04-11 22:12:12.66 Server      Detected 12832 MB of RAM. This is an informational message; no user action is required.
2021-04-11 22:12:12.67 Server      Using conventional memory in the memory manager.
2021-04-11 22:12:12.68 Server      Page exclusion bitmap is enabled.
2021-04-11 22:12:12.71 Server      Buffer pool extension is not supported on Linux platform.
2021-04-11 22:12:12.72 Server      Buffer Pool: Allocating 2097152 bytes for 1975208 hashPages.
2021-04-11 22:12:13.03 Server      Buffer pool extension is already disabled. No action is necessary.
2021-04-11 22:12:13.94 Server      Successfully initialized the TLS configuration. Allowed TLS protocol versions are ['1.0 1.1 1.2']. Allowed TLS ciphers are ['ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:!DHE-RSA-AES256-GCM-SHA384:!DHE-RSA-AES128-GCM-SHA256:!DHE-RSA-AES256-SHA:!DHE-RSA-AES128-SHA'].
2021-04-11 22:12:13.99 Server      Query Store settings initialized with enabled = 1, 
2021-04-11 22:12:14.01 Server      The maximum number of dedicated administrator connections for this instance is '1'
2021-04-11 22:12:14.01 Server      Node configuration: node 0: CPU mask: 0x000000000000000f:0 Active CPU mask: 0x000000000000000f:0. This message provides a description of the NUMA configuration for this computer. This is an informational message only. No user action is required.
2021-04-11 22:12:14.04 Server      Using dynamic lock allocation.  Initial allocation of 2500 Lock blocks and 5000 Lock Owner blocks per node.  This is an informational message only.  No user action is required.
2021-04-11 22:12:14.07 Server      In-Memory OLTP initialized on lowend machine.
2021-04-11 22:12:14.09 Server      [INFO] Created Extended Events session 'hkenginexesession'
2021-04-11 22:12:14.10 Server      Database Instant File Initialization: enabled. For security and performance considerations see the topic 'Database Instant File Initialization' in SQL Server Books Online. This is an informational message only. No user action is required.
ForceFlush is enabled for this instance. 
2021-04-11 22:12:14.12 Server      Total Log Writer threads: 2. This is an informational message; no user action is required.
2021-04-11 22:12:14.15 Server      clflush is selected for pmem flush operation.
2021-04-11 22:12:14.16 Server      Software Usage Metrics is disabled.
2021-04-11 22:12:14.18 spid10s     [1]. Feature Status: PVS: 0. CTR: 0. ConcurrentPFSUpdate: 1.
2021-04-11 22:12:14.18 Server      CLR version v4.0.30319 loaded.
2021-04-11 22:12:14.18 spid10s     Starting up database 'master'.
ForceFlush feature is enabled for log durability.
2021-04-11 22:12:14.30 spid10s     The tail of the log for database master is being rewritten to match the new sector size of 4096 bytes.  3584 bytes at offset 393728 in file /var/opt/mssql/data/mastlog.ldf will be written.
2021-04-11 22:12:14.42 spid10s     Converting database 'master' from version 897 to the current version 904.
2021-04-11 22:12:14.43 spid10s     Database 'master' running the upgrade step from version 897 to version 898.
2021-04-11 22:12:14.48 spid10s     Database 'master' running the upgrade step from version 898 to version 899.
2021-04-11 22:12:14.54 spid10s     Database 'master' running the upgrade step from version 899 to version 900.
2021-04-11 22:12:14.55 Server      Common language runtime (CLR) functionality initialized.
2021-04-11 22:12:14.59 spid10s     Database 'master' running the upgrade step from version 900 to version 901.
2021-04-11 22:12:14.62 spid10s     Database 'master' running the upgrade step from version 901 to version 902.
2021-04-11 22:12:14.70 spid10s     Database 'master' running the upgrade step from version 902 to version 903.
2021-04-11 22:12:14.76 spid10s     Database 'master' running the upgrade step from version 903 to version 904.
2021-04-11 22:12:15.16 spid10s     Resource governor reconfiguration succeeded.
2021-04-11 22:12:15.17 spid10s     SQL Server Audit is starting the audits. This is an informational message. No user action is required.
2021-04-11 22:12:15.18 spid10s     SQL Server Audit has started the audits. This is an informational message. No user action is required.
2021-04-11 22:12:15.24 spid10s     SQL Trace ID 1 was started by login "sa".
2021-04-11 22:12:15.27 spid26s     Password policy update was successful.
2021-04-11 22:12:15.33 spid10s     Server name is 'mssqlinst'. This is an informational message only. No user action is required.
2021-04-11 22:12:15.36 spid28s     Always On: The availability replica manager is starting. This is an informational message only. No user action is required.
2021-04-11 22:12:15.37 spid12s     [32767]. Feature Status: PVS: 0. CTR: 0. ConcurrentPFSUpdate: 1.
2021-04-11 22:12:15.38 spid12s     Starting up database 'mssqlsystemresource'.
2021-04-11 22:12:15.37 spid10s     [4]. Feature Status: PVS: 0. CTR: 0. ConcurrentPFSUpdate: 1.
2021-04-11 22:12:15.38 spid28s     Always On: The availability replica manager is waiting for the instance of SQL Server to allow client connections. This is an informational message only. No user action is required.
2021-04-11 22:12:15.38 spid10s     Starting up database 'msdb'.
2021-04-11 22:12:15.39 spid12s     The resource database build version is 15.00.4123. This is an informational message only. No user action is required.
2021-04-11 22:12:15.44 spid12s     [3]. Feature Status: PVS: 0. CTR: 0. ConcurrentPFSUpdate: 1.
2021-04-11 22:12:15.44 spid12s     Starting up database 'model'.
2021-04-11 22:12:15.50 spid10s     The tail of the log for database msdb is being rewritten to match the new sector size of 4096 bytes.  3072 bytes at offset 50176 in file /var/opt/mssql/data/MSDBLog.ldf will be written.
2021-04-11 22:12:15.53 spid26s     A self-generated certificate was successfully loaded for encryption.
2021-04-11 22:12:15.53 spid12s     The tail of the log for database model is being rewritten to match the new sector size of 4096 bytes.  512 bytes at offset 73216 in file /var/opt/mssql/data/modellog.ldf will be written.
2021-04-11 22:12:15.55 spid26s     Server is listening on [ 'any' <ipv6> 1433].
2021-04-11 22:12:15.56 spid26s     Server is listening on [ 'any' <ipv4> 1433].
2021-04-11 22:12:15.58 Server      Server is listening on [ ::1 <ipv6> 1434].
2021-04-11 22:12:15.59 spid10s     Converting database 'msdb' from version 897 to the current version 904.
2021-04-11 22:12:15.59 Server      Server is listening on [ 127.0.0.1 <ipv4> 1434].
2021-04-11 22:12:15.59 spid10s     Database 'msdb' running the upgrade step from version 897 to version 898.
2021-04-11 22:12:15.60 spid12s     Converting database 'model' from version 897 to the current version 904.
2021-04-11 22:12:15.60 Server      Dedicated admin connection support was established for listening locally on port 1434.
2021-04-11 22:12:15.61 spid12s     Database 'model' running the upgrade step from version 897 to version 898.
2021-04-11 22:12:15.62 spid26s     Server is listening on [ ::1 <ipv6> 1431].
2021-04-11 22:12:15.62 spid26s     Server is listening on [ 127.0.0.1 <ipv4> 1431].
2021-04-11 22:12:15.63 spid26s     SQL Server is now ready for client connections. This is an informational message; no user action is required.
2021-04-11 22:12:15.64 spid10s     Database 'msdb' running the upgrade step from version 898 to version 899.
2021-04-11 22:12:15.66 spid12s     Database 'model' running the upgrade step from version 898 to version 899.
2021-04-11 22:12:15.72 spid10s     Database 'msdb' running the upgrade step from version 899 to version 900.
2021-04-11 22:12:15.73 spid12s     Database 'model' running the upgrade step from version 899 to version 900.
2021-04-11 22:12:15.78 spid10s     Database 'msdb' running the upgrade step from version 900 to version 901.
2021-04-11 22:12:15.79 spid12s     Database 'model' running the upgrade step from version 900 to version 901.
2021-04-11 22:12:15.82 spid10s     Database 'msdb' running the upgrade step from version 901 to version 902.
2021-04-11 22:12:15.83 spid12s     Database 'model' running the upgrade step from version 901 to version 902.
2021-04-11 22:12:15.90 spid12s     Database 'model' running the upgrade step from version 902 to version 903.
2021-04-11 22:12:15.93 spid12s     Database 'model' running the upgrade step from version 903 to version 904.
2021-04-11 22:12:16.08 spid12s     Clearing tempdb database.
2021-04-11 22:12:16.40 spid12s     [2]. Feature Status: PVS: 0. CTR: 0. ConcurrentPFSUpdate: 1.
2021-04-11 22:12:16.40 spid12s     Starting up database 'tempdb'.
2021-04-11 22:12:16.51 spid12s     The tempdb database has 1 data file(s).
2021-04-11 22:12:16.52 spid31s     The Service Broker endpoint is in disabled or stopped state.
2021-04-11 22:12:16.54 spid31s     The Database Mirroring endpoint is in disabled or stopped state.
2021-04-11 22:12:16.56 spid31s     Service Broker manager has started.
2021-04-11 22:12:16.61 spid10s     Database 'msdb' running the upgrade step from version 902 to version 903.
2021-04-11 22:12:16.64 spid10s     Database 'msdb' running the upgrade step from version 903 to version 904.
2021-04-11 22:12:16.83 spid10s     Recovery is complete. This is an informational message only. No user action is required.
2021-04-11 22:12:16.86 spid28s     The default language (LCID 0) has been set for engine and full-text services.
2021-04-11 22:12:17.15 spid28s     The tempdb database has 4 data file(s).
```

# Additional URLs with relevant information:

- https://www.cdata.com/kb/tech/sql-odbc-polybase.rst 
- https://docs.microsoft.com/en-us/sql/linux/tutorial-sql-server-containers-kubernetes?view=sql-server-ver15
- https://blog.dbi-services.com/using-non-root-sql-server-containers-on-docker-and-k8s/










