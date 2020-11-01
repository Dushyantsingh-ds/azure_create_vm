# azure_create_vm

## Launch Azure Cloud Shell
The Azure Cloud Shell is a free interactive shell that you can use to run the steps in this article. It has common Azure tools preinstalled and configured to use with your account.

To open the Cloud Shell, just select Try it from the upper right corner of a code block. You can also launch Cloud Shell in a separate browser tab by going to https://shell.azure.com/powershell. Select Copy to copy the blocks of code, paste it into the Cloud Shell, and press enter to run it.

## Create a resource group
In Azure, all resources are allocated in a resource management group. Resource groups provide logical groupings of resources that make them easier to work with as a collection. For this tutorial, all of the created resources go into a single group named TutorialResources.

``` New-AzResourceGroup -Name TutorialResources -Location eastus ```

## Create admin credentials for the VM
Before you can create a new virtual machine, you must create a credential object containing the username and password for the administrator account of the Windows VM.

``` $cred = Get-Credential -Message "Enter a username and password for the virtual machine." ```

## Create a virtual machine
Virtual machines in Azure have a large number of dependencies. The Azure PowerShell creates these resources for you based on the command-line arguments you specify. For readability, we are using PowerShell splatting to pass parameters to the Azure PowerShell cmdlets.

Create a new virtual machine running Windows.

``` 
$vmParams = @{
  ResourceGroupName = 'TutorialResources'
  Name = 'TutorialVM1'
  Location = 'eastus'
  ImageName = 'Win2016Datacenter'
  PublicIpAddressName = 'tutorialPublicIp'
  Credential = $cred
  OpenPorts = 3389
}
$newVM1 = New-AzVM @vmParams
```
As the VM is created, you see the parameter values used and Azure resources being created. PowerShell will display a progress bar as shown below.
``` Creating Azure resources
  39% \
  [ooooooooooooooooooooooooooooooooooo                                                                 ]

  Creating TutorialVM1 virtual machine. ```

Once the VM is ready, we can view the results in the Azure Portal or by inspecting the $newVM1 variable.
``` $newVM1 ```

OUTPUT
``` 
ResourceGroupName : TutorialResources
Id                : /subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/TutorialResources/providers/Microsoft.Compute/virtualMachines/TutorialVM1
VmId              : 12345678-9abc-def0-1234-56789abcedf0
Name              : TutorialVM1
Type              : Microsoft.Compute/virtualMachines
Location          : eastus
Tags              : {}
HardwareProfile   : {VmSize}
NetworkProfile    : {NetworkInterfaces}
OSProfile         : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState : Succeeded
StorageProfile    : {ImageReference, OsDisk, DataDisks} 
```
Property values listed inside of braces are nested objects. In the next step we will show you how to view specific values in these nested objects.


## Get VM information with queries
Let's get some more detailed information from the VM we just created. In this example, we verify the Name of the VM and the admin account we created.
``` $newVM1.OSProfile | Select-Object ComputerName,AdminUserName ```

Output
``` ComputerName AdminUsername
------------ -------------
TutorialVM1  tutorialAdmin ``

We can use other Azure PowerShell commands to get specific information about the network configuration.

```
$newVM1 | Get-AzNetworkInterface |
  Select-Object -ExpandProperty IpConfigurations |
    Select-Object Name,PrivateIpAddress
    ```
Output
In this example we are using the PowerShell pipeline to send the $newVM1 object to the Get-AzNetworkInterface cmdlet. From the resulting network interface object we are selecting the nested IpConfigurations object. From the IpConfigurations object we are selecting the Name and PrivateIpAddress properties.
```
Name        PrivateIpAddress
----        ----------------
TutorialVM1 192.168.1.4
```
To confirm that the VM is running, we need to connect via Remote Desktop. For that, we need to know the Public IP address.
```
$publicIp = Get-AzPublicIpAddress -Name tutorialPublicIp -ResourceGroupName TutorialResources
$publicIp | Select-Object Name,IpAddress,@{label='FQDN';expression={$_.DnsSettings.Fqdn}}
```
In this example, we use the Get-AzPublicIpAddress and store the results in the $publicIp variable. From this variable we are selecting properties and using an expression to retrieve the nested Fqdn property.
```
Name             IpAddress           FQDN
----             ---------           ----
tutorialPublicIp <PUBLIC_IP_ADDRESS> tutorialvm1-8a0999.eastus.cloudapp.azure.com
```
### From your local machine you can run the following command to connect to the VM over Remote Desktop.

```
mstsc.exe /v <PUBLIC_IP_ADDRESS>
```
---------------------------------------------------------------------------
## Creating a new VM on the existing subnet
The second VM uses the existing subnet.
```
$vm2Params = @{
  ResourceGroupName = 'TutorialResources'
  Name = 'TutorialVM2'
  ImageName = 'Win2016Datacenter'
  VirtualNetworkName = 'TutorialVM1'
  SubnetName = 'TutorialVM1'
  PublicIpAddressName = 'tutorialPublicIp2'
  Credential = $cred
  OpenPorts = 3389
}
$newVM2 = New-AzVM @vm2Params

