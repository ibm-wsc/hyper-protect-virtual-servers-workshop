# Securely Build your Application

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Create repository registration GPG signing key
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

3. Create directory to store Docker repository registration signing key material

    ``` bash
    mkdir -p "${SB_DIR}/registration_keys"
    ```

4. Create Key Definition

    ``` bash
    cat > "${SB_DIR}/registration_keys/${keyName}_definition_keys" <<EOF
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
    gpg --armor --batch \
    --generate-key "${SB_DIR}/registration_keys/${keyName}_definition_keys"
    ```

    ???+ example "Example Output"
        ```bash
        gpg: directory '/home/multiarch-lab/.gnupg' created
        gpg: keybox '/home/multiarch-lab/.gnupg/pubring.kbx' created
        gpg: Generating registration definition key
        gpg: /home/multiarch-lab/.gnupg/trustdb.gpg: trustdb created
        gpg: key 7E05CE05DFEBA2BC marked as ultimately trusted
        gpg: directory '/home/multiarch-lab/.gnupg/openpgp-revocs.d' created
        gpg: revocation certificate stored as '/home/multiarch-lab/.gnupg/openpgp-revocs.d/27FB55DC5F7FDF0598C9B1007E05CE05DFEBA2BC.rev'
        gpg: done
        ```

6. Export Private key

    ``` bash
    gpg --armor --pinentry-mode=loopback --passphrase  "${passphrase}" \
    --export-secret-keys "${keyName}" > "${SB_DIR}/registration_keys/${keyName}.private"
    ```

7. Export Public key

    ``` bash
    gpg --armor --export ${keyName} > "${SB_DIR}/registration_keys/${keyName}.pub"
    ```

8. List newly generated key files 

    ``` bash
     ls ${SB_DIR}/registration_keys/
    ``` 

    ???+ example "Example Output"
        ``` bash
        secure_bitcoin_key_definition_keys  secure_bitcoin_key.pub secure_bitcoin_key.private
        ```

## Set Build Configuration

1. Set Secure Build Lab IP Address

    ``` bash
    export SB_IP=192.168.22.120
    ```

2. Save Secure Build Lab IP Address for later use

    ``` bash
    echo "export SB_IP='${SB_IP}'" >> "${HOME}/.bashrc"
    ```

3. Set Secure Build Server Port

    ``` bash
    export SB_PORT=213${HPVS_NUMBER}
    ```

4. Set Secure Build GitHub repository

    ``` bash
    export GH_REPO="git@github.com:IBM/secure-bitcoin-wallet.git"
    ```

5. Set Docker Image Name

    ``` bash
    export IMAGE_NAME="hpvs_bc"
    ```

6. Set repository registration name

    ``` bash
    export REPO_ID="${IMAGE_NAME}_${HPVS_NUMBER}"
    ```

7. Save repository registration name for later use

    ``` bash
    echo "export REPO_ID='${REPO_ID}'" >> "${HOME}/.bashrc"
    ```

