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
