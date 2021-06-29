# Clean up your Environment

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Clean up Secure Build Server

1. Clean up virtual server

    ``` bash
    hpvs vs delete --name sbserver_${HPVS_NUMBER}
    ```

2. Clean up Quotagroup

    ``` bash
    hpvs quotagroup delete --name "sb_user${HPVS_NUMBER}"
    ```

## Clean up Application

1. Clean up HPVS deployment

    ``` bash
    hpvs vs delete --name ${REPO_ID}
    ```

2. Clean up Quotagroup

    ``` bash
    hpvs quotagroup delete --name "${REPO_ID}"
    ```

## Clean up Repository

``` bash
hpvs repository  delete --id ${REPO_ID} --force
```

???+ example "Example Output"
    ``` bash
    +-------------------+-------------------------------------+
    | ContainersDeleted | []                                  |
    | ImagesDeleted     | [gmoney23/hpvs_bc_a:latest]         |
    | RepositoryDeleted | docker.io/gmoney23/hpvs_bc_a        |
    +-------------------+-------------------------------------+
    ```

## Clean up Docker Token

Delete the existing Docker Token by logging into your `Docker Hub` account and following [these instructions](https://docs.docker.com/docker-hub/access-tokens/#modify-existing-tokens){target=_blank}

## Clean up GitHub ssh key

Delete the ssh key you added for the lab following steps 1-3 of [these instructions](https://help.github.com/en/github/authenticating-to-github/reviewing-your-ssh-keys){target=_blank}. Stay logged in to GitHub as the next section also deals with your GitHub account. 

## Clean up GitHub personal access token
You should still be logged in to GitHub from the previous step.  Click on your picture or avatar for your account and navigate to _Settings->Developer Settings->Personal access tokens_ and delete the personal access token that you created for this lab. Ask an instructor for help if you have difficulty deleting this.

## Clean up Lab directory

You can remove the directory you stored your files in throughout the lab. This is a personal choice as the lab machine will be deleted after the lab anyway.

``` bash
rm -rf "${SB_DIR}"
```
