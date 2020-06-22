# Clean up Instructions

## Export Variables set in previous sections to current terminal session

Source `bashrc` to set necessary variables if unset

``` bash
source "${HOME}/.bashrc"
```

## Cleanup Secure Build Server

1. Cleanup virtual server

    ``` bash
    hpvs vs delete --name sbserver-${HPVS_NUMBER}
    ```

2. Cleanup quotagroup

    ``` bash
    hpvs quotagroup delete --name "sb-user${HPVS_NUMBER}"
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