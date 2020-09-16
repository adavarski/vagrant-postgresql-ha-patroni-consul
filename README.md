vagrant-postgresql-patroni-consul-setup
======================================

# Overview

This repo it helps you quickly spin up a 3-node cluster of PostgreSQL, managed by Patroni using Consul.

# What's in the cluster?

When you start the cluster, you get 2 nodes (pg01 and pg02), each running:

  - PostgreSQL
  - Patroni
  - Consul agent

and a third node (pg03) running:

  - Consul (server)
  - HAProxy
  - pgBouncer

The connection/data path is the following:

1. pgBouncer (where the app will connect)
2. HAProxy
3. PostgreSQL

All packages are from Ubuntu 18.04, except for PostgreSQL itself, which is at version 12.x.

The cluster is configured with a single primary and one asynchronous streaming replica.

# Dependencies
1. [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
2. [Vagrant](http://www.vagrantup.com/downloads.html)
3. `git clone https://github.com/vtomasr5/postgresql-patroni-setup.git`

# Getting started

1.  On 3 separate windows:
2.  `vagrant up pg03 && vagrant ssh pg03 # pg03 contains the consul server and it must start first` 
3.  `vagrant up pg01 && vagrant ssh pg01`
4.  `vagrant up pg02 && vagrant ssh pg02`

# Viewing cluster status

Get patroni information from his members
  - `patronictl -c /etc/patroni/patroni.yml list`

# Connecting to PostgreSQL

You should use the pgBouncer connection pool
  - 172.28.33.13:6432

But you can also connect via HAproxy using the balancing IP
  - 172.28.33.13:5000 (postgresql)
  - 172.28.33.13:7000 (HAProxy stats)

# Test HA

```
vagrant@pg01:~$ patronictl -c /etc/patroni/patroni.yml list
+ Cluster: clustertest (6873022352597197692) ---+-----------+
| Member |     Host     |  Role  |  State  | TL | Lag in MB |
+--------+--------------+--------+---------+----+-----------+
|  pg01  | 172.28.33.11 | Leader | running |  1 |           |
|  pg02  | 172.28.33.12 |        | running |  1 |       0.0 |
+--------+--------------+--------+---------+----+-----------+

# vagrant halt pg01
==> pg01: Attempting graceful shutdown of VM...


vagrant@pg02:~$ patronictl -c /etc/patroni/patroni.yml list
+ Cluster: clustertest (6873022352597197692) ---+-----------+
| Member |     Host     |  Role  |  State  | TL | Lag in MB |
+--------+--------------+--------+---------+----+-----------+
|  pg02  | 172.28.33.12 | Leader | running |  3 |           |
+--------+--------------+--------+---------+----+-----------+

# vagrant up pg01

# vagrant ssh pg01
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-117-generic x86_64)

vagrant@pg01:~$ patronictl -c /etc/patroni/patroni.yml list
+ Cluster: clustertest (6873022352597197692) ---+-----------+
| Member |     Host     |  Role  |  State  | TL | Lag in MB |
+--------+--------------+--------+---------+----+-----------+
|  pg01  | 172.28.33.11 |        | running |  3 |       0.0 |
|  pg02  | 172.28.33.12 | Leader | running |  3 |           |
+--------+--------------+--------+---------+----+-----------+
```

# TODO

- [X] Add pgBouncer support
- [ ] Improve documentation

# Further reading

* [PostgreSQL and Patroni cluster](https://www.linode.com/docs/databases/postgresql/create-a-highly-available-postgresql-cluster-using-patroni-and-haproxy/#before-you-begin)

# References
* [PostgreSQL](https://www.postgresql.org)
* [Patroni](https://patroni.readthedocs.io/en/latest/)
* [HAProxy](https://www.haproxy.org/)
* [Vagrant](http://vagrantup.com)
* [VirtualBox](http://www.virtualbox.org)