$newVM2
```
Output

```
ResourceGroupName        : TutorialResources
Id                       : /subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/TutorialResources/providers/Microsoft.Compute/virtualMachines/TutorialVM2
VmId                     : 12345678-9abc-def0-1234-56789abcedf1
Name                     : TutorialVM2
Type                     : Microsoft.Compute/virtualMachines
Location                 : eastus
Tags                     : {}
HardwareProfile          : {VmSize}
NetworkProfile           : {NetworkInterfaces}
OSProfile                : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState        : Succeeded
StorageProfile           : {ImageReference, OsDisk, DataDisks}
FullyQualifiedDomainName : tutorialvm2-dfa5af.eastus.cloudapp.azure.com
```
You can skip a few steps to get the public IP address of the new VM since it's returned in the FullyQualifiedDomainName property of the $newVM2 object. Use the following command to connect using Remote Desktop.

```
mstsc.exe /v $newVM2.FullyQualifiedDomainName
```

--------------------------------------------------
# Using Sub/Vnet Create Vm
# azure_create_vm


## Launch Azure Cloud Shell
The Azure Cloud Shell is a free interactive shell that you can use to run the steps in this article. It has common Azure tools preinstalled and configured to use with your account.

To open the Cloud Shell, just select Try it from the upper right corner of a code block. You can also launch Cloud Shell in a separate browser tab by going to https://shell.azure.com/powershell. Select Copy to copy the blocks of code, paste it into the Cloud Shell, and press enter to run it.

## Create a resource group
In Azure, all resources are allocated in a resource management group. Resource groups provide logical groupings of resources that make them easier to work with as a collection. For this tutorial, all of the created resources go into a single group named TutorialResources.

``` New-AzResourceGroup -Name TutorialResources -Location eastus ```

## Create admin credentials for the VM
Before you can create a new virtual machine, you must create a credential object containing the username and password for the administrator account of the Windows VM.

``` $cred = Get-Credential -Message "Enter a username and password for the virtual machine." ```

## Create a virtual machine
Virtual machines in Azure have a large number of dependencies. The Azure PowerShell creates these resources for you based on the command-line arguments you specify. For readability, we are using PowerShell splatting to pass parameters to the Azure PowerShell cmdlets.

Create a new virtual machine running Windows.

``` 
$vmParams = @{
  ResourceGroupName = 'TutorialResources'
  Name = 'TutorialVM1'
  Location = 'eastus'
  ImageName = 'Win2016Datacenter'
  PublicIpAddressName = 'tutorialPublicIp'
  Credential = $cred
  OpenPorts = 3389
}
$newVM1 = New-AzVM @vmParams
```
As the VM is created, you see the parameter values used and Azure resources being created. PowerShell will display a progress bar as shown below.
``` Creating Azure resources
  39% \
  [ooooooooooooooooooooooooooooooooooo                                                                 ]

  Creating TutorialVM1 virtual machine. ```

Once the VM is ready, we can view the results in the Azure Portal or by inspecting the $newVM1 variable.
``` $newVM1 ```

OUTPUT
``` 
ResourceGroupName : TutorialResources
Id                : /subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/TutorialResources/providers/Microsoft.Compute/virtualMachines/TutorialVM1
VmId              : 12345678-9abc-def0-1234-56789abcedf0
Name              : TutorialVM1
Type              : Microsoft.Compute/virtualMachines
Location          : eastus
Tags              : {}
HardwareProfile   : {VmSize}
NetworkProfile    : {NetworkInterfaces}
OSProfile         : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState : Succeeded
StorageProfile    : {ImageReference, OsDisk, DataDisks} 
```
Property values listed inside of braces are nested objects. In the next step we will show you how to view specific values in these nested objects.


## Get VM information with queries
Let's get some more detailed information from the VM we just created. In this example, we verify the Name of the VM and the admin account we created.
``` $newVM1.OSProfile | Select-Object ComputerName,AdminUserName ```

Output
``` ComputerName AdminUsername
------------ -------------
TutorialVM1  tutorialAdmin ``

