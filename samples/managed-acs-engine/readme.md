# Managed ACS Engine with Azure management services

>Note: This sample is for Managed Application in Service Catalog. For Marketplace, please see these instructions:
[**Marketplace Managed Application**](https://docs.microsoft.com/en-us/azure/managed-applications/publish-marketplace-app)

## Deploy this sample to your Service Catalog

### Deploy using AzureCLI

Modify the snippet below to deploy Managed Application definition to a Resource Group in your Azure subscription

```azureCLI
az managedapp definition create \
  --name "ManagedACSEngine" \
  --location <rgLocation> \
  --resource-group <yourRgName> \
  --lock-level ReadOnly \
  --display-name "Managed ACS Engine" \
  --description "Managed ACS Engine" \
  --authorizations "<userOrGroupId>:<RBACRoleDefinitionId>" \
  --package-file-uri "https://github.com/sozercan/azure-managedapp-samples/raw/acsengine/samples/managed-acs-engine/managedacsengine.zip"
```

az managedapp create \
  --name managedACSEngine \
  --location <rgLocation> \
  --kind "Servicecatalog" \
  --resource-group <yourRgName> \
  --managedapp-definition-id $appid \
  --managed-rg-id $managedGroupId \
  --parameters "samples/managed-acs-engine/azuredeploy.parameters.json"


![alt text](images/appliance.png "Azure Managed Application")