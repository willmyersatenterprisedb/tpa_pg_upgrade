---
customer: EDB Internal
title: BDR4 In-Place Major Postgres Upgrade
copyright-years: 2023
author: ['Will Myers <will.myers@enterprisedb.com>']
date: 22 Sep 2023
toc: True
cluster: {'type': 'BDR', 'subtype': 'Active-Active-Active'}
---

# Overview

Deploy a cluster then install and upgrade the nodes in-place to the
next major version of Postgres. This demo will be upgrading 4 data nodes
and 1 witness node from EPAS 13 to EPAS 14.

The strategy is to :
1) Update the configuration with new inventory variables
for those nodes slated for the major upgrade. For a BDR cluster, we can
simply set the `postgres_version` and `postgres_data_dir` variables in
the intermediary configuration files (see config.2.yml for an example).
2) Upgrade the shadows first, then upgrade the leader before finally upgrading
the witness. Run deploy between each set of upgrades to bring the upgraded nodes
properly back online with the cluster.
3) Run checks on the cluster and Postgres version.

Note:
- The binaries and data directory of the previous version are not removed
by the upgrade process.

Note: long command lines are wrapped for readability.

1.  **Initial deployment**

    ```
    cp config.1.yml config.yml
    tpaexec provision .
    tpaexec deploy .
    ```

2.  **Provision updated instance variables for nodes ready for upgrade**

    ```
    cp config.2.yml config.yml
    tpaexec provision .   
    ```

3.  **Upgrade the first data node**

    ```
    tpaexec upgrade-postgres . -e update_hosts=oltp01
    ```

4.  **Run deploy to bring the cluster to a good working state**

    ```
    tpaexec deploy .
    ```

5.  **Provision updated instance variables for nodes ready for upgrade**

    ```
    cp config.3.yml config.yml
    tpaexec provision .
    ```

6.  **Upgrade the second data node then the logical standby**

    ```
    tpaexec upgrade-postgres . -e update_hosts=oltp02,orr02
    ```

7.  **Run deploy to bring the cluster to a good working state**

    ```
    tpaexec deploy .
    ```

8.  **Provision updated instance variables for nodes ready for upgrade**

    ```
    cp config.4.yml config.yml
    tpaexec provision .
    ```


9.  **Upgrade the third data node**

    ```
    tpaexec upgrade-postgres . -e update_hosts=orr01
    ```

10.  **Run deploy to bring the cluster to a good working state**

    ```
    tpaexec deploy .
    ```

11.  **Provision updated instance variables for nodes ready for upgrade**

    ```
    cp config.5.yml config.yml
    tpaexec provision . 
    ```

12.  **Upgrade the witness node**

    ```
    tpaexec upgrade-postgres . -e update_hosts=wit
    ```

13.  **Run deploy to bring the cluster to a good working state**

    ```
    tpaexec deploy .
    ```

14.  **Verify cluster state and Postgres version**

    ```
    tpaexec cmd . oltp01,orr01,oltp02,orr02,wit  -b --become-user enterprisedb -a \
      'psql -p 5444 bdrdb -c "SELECT current_setting($$server_version$$) AS server_version;"'
    ```
