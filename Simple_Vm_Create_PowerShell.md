# Create a Windows virtual machine in Azure with PowerShell

## Launch Azure Cloud Shell
The Azure Cloud Shell is a free interactive shell that you can use to run the steps in this article. It has common Azure tools preinstalled and configured to use with your account.

To open the Cloud Shell, just select Try it from the upper right corner of a code block. You can also launch Cloud Shell in a separate browser tab by going to https://shell.azure.com/powershell. Select Copy to copy the blocks of code, paste it into the Cloud Shell, and press enter to run it.

## Create resource group
Create an Azure resource group with New-AzResourceGroup. A resource group is a logical container into which Azure resources are deployed and managed.
``` New-AzResourceGroup -Name myResourceGroup -Location EastUS ```
## Create virtual machine
Create a VM with New-AzVM. Provide names for each of the resources and the New-AzVM cmdlet creates if they don't already exist.

When prompted, provide a username and password to be used as the sign-in credentials for the VM:
``` New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80,3389
```
### Connect to virtual machine
After the deployment has completed, RDP to the VM. To see your VM in action, the IIS web server is then installed.

To see the public IP address of the VM, use the Get-AzPublicIpAddress cmdlet:
``` Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress" ```

Use the following command to create a remote desktop session from your local computer. Replace the IP address with the public IP address of your VM.
``` mstsc /v:publicIpAddress ```
In the Windows Security window, select More choices, and then select Use a different account. Type the username as localhost\username, enter password you created for the virtual machine, and then click OK.

You may receive a certificate warning during the sign-in process. Click Yes or Continue to create the connection

## Install web server
To see your VM in action, install the IIS web server. Open a PowerShell prompt on the VM and run the following command:
``` Install-WindowsFeature -name Web-Server -IncludeManagementTools ```

## Start a VM
``` Start-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" ``
   
## Stop a VM
 ``` Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force ``
