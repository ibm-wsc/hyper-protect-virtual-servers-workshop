# Create your HPVS Secure Build Server

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Set your provided number and save it for later use

You will be assigned a number for the lab so as not to interfere with other users.

!!! note
    [This table](../assignment.md){target=_blank} tells you which number you are assigned.

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

## Create Certificate and Key for server-side certificate checking in TLS

!!! info

    In this process you will create the `Secure Build Server Client Certificate and Key` referenced in the [key table](overview.md#fnref:2). You will keep the private key on your Linux vm and pass the corresponding certificate to the secure build server you are deploying in this section. Then, whenever you interact with the secure build server to build your applications and get your manifest files it will check that your private key matches the public key of the certificate (i.e., server-side checking in mutual TLS) so that only you (or others / CI tools with the necessary private key) can access the server to perform secure build operations (`hpvs sb` commands) on it (i.e., build, log, status, manifest, etc.).

1. Generate rand file

    ``` bash
    openssl rand -out "${HOME}/.rnd" -hex 256
    ```

2. Make `SB_DIR/sbs_keys` directory to store secure build server keys and switch to it.

    ``` bash
    mkdir -p "${SB_DIR}/sbs_keys" && cd ${SB_DIR}/sbs_keys
    ```

3. Create a private key that will be used by the Secure Build Server certification authority

    ``` bash
    openssl genrsa -out ca.key 2048
    ```

4. Create the Secure Build Server certification authority

    ``` bash
    openssl req -new -x509 -key ca.key -days 730 -out ca.pem
    ```

5. Set environment variables that will be referenced in a configuration file used by subsequent openssl commands in the lab

    ``` bash
    export COMMON_NAME=server
    export PATHLEN=CA:true
    export SUBJECT_ALT_NAME=DNS:192.168.22.79:${SB_PORT},IP:192.168.22.79
    ```

6. Create the configuration file which will be used by subsequent openssl commands in the lab

    ``` yaml
    cat << EOF > ${SB_DIR}/sbs_keys/openssl.cnf
    # OpenSSL configuration file.
    #

    # Establish working directory.

    dir   = .

    [ ca ]
    default_ca  = CA_default

    [ CA_default ]
    serial   = \$dir/serial
    #database  = \${ENV::DIR}/index.txt
    #new_certs_dir  = \$dir/newcerts
    #private_key       = \$dir/ca.key
    #certificate       = \$dir/ca.cer
    default_days  = 730
    default_md  = sha256
    preserve  = no
    email_in_dn  = no
    nameopt   = default_ca
    certopt   = default_ca
    default_crl_days = 45
    policy   = policy_match

    [ policy_match ]
    countryName  = match
    stateOrProvinceName = optional
    organizationName = match
    organizationalUnitName = optional
    commonName  = supplied
    emailAddress  = optional
   
    [ req ]
    default_md  = sha256
    distinguished_name = req_distinguished_name
    prompt             = yes
   
    [ req_distinguished_name ]
    #countryName = Country
    #countryName_default = US
    #countryName_min = 2
    #countryName_max = 2
    #localityName = Locality
    #localityName_default = Los Angeles
    #organizationName = Organization
    #organizationName_default = IBM
    #commonName = Common Name
    #commonName_max = 64
   
    C  = US
    ST = California
    L  = Los Angeles
    O  = IBM
    CN = \${ENV::COMMON_NAME}

    [ certauth ]
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid:always,issuer:always
    keyUsage = digitalSignature, keyEncipherment, dataEncipherment, keyCertSign, cRLSign
    keyUsage = digitalSignature, keyEncipherment, dataEncipherment, keyCertSign, cRLSign
    basicConstraints = \${ENV::PATHLEN}
    #crlDistributionPoints = @crl

    [ server ]
    basicConstraints = CA:FALSE
    keyUsage = digitalSignature, keyEncipherment, dataEncipherment
    extendedKeyUsage = serverAuth
    nsCertType = server
    crlDistributionPoints = @crl
    subjectAltName = \${ENV::SUBJECT_ALT_NAME}
   
    [ client ]
    basicConstraints = CA:FALSE
    keyUsage = digitalSignature, keyEncipherment, dataEncipherment
    extendedKeyUsage = clientAuth,msSmartcardLogin
    nsCertType = client
    crlDistributionPoints = @crl
    authorityInfoAccess = @ocsp_section
    subjectAltName = @alt_names

    [ selfSignedServer ]
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid:always,issuer:always
    keyUsage = digitalSignature, keyEncipherment, dataEncipherment
    basicConstraints = CA:FALSE
    subjectAltName = \${ENV::SUBJECT_ALT_NAME}
    extendedKeyUsage = serverAuth
   
    [ selfSignedClient ]
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid:always,issuer:always
    keyUsage = digitalSignature, keyEncipherment, dataEncipherment
    basicConstraints = CA:FALSE
    subjectAltName = @alt_names
    extendedKeyUsage = clientAuth

    [ server_client ]
    subjectKeyIdentifier = hash
    keyUsage = digitalSignature, keyEncipherment, dataEncipherment
    basicConstraints = CA:FALSE
    subjectAltName = \${ENV::SUBJECT_ALT_NAME}
    crlDistributionPoints = @crl
    extendedKeyUsage = serverAuth,clientAuth

    [ v3_intermediate_ca ]
    # Extensions for a typical intermediate CA 
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid:always,issuer
    basicConstraints = critical, \${ENV::PATHLEN}
    keyUsage = critical, digitalSignature, cRLSign, keyCertSign
    crlDistributionPoints = @crl
    authorityInfoAccess = @ocsp_section

    [ crl ]
    URI=http://localhost/ca.crl
   
    [ ocsp_section ]
    OCSP;URI.0 = http://localhost:2560/ocsp

    [ ocsp ]
    # Extension for OCSP signing certificates 
    basicConstraints = CA:FALSE
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid,issuer
    keyUsage = critical, digitalSignature
    extendedKeyUsage = critical, OCSPSigning

    [alt_names]
    # email= \${ENV::SUBJECT_ALT_NAME}
    otherName=msUPN;UTF8:\${ENV::SUBJECT_ALT_NAME}
   
    [v3_conf]
    keyUsage = digitalSignature, keyEncipherment, dataEncipherment, keyCertSign, cRLSign
    basicConstraints = CA:FALSE
    EOF
    ```
   
7. Create a private key that will be used by the Secure Build Server

    ``` bash
    openssl genrsa -out server-key.pem 2048
    ``` 

8. Create the Secure Build Server certificate signing request

    ``` bash
    openssl req -new -key server-key.pem -out server.csr   
    ```

9. Create the Secure Build Server certificate

    ``` bash
    openssl x509 -sha256 -req -in server.csr \
    -CA ca.pem \
    -CAkey ca.key \
    -set_serial 8086 -extfile openssl.cnf \
    -extensions server -days 730 -outform PEM \
    -out server.pem
    ```

10. Create a private key for use by the client certificate

    ``` bash
    openssl genrsa -out client-key.pem 2048
    ```

11. Create the client certificate signing request

    ``` bash
    openssl req -new \
    -key client-key.pem \
    -out client.csr
    ```

12. Create the client certificate

    ``` bash
    openssl x509 -req -days 730 \
    -in client.csr \
    -CA ca.pem \
    -CAcreateserial \
    -CAkey ca.key \
    -out client_cert.pem
    ```

13. Create environment variables

    ``` bash
    export ca_cert=$(echo $(cat ca.pem | base64) | tr -d ' ')
    export client_cert=$(echo $(cat client_cert.pem | base64) | tr -d ' ')
    export server_cert=$(echo $(cat server.pem | base64) | tr -d ' ')
    export server_key=$(echo $(cat server-key.pem | base64) | tr -d ' ')
    ```

14. Switch back to main lab working directory

   ``` bash
   cd ${SB_DIR}
   ```

## Create Quotagroup with storage for secure build server

``` bash
hpvs quotagroup create --name "sb_user${HPVS_NUMBER}" --size=40GB
```

???+ example "Example Output"

    ``` bash
    +-------------+--------------+
    | PROPERTIES  | VALUES       |
    +-------------+--------------+
    | Name        | sb_user00    |
    | Filesystem  | btrfs        |
    | Passthrough | false        |
    | PoolID      | lv_data_pool |
    | Size        | 40 GB        |
    | Containers  |              |
    | Available   | 39 GB        |
    +-------------+--------------+
    ```

## Create Secure Build Server

``` bash
hpvs vs create --name sbserver_${HPVS_NUMBER} --repo SecureDockerBuild \
--tag 1.2.7.5 --cpu 2 --ram 2048 \
--quotagroup "{quotagroup = sb_user${HPVS_NUMBER}, mountid = new, mount = /newroot, filesystem = ext4, size = 4GB, reset_root=true}" \
--quotagroup "{quotagroup = sb_user${HPVS_NUMBER}, mountid = data, mount = /data, filesystem = ext4, size = 4GB}" \
--quotagroup "{quotagroup = sb_user${HPVS_NUMBER}, mountid = docker, mount = /docker, filesystem = ext4, size = 16GB}" \
--env={EX_VOLUMES="/docker,/data",ROOTFS_LOCK=y,CLIENT_CRT=$client_cert,CLIENT_CA=$ca_cert,SERVER_CRT=$server_cert,SERVER_KEY=$server_key,RUNQ_ROOTDISK=new} \
--ports "{containerport = 443, protocol = tcp, hostport = ${SB_PORT}}"
```


???+ example "Example Output"

    ``` bash
    +-------------+------------------------------+
    | PROPERTIES  | VALUES                       |
    +-------------+------------------------------+
    | Name        | sbserver_00                  |
    | CPU         | 2                            |
    | Memory      | 2048                         |
    | State       | running                      |
    | Status      | Up Less than a second        |
    | Networks    | Gateway:172.31.0.1           |
    |             | IPAddress:172.31.0.2         |
    |             | MacAddress:02:42:ac:1f:00:02 |
    |             | Network:bridge               |
    |             | Subnet:16                    |
    | Ports       | GuestPort:30000              |
    |             | LocalPort:443/tcp            |
    | Quotagroups | [sb_user00]                  |
    +-------------+------------------------------+
    ```

We can see the quotagroup is now being used with

``` bash
hpvs quotagroup show --name "sb_user${HPVS_NUMBER}"
```

???+ example "Example Output"

    ``` bash
    +-------------+-----------------------------+
    | PROPERTIES  | VALUES                      |
    +-------------+-----------------------------+
    | Name        | sb_user00                   |
    | Filesystem  | btrfs                       |
    | Passthrough | false                       |
    | PoolID      | lv_data_pool                |
    | Size        | 40 GB                       |
    | Containers  | container_name:sbserver_00  |
    |             | mount_ids:[new data docker] |
    | Available   | 11 GB                       |
    +-------------+-----------------------------+
    ```

The show output for the Hyper Protect Virtual Server was shown when it was deployed but we can bring it back up with

``` bash
hpvs vs show --name "sbserver_${HPVS_NUMBER}"
```

???+ example "Example Output"

    ``` bash
    +-------------+------------------------------+
    | PROPERTIES  | VALUES                       |
    +-------------+------------------------------+
    | Name        | sbserver_00                  |
    | CPU         | 2                            |
    | Memory      | 2048                         |
    | State       | running                      |
    | Status      | Up 2 minutes                 |
    | Networks    | Gateway:172.31.0.1           |
    |             | IPAddress:172.31.0.2         |
    |             | MacAddress:02:42:ac:1f:00:02 |
    |             | Network:bridge               |
    |             | Subnet:16                    |
    | Ports       | GuestPort:30000              |
    |             | LocalPort:443/tcp            |
    | Quotagroups | [sb_user00]                  |
    +-------------+------------------------------+
    ```

Your secure build server is now up and running! :fire:

It is available at the IP Address of the Hyper Protect Virtual Server LPAR and port (GuestPort) specified.

You will use this secure build server to securely build your application in the next section.

!!! note
    You can assign IP addresses and hostnames for containers as necessary for your purposes but using the docker network and host ports is a nice way to quickly get running without having to use up IP addresses on your network.
