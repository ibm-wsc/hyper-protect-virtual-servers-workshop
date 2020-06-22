# Securely Build your Application

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Create Trust GPG signing key for your Docker Repository
1. Set key name

    === "Command Syntax"

        ``` bash
        export keyName="your_keyname"
        ```

    === "Example Command"

        ``` bash
        export keyName="secure_bitcoin_key"
        ```

2. Set key passphrase

    === "Command Syntax"

        ``` bash
        export passphrase="your_passphrase"
        ```
    
    === "Example Command"

        ``` bash
        export passphrase="most_secure_pw_i_could_think_of"
        ```

3. Create directory to store Docker Content Trust repository signing key material

    ``` bash
    mkdir -p "${SB_DIR}/dctrust_keys"
    ```

4. Create Key Definition

    ``` bash
    cat > "${SB_DIR}/dctrust_keys/${keyName}_definition_keys" <<EOF
        %echo Generating registration definition key
        Key-Type: RSA
        Key-Length: 4096
        Subkey-Type: RSA
        Subkey-Length: 4096
        Name-Real: ${keyName}
        Expire-Date: 0
        Passphrase: ${passphrase}
        # Do a commit here, so that we can later print "done" :-)
        %commit
        %echo done
    EOF
    ```

5. Generate Key pair

    ``` bash
    gpg --armor --batch --generate-key "${SB_DIR}/dctrust_keys/${keyName}_definition_keys"
    ```

    ???+ example "Example Output"
        ```bash

        ```

6. Export Private key

    ``` bash
    gpg --armor --pinentry-mode=loopback --passphrase  "${passphrase}" \
    --export-secret-keys "${keyName}" > "${SB_DIR}/dctrust_keys/${keyName}.private"
    ```

    ???+ example "Example Output"
        ``` bash
        ```

7. Export Public key

    ``` bash
    gpg --armor --export ${keyName} > "${SB_DIR}/dctrust_keys/${keyName}.pub"
    ```

    ???+ example "Example Ouput"

        ``` bash
        ```

8. List newly geneerated key files 

    ``` bash
     ls ${SB_DIR}/dctrust_keys/
    ``` 

    ???+ example "Example Output"
        ``` bash
        ```

## Set Build Configuration

1. Set Secure Build Server IP ADDRESS

    ``` bash
    export SB_IP=192.168.22.79
    ```

2. Set Secure Build Server Port

    ``` bash
    export SB_PORT=213${HPVS_NUMBER}
    ```

3. Set Secure Build GitHub repository

    ``` bash
    export GH_REPO="https://github.com/IBM/secure-bitcoin-wallet.git"
    ```

4. Set Docker Image Name

    ``` bash
    export IMAGE_NAME="hpvs_bc"
    ```

5. Set repository registration name

    ``` bash
    export REPO_ID="${REGISTRY_NAME}-${HPVS_NUMBER}"
    ```

6. Save repository registration name for later use

    ``` bash
    echo "export REPO_ID='${REPO_ID}'" >> "${HOME}/.bashrc"
    ```

5. Create config file

    ```
    cat > "${SB_DIR}/sb_config.yaml" <<EOF
        secure_build_workers:
        sbs:
            url: 'https://${SB_IP}'
            port: '${SB_PORT}'
            cert_path: '${SB_DIR}/sbs_keys/sbs.cert'
            key_path: '${SB_DIR}/sbs_keys/sbs.key'
        regfile:
            id: '${REPO_ID}'
        github:
            url: '${GH_REPO}'
            branch: 'master'
            recurse_submodules: 'False'
            dockerfile_path: './Dockerfile'
            docker_build_path: './'
        docker:
            push_server: '${REGISTRY_NAME}'
            #base_server: '${REGISTRY_NAME}'
            pull_server: '${REGISTRY_NAME}'
            repo: '${DOCKER_USERNAME}/${IMAGE_NAME}'
            image_tag_prefix: 'latest'
            content_trust_base: 'False'
        env:
            whitelist: []
        build:
            args: []
        signing_key:
            private_key_path: '${SB_DIR}/dctrust_keys/${keyName}.private'
            public_key_path: '${SB_DIR}/dctrust_keys/${keyName}.pub'
    EOF
    ```

## Build Application

1. Launch secure build with a timeout of 15 minutes (900 seconds) to complete using the above generated configuration file.

    ``` bash
    hpvs sb init --config "${SB_DIR}/sb_config.yaml" --out "${SB_DIR}/yaml.${REPO_ID}.enc" --timeout 900 --build
    ```

2. You can look at the logs if desired in another terminal window while the secure build is running (don't intterrupt the current terminal window which is waiting for the secure build) 

    ``` bash
    hpvs sb log --config "${SB_DIR}/sb_config.yaml"
    ```

3. You can also look at the secure build status in another window. This is useful if you accidentally interrupted the secure build command or if it times out due to the timeout not being long enough.

    ``` bash
    hpvs sb status --config "${SB_DIR}/sb_config.yaml"
    ```

4. When the seecure build completes successfully it will have the following status

    ??? exmaple "Example Output"
    ```
    ```

## Verify your application

1. Get your application manifest

2. Verify you application manifest

3. What does this give me?

    This gives you awesomeness .... 

## Summary

Congratulations!!! :tada: 

You have securely built your application and are now ready to deploy it into a Hyper Protect Virtual Server in the next section.