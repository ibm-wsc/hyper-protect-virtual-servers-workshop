# Deploy your Securely built application into a Hyper Protect Virtual Server

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Set up you Docker Image Repository

1. Register your repository

    ``` bash
    hpvs repository register \
    --pgp="${SB_DIR}/yaml.${REPO_ID}.enc" \
    --id="${REPO_ID}"
    ```

2. View your newly registered repository

    ``` bash
    hpvs repository show --id "${REPO_ID}"
    ```

##  Set up your quota group for storage

1. Create Quota Group

    ``` bash
    hpvs quotagroup create --name="${REPO_ID}" --size=5GB
    ```

2. View your newly created quotagroup

    ``` bash
    hpvs quotagroup list | grep "${REPO_ID}"
    ```

## Deploy your Application

1. Set App port

    ``` bash
    export APP_PORT=214${HPVS_NUMBER}
    ```

2. Create your Hyper Protect Virtual Server

    ``` bash
    hpvs vs create --name=${REPO_ID} --repo ${REPO_ID} \
    --tag latest --cpu 2 --ram 2048 \
    --quotagroup "{quotagroup = ${REPO_ID}, mountid = new, mount = /newroot, filesystem = btrfs, size = 4GB}" \
    --ports "{containerport = 443, protocol = tcp, hostport = ${APP_PORT}}"
    ```

## Access your application

Run the following command to print the address for your application:

``` bash
echo "https://${SB_IP}:${APP_PORT}"
```

???+ example "Example Output"

    ```
    ```

Your app is now up and running! In the final part of this lab you will use your newly built secure application...
