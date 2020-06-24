# Create Secure Build Hyper Protect Virtual Server

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Create Certificate and Key for mutual tls

1. Generate rand file

    ``` bash
    openssl rand -out "${HOME}/.rnd" -hex 256
    ```

2. Make `SB_DIR/sbs_keys` directory to store secure build server keys.

    ``` bash
    mkdir -p "${SB_DIR}/sbs_keys"
    ```

3. Generate cert and key to use mutual tls 

    ``` bash
    openssl req -newkey rsa:2048 \
    -new -nodes -x509 \
    -days 3650 \
    -out "${SB_DIR}/sbs_keys/sbs.cert" \
    -keyout "${SB_DIR}/sbs_keys/sbs.key" \
    -subj "/C=US/O=IBM/CN=hpvs.example.com"
    ```

    ???+ example "Example Output"

        ``` bash
        Generating a RSA private key
        ................................+++++
        ...............................+++++
        writing new private key to '/home/multiarch-lab/securebuild-lab/sbs_keys/sbs.key'
        -----
        ```

4. Change cert to base64 encoding and save to new file.

    ``` bash
    echo $(cat "${SB_DIR}/sbs_keys/sbs.cert" | base64) \
    | tr -d ' ' >> "${SB_DIR}/sbs_keys/sbs_base64.cert"
    ```

5. Save the cert as an environment variable 

    ``` bash
    export cert=$(cat "${SB_DIR}/sbs_keys/sbs_base64.cert")
    ```

## Set your provided number and save it for later use

You will be assigned a number for the lab so as not to interfere with other users.

!!! note 
    [This table](assignment.md){target=_blank} tells you which number you are assigned.

1. Set the `HPVS_NUMBER` variable with your assigned 2 digit number

    === "Command Syntax"

        ``` bash
        export HPVS_NUMBER="your_assigned_number"
        ```

    === "Example Command"

        ``` bash
        export HPVS_NUMBER="00"
        ```
        
    !!! warning
        Your user will **NOT** be `00`. Please set the appropriate user you have been assigned.

2. Save your number to `bashrc` for later use.

    ``` bash
    echo "export HPVS_NUMBER='${HPVS_NUMBER}'" >> "${HOME}/.bashrc"
    ```
3. Set Secure Build Server Port

    ``` bash
    export SB_PORT=300${HPVS_NUMBER}
    ```
    
4. Save `SB_PORT` to `bashrc` for later use.

    ``` bash
    echo "export SB_PORT='${SB_PORT}'" >> "${HOME}/.bashrc"
    ```

## Create Quotagroup with storage for secure build server

``` bash
hpvs quotagroup create --name "sb_user${HPVS_NUMBER}" --size=30GB
```

???+ example "Example Ouput"

    ``` bash
    +-------------+--------------+
    | name        | sb_user00    |
    | filesystem  | btrfs        |
    | passthrough | false        |
    | pool_id     | lv_data_pool |
    | size        | 30GB         |
    | available   | 30GB         |
    | containers  | []           |
    +-------------+--------------+
    ```

## Create Securebuild server

``` bash
hpvs vs create --name sbserver_${HPVS_NUMBER} --repo SecureDockerBuild \
--tag 1.2.1-release-9b63b43 --cpu 2 --ram 2048 \
--quotagroup "{quotagroup = sb_user${HPVS_NUMBER}, mountid = new, mount = /newroot, filesystem = ext4, size = 10GB}" \
--quotagroup "{quotagroup = sb_user${HPVS_NUMBER}, mountid = data, mount = /data, filesystem = ext4, size = 2GB}" \
--quotagroup "{quotagroup = sb_user${HPVS_NUMBER}, mountid = docker, mount = /docker, filesystem = ext4, size = 16GB}" \
--env={EX_VOLUMES="/docker,/data",ROOTFS_LOCK=y,CLIENT_CRT=$cert} \
--ports "{containerport = 443, protocol = tcp, hostport = ${SB_PORT}}"
```

???+ example "Example Output"

    ``` bash
    ╭─────────────┬──────────────────────────────╮
    │ PROPERTIES  │ VALUES                       │
    ├─────────────┼──────────────────────────────┤
    │ Name        │ sbserver_00                  │
    │ Status      │ Up Less than a second        │
    │ CPU         │ 2                            │
    │ Memory      │ 2048                         │
    │ Networks    │ Network:bridge               │
    │             │ IPAddress:172.31.0.5         │
    │             │ Gateway:172.31.0.1           │
    │             │ Subnet:16                    │
    │             │ MacAddress:02:42:ac:1f:00:05 │
    │             │                              │
    │             │                              │
    │ Ports       │ LocalPort:443/tcp            │
    │             │ GuestPort:30000              │
    │             │                              │
    │ Quotagroups │ appliance_data               │
    │             │ sb_user00                    │
    │             │                              │
    │ State       │ running                      │
    ╰─────────────┴──────────────────────────────╯
    ```

We can see the quotagroup is now being used with

``` bash
hpvs quotagroup show --name "sb_user${HPVS_NUMBER}"
```

???+ example "Example Output"

    ``` bash
    +-------------+--------------------------------+
    | PROPERTIES  | VALUES                         |
    +-------------+--------------------------------+
    | name        | sb_user00                      |
    | filesystem  | btrfs                          |
    | passthrough | false                          |
    | pool_id     | lv_data_pool                   |
    | size        | 30GB                           |
    | available   | 2GB                            |
    | containers  | Container:sbserver_00          |
    |             | Mountids:"new","data","docker" |
    |             |                                |
    |             |                                |
    +-------------+--------------------------------+
    ```

The show output for the Hyper Protect Virtual Server was shown when it was deployed but we can bring it back up with

``` bash
hpvs vs show --name "sbserver_${HPVS_NUMBER}"
```

???+ example "Example Output"

    ``` bash
    ╭─────────────┬──────────────────────────────╮
    │ PROPERTIES  │ VALUES                       │
    ├─────────────┼──────────────────────────────┤
    │ Name        │ sbserver_00                  │
    │ Status      │ Up About a minute            │
    │ CPU         │ 2                            │
    │ Memory      │ 2048                         │
    │ Networks    │ Network:bridge               │
    │             │ IPAddress:172.31.0.5         │
    │             │ Gateway:172.31.0.1           │
    │             │ Subnet:16                    │
    │             │ MacAddress:02:42:ac:1f:00:05 │
    │             │                              │
    │             │                              │
    │ Ports       │ LocalPort:443/tcp            │
    │             │ GuestPort:30000              │
    │             │                              │
    │ Quotagroups │ appliance_data               │
    │             │ sb_user00                    │
    │             │                              │
    │ State       │ running                      │
    ╰─────────────┴──────────────────────────────╯
    ```


Your secure build server is now up and running! :fire:

It is available at the IP Address of the Hyper Protect Virtual Server LPAR and port (GuestPort) specified.

You will use this secure build server to securely build your application in the next section. 

!!! note
    You can assign IP addresses and hostnames for containers as necessary for your purposes but using the docker network and host ports is a nice way to quickly get running without having to use up IP addresses on your network.

