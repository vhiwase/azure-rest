## Overview
- Azure REST APIs authenticate requests via a Bearer Token in the Authorization Header:
    
    `Authorization: Bearer {token}`

- You need the following to create a Bearer Token:
    - Tenant Id: `az account show --query tenantId -otsv`
    - Service Principal Id and Secret: `az ad sp create-for-rbac -n name -p password`
    - Resource Uri: Typically: `https://management.azure.com/`, but for Key Vault: `https://vault.azure.net`
- You HTTP POST to `https://login.microsoft.com/{tenantId}/oauth2/token` to create a Bearer Token.
- You use that Bearer Token on subsequent requests.

## Step-by-Step Instructions
1. **Open [Azure Cloud Shell](https://azure.microsoft.com/en-us/features/cloud-shell/) or Install [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)**

1. **Login**: Run the following command to login to the Azure CLI.

    `az login`

1. **Set Subscription**: If you have more than one subscription, you need to pick which one you want to work with.

    `az account set -s {subscriptionId}`

1. **Get Tenant Id**: Run the following command to get your Tenant Id:

    `az account show --query tenantId -otsv`

    > Copy that Id to a safe place.

1. **Create Service Principal**: Run the following command to create a new Service Principal and get its Id and Secret:

    `az ad sp create-for-rbac -n {appName} -p {password}`

    That will output the following: 
    ```
    {
        "appId": "8ef35921-03a8-4291-a57f-d489a62d27dc",
        "displayName": "azurekeyvaultapp6",
        "name": "http://azurekeyvaultapp6",
        "password": "jongpassword",
        "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
    }
    ```

    > Copy the appId and password to a safe place.

1. **Get Bearer Token**: Execute the following HTTP POST to get your Bearer Token.

    Request:
    ```
    curl --request POST \
    --url 'https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db47/oauth2/token' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --data 'grant_type=client_credentials&client_id=8ef35921-03a8-4291-a57f-d489a62d27dc&client_secret=jongpassword&resource=https://management.azure.com/'
    ```

    Result:
    ```javascript
        {"token_type":"Bearer","expires_in":"3600","ext_expires_in":"0","expires_on":"1516166046","not_before":"1516162146","resource":"https://management.azure.com/","access_token":"...wU6pYyg"}
    ```

    > Copy `access_token` to a safe place. This is your Bearer Token.

1. **Use Bearer Token**: Execute the following request with the Bearer Token to Get Resource Groups in your Subscription.

    Request:
    ```
    curl --request GET \
    --url 'https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups?api-version=2017-05-10' \
    --header 'Authorization: Bearer {bearerToken}'
    ```

    Result:
    ```
    {"value":[{"id":"/subscriptions/f9766876-e50b-436f-9ad3-5afb7bb8cf45/resourceGroups/jongdemo","name":"jongdemo","location":"westus","properties":{"provisioningState":"Succeeded"}}]
    ```
