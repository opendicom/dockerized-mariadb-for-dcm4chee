## dockerized-mariadb-for-dcm4chee
Docker image that provides a MariaDB instance for DCM4CHEE 5.x

Based on https://github.com/dcm4che-dockerfiles/postgres-dcm4chee

Migrated from https://github.com/opendicom/dockerized-dcm4chee-mariadb

There are a couple of compose files available on the `docker-compose` directory that bootstap replicated nodes.
The simplest one starts a local environment with two containers. The purpose of the other two (`host_one.yml` and `host_two.yml`) is to test the replication with containers running on separate docker hosts.

Let's start with the local environment. Get it up by running
`docker-compose up`. After that, get inside the first container by executing
`docker exec -ti db_node_one bash` and:

- Start a mysql session: `mysql -psecure awesome`
- Get the status of the cluster: `SHOW STATUS LIKE 'wsrep_cluster_size';`
- Insert a row in some random table: `insert into code (pk) values(123);`

Now read that row from the other node. Get inside it by executing `docker exec -ti db_node_two bash` and:

- Start a mysql session: `mysql -psecure awesome`
- Get the status of the cluster: `SHOW STATUS LIKE 'wsrep_cluster_size';`
- Read from the table: `select * from code;`

##### Node One:
```
root@11dfe3540057:/# mysql -psecure awesome
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 10.1.14-MariaDB-1~jessie mariadb.org binary distribution

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [awesome]> SHOW STATUS LIKE 'wsrep_cluster_size';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+
1 row in set (0.01 sec)

MariaDB [awesome]> insert into code (pk) values(123);
Query OK, 1 row affected, 4 warnings (0.01 sec)
```

##### Node Two:
```
root@5099b8d519ec:/# mysql -psecure awesome                                                                                                                                                               
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.1.14-MariaDB-1~jessie mariadb.org binary distribution

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [awesome]> select * from code;
+-----+--------------+------------+-----------------+--------------+
| pk  | code_meaning | code_value | code_designator | code_version |
+-----+--------------+------------+-----------------+--------------+
| 123 |              |            |                 |              |
+-----+--------------+------------+-----------------+--------------+
1 row in set (0.00 sec)
```


**Sources**

http://galeracluster.com/2015/05/getting-started-galera-with-docker-part-1/
https://www.digitalocean.com/community/tutorials/how-to-configure-a-galera-cluster-with-mariadb-on-ubuntu-12-04-servers
https://mariadb.com/kb/en/mariadb/installing-mariadb-deb-files/
https://github.com/docker-library/mariadb/tree/master/10.1
http://wiki.centos-webpanel.com/mysql-cluster-mariadb-galera

**Galera Cluster System Variables:**

https://mariadb.com/kb/en/mariadb/galera-cluster-system-variables
http://galeracluster.com/documentation-webpages/mysqlwsrepoptions.html
