# Clean up Instructions

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Cleanup Secure Build Server

1. Cleanup virtual server

    ``` bash
    hpvs vs delete --name sbserver_${HPVS_NUMBER}
    ```

2. Cleanup quotagroup

    ``` bash
    hpvs quotagroup delete --name "sb_user${HPVS_NUMBER}"
    ```
    
## Cleanup Application

1. Cleanup HPVS deployment

    ``` bash
    hpvs vs delete --name ${REPO_ID}
    ```

2. Cleanup quotagroup

    ``` bash
    hpvs quotagroup delete --name "${REPO_ID}"
    ```

## Cleanup Repository

``` bash
hpvs repository  delete --id ${REPO_ID}
```

???+ example "Example Output"
    ``` bash
    ```

## Cleanup Docker Token

Delete the existing Docker Token by logging into your `Docker Hub` account and following [these instructions](https://docs.docker.com/docker-hub/access-tokens/#modify-existing-tokens){target=_blank}

## Cleanup GitHub ssh key

Delete the ssh key you added for the lab following steps 1-3 of [these instructions](https://help.github.com/en/github/authenticating-to-github/reviewing-your-ssh-keys){target=_blank}

## Cleanup Lab directory 

You can remove the directory you stored your files in throughout the lab. This is a personal choice as the lab machine will be deleted after the lab anyway.

``` bash
rm -rf "${SB_DIR}"
```