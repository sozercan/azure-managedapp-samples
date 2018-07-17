# Managed ACS Engine with Azure management services

>Note: This sample is for Managed Application in Service Catalog. For Marketplace, please see these instructions:
[**Marketplace Managed Application**](https://docs.microsoft.com/en-us/azure/managed-applications/publish-marketplace-app)

## Deploy this sample to your Service Catalog

### Deploy using AzureCLI

Modify the snippets below to deploy Managed Application definition to a Resource Group in your Azure subscription

```azureCLI
./acs-engine generate apimodel.json
```

```azureCLI
EMAIL=<your email>
userid=$(az ad user show --upn-or-object-id $EMAIL --query objectId --output tsv)
roleid=$(az role definition list --name Owner --query [].name --output tsv)
```

```azureCLI
RESOURCE_GROUP=<your resource group name>
LOCATION=<your location name>
az group create --name $RESOURCE_GROUP --location $LOCATION
```

#### Masters + Agents

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

```azureCLI
managedGroupId=/subscriptions/$subid/resourceGroups/acsEngineGroup
appid=$(az managedapp definition show --name ManagedACSEngine --resource-group $RESOURCE_GROUP --query id --output tsv)
```

```azureCLI
az managedapp create \
  --name managedACSEngine \
  --location <rgLocation> \
  --kind "Servicecatalog" \
  --resource-group <yourRgName> \
  --managedapp-definition-id $appid \
  --managed-rg-id $managedGroupId \
  --parameters "samples/managed-acs-engine/params.json"
```

#### Masters Only

```azureCLI
az managedapp definition create \
  --name "ManagedACSEngineMasters" \
  --location $LOCATION \
  --resource-group $RESOURCE_GROUP \
  --lock-level ReadOnly \
  --display-name "Managed ACS Engine (Masters Only)" \
  --description "Managed ACS Engine (Masters Only)" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "https://github.com/sozercan/azure-managedapp-samples/raw/acsengine/samples/managed-acs-engine/mastersonly/managedacsengine-mastersonly.zip"
```

```azureCLI
managedGroupId=/subscriptions/$subid/resourceGroups/masters1
appid=$(az managedapp definition show --name ManagedACSEngineMasters --resource-group $RESOURCE_GROUP --query id --output tsv)
```

```azureCLI
az managedapp create \
  --name managedACSEngineMasters1 \
  --location $LOCATION \
  --kind "Servicecatalog" \
  --resource-group $RESOURCE_GROUP \
  --managedapp-definition-id $appid \
  --managed-rg-id $managedGroupId \
  --parameters "samples/managed-acs-engine/params.json"
```

#### Agent Pools

```azureCLI
AGENT_RESOURCE_GROUP=<your resource group name>

az group create --name $AGENT_RESOURCE_GROUP --location $LOCATION
```

```azureCLI
az group deployment create \
  --resource-group "$AGENT_RESOURCE_GROUP" \
  --template-file "./samples/managed-acs-engine/agentsonly/mainTemplate-agentonly.json" \
  --parameters "./samples/managed-acs-engine/azuredeploy.parameters.json"
```