# Managed ACS Engine with Azure management services

>Note: This sample is for Managed Application in Service Catalog. For Marketplace, please see these instructions:
[**Marketplace Managed Application**](https://docs.microsoft.com/en-us/azure/managed-applications/publish-marketplace-app)

## Deploy this sample to your Service Catalog

### Deploy using AzureCLI

Modify the snippets below to deploy Managed Application definition to a Resource Group in your Azure subscription

* Download [ACS-Engine](https://github.com/Azure/acs-engine/releases) binary or build from source.

```azureCLI
./acs-engine generate apimodel.json
```

This will create `./_output/<your dnsPrefix>/` and will output generated templates and certificates in there. For more details, check out [ACS-Engine](https://github.com/Azure/acs-engine).

```azureCLI
DNS_PREFIX=<your dns prefix>
EMAIL=<your AD email (for example, `alias@microsoft.com`) or Service Principal>
userid=$(az ad user show --upn-or-object-id $EMAIL --query objectId --output tsv)
roleid=$(az role definition list --name Owner --query [].name --output tsv)
subid=$(az account show --query id --output tsv)
```

```azureCLI
SERVICE_CATALOG_RESOURCE_GROUP=<resource group name for the service catalog>
LOCATION=<azure region name>
az group create --name $SERVICE_CATALOG_RESOURCE_GROUP --location $LOCATION
```

### Deployment

Select either Masters+Agents or Masters only (followed by non-managed agent pools).

#### Masters + Agents

This section will create both master and agent pools in the same managed app resource group.

```azureCLI
az managedapp definition create \
  --name "ManagedACSEngine" \
  --location $LOCATION \
  --resource-group $SERVICE_CATALOG_RESOURCE_GROUP \
  --lock-level ReadOnly \
  --display-name "Managed ACS Engine (Masters+Agents)" \
  --description "Managed ACS Engine (Masters+Agents)" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "https://github.com/sozercan/azure-managedapp-samples/raw/acsengine/samples/managed-acs-engine/managedacsengine.zip"
```

```azureCLI
MANAGED_APP_RESOURCE_GROUP=<resource group name for the managed app>
PARAMS=$(cat _output/$DNS_PREFIX/azuredeploy.parameters.json | jq '.parameters')

managedGroupId=/subscriptions/$subid/resourceGroups/$MANAGED_APP_RESOURCE_GROUP
appid=$(az managedapp definition show --name ManagedACSEngine --resource-group $SERVICE_CATALOG_RESOURCE_GROUP --query id --output tsv)
```

```azureCLI
az managedapp create \
  --name managedACSEngine \
  --location $LOCATION \
  --kind "Servicecatalog" \
  --resource-group $SERVICE_CATALOG_RESOURCE_GROUP \
  --managedapp-definition-id $appid \
  --managed-rg-id $managedGroupId \
  --parameters $PARAMS
```

#### Masters Only

This section will create masters in a managed app resource group. And then we'll deploy agents in a seperate non-managed resource group in the next section.

```azureCLI
az managedapp definition create \
  --name "ManagedACSEngineMasters" \
  --location $LOCATION \
  --resource-group $SERVICE_CATALOG_RESOURCE_GROUP \
  --lock-level ReadOnly \
  --display-name "Managed ACS Engine (Masters Only)" \
  --description "Managed ACS Engine (Masters Only)" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "https://github.com/sozercan/azure-managedapp-samples/raw/acsengine/samples/managed-acs-engine/mastersonly/managedacsengine-mastersonly.zip"
```

```azureCLI
MANAGED_APP_RESOURCE_GROUP=<resource group name for the managed app>
PARAMS=$(cat _output/$DNS_PREFIX/azuredeploy.parameters.json | jq '.parameters')

managedGroupId=/subscriptions/$subid/resourceGroups/$MANAGED_APP_RESOURCE_GROUP
appid=$(az managedapp definition show --name ManagedACSEngineMasters --resource-group $SERVICE_CATALOG_RESOURCE_GROUP --query id --output tsv)
```

```azureCLI
az managedapp create \
  --name managedACSEngineMasters \
  --location $LOCATION \
  --kind "Servicecatalog" \
  --resource-group $SERVICE_CATALOG_RESOURCE_GROUP \
  --managedapp-definition-id $appid \
  --managed-rg-id $managedGroupId \
  --parameters $PARAMS
```

#### Agent Pools

These are the unmanaged agent pools.

```azureCLI
AGENT_RESOURCE_GROUP=<your resource group name for agent pools>

az group create --name $AGENT_RESOURCE_GROUP --location $LOCATION
```

```azureCLI
az group deployment create \
  --resource-group "$AGENT_RESOURCE_GROUP" \
  --template-file "./samples/managed-acs-engine/agentsonly/mainTemplate-agentonly.json" \
  --parameters $PARAMS
```