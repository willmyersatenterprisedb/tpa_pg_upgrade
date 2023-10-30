---
customer: EDB Internal
title: M1 In-Place Major Postgres Upgrade
copyright-years: 2023
author: ['Will Myers <will.myers@enterprisedb.com>']
date: 22 Sep 2023
toc: True
cluster: {'type': 'M1', 'subtype': 'Active-Passive-Passive'}
---

# Overview

Deploy a cluster then install and upgrade the nodes in-place to the 
next major version of Postgres. This demo will be upgrading two replicas
and a primary from Postgres 14 to Postgres 15.

The strategy is to :
1) Update the configuration with new inventory variables
for those nodes slated for the major upgrade. For an M1 cluster, we can 
simply set the `postgres_version` and `postgres_data_dir` variables in 
the intermediary configuration file (see config.2.yml for an example).
2) Upgrade replicas first then the primary. The replicas won't actually be
upgraded but will need to have the binaries and some degree of configuration
in place for the primary to upgrade them in-place after itself is upgraded.
3) Run deploy to bring the cluster to a good working order.
4) Provision and deploy our final configuration config.3.yml for good measure. 
5) Run checks on the cluster and Postgres version.

Note: 
- The last p/d step above is optional but highly recommended. Although config.2.yml
and config.3.yml are operationally equivalent, the readability of config.3.yml is
preferable. 
- The binaries and data directory of the previous version are not removed
by the upgrade process. 

Note: long command lines are wrapped for readability.


1.  **Initial deployment**

    ```
    cp config.1.yml config.yml
    tpaexec provision .
    tpaexec deploy .
    ```

2.  **Provision updated instance variables to nodes ready for upgrade**

    ```
    cp config.2.yml config.yml
    tpaexec provision .   
    ```

3.  **Upgrade replicas then primary to new major version of Postgres**

    ```
    tpaexec upgrade-postgres . -e update_hosts=charlie,bravo,alpha
    ```

4.  **Run deploy to bring the cluster to a good working state**

    ```
    tpaexec deploy .
    ```

5.  **Provision and deploy a more readable version of our new configuration**

    ```
    cp config.3.yml config.yml
    tpaexec provision . && tpaexec deploy .
    ```

6.  **Verify cluster state and Postgres version**

    ```
    tpaexec cmd . alpha,bravo,charlie -b --become-user postgres -a \
    '/usr/pgsql-15/bin/repmgr cluster show -f /etc/repmgr/15/repmgr.conf'

    tpaexec cmd . alpha,bravo,charlie -b --become-user postgres -a \
    'psql -p 5432 postgres -c "SELECT current_setting($$server_version$$) AS server_version;"'
    ```
