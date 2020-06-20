# Create Secure Build Hyper Protect Virtual Server

## Create directory for secure build lab and change directory

``` bash 
export SB_DIR="$HOME/securebuild-lab"
```

``` bash
mkdir -p "$SB_DIR" && cd "$SB_DIR"
```

## Create Certificate and Key for mutual tls

1. Generate rand file

    ``` bash
    openssl rand -out $HOME/.rnd -hex 256
    ```
2. Generate cert and key to use mutual tls 

    ``` bash
    openssl req -newkey rsa:2048 \
    -new -nodes -x509 \
    -days 3650 \
    -out "$SB_DIR"/sbs.cert \
    -keyout "$SB_DIR"/sbs.key \
    -subj "/C=US/O=IBM/CN=hpvs.example.com"
    ```

    ???+ example "Example Output"

        ``` bash
        Generating a RSA private key
        .......................................................+++++
        .......................+++++
        writing new private key to '/home/multiarch-lab/securebuild-lab/sbs.key'
        -----
        ```
3. Save the cert as an environment variable 

    ``` bash
    export cert=$(echo $(cat "$SB_DIR"/sbs.cert | base64) | tr -d ' ')
    ```

## Set your provided number 

You will be assigned a number for the lab so as not to interfere with other users.

=== "Command Syntax"

    ``` bash
    export HPVS_NUMBER="your_assigned_number"
    ```

=== "Example Command"

    ``` bash
    export HPVS_NUMBER="00"
    ```
    
!!! warning
    Your user will NOT be 00. Please set the appropriate user you have been assigned.

!!! note 
    [This table](assignment.md){target=_blank} tells you which number you are assigned.

## Create Quota Group with storage for secure build server

``` bash
hpvs quotagroup create --name "sb-user${HPVS_NUMBER}" --size=29GB
```

???+ example "Example Ouput"

    ``` bash
    +-------------+--------------+
    | name        | sb-user00    |
    | filesystem  | btrfs        |
    | passthrough | false        |
    | pool_id     | lv_data_pool |
    | size        | 29GB         |
    | available   | 29GB         |
    | containers  | []           |
    +-------------+--------------+
    ```

## Create Securebuild server

``` bash
hpvs vs create --name sbserver-${HPVS_NUMBER} --repo SecureDockerBuild \
--tag 1.2.1-release-9b63b43 --cpu 2 --ram 2048 \
--quotagroup "{quotagroup = sb-user${HPVS_NUMBER}, mountid = new, mount = /newroot, filesystem = ext4, size = 10GB}" \
--quotagroup "{quotagroup = sb-user${HPVS_NUMBER}, mountid = data, mount = /data, filesystem = ext4, size = 2GB}" \
--quotagroup "{quotagroup = sb-user${HPVS_NUMBER}, mountid = docker, mount = /docker, filesystem = ext4, size = 16GB}" \
--env={EX_VOLUMES="/docker,/data",ROOTFS_LOCK=y,CLIENT_CRT=$cert} \
--ports "{containerport = 443, protocol = tcp, hostport = 213${HPVS_NUMBER}}"
```

???+ example "Example Output"

    ``` bash
    ╭─────────────┬──────────────────────────────╮
    │ PROPERTIES  │ VALUES                       │
    ├─────────────┼──────────────────────────────┤
    │ Name        │ sbserver-00                  │
    │ Status      │ Up Less than a second        │
    │ CPU         │ 2                            │
    │ Memory      │ 2048                         │
    │ Networks    │ Network:bridge               │
    │             │ IPAddress:172.31.0.7         │
    │             │ Gateway:172.31.0.1           │
    │             │ Subnet:16                    │
    │             │ MacAddress:02:42:ac:1f:00:07 │
    │             │                              │
    │             │                              │
    │ Ports       │ LocalPort:443/tcp            │
    │             │ GuestPort:21300              │
    │             │                              │
    │ Quotagroups │ appliance_data               │
    │             │ sb-user00                    │
    │             │                              │
    │ State       │ running                      │
    ╰─────────────┴──────────────────────────────╯
    ```

We can see the quotagroup is now being used with

```
hpvs quotagroup show --name "sb-user${HPVS_NUMBER}"
```

???+ example "Example Output"

    ```
    +-------------+--------------------------------+
    | PROPERTIES  | VALUES                         |
    +-------------+--------------------------------+
    | name        | sb-user00                      |
    | filesystem  | btrfs                          |
    | passthrough | false                          |
    | pool_id     | lv_data_pool                   |
    | size        | 29GB                           |
    | available   | 701MB                          |
    | containers  | Mountids:"new","data","docker" |
    |             |                                |
    |             | Container:sbserver-00          |
    |             |                                |
    +-------------+--------------------------------+
    ```

The show output for the Hyper Protect Virtual Server was shown when it was deployed but when can bring it back up with

```
hpvs vs show --name sbserver-${HPVS_NUMBER}
```

???+ example "Example Output"

    ```
    ╭─────────────┬──────────────────────────────╮
    │ PROPERTIES  │ VALUES                       │
    ├─────────────┼──────────────────────────────┤
    │ Name        │ sbserver-00                  │
    │ Status      │ Up 3 minutes                 │
    │ CPU         │ 2                            │
    │ Memory      │ 2048                         │
    │ Networks    │ Network:bridge               │
    │             │ IPAddress:172.31.0.7         │
    │             │ Gateway:172.31.0.1           │
    │             │ Subnet:16                    │
    │             │ MacAddress:02:42:ac:1f:00:07 │
    │             │                              │
    │             │                              │
    │ Ports       │ LocalPort:443/tcp            │
    │             │ GuestPort:21300              │
    │             │                              │
    │ Quotagroups │ appliance_data               │
    │             │ sb-user00                    │
    │             │                              │
    │ State       │ running                      │
    ╰─────────────┴──────────────────────────────╯
    ```


Your secure build server is now up and running! :fire:

It is available at the IP Address of the Hyper Protect Virtual Server LPAR and port (GuestPort) specified (In this case https://192.168.22.79:21300)

You will use this secure build server to securely build your application in the next section. 

!!! note
    You can assign ips and hostnames for containers as necessary for your purposes but using the docker network and host ports is a nice way to quickly get running without having to use up IP addresses on your network.

