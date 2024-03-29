# Securely Build your Application

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Create repository registration GPG signing key

!!! info
    This section creates the `Repository Registration Signing Key` referenced in the [key table](overview.md#fnref:2). [The GNU Privacy Guard (GPG)](https://gnupg.org/){target=_blank} utility (a free implementation of the OpenPGP standard or PGP) generates this key pair (protected by a user defined password) and then exports the public and private keys to files in the `${SB_DIR}/registration_keys/` directory. Later, you will employ these keys to sign a registration file for your securely built image to register it with your Hyper Protect Virtual Servers appliance such that only the holder of the private key can change the registered repository on the Hyper Protect Virtual Servers appliance.

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
        secure_bitcoin_key7750_definition_keys  secure_bitcoin_key7750.private  secure_bitcoin_key7750.pub
        ```

## Set Build Configuration

1. Set Secure Build Lab IP Address

    ``` bash
    export SB_IP=192.168.22.79
    ```

2. Save Secure Build Lab IP Address for later use

    ``` bash
    echo "export SB_IP='${SB_IP}'" >> "${HOME}/.bashrc"
    ```

3. Set Secure Build GitHub repository

    ``` bash
    export GH_REPO="git@github.com:ibm-wsc/secure-bitcoin-wallet.git"
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

    ``` yaml
    cat > "${SB_DIR}/sb_config.yaml" <<EOF
    secure_build_workers:
        sbs:
            url: 'https://${SB_IP}'
            port: '${SB_PORT}'
            cert_path: '${SB_DIR}/sbs_keys/client_cert.pem'
            key_path: '${SB_DIR}/sbs_keys/client-key.pem'
            ca_cert_path: '${SB_DIR}/sbs_keys/ca.pem'
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
		# You would enter build parameters such as build args you want to use in your application container
        #    args: { 'ARG1' : 'VALUE1' }
        signing_key:
            private_key_path: '${SB_DIR}/registration_keys/${keyName}.private'
            public_key_path: '${SB_DIR}/registration_keys/${keyName}.pub'
    EOF
    ```

8. Continue to the [Build Application section](#build-application) if you are launching secure build for the first time or the [Troubleshooting Secure Build section](#troubleshooting-secure-build) if things didn't go as planned your first go-round and you are trying to get back on the right track.

## Build Application

!!! info
    In this section, you finally use all of your configuration information to build your application securely over TLS by connecting to the secure build server you deployed in the [previous part of the lab](create-server.md){target=_blank} with the `Secure Build Server Client Certificate and Key`. This build securely updates the application using [Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/){target=_blank}, an implementation of [The Update Framework](https://theupdateframework.io/overview/){target=_blank}. This involves securely signing a series of [metadata files](https://theupdateframework.io/metadata/){target=_blank} with corresponding keys (`Root Key`, `Image Signing "Targets" Key`, `Snapshot Key`, and `Timestamp Key` referenced in the [key table](overview.md#fnref:2)) and pushing this trust metadata to a [Notary Server](https://docs.docker.com/notary/service_architecture/){target=_blank} to provide assurances regarding the integrity of the Docker Images when users pull them to run in their environments. This conforms to [The Update Framework Specification](https://github.com/theupdateframework/specification/blob/master/tuf-spec.md#the-update-framework-specification){target=_blank}

    Additionally, this process generates a `Manifest Signing Key` which it employs to sign the contents of the build for attestation of the build contents by internal and/or third party auditors. The `Manifest Signing Key` is discussed in more detail in the [`Verify your application section`](#verify-your-application) later in the lab.

1. Initialize Secure Build Hyper Protect Virtual Server with configuration file generated in the [Set Build Configuration section](#set-build-configuration)

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
        The following build will take anywhere from **15-20 minutes** to complete. While this is ongoing, you should open a new tab in your terminal to check the automatically updating logs and build status (steps for doing this are detailed in the next few steps). If this command times out please check the status as in `step 4` to make sure nothing went wrong. It might just be that your build took too long and everything will be ok. If something did go wrong, visit the [Troubleshooting Secure Build section](#troubleshooting-secure-build).

    !!! note
        The secure build is asynchronous so if the command gets interrupted here don't worry! :grin:

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

3. Please look at the logs in another terminal window while the secure build is running (don't interrupt the current terminal window which is waiting for the secure build)

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

4. You can also look at the secure build status in another window. This can be useful if you accidentally interrupted the secure build command or it times out due to the timeout not being long enough. It will also fill out the additional fields that are initially blank when the build completes (such as `build_name` and `manifest_public_key`, etc.).

    ``` bash
    hpvs sb status --config "${SB_DIR}/sb_config.yaml"
    ```

    ???+ example "Example Output"

        ``` bash
        +---------------------+---------------+
        | build_name          |               |
        | image_tag           |               |
        | manifest_key_gen    |               |
        | manifest_public_key |               |
        | root_ssh_enabled    | false         |
        | status              | github cloned |
        +---------------------+---------------+
        ```

    !!! error "If you see an error in your status"
        If you see that your build ran into an error please visit the [Troubleshooting Secure Build section](#troubleshooting-secure-build)

5. You can continue to check the logs to monitor the progress of your secure build with the previous logs command

    ``` bash
    hpvs sb log --config "${SB_DIR}/sb_config.yaml"
    ```

6. When the secure build successfully completes, it will have ending logs similar to the following:

    ???+ example "Example Output"

        ``` bash
        2022-04-26 18:09:28,843  root       INFO    run: latest: digest: sha256:45cd2d06a2ef978cf1ae60b58833689ff2af1aff0b3d6aa2e69ea83378f6f22d size: 5549
        2022-04-26 18:09:28,843  root       INFO    run: Signing and pushing trust metadata
        2022-04-26 18:09:29,656  root       INFO    run: Successfully signed docker.io/gmoney23/hpvs_bc:latest
        2022-04-26 18:09:29,661  root       INFO    run: return code = 0
        2022-04-26 18:09:29,662  root       INFO    extracting an image keyid and key
        2022-04-26 18:09:29,663  root       INFO    keyid=2092f4e8a72ab297e94a05c18a1ad5b558b9cdee3531018752cd23e238c02889
        2022-04-26 18:09:29,663  root       INFO    publickey=LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJrVENDQVRhZ0F3SUJBZ0lSQUlSUVpKVmR0QVJycGNnc1A3T0diRVF3Q2dZSUtvWkl6ajBFQXdJd0xqRXMKTUNvR0ExVUVBd3dqWkc5amEyVnlMbWx2TDNOcGJHeHBiV0Z1TDJod2RuTmZZbU5mTWpBeU1qQTBNall3SGhjTgpNakl3TkRJMk1UZ3dPVEkwV2hjTk16SXdOREl6TVRnd09USTBXakF1TVN3d0tnWURWUVFERENOa2IyTnJaWEl1CmFXOHZjMmxzYkdsdFlXNHZhSEIyYzE5aVkxOHlNREl5TURReU5qQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDkKQXdFSEEwSUFCSUhqM1RLaDZPYmp4Z214allOZ3ZDSGlDSTNOOTlBZy8vS0lPc0hxR25SYUR6NU5LbEdMWktvOAoxaDRYaUgwRG92d1ZFMGUvdlZzM3V3dnlwT0JEYzNtak5UQXpNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WCkhTVUVEREFLQmdnckJnRUZCUWNEQXpBTUJnTlZIUk1CQWY4RUFqQUFNQW9HQ0NxR1NNNDlCQU1DQTBrQU1FWUMKSVFEU2o4SzcvVDhRSWFLbVoyaEdSTTZnTkRsRE01L21UcUdQK09qc3NwWS9jZ0loQU43ZDM5RXlITkV3bFRUMApkMC9XaVcwejRCMWdMM1RyQVhFRDlwVUhoMVR5Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
        2022-04-26 18:09:29,663  root       INFO    generating a config file
        2022-04-26 18:09:29,664  root       INFO    create_and_push: SoftCrypto
        2022-04-26 18:09:30,627  root       INFO    digest=7b997311130b0b0984e1b7cabd45e7b4a5461337d52dcb1b99be26849d7172d9
        2022-04-26 18:09:30,627  root       INFO    block_size=64
        2022-04-26 18:09:30,648  root       INFO    digest=7b997311130b0b0984e1b7cabd45e7b4a5461337d52dcb1b99be26849d7172d9
        2022-04-26 18:09:30,650  root       INFO    signature=4e6b43530773245b8c04e016e8b112cfa8efab6621b84c9825d06c7fa7faba525c8cc97618313205bf46db123d8f53ca2377871b3daa6dec62e48fc539fd17c9796f5c395bd1a82b05e60d69796fea1baa7fb48b4f9ae0cd72c329121241b2e4f7a21d8bc6456fa921df2ab9b9a47974bfb841c540ff1bd9e9e350e9f9d1c5f4da3bef0857e3f21605e1131f4b6ca0a12db33ae57149e37718ef8e54b978769cbf8cf025d32323f1f488df6db9ffd6ec22ebad7daa67e99c3506b47bdefc0ebe3d0fe76edc6d19d38283a355f9d94eab20c3e3d32cdd6b5ae8f8e0f4748ffbfec10661f7aad9dbc3dd5017e42f4468f7ef5443303c694761d6ff8646382fda14
        2022-04-26 18:09:30,650  root       INFO    verify=OK
        2022-04-26 18:09:31,049  root       WARNING undefined MANIFEST_BUCKET_NAME
        2022-04-26 18:09:31,049  root       WARNING skipping transferring a manifest to COS
        2022-04-26 18:09:31,049  root       INFO    cleaning up the build environment
        2022-04-26 18:09:31,049  root       INFO    github_dir=secure-bitcoin-wallet
        2022-04-26 18:09:31,110  root       INFO    completed a build
        ```

7. When the secure build successfully completes, you can check the status again to see a status of `success`.

    ``` bash
    hpvs sb status --config "${SB_DIR}/sb_config.yaml"
    ```

    ???+ example "Example Output"

        ``` bash
        +---------------------+---------------------------------------------------------------------------------------------------+
        | manifest_public_key | manifest.docker.io.gmoney23.hpvs_bc.latest-941893e.2022-04-26_18-09-29.663558-public.pem |
        | root_ssh_enabled    | false                                                                                    |
        | status              | success                                                                                  |
        | build_name          | docker.io.gmoney23.hpvs_bc.latest-941893e.2022-04-26_18-09-29.663558                     |
        | image_tag           | latest-941893e                                                                           |
        | manifest_key_gen    | soft_crypto                                                                              |
        +---------------------+------------------------------------------------------------------------------------------+
        ```

    !!! error "If you see an error in your status"
        If you see that your build ran into an error please visit the [Troubleshooting Secure Build section](#troubleshooting-secure-build)

8. From the original terminal window, where you ran `hpvs sb init`, output the repository registration file.

    ``` bash
    echo "${passphrase}" | hpvs sb regfile \
    --config "${SB_DIR}/sb_config.yaml" \
    --out "${SB_DIR}/yaml.${REPO_ID}.enc"
    ```

    ???+ example "Example Output"

        ``` bash
        Enter Signing Private key passphrase:
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

!!! info
    On the secure build server, the `.tbz` manifest file was hashed and then signed by the `Manifest Signing Key` referenced in the [key table](overview.md#fnref:2) which only exists on the server (inside its secure environment). In steps 1-11 below, you are using the matching public key you will receive from the secure build server (using a secure TLS connection) in `step 4` to "undo" this signature to reveal the original hash of the file. We then compare this hash to the hash of the file we have using the `verify` command in `step 11`. `Verification OK` means they are the same, implying that the `.tbz` file we have now was the same one that was signed by the manifest private key inside the safe confines of the secure build server.

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
        docker.io.gmoney23.hpvs_bc.latest-941893e.2022-04-26_18-09-29.663558
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

        Connections to the secure build server are secured using mutual TLS. This means that proper certificates and keys have to be used on the client side to access these files (as well as when doing the previous secure build commands). This adds a layer of security to make sure that the `public key` and `manifest files` are authentic and securely distributed.

5. Check that your application manifest and signing key were retrieved

    ``` bash
    ls
    ```

    ???+ example "Example Output"

        ```bash 
        docker.io.gmoney23.hpvs_bc.latest-941893e.2022-04-26_18-09-29.663558-public.pem
        manifest.docker.io.gmoney23.hpvs_bc.latest-941893e.2022-04-26_18-09-29.663558.sig.tbz
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
        manifest.docker.io.gmoney23.hpvs_bc.latest-941893e.2022-04-26_18-09-29.663558.tbz
        manifest.docker.io.gmoney23.hpvs_bc.latest-941893e.2022-04-26_18-09-29.663558.sig
        ```

    !!! note

        The `.sig.tbz` was a tarball compressed using bzip2 compression of both a nested `.tbz` file (containing the manifest files) and a `.sig` file containing a signature of the nested `.tbz` to verify it with the public key retrieved using the `hpvs sb pubkey` command above saved in the file referenced by `MAN_PUBKEY`.

9. Convert the signature file to binary format

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

    !!! note
        In this case, we are double hashing the `.tbz` file on the signature side which is why we hash the `.tbz` before the verify command here.

12. Untar and unzip the manifest `.tbz` file into the `manifest_files` directory.

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

    This gives you a way to see the collection of files used for your build at the time of the build and signed by the manifest private key which is secured in your secure build container. By retrieving the public key and verifying the signature of the package we *and auditors* can verify what was used for our secure image build. (Since the private key was generated in the secure build server we can trust it). This gives us verification of what was used in the build as well as the verification of the images themselves we get from [Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/){target=_blank}.

    !!! note
        The secure build server also generates the keys for Docker Content Trust and stores them safely to provide a secure root of trust.

        _Remember the secure build server is running as a Hyper Protect Virtual Server and thus inherits all of the security features and assurances of Hyper Protect Virtual Servers._

## Summary

Congratulations!!! :tada:

You have securely built your application and are ready to deploy it into a Hyper Protect Virtual Server in the next section.
