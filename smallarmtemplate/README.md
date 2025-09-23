# Azure ARM Template - Small Infrastructure

This ARM template deploys a simple Azure infrastructure containing:

- 1 Virtual Network (10.0.0.0/16)
- 2 Subnets:
  - Subnet: 10.0.0.0/24
  - Subnet2: 10.0.1.0/24
- 1 Storage Account (Standard LRS)
- 1 Virtual Machine (Standard_B1s)
- Supporting resources:
  - Network Security Group (with RDP rule)
  - Public IP Address
  - Network Interface

## Template Resources

| Resource Type | Resource Name | Description |
|---------------|---------------|-------------|
| Virtual Network | MyVNET | Main virtual network with address space 10.0.0.0/16 |
| Subnets | Subnet, Subnet2 | Two subnets within the virtual network |
| Storage Account | [unique] | General-purpose storage account for VM diagnostics |
| Virtual Machine | simple-vm | Windows Server VM with Standard_B1s size |
| Network Security Group | default-NSG | Security rules for the subnets |
| Public IP | myPublicIP | Dynamic public IP for VM access |
| Network Interface | myVMNic | Network interface connecting VM to subnet |

## Parameters

- `adminUsername`: Administrator username for the VM
- `adminPassword`: Administrator password for the VM (must be at least 12 characters)
- `vmName`: Name of the virtual machine (default: simple-vm)
- `virtualNetworkName`: Name of the virtual network (default: MyVNET)
- `subnetName`: Name of the first subnet (default: Subnet)
- `subnet2Name`: Name of the second subnet (default: Subnet2)
- `storageAccountName`: Name of the storage account (auto-generated unique name)
- `OSVersion`: Windows Server version (default: 2022-datacenter-azure-edition)
- `vmSize`: VM size (default: Standard_B1s)
- `location`: Location for all resources (default: resource group location)

## Deployment

### Using Azure CLI

```bash
az group create --name myResourceGroup --location eastus

az deployment group create \
  --resource-group myResourceGroup \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json
```

### Using PowerShell

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "East US"

New-AzResourceGroupDeployment `
  -ResourceGroupName "myResourceGroup" `
  -TemplateFile "azuredeploy.json" `
  -TemplateParameterFile "azuredeploy.parameters.json"
```

### Using Azure Portal

1. Click the "Deploy to Azure" button (if configured)
2. Fill in the required parameters
3. Review and create the deployment

## Notes

- The VM will have RDP (port 3389) access enabled through the Network Security Group
- Boot diagnostics are enabled and use the created storage account
- The public IP address is dynamic by default
- The storage account name must be globally unique
- Both subnets use the same Network Security Group for simplicity

## Outputs

- `hostname`: The fully qualified domain name of the VM's public IP address

## Cost Considerations

- Standard_B1s VM: Low-cost burstable VM suitable for light workloads
- Standard LRS storage: Most cost-effective storage option
- Dynamic public IP: No charge when VM is deallocated

Remember to deallocate the VM when not in use to minimize costs.

