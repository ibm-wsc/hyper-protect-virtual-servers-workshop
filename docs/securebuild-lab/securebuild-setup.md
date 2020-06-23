# Secure Build Setup

## Add Docker registry to use for secure build

1. See your current docker registries with:

    ``` bash
    hpvs registry list
    ```

    ???+ example "Example Output"

        ```
        +---------------+
        | REGISTRY NAME |
        +---------------+
        +---------------+
        ```
    

2. Set your Docker username to the username for your account on Docker Hub

    === "Command Syntax"

        ``` bash
        export DOCKER_USERNAME="my_username"
        ```

    === "Example Command"

        ``` bash
        export DOCKER_USERNAME="gmoney23"
        ```
    
    !!! note 
        This will be the username you used when you created your Docker Hub account in the [Prerequisites](../prerequisites.md#Create-a-Docker-Hub){target=_blank}

3. Set your Docker registry placeholder name 

    === "Command Syntax"

        ``` bash
        export REGISTRY_NAME="my_registry"
        ```

    === "Example Command"

        ``` bash
        export REGISTRY_NAME="g_docker_hub"
        ```

4. Save your `REGISTRY_NAME` to bashrc for future shells (in case you open new terminals)

    ``` bash
    echo "export REGISTRY_NAME='${REGISTRY_NAME}'" >> "${HOME}/.bashrc"
    ```

5. Add your Docker registry with:

    ``` bash
    hpvs registry add --name "${REGISTRY_NAME}" --dct https://notary.docker.io \
    --url docker.io --user "${DOCKER_USERNAME}"
    ```

    !!! note 
        It will prompt you to enter your password. Use the Docker Hub token you have created for the lab in the [Prerequisites](../prerequisites.md#create-a-docker-access-token){target=_blank}

    ???+ example "Example Output"

        ``` bash
        Enter Password: 
        ```

6. List your registered Docker registries again to confirm your registry has been added.

    ``` bash
    hpvs registry list
    ```

    ???+ example "Example Output"

        ``` bash
        +---------------+
        | REGISTRY NAME |
        +---------------+
        | g_docker_hub  |
        +---------------+
        ```

7. Check the details of your added registry with 

    ``` bash
    hpvs registry show --name "${REGISTRY_NAME}"
    ```

    ???+ example "Example Output"

        ``` bash
        +------+--------------------------+
        | name | g_docker_hub             |
        | user | gmoney23                 |
        | dct  | https://notary.docker.io |
        | url  | docker.io                |
        +------+--------------------------+
        ```

## Create directory for secure build lab and change directory

1. Set your secure build directory

    ``` bash 
    export SB_DIR="$HOME/securebuild-lab"
    ```

2. Save `SB_DIR` to `$HOME/.bashrc` for future shells (in case you open new terminals)

    ``` bash
    echo "export SB_DIR='${SB_DIR}'" >> "${HOME}/.bashrc"
    ```

3. Create `SB_DIR` and change into it

    ``` bash
    mkdir -p "${SB_DIR}" && cd "${SB_DIR}"
    ```

## Create SSH Key and Grant GitHub access

1. Create GitHub keys directory

    ``` bash
    mkdir -p "${SB_DIR}/github_keys"
    ```

2. Set `GITHUB_SSH_KEY` variable

    ``` bash
    export GITHUB_SSH_KEY="${SB_DIR}/github_keys/github_rsa"
    ```

3. Save `GITHUB_SSH_KEY` variable in `bashrc` for later use

    ``` bash
    echo "export GITHUB_SSH_KEY='${GITHUB_SSH_KEY}'" >> "${HOME}/.bashrc"
    ```

4. Create Public and private key for GitHub access

    ``` bash
    ssh-keygen -t rsa -b 4096 -f "${GITHUB_SSH_KEY}" -N ''
    ```

    ???+ example "Example Output"

        ``` bash
        Generating public/private rsa key pair.
        Your identification has been saved in /home/multiarch-lab/securebuild-lab/github_keys/github_rsa.
        Your public key has been saved in /home/multiarch-lab/securebuild-lab/github_keys/github_rsa.pub.
        The key fingerprint is:
        SHA256:sbEfCzwyOz/LaZ4jSeKT+Aw7IWUr7YT6m8Hxv9Dq90k multiarch-lab@ubuntu
        The key's randomart image is:
        +---[RSA 4096]----+
        |                 |
        |                 |
        |        o        |
        |  o    . =       |
        | =..  o S .      |
        |+.=o...+ + o     |
        |.=++o++.E o      |
        |. +*+++=++       |
        | .===oo*X+       |
        +----[SHA256]-----+
        ```

5. Cat GitHub public key to terminal

    ``` bash
    cat "${GITHUB_SSH_KEY}.pub"
    ```

    ???+ example "Example Output"

        ``` bash
        ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC6zsnjH4fXaR/imYkzRaYVgWsNVIY4LzCftygKGTFJVBDMqVErsbkvF810RUTPIIowwe7Bx2UGLtwv3kL2omUJaHjol/+nzQjdmFrV2qnZosMUCn4xSpdffCmMyFWE8FnWw1ZSc7STcTD/NFzLjrN/vbJgMvla0aSxsENio2RvyFQkMKbwfojE22Q/MOtJAg0wyr9/0JNXiAyLYUEwfi1qVoF4F1mvXgLtNgTSx4VzokUKBsiSaDDbEa70ik154dtWLY9nemUsrSfUluCrHLaJhUy9cg6Jc+/9tco0cZhyZDnLIhT/4+n1XGfZxb83c5WElrg/IEYHGOHfqp+lMfusMj/0ppopz+f66dWDabVQCbrkqa+yfbw0ItC0+sZ7opIYGRLt+OelkrIECo7utzCTxdCCJ+2iUIiBV14my7xAFR+fNLKzl9KD+FTT/QoUPlqTmUQNo6NFNGG15znWVRtMDOdDiA6fugl6RrFJHDpaw9/lh+7g7tqwHGF83ZomvvFdKFMsXY6JTpj6rb7gtKzEapB9imCr0aw0ZTWTdlzTs9ksSNr1gZRf/eiztP5puC4weGDRXVjPwHOFOGbUH7Wk8ywQAmUJdEg03bz01U1htYryiNTD4VD2QfVTmxTltocj/2yv7Apv6gQ+Gb0LsgTp2aWbAnbeWX7qfjpYt0j/iw== multiarch-lab@ubuntu
        ```

6. Copy your output to the clipboard with `ctrl+shift+c`.

7. Add this key to your public GitHub (paste copied content into GitHub keys textbox)

    **Follow [these instructions](https://help.github.com/en/enterprise/2.15/user/articles/adding-a-new-ssh-key-to-your-github-account){target=_blank}** for adding this *public* key to your `github.com` GitHub account.

    !!! note 
        You created this *public* `github.com` GitHub in the [Prerequisites](../prerequisites.md#Create-a-GitHub){target=_blank} (or already had one).

    !!! warning
        The above copy command (`ctrl+shift+c`) is saving the `.pub` public key **NOT** the `private` key. You keep the private key and GitHub uses the public key to verify that it is communicating with the owner of the private key (i.e. you).

8. Feel at ease knowing you will delete this key from your Github account in the cleanup phase of this lab so access will be revoked soon enough :relaxed: