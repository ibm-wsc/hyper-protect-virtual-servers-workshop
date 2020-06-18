# Secure Build Lab

## Create a Docker Hub account and token to use 

## Add Docker registry to secure build

1. See your current docker registries with:

    ``` bash
    hpvs registry list
    ```

    ??? example "Example Output"
        ```
        hpvs registry list
        +---------------+
        | REGISTRY NAME |
        +---------------+
        +---------------+
        ```
    

2. Set your Docker username to the username for your account on Docker Hub

    ``` bash
    export DOCKER_USERNAME="my_username"
    ```

    !!! note 
        This will be the username you used when you created your Docker Hub account in the [Prerequisites](prerequisites.md#Create-a-Docker-Hub){target=_blank}

    ??? example "Example"

        ``` bash
        export DOCKER_USERNAME="gmoney23"
        ```

3. Set your Docker registry placeholder name 

    ``` bash
    export REGISTRY_NAME="my_registry"
    ```

    ??? example "Example"

        ``` bash
        export REGISTRY_NAME="g_docker_hub"
        ```

2. Add you docker registry with:

    ``` bash
    hpvs registry add --name "${REGISTRY_NAME}" --dct https://notary.docker.io --url docker.io --user "${DOCKER_USERNAME}"
    ```

    !!! note 
        It will prompt you to enter your password. Use the Docker Hub token you have created for the lab in the [Prerequisites](prerequisites.md#Create-a-GitHub){target=_blank}

    ??? example "Example Output"

        ``` bash
        hpvs registry add --name "${REGISTRY_NAME}" --dct https://notary.docker.io --url docker.io --user "${DOCKER_USERNAME}"
        Enter Password: 
        ```

4. Run hpvs registry list again to confirm your registry has been added

    ``` bash
    hpvs registry list
    ```

5. Check the details of your added registry with 

    ``` bash
    hpvs registry show --name "${REGISTRY_NAME}"
    ```

    ??? example "`hpvs registry show --name "${REGISTRY_NAME}`""

        ``` bash
        +------+--------------------------+
        | name | g_docker_hub             |
        | user | gmoney23                 |
        | dct  | https://notary.docker.io |
        | url  | docker.io                |
        +------+--------------------------+
        ```
