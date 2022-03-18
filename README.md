# azure-pipelines-secrets-key-vault

Demoing the use of Azure Key Vault Secrets within Azure DevOps Pipelines.

1) Create Azure Key Vault with Secrets and persmissions

KEYVAULT_RG="rg-keyvault-devops"
KEYVAULT_NAME="keyvault019"
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

az group create -n rg-keyvault-devops -l westeurope

az keyvault create --name $KEYVAULT_NAME \
   --resource-group $KEYVAULT_RG \
   --enable-rbac-authorization

USER_ID=$(az ad signed-in-user show --query objectId -o tsv)

KEYVAULT_ID=$(az keyvault show --name $KEYVAULT_NAME \
   --resource-group $KEYVAULT_RG \
   --query id \
   --output tsv)

az role assignment create --role "Key Vault Secrets Officer" \
   --scope $KEYVAULT_ID \
   --assignee-object-id $USER_ID

az keyvault secret set --name "DatabasePassword" \
  --value "mySecretPassword" \
  --vault-name $KEYVAULT_NAME

2) Create Service Principal to access Key Vault from Azure DevOps Pipelines

SPN=$(az ad sp create-for-rbac -n "spn-keyvault-devops")

echo $SPN

SPN_APPID=$(echo $SPN | jq .appId)

SPN_ID=$(az ad sp list --display-name "spn-keyvault-devops" --query [0].objectId --out tsv)
<!-- SPN_ID=$(az ad sp show --id $SPN_APPID --query objectId --out tsv) -->

az role assignment create --role "Key Vault Secrets Officer" \
   --scope $KEYVAULT_ID \
   --assignee-object-id $SPN_ID