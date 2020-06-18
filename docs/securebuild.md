# Secure Build Lab

## Create a Docker Hub account and token to use 

## Add Docker registry to secure build

1. See your current docker registries with:

    ``` bash
    hpvs registry list list
    ```

2. Set your Docker username to the username for your account on Docker Hub

    ``` bash
    export DOCKER_USERNAME="my_username"
    ```

    ??? example

        ``` bash
        export DOCKER_USERNAME="gmoney23"
        ```

3. Set your Docker registry placeholder name 

    ``` bash
    export REGISTRY_NAME=
    ```

    ??? example

        ``` bash
        export REGISTRY_NAME=g_docker_hub
        ```

2. Add you docker registry with:

    ``` bash
    hpvs registry add --name Docker_Hub --dct https://notary.docker.io --url docker.io --user ${DOCKER_USERNAME}
    ```

3. It will prompt you to enter your password. Use the DockerHub token you have created for the lab

4. Run hpvs registry list again to confirm your registry has been added

    ``` bash
    hpvs registry list
    ```

5. Check the details of your added registry with 

    ``` bash
    hpvs registry show --name 
    ```