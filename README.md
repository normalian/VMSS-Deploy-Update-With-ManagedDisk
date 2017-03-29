# How to deploy and update VMSS with custom images using Managed Disk
1. Create custom images with Managed Disks
1. Deploy your custom image using ARM template in this repository, and you can create a VMSS
1. Create a new custom image with Managed Disks for updating the VMSS
1. Update the VMSS with a PowerShell script in this repository

## 1. Create custom images with Managed Disks
Watch https://docs.microsoft.com/en-us/azure/storage/storage-managed-disks-overview#images for creating your custom images.
After creating the custom image, take its resource id. The id will be needed when you deploy VMSS using ARM templates.


## 2. Deploy your custom image using ARM template in this repository, and you can create a VMSS
At first, edit parameters and variables in azuredeploy.json and azuredeploy.parameters.json. 

Update below variables in azuredeploy.json, especially put the resource id into "ManageddiskCustomimageId".

    "VMSSPrefix": "[YOUR SUBNET PREFIX. SUCH LIKE vmss-linux]",
    "existingVNETName": "[YOUR SUBNET PREFIX. SUCH LIKE vmss-vnet]",
    "addressPrefix": "[YOUR SUBNET PREFIX. SUCH LIKE 172.19.0.0/16]",
    "subnetName": "[YOUR SUBNET PREFIX. SUCH LIKE dmz-subnet]",
    "subnetPrefix": "[YOUR SUBNET PREFIX. SUCH LIKE 172.19.0.0/24]",
    "ManageddiskCustomimageId": "/subscriptions/[YOUR SUBSCRIPTION ID]/resourceGroups/[YOUR RESOURCEGROUP NAME]/providers/Microsoft.Compute/images/[YOUR IMAGE NAME]",


Update below parameters in azuredeploy.parameters.json.

    "parameters": {
      "instanceCountPerVMSS": {
        "value": 5
      },
      "adminUsername": {
        "value": "[YOUR OS ADMIN USER]"
      },
      "adminPassword": {
        "value": "[YOUR OS ADMIN USER PASSWORD]"
      }
    }

After updating azuredeploy.json and azuredeploy.parameters.json, deploy VMSS with below PowerShell script.

    $rgName = "[your resource group name]"
    $vmssname = "[your vmss name]"
    
    Test-AzureRmResourceGroupDeployment -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json -ResourceGroupName $rgName -Debug
    New-AzureRmResourceGroupDeployment -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json -ResourceGroupName $rgName


## 3. Create a new custom image with Managed Disks for updating the VMSS
Same procedure with #1.

## 4. Update the VMSS with a PowerShell script in this repository
Run below script to update VMSS.

    $rgName = "[your resource group name]"
    $vmssname = "[your vmss name]"
    $manageddiskCustomimageId = "/subscriptions/[YOUR SUBSCRIPTION ID]/resourceGroups/[YOUR RESOURCEGROUP NAME]/providers/Microsoft.Compute/images/[YOUR IMAGE NAME]"
    
    $vmss = Get-AzureRmVmss -ResourceGroupName $rgname -VMScaleSetName $vmssname
    $vmss.virtualMachineProfile.storageProfile.ImageReference.Id = $manageddiskCustomimageId
    Update-AzureRmVmss -ResourceGroupName $rgname -Name $vmssname -VirtualMachineScaleSet $vmss

After running the script, watch your vmss via Azure portal. The script was succeeded if you can watch "LATEST MODEL" of your vmss are "No" via Azure portal like below.
![VMSS Model Update](https://raw.githubusercontent.com/normalian/VMSS-Deploy-Update-With-ManagedDisk/master/fig.png "VMSS Model Update")

Upgrade your instances to "LATEST MODEL" via Azure portal. Your update will be finished when "LATEST MODEL" of all instances are "Yes".


