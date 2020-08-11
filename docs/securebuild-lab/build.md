# Securely Build your Application

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Create repository registration GPG signing key
1. Set key name

    ``` bash
    export keyName="secure_bitcoin_key${RANDOM}"
    ```

    !!! info
        `${RANDOM}` stores a pseudo-random number to use since GPG keys on a system must each have a unique key name (uid). This will make sure users can safely re-run commands for multiple runs if they so choose.

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
    export SB_IP=192.168.22.80
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
        #env:
        # You would enter environment variables that you want to use in your application container in the whitelist array.
        #    whitelist: []
        #build:
        # You would enter any desired docker build arguments in the args array.
        #    args: []
        signing_key:
            private_key_path: '${SB_DIR}/registration_keys/${keyName}.private'
            public_key_path: '${SB_DIR}/registration_keys/${keyName}.pub'
    EOF
    ```
8. Continue to the [Build Application section](#build-application) if you are launching secure build for the first time or the [Troubleshooting Secure Build section](#troubleshooting-secure-build) if things didn't go as planned your first go-round and you are trying to get back on the right track.

## Build Application

1. Initialize Secure Build Hyper Protect Virtual server with configuration file generated in the [Set Build Configuration section](#set-build-configuration)

    ``` bash
    hpvs sb init --config "${SB_DIR}/sb_config.yaml"
    ```

    !!! example "Example Output"

        ``` bash
        {"status":"OK"}
        ```

2. Launch secure build with a timeout of 20 minutes to complete using the configuration file generated in the [Set Build Configuration section](#set-build-configuration)

    ``` bash
    hpvs sb build \
    --timeout 20 \
    --config "${SB_DIR}/sb_config.yaml"
    ```

    !!! Tip
        The following build will take anywhere from **15-20 minutes** to complete. While this is ongoing, you should open a new tab in your terminal to check the automatically updating logs and build status (steps for doing this are detailed in the next few steps). If this command times out please check the status in the `step 3` to make sure nothing went wrong. It might just be that your build took to long and everything will be ok. If something did go wrong, visit the [Troubleshooting Secure Build section](#troubleshooting-secure-build)

    !!! note
        The secure build is asynchronous so if the command gets interrupted here don't worry! :grin:

    !!! warning
        If the secure build command gets an error this command will keep on waiting until eventually timing out. If you see an error in the `status` you can interrupt this command with `ctrl+c` instead of waiting for it to time out. 

    ???+ example "Example Output after running 15-20 minutes to completion"
    
        ``` bash 
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

    !!! error "If you see an error in your status"
        If you see that your build ran into an error please visit the [Troubleshooting Secure Build section](#troubleshooting-secure-build)

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

    !!! error "If you see an error in your status"
        If you see that your build ran into an error please visit the [Troubleshooting Secure Build section](#troubleshooting-secure-build)

7. From the original terminal window that you ran `hpvs sb init`, output the repository registration file.

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
    
    !!! warning
        If you run this command from a terminal window where you did not set the `passphrase` environment variable you will get an error saying `openpgp: invalid data: private key checksum failure`. You can also check to make sure you are in the right terminal window by running `echo "${passphrase}"` and the output should be the passphrase you set in the beginning of the "Create repository registration GPG signing key" section. 

## Troubleshooting Secure Build

???+ "Troubleshooting Instructions"

    If your build completed successfully, feel free to [SKIP this section](#verify-your-application) :smile:

    !!! error "If your build failed continue for instructions on how to find the happy path yet again."
        
        If at any point you see an error in the status command you will need to restart your build with the following steps.

        1. If the secure build command is still running in its own window please interrupt it with `ctrl + c`
        2. Check the logs for more error information using:
            ``` bash
            hpvs sb log --config "${SB_DIR}/sb_config.yaml"
            ```
        3. Correct whatever information was incorrect and go back through the steps in the [Set Build Configuration section](#set-build-configuration)
        4. Clean up the data (i.e. logs) from the old build with:
            ``` bash
            hpvs sb clean --config "${SB_DIR}/sb_config.yaml"
            ```
        5. Update the secure build server with the new configuration
            ``` bash
            hpvs sb update --config "${SB_DIR}/sb_config.yaml"
            ```
        6. Continue onward like you just initialized your build server (You did ... just for another time :grin:) at `Step 2` in the [Build Application Section](#build-application)

        !!! info
            The next command you should run will be `hpvs sb build` (`Step 2` in the [Build Application Section](#build-application))

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

    !!! info

        Connections to the secure build server are secured using mutual tls. This means that proper certificates and keys have to be used on the client side to access these files (as well as when doing the previous secure build commands). This adds a layer of security to make sure that the `public key` and `manifest files` are authentic and securely distributed.

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

7. Set `MAN_PUBKEY` to point to your manifest public key.

    ``` bash
    export MAN_PUBKEY="${SB_DIR}/manifest/${BUILD_NAME}-public.pem"
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

        The `.sig.tbz` was a tarball compressed using bzip2 compression of both a nested `.tbz` file (containing the manifest files) and a `.sig` file containing a signature of the nested `.tbz` to verify it with the public key retrieved using the `hpvs sb pubkey` command above saved in the file referenced by `MAN_PUBKEY`.

9.  Convert the signature file to binary format

    ``` bash
    cat "${MANIFEST}.sig" | xxd -r -p > "${MANIFEST}.sig.bin"
    ```

10. Hash (`SHA256`) the manifest `.tbz` file (containing the manifest files)

    ``` bash
    openssl dgst -sha256 -binary -out "${MANIFEST}.tbz.sha256" "${MANIFEST}.tbz"
    ```

11. Verify that the `.tbz` manifest package was the one signed on the secure build server.

    ``` bash
    openssl dgst -sha256 -verify "${MAN_PUBKEY}" \
    -signature "${MANIFEST}.sig.bin" "${MANIFEST}.tbz.sha256"
    ```

    ???+ success

        ``` bash
        Verified OK
        ```

    !!! info
        On the secure build server, the `.tbz` manifest file was hashed and then signed by the manifest private key which only exists on the server (inside its secure environment). Here, you are using the matching public key you received from the secure build server (using a mutual tls connection) in `step 4` to "undo" this signature to reveal the original hash of the file. We then compare this hash to the hash of the file we have using the above `verify` command. `Verification OK` means they are the same, implying that the `.tbz` file we have now was the same one that was signed by the manifest private key inside the safe confines of the secure build server.

    !!! note
        In this case, we are double hashing the `.tbz` file on the signature side which is why we hash the `.tbz` before the verify command here.

12.  Untar and unzip the manifest `.tbz` file into the `manifest_files` directory.

    ``` bash
    tar -xjf "${MANIFEST}.tbz" -C "${SB_DIR}/manifest/manifest_files"
    ```

13. Change into the `manifest_files` directory with your manifest files and view the files in the directory.

    ``` bash
    cd "${SB_DIR}/manifest/manifest_files" && ls
    ```

    ???+ example "Example Output"

        ``` bash
        data  git  root_ssh
        ```
    
    !!! info "Manifest explanation"
        This `manifest` package contains three directories as shown above. The `data` directory contains the `build.json` (containing the build status of the directory updated after the image was pushed to its Docker Registry) and `build.log` (containing the logs from the secure build). The `git` directory contains the source code used for the build (cloned from git). Finally, the `root_ssh` directory contains any `ssh` material provided (if the user chose to use ssh) which is empty for us because we didn't enable ssh and provide ssh keys. This way no one can ssh into our secure build container. 

14. What does this give me?

    This gives you a way to see the collection of files used for your build at the time of the build and signed by the manifest private key which is secured in your secure build container. By retrieving the public key and verifying the signature of the package we *and auditors* can verify what was used for our secure image build. (Since the private key was generated in the secure build server we can trust it) This gives us verification of what was used in the build as well as the verification of the images themselves we get from [Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/){target=_blank}.

    !!! note
        The secure build server also generates the keys for Docker Content Trust and stores them safely to provide a secure root of trust.

        _Remember the secure build server is running as a Hyper Protect Virtual Server and thus inherits all of the security features and assurances of Hyper Protect Virtual Servers._

## Summary

Congratulations!!! :tada: 

You have securely built your application and are ready to deploy it into a Hyper Protect Virtual Server in the next section.