We can use other Azure PowerShell commands to get specific information about the network configuration.

```
$newVM1 | Get-AzNetworkInterface |
  Select-Object -ExpandProperty IpConfigurations |
    Select-Object Name,PrivateIpAddress
    ```
Output
In this example we are using the PowerShell pipeline to send the $newVM1 object to the Get-AzNetworkInterface cmdlet. From the resulting network interface object we are selecting the nested IpConfigurations object. From the IpConfigurations object we are selecting the Name and PrivateIpAddress properties.
```
Name        PrivateIpAddress
----        ----------------
TutorialVM1 192.168.1.4
```
To confirm that the VM is running, we need to connect via Remote Desktop. For that, we need to know the Public IP address.
```
$publicIp = Get-AzPublicIpAddress -Name tutorialPublicIp -ResourceGroupName TutorialResources
$publicIp | Select-Object Name,IpAddress,@{label='FQDN';expression={$_.DnsSettings.Fqdn}}
```
In this example, we use the Get-AzPublicIpAddress and store the results in the $publicIp variable. From this variable we are selecting properties and using an expression to retrieve the nested Fqdn property.
```
Name             IpAddress           FQDN
----             ---------           ----
tutorialPublicIp <PUBLIC_IP_ADDRESS> tutorialvm1-8a0999.eastus.cloudapp.azure.com
```
### From your local machine you can run the following command to connect to the VM over Remote Desktop.

```
mstsc.exe /v <PUBLIC_IP_ADDRESS>
```
---------------------------------------------------------------------------
## Creating a new VM on the existing subnet
The second VM uses the existing subnet.
```
$vm2Params = @{
  ResourceGroupName = 'TutorialResources'
  Name = 'TutorialVM2'
  ImageName = 'Win2016Datacenter'
  VirtualNetworkName = 'TutorialVM1'
  SubnetName = 'TutorialVM1'
  PublicIpAddressName = 'tutorialPublicIp2'
  Credential = $cred
  OpenPorts = 3389
}
$newVM2 = New-AzVM @vm2Params

$newVM2
```
Output

```
ResourceGroupName        : TutorialResources
Id                       : /subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/TutorialResources/providers/Microsoft.Compute/virtualMachines/TutorialVM2
VmId                     : 12345678-9abc-def0-1234-56789abcedf1
Name                     : TutorialVM2
Type                     : Microsoft.Compute/virtualMachines
Location                 : eastus
Tags                     : {}
HardwareProfile          : {VmSize}
NetworkProfile           : {NetworkInterfaces}
OSProfile                : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState        : Succeeded
StorageProfile           : {ImageReference, OsDisk, DataDisks}
FullyQualifiedDomainName : tutorialvm2-dfa5af.eastus.cloudapp.azure.com
```
You can skip a few steps to get the public IP address of the new VM since it's returned in the FullyQualifiedDomainName property of the $newVM2 object. Use the following command to connect using Remote Desktop.

```
mstsc.exe /v $newVM2.FullyQualifiedDomainName
```







