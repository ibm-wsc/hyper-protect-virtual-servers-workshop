# Displaying your GREP11 environment

## List Crypto Domains on your Hyper Protect Virtual Servers LPAR

1. List your available domains with the Hyper Protect Virtual Server CLI:

    ``` bash
    hpvs crypto list
    ```

    ???+ example "Example output"

        ```
        hpvs crypto list
        +---------------+--------+
        | CRYPTO.DOMAIN | STATUS |
        +---------------+--------+
        | 08.0016       | in use |
        | 0a.0016       | in use |
        +---------------+--------+
        ```

    A value of *in use* in the **STATUS** column indicates that each crypto domain has a GREP11 server connected to it.

## Display the GREP11 servers

2. Display the virtual servers running on the Hyper Protect Virtual Servers LPAR:

    ``` bash
    hpvs vs list
    ```

    ???+ example "Example output"
    
        ```
        +----------------------+---------+----------------------+----------------------------------------------------------+
        | NAMES                | STATE   | STATUS               | IMAGE                                                    |
        +----------------------+---------+----------------------+----------------------------------------------------------+
        | collectd             | running | Up 2 weeks           | ibmzcontainers/collectd-host:1.2.1                       |
        | grep11-08-0016-9876  | running | Up 9 days            | ibmzcontainers/hpcs-grep11-prod:1.2.1                    |
        | sbserver_00          | running | Up 9 days            | ibmzcontainers/secure-docker-build:1.2.1-release-9b63b43 |
        | sbserver_19          | running | Up 9 days            | ibmzcontainers/secure-docker-build:1.2.1-release-9b63b43 |
        | grep11-0a-0016-19876 | running | Up 9 days            | ibmzcontainers/hpcs-grep11-prod:1.2.1                    |
        | hpvs_bc_08           | running | Up 2 weeks           | mplccblockchain/hpvs_bc:latest                           |
        | sbserver_08          | running | Up 2 weeks           | ibmzcontainers/secure-docker-build:1.2.1-release-9b63b43 |
        | monitoring           | running | Up 2 weeks           | ibmzcontainers/monitoring:1.2.1                          |
        | hpvs_grafana         | running | Up 8 days            | jinxiong/hpvs_grafana:latest                             |
        | prom0630_19          | running | Up 9 days            | jinxiong/prom0630:latest                                 |
        | sbserver_07          | running | Up 10 days           | ibmzcontainers/secure-docker-build:1.2.1-release-9b63b43 |
        +----------------------+---------+----------------------+----------------------------------------------------------+
        ```

    Observe that there are two servers listed whose name starts with `grep11`.  These are the two GREP11 servers that we have provided. One accessible from host port 9876 and one from host port 19876.

3. Display details about one of the GREP11 servers with this command:

    ``` bash
    hpvs vs show --name grep11-08-0016-9876
    ```

    ???+ example "Example output"

        ```
        ╭─────────────┬──────────────────────────────╮
        │ PROPERTIES  │ VALUES                       │
        ├─────────────┼──────────────────────────────┤
        │ Name        │ grep11-08-0016-9876          │
        │ Status      │ Up 9 days                    │
        │ CPU         │ 1                            │
        │ Memory      │ 4096                         │
        │ Networks    │ Network:bridge               │
        │             │ IPAddress:172.31.0.10        │
        │             │ Gateway:172.31.0.1           │
        │             │ Subnet:16                    │
        │             │ MacAddress:02:42:ac:1f:00:0a │
        │             │                              │
        │             │                              │
        │ Ports       │ LocalPort:9876/tcp           │
        │             │ GuestPort:9876               │
        │             │                              │
        │ Quotagroups │ appliance_data               │
        │             │                              │
        │ State       │ running                      │
        ╰─────────────┴──────────────────────────────╯
        ```