8. Create config file

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
            ssh_private_key_path: '${GITHUB_SSH_KEY}'
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
            private_key_path: '${SB_DIR}/registration_keys/${keyName}.private'
            public_key_path: '${SB_DIR}/registration_keys/${keyName}.pub'
    EOF
    ```

## Build Application

1. Launch secure build with a timeout of 20 minutes (1200 seconds) to complete using the above generated configuration file.

    ``` bash
    echo "${passphrase}" | hpvs sb init \
    --config "${SB_DIR}/sb_config.yaml" \
    --out "${SB_DIR}/yaml.${REPO_ID}.enc" --timeout 1200 --build
    ```

2. You can look at the logs if desired in another terminal window while the secure build is running (don't intterrupt the current terminal window which is waiting for the secure build) 

    ``` bash
    hpvs sb log --config "${SB_DIR}/sb_config.yaml"
    ```

    ???+ exmaple "Example Truncated Output"

        ```
        2020-06-23 05:25:42,453  root       INFO    starting a build
        2020-06-23 05:25:42,454  root       INFO    cleaning up the local github repo and the github access credential
        2020-06-23 05:25:42,454  root       INFO    github_dir=secure-bitcoin-wallet
        2020-06-23 05:25:42,454  root       INFO    cloning a github repo
        2020-06-23 05:25:42,454  root       INFO    github_host=github.com
        2020-06-23 05:25:42,454  root       INFO    cmd=ssh-keyscan github.com
        2020-06-23 05:25:47,659  root       INFO    # github.com:22 SSH-2.0-babeld-7c96ae41
        # github.com:22 SSH-2.0-babeld-b6072416
        # github.com:22 SSH-2.0-babeld-b6072416
        github.com ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pL
        .....
        ```

3. You can also look at the secure build status in another window. This is useful if you accidentally interrupted the secure build command or if it times out due to the timeout not being long enough.

    ``` bash
    hpvs sb status --config "${SB_DIR}/sb_config.yaml"
    ```

    ???+ example "Example Output"

        ``` bash
        +---------------------+---------------+
        | manifest_public_key |               |
        | root_ssh_enabled    | false         |
        | status              | github cloned |
        | build_name          |               |
        | image_tag           |               |
        | manifest_key_gen    |               |
        +---------------------+---------------+
        ```

4. You can continue to check the logs to monitor the progress of your secure build with the previous logs command

    ``` bash
    hpvs sb log --config "${SB_DIR}/sb_config.yaml"
    ```

5. When the seecure build completes successfully it will have the following logs

    ???+ example "Example Output"

        ``` bash
        2020-06-23 08:39:46,463  root       INFO    run: latest: digest: sha256:ffbaf396807659d5a4d66fe54c0ebf382a9f170c4eaf187b9b4c8582ca8fdec2 size: 5133
        2020-06-23 08:39:46,463  root       INFO    run: Signing and pushing trust metadata
        2020-06-23 08:39:48,299  root       INFO    run: Successfully signed docker.io/gmoney23/hpvs_bc:latest
        2020-06-23 08:39:48,300  root       INFO    run: return code = 0
        2020-06-23 08:39:48,300  root       INFO    extracting an image keyid and key
        2020-06-23 08:39:48,301  root       INFO    keyid=0cc264a565c452ea6aca776b2787be54e94b905113e77edae00d5ec267a07ffc
        2020-06-23 08:39:48,301  root       INFO    publickey=LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJmVENDQVNTZ0F3SUJBZ0lSQUt3cTlRWEZhMzRYMXdIRmZST2NJZXN3Q2dZSUtvWkl6ajBFQXdJd0pURWoKTUNFR0ExVUVBd3dhWkc5amEyVnlMbWx2TDJkdGIyNWxlVEl6TDJod2RuTmZZbU13SGhjTk1qQXdOakl6TURnegpPVFF4V2hjTk16QXdOakl4TURnek9UUXhXakFsTVNNd0lRWURWUVFEREJwa2IyTnJaWEl1YVc4dloyMXZibVY1Ck1qTXZhSEIyYzE5aVl6QlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJKU3ZPdWlSaHJNcjJmQVYKcmZLcHZncVRYNXZwSFlodnEvZXc1SFRuMzRMcnBrQ0xJMjBkdmxjcTUyc1ZvQjVxYkpzeEdTbkphOU5sM0tYYQpZUlRjQ2Z1ak5UQXpNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUtCZ2dyQmdFRkJRY0RBekFNCkJnTlZIUk1CQWY4RUFqQUFNQW9HQ0NxR1NNNDlCQU1DQTBjQU1FUUNJRlhFWE9iZTdHR1NsSjEzTzZicTR6T0IKamhoMlZSbmRYOVJYMytrSFpnVVVBaUJBWXAyNTkwTkpVelJIK1lhR2JzQ1hMRmxOUWRzT2ltWW9NMzlqR0IwRAp5dz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
        2020-06-23 08:39:48,301  root       INFO    generating a config file
        2020-06-23 08:39:48,301  root       INFO    create_and_push: SoftCrypto
        2020-06-23 08:39:48,819  root       INFO    digest=1f0b1f65b1462f16930b200c77ce5fbe654e1d624405bc243d713de1aade36b7
        2020-06-23 08:39:48,819  root       INFO    block_size=64
        2020-06-23 08:39:48,821  root       INFO    digest=1f0b1f65b1462f16930b200c77ce5fbe654e1d624405bc243d713de1aade36b7
        2020-06-23 08:39:48,824  root       INFO    signature=114157a8bd98d7f5a5c2ca33f81496563af7f18d23123fc35c8aa84a5bfadc709ecb17e5e79d42d6bd6ac8a815053d9cfd039b7f01ab84b9a75a23e2917b0bc4f0c1b5bd5664dfebd573c2355c34115762b8fea56285d65cd8db4877c9b95ab3149b65d14ce1a23b1065a34c2d4ba9a1526286a03d87d307a5972cf1425e586c9d213b34fe53407c79a527e78779b7a70b426516db35f22a09329dfac76a8505613249ad2b46070ad7d932a8c4bbe1981d0370150528cb9e6f5a426be6734405435393a8d6d8e145418398f85bc28be6c332d2fa2e84f5465618051b110a3efcd25600dc95ee0d7f8bc0d36b8ddaa9ce1c4be78f38928d9213e5171078e22930
        2020-06-23 08:39:48,824  root       INFO    verify=OK
        2020-06-23 08:39:49,214  root       WARNING undefined MANIFEST_BUCKET_NAME
        2020-06-23 08:39:49,214  root       WARNING skipping transferring a manifest to COS
        2020-06-23 08:39:49,214  root       INFO    cleaning up the build environment
        2020-06-23 08:39:49,214  root       INFO    github_dir=secure-bitcoin-wallet
        2020-06-23 08:39:49,217  root       INFO    completed a build
        ```

6. When the secure build completes successfully you can check the status again to see a completed status

    ``` bash
    hpvs sb status --config "${SB_DIR}/sb_config.yaml"
    ```

    ???+ example "Example Output"

        ``` bash
        +---------------------+------------------------------------------------------------------------------------------+
        | build_name          | docker.io.gmoney23.hpvs_bc.latest-b3416d8.2020-06-23_08-39-48.301849                     |
        | image_tag           | latest-b3416d8                                                                           |
        | manifest_key_gen    | soft_crypto                                                                              |
        | manifest_public_key | manifest.docker.io.gmoney23.hpvs_bc.latest-b3416d8.2020-06-23_08-39-48.301849-public.pem |
        | root_ssh_enabled    | false                                                                                    |
        | status              | success                                                                                  |
        +---------------------+------------------------------------------------------------------------------------------+
        ```

## Verify your application

1. Create a directory for your manifest file information and change into it

    ``` bash
    mkdir -p "${SB_DIR}/manifest" && cd "${SB_DIR}/manifest"
    ```

2. Save the build name from the status command as a variable

    ``` bash
    export BUILD_NAME="$(hpvs sb status --config "${SB_DIR}/sb_config.yaml" \
    | grep build_name | egrep -ow 'docker.*[[:digit:]]')" && echo "${BUILD_NAME}"
    ```

    ???+ example "Example Output"

        ``` bash
        docker.io.gmoney23.hpvs_bc.latest-b3416d8.2020-06-23_08-39-48.301849
        ```

3. Get your application manifest

    ``` bash
    hpvs sb manifest --config "${SB_DIR}/sb_config.yaml" --name "${BUILD_NAME}"
    ```

4. Get your manifest public verification key

    ``` bash 
    hpvs sb pubkey --config "${SB_DIR}/sb_config.yaml" --name "${BUILD_NAME}"
    ```

5. Check that your application manifest and sigining key were retrieved

    ``` bash
    ls
    ```

    ???+ example "Example Output"

        ```bash 
        docker.io.gmoney23.hpvs_bc.latest-b3416d8.2020-06-23_08-39-48.301849-public.pem
        manifest.docker.io.gmoney23.hpvs_bc.latest-b3416d8.2020-06-23_08-39-48.301849.sig.tbz
        ```

6. Verify your manifest file with the public key
 ... continued 

7. Further inspect manifest files .... (I will fill this out)

8. What does this give me?

    This gives you awesomeness ....  (Obviously I will fill this out)

## Summary

Congratulations!!! :tada: 

You have securely built your application and are now ready to deploy it into a Hyper Protect Virtual Server in the next section.