## Notes:
Starting CloudShell in new account results in this message:
```
Subscription used to launch your CloudShell 51c75c0b-6e17-4ba3-bbfa-eecef136d676 is not registered to Microsoft.CloudShell Namespace. Please follow these instructions "https://aka.ms/RegisterCloudShell" to register. In future, unregistered subscriptions will have restricted access to CloudShell service.

Your Cloud Shell session will be ephemeral so no files or system changes will persist beyond your current session.
```
Working on this tutorial from Pluralsight:
https://app.pluralsight.com/hands-on/labs/3812f2d3-ede3-48f6-8512-aea20cf4801a?ilx=true
Attempted to execute the instructions at the link, but it's already registered.
Attempted to deploy this template by uploading parameters and template to Cloud Shell.
Ran the following command:
```
az deployment group create --resource-group SmallArmTemplate --template-file azuredeploy.json
```
This generated the error.
```
The content for this response was already consumed.
```
Ran this debugging command:
```
az deployment group what-if --resource-group SmallArmTemplate --template-file azuredeploy.json --parameters azuredeploy.parameters.json 
```
Got this output:
```
InvalidTemplateDeployment - The template deployment 'azuredeploy' is not valid according to the validation procedure. The tracking id is 'e8e495c0-3273-4906-adef-6f7ef54c0abd'. See inner errors for details.
SkuNotAvailable - The requested VM size for resource 'Following SKUs have failed for Capacity Restrictions: Standard_B1s' is currently not available in location 'southcentralus'. Please try another size or deploy to a different location or different zone. See https://aka.ms/azureskunotavailable for details.
```
Get a command to list the SKUs in a given zone.
```
az vm list-skus --location southcentralus
```
Maybe it's that I'm not allowed to use that region at all.
Going through the screen to create a VM in the Console to figure out what regions and instance types it suggests.
The regions it suggests are:
(US) East US 2 | eastus2
(US) East US 3 | eastus3
(US) West US   | westus
Figuring out the command-line symbols for those regions
Worked my way to the following command-line:
```
az deployment group create --template-file azuredeploy.json --parameters azuredeploy.parameters.json --resource-group SmallArmTemplate
```
Which generates the following error:
```
The content for this response was already consumed
```
And when I do:
```
az deployment group what-if --template-file azuredeploy.json --parameters azuredeploy.parameters.json --resource-group SmallArmTemplate
```
Then I get...
```
InvalidTemplateDeployment - The template deployment 'azuredeploy' is not valid according to the validation procedure. The tracking id is 'abac5c0e-7a5a-40c6-8163-d04c841a992e'. See inner errors for details.
SkuNotAvailable - The requested VM size for resource 'Following SKUs have failed for Capacity Restrictions: Standard_B1s' is currently not available in location 'southcentralus'. Please try another size or deploy to a different location or different zone. See https://aka.ms/azureskunotavailable for details.

```
I looked up and found several ways to specify the region. I added "location" to the parameters file. Then I successfully ran the "what-if" command.
```
az deployment group what-if --template-file azuredeploy.json --parameters azuredeploy.parameters.json --resource-group SmallArmTemplate
```
That gave me a non-error output.
But running the actual creation gave me an error.
I asked AI, which corrected "US East" to "East US" as my location parameter.
Then I ran this creation command:
```
az deployment group create --template-file azuredeploy.json --parameters azuredeploy.parameters.json --resource-group SmallArmTemplate
```
I worked through the uniqueness of the storage account name, the uniqueness of the domain name, the need for public IP (no, don't need it).
The following command actually and finally worked.
```
az deployment group create --template-file azuredeploy.json --parameters azuredeploy.parameters.json --resource-group SmallArmTemplate
```
Output
```
{
  "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Resources/deployments/azuredeploy",
  "location": null,
  "name": "azuredeploy",
  "properties": {
    "correlationId": "dce6292d-667c-4404-b4c3-1f8d64f1a062",
    "debugSetting": null,
    "dependencies": [
      {
        "dependsOn": [
          {
            "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Network/networkSecurityGroups/default-NSG",
            "resourceGroup": "SmallArmTemplate",
            "resourceName": "default-NSG",
            "resourceType": "Microsoft.Network/networkSecurityGroups"
          }
        ],
        "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Network/virtualNetworks/MyVNET",
        "resourceGroup": "SmallArmTemplate",
        "resourceName": "MyVNET",
        "resourceType": "Microsoft.Network/virtualNetworks"
      },
      {
        "dependsOn": [
          {
            "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Network/virtualNetworks/MyVNET",
            "resourceGroup": "SmallArmTemplate",
            "resourceName": "MyVNET",
            "resourceType": "Microsoft.Network/virtualNetworks"
          }
        ],
        "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Network/networkInterfaces/myVMNic",
        "resourceGroup": "SmallArmTemplate",
        "resourceName": "myVMNic",
        "resourceType": "Microsoft.Network/networkInterfaces"
      },
      {
        "dependsOn": [
          {
            "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Storage/storageAccounts/mystoragedutchovenbread",
            "resourceGroup": "SmallArmTemplate",
            "resourceName": "mystoragedutchovenbread",
            "resourceType": "Microsoft.Storage/storageAccounts"
          },
          {
            "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Network/networkInterfaces/myVMNic",
            "resourceGroup": "SmallArmTemplate",
            "resourceName": "myVMNic",
            "resourceType": "Microsoft.Network/networkInterfaces"
          },
          {
            "apiVersion": "2022-05-01",
            "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Storage/storageAccounts/mystoragedutchovenbread",
            "resourceGroup": "SmallArmTemplate",
            "resourceName": "mystoragedutchovenbread",
            "resourceType": "Microsoft.Storage/storageAccounts"
          }
        ],
        "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Compute/virtualMachines/simple-vm",
        "resourceGroup": "SmallArmTemplate",
        "resourceName": "simple-vm",
        "resourceType": "Microsoft.Compute/virtualMachines"
      }
    ],
    "diagnostics": null,
    "duration": "PT4.0657626S",
    "error": null,
    "extensions": null,
    "mode": "Incremental",
    "onErrorDeployment": null,
    "outputResources": [
      {
        "apiVersion": null,
        "extension": null,
        "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Compute/virtualMachines/simple-vm",
        "identifiers": null,
        "resourceGroup": "SmallArmTemplate",
        "resourceType": null
      },
      {
        "apiVersion": null,
        "extension": null,
        "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Network/networkInterfaces/myVMNic",
        "identifiers": null,
        "resourceGroup": "SmallArmTemplate",
        "resourceType": null
      },
      {
        "apiVersion": null,
        "extension": null,
        "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Network/networkSecurityGroups/default-NSG",
        "identifiers": null,
        "resourceGroup": "SmallArmTemplate",
        "resourceType": null
      },
      {
        "apiVersion": null,
        "extension": null,
        "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Network/virtualNetworks/MyVNET",
        "identifiers": null,
        "resourceGroup": "SmallArmTemplate",
        "resourceType": null
      },
      {
        "apiVersion": null,
        "extension": null,
        "id": "/subscriptions/51c75c0b-6e17-4ba3-bbfa-eecef136d676/resourceGroups/SmallArmTemplate/providers/Microsoft.Storage/storageAccounts/mystoragedutchovenbread",
        "identifiers": null,
        "resourceGroup": "SmallArmTemplate",
        "resourceType": null
      }
    ],
    "outputs": {
      "vmPrivateIP": {
        "type": "String",
        "value": "10.0.0.4"
      }
    },
    "parameters": {
      "adminPassword": {
        "type": "SecureString"
      },
      "adminUsername": {
        "type": "String",
        "value": "azureuser"
      },
      "location": {
        "type": "String",
        "value": "East US"
      },
      "networkSecurityGroupName": {
        "type": "String",
        "value": "default-NSG"
      },
      "osVersion": {
        "type": "String",
        "value": "2022-datacenter-azure-edition"
      },
      "storageAccountName": {
        "type": "String",
        "value": "mystoragedutchovenbread"
      },
      "subnet2Name": {
        "type": "String",
        "value": "Subnet2"
      },
      "subnetName": {
        "type": "String",
        "value": "Subnet"
      },
      "virtualNetworkName": {
        "type": "String",
        "value": "MyVNET"
      },
      "vmName": {
        "type": "String",
        "value": "simple-vm"
      },
      "vmSize": {
        "type": "String",
        "value": "Standard_B1s"
      }
    },
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.Storage",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              "eastus"
            ],
            "properties": null,
            "resourceType": "storageAccounts",
            "zoneMappings": null
          }
        ]
      },
      {
        "id": null,
        "namespace": "Microsoft.Network",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              "eastus"
            ],
            "properties": null,
            "resourceType": "networkSecurityGroups",
            "zoneMappings": null
          },
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              "eastus"
            ],
            "properties": null,
            "resourceType": "virtualNetworks",
            "zoneMappings": null
          },
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              "eastus"
            ],
            "properties": null,
            "resourceType": "networkInterfaces",
            "zoneMappings": null
          }
        ]
      },
      {
        "id": null,
        "namespace": "Microsoft.Compute",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              "eastus"
            ],
            "properties": null,
            "resourceType": "virtualMachines",
            "zoneMappings": null
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "templateHash": "8349491988158399769",
    "templateLink": null,
    "timestamp": "2025-09-23T19:17:35.076103+00:00",
    "validatedResources": null,
    "validationLevel": null
  },
  "resourceGroup": "SmallArmTemplate",
  "tags": null,
  "type": "Microsoft.Resources/deployments"
}
```
Now I'll try to connect via Azure Bastion.
Bastion is a service which requires its own deployment. It isn't built into the console.
I was able to create a Bastion and connect to the VM. I reached the desktop, but the target didn't play well with my browser.

## Deletion

List the deployments
```
az deployment group list --resource-group SmallArmTemplate -o table
```
Delete the deployments
```
az deployment group delete --resource-group SmallArmTemplate --name azuredeploy
```
