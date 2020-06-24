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

3. Set Secure Build GitHub repository

    ``` bash
    export GH_REPO="git@github.com:IBM/secure-bitcoin-wallet.git"
    ```

4. Set Docker Image Name

    ``` bash
    export IMAGE_NAME="hpvs_bc"
    ```

5. Set repository registration name

    ``` bash
    export REPO_ID="${IMAGE_NAME}_${HPVS_NUMBER}"
    ```

6. Save repository registration name for later use

    ``` bash
    echo "export REPO_ID='${REPO_ID}'" >> "${HOME}/.bashrc"
    ```

7. Create config file

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

    !!! Tip
        The following build will take anywhere from **15-20 minutes** to complete. While this is ongoing, you should open a new tab in your terminal to check the automatically updating logs and build status (steps for doing this are detailed in the next few steps).

    !!! note
        The secure build is asynchronous so if the command gets interrupted here don't worry! :grin:

        That just means you will need to retrieve the registration file later since the cli couldn't grab it after the build. 
        
        (We will do this anyway in `Step 7` to cover our bases)

    ???+ example "Example Output after running 15-20 minutes to completion"
    
        ``` bash
        > --config "${SB_DIR}/sb_config.yaml" \
        > --out "${SB_DIR}/yaml.${REPO_ID}.enc" --timeout 1200 --build
        Enter Sigining Private key passphrase: 
        {"status":"OK"}

        +--------+-------------------------+
        | status | OK: async build started |
        +--------+-------------------------+
        ##############################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################################
        +---------------------+--------------------------------------------------------------------------------------------+
        | root_ssh_enabled    | false                                                                                      |
        | status              | success                                                                                    |
        | build_name          | docker.io.gmoney23.hpvs_bc_a.latest-b3416d8.2020-06-23_22-12-54.183641                     |
        | image_tag           | latest-b3416d8                                                                             |
        | manifest_key_gen    | soft_crypto                                                                                |
        | manifest_public_key | manifest.docker.io.gmoney23.hpvs_bc_a.latest-b3416d8.2020-06-23_22-12-54.183641-public.pem |
        +---------------------+--------------------------------------------------------------------------------------------+
        ```

2. Please look at the logs in another terminal window while the secure build is running (don't interrupt the current terminal window which is waiting for the secure build) 

    ``` bash
    hpvs sb log --config "${SB_DIR}/sb_config.yaml"
    ```

    ???+ example "Example Truncated Output"

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

3. You can also look at the secure build status in another window. This can be useful if you accidentally interrupted the secure build command or it times out due to the timeout not being long enough. It will also fill out the additional fields that are initially blank when the build completes (such as `build_name` and `manifest_public_key`, etc.).

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

5. When the secure build successfully completes, it will have ending logs similar to the following:

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

6. When the secure build successfully completes, you can check the status again to see a `completed` status.

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

7. From the original terminal window that you ran `hpvs sb init`, output the repository registration file (just in case the `sb init` command got interrupted before completing)

    ``` bash
    echo "${passphrase}" | hpvs sb regfile \
    --config "${SB_DIR}/sb_config.yaml" \
    --out "${SB_DIR}/yaml.${REPO_ID}.enc"
    ```

    ???+ example "Example Output"

    ``` bash
    Enter Sigining Private key passphrase:
    ```

    !!! Tip 
        The `echo` command takes care of the passphrase so you don't need to enter it manually.
    
    !!! note
        If you run this command from a terminal window where you did not set the `$passphrase` environment variable you will get an error saying `openpgp: invalid data: private key checksum failure`. You can also check to make sure you are in the right terminal window by running `echo $passphrase` and the output should be the passphrase you set in the beginning of the "Create repository registration GPG signing key" section.

    !!! note
        This registration file should be created at the end of the `sb init` command. However, given that the build is asynchronous it will complete even if you accidentally interrupt it. However, if you do interrupt it then the repository registration file won't be created. We grab it again here just to cover our bases if that command got interrupted. If it didn't get interrupted you could skip this step. However, it doesn't hurt to grab it again here as it will just retrieve it again. 

## Verify your application

1. Create directories for your manifest file information and change into your new `manifest` directory.

    ``` bash
    mkdir -p "${SB_DIR}/manifest/manifest_files" && cd "${SB_DIR}/manifest"
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

5. Check that your application manifest and signing key were retrieved

    ``` bash
    ls
    ```

    ???+ example "Example Output"

        ```bash 
        docker.io.gmoney23.hpvs_bc_a.latest-b3416d8.2020-06-23_22-12-54.183641-public.pem
        manifest.docker.io.gmoney23.hpvs_bc_a.latest-b3416d8.2020-06-23_22-12-54.183641.sig.tbz
        manifest_files
        ```

6. Set `MANIFEST` to point to your manifest files.

    ``` bash
    export MANIFEST="${SB_DIR}/manifest/manifest.${BUILD_NAME}"
    ```

7. Set `MAN_KEY` to point to your manifest public key.

    ``` bash
    export MAN_KEY="${SB_DIR}/manifest/${MAN_KEY}-public.pem"
    ```

8. Untar and unzip the manifest `.sig.tbz` file to reveal the `.sig` and `.tbz` files (and remove the original `.sig.tbz`)

    ``` bash
    tar -xjvf "${MANIFEST}.sig.tbz" && rm "${MANIFEST}.sig.tbz"
    ```

    ???+ example "Example Output"

        ```
        manifest.docker.io.gmoney23.hpvs_bc_a.latest-b3416d8.2020-06-23_22-12-54.183641.tbz
        manifest.docker.io.gmoney23.hpvs_bc_a.latest-b3416d8.2020-06-23_22-12-54.183641.sig
        ```

    !!! note

        The `.sig.tbz` was a tarball compressed using bzip2 compression of both a nested `.tbz` file (containing the manifest files) and a `.sig` file containing a signature of the nested `.tbz` to verify it with the public key retrieved using the `hpvs sb pubkey` command above saved in the file referenced by `MAN_KEY`.

9.  Untar and unzip the manifest `.tbz` file into the `manifest_files` directory.

    ``` bash
    tar -xjf "${MANIFEST}.tbz" -C "${SB_DIR}/manifest/manifest_files"
    ```

10. Change into the `manifest_files` directory with your manifest files and view the files in the directory.

    ``` bash
    cd "${SB_DIR}/manifest/manifest_files" && ls
    ```

    ???+ example "Example Output"

        ``` bash
        data  git  root_ssh
        ```
    
    !!! info "Manifest explanation"
        This `manifest` package contains three directories as shown above. The `data` directory contains the `build.json` (containing the build status of the directory updated after the image was pushed to its Docker Registry) and `build.log` (containing the logs from the secure build). The `git` directory contains the source code used for the build (cloned from git). Finally, the `root_ssh` directory contains any `ssh` material provided (if the user chose to use ssh) which is empty for us because we didn't enable ssh and provide ssh keys. This way no one can ssh into our secure build container. 

11. What does this give me?

    This gives you a way to see the collection of files used for your build at the time of the build and signed by the manifest private key which is secured in your secure build container. By retrieving the public key and verifying the signature of the package we *and auditors* can verify what was used for our secure image build. (Since the private key was generated in the secure build server we can trust it) This gives us verification of what was used in the build as well as the verification of the images themselves we get from [Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/){target=_blank}.

    !!! note
        The secure build server also generates the keys for Docker Content Trust and stores them safely to provide a secure root of trust.

        _Remember the secure build server is running as a Hyper Protect Virtual Server and thus inherits all of the security features and assurances of Hyper Protect Virtual Servers._

## Summary

Congratulations!!! :tada: 

You have securely built your application and are ready to deploy it into a Hyper Protect Virtual Server in the next section.
