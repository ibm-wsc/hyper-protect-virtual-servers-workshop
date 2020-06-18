# Secure Build Lab


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
        This will be the username you used when you created your Docker Hub account in the [Prerequisites](prerequisites.md#Create-a-Docker-Hub){target=_blank}

3. Set your Docker registry placeholder name 

    === "Command Syntax"

        ``` bash
        export REGISTRY_NAME="my_registry"
        ```

    === "Example Command"

        ``` bash
        export REGISTRY_NAME="g_docker_hub"
        ```

4. Add your Docker registry with:

    ``` bash
    hpvs registry add --name "${REGISTRY_NAME}" --dct https://notary.docker.io --url docker.io --user "${DOCKER_USERNAME}"
    ```

    !!! note 
        It will prompt you to enter your password. Use the Docker Hub token you have created for the lab in the [Prerequisites](prerequisites.md#Create-a-GitHub){target=_blank}

    ???+ example "Example Output"

        ``` bash
        Enter Password: 
        ```

5. List your registered Docker registries again to confirm your registry has been added.

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

6. Check the details of your added registry with 

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

## Create Secure Build Hyper Proteect Virtual Server
