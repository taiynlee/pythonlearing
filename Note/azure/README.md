# VM Start
```powershell
# Input Parameters for
# - VmName: name of the vm to perform action to
# - ResourceGroupName: resource group where the vm belongs to
# - VmAction:action to perform (startup or shutdown)
Param(
[string]$VmName,
[string]$ResourceGroupName
)
 
# Authenticate with your Automation Account
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID `
-ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
 
# Startup VM
Start-AzureRmVM -Name $VmName -ResourceGroupName $ResourceGroupName
```
# Analysis Service Pause
### Install Module AzureRM.AnalysisServices is Required
```powershell
Param(
[string]$ResourceGroupName,
[string]$ServerName
)

import-module AzureRM.AnalysisServices

# Authenticate with your Automation Account
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID `
-ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

Suspend-AzureRmAnalysisServicesServer -Name $ServerName -ResourceGroupName $ResourceGroupName
```

# Analysis Service Resume
```powershell
Param(
[string]$ResourceGroupName,
[string]$Name
)
 
# Authenticate with your Automation Account
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID `
-ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

Resume-AzureRmAnalysisServicesServer -Name $Name -ResourceGroupName $ResourceGroupName
```

# Analysis Service Refresh By [TMSL](https://docs.microsoft.com/en-us/sql/analysis-services/tabular-models-scripting-language-commands/refresh-command-tmsl)
### Install Module SqlServer is Required
```powershell
import-module SQLServer

#$Conn = Get-AutomationConnection -Name AzureRunAsConnection
#Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID `
#-ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

$pass="<your_password>"|ConvertTo-SecureString -AsPlainText -Force
$cred = New-Object   System.Management.Automation.PsCredential('<your_account>',$pass)
Login-AzureRmAccount -Credential $cred

Invoke-ASCmd `
    -Server "asazure://southeastasia.asazure.windows.net/<analysis_service_name>" `
    -Database "Object_Level_Security_Sample" `
    -Query "{
              `"refresh`": {
                `"type`": `"full`",
                `"objects`": [
                  {
                    `"database`": `"Object_Level_Security_Sample`",
                    `"table`": `"SanTable`",
                    `"partition`": `"DT_2017`"
                  }
                ]
              }
            }" `
	-Credential $cred
```
### [Anothor Approach By RESTFul](https://docs.microsoft.com/en-us/azure/analysis-services/analysis-services-async-refresh)

# DataWarehouse Pause
```powershell
Param(
[string]$ResourceGroupName,
[string]$ServerName,
[string]$DatabaseName
)
 
# Authenticate with your Automation Account
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID `
-ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

Suspend-AzureRmSqlDatabase –ResourceGroupName $ResourceGroupName `
–ServerName $ServerName –DatabaseName $DatabaseName
```

# DataWarehouse Resume
```powershell
Param(
[string]$ResourceGroupName,
[string]$ServerName,
[string]$DatabaseName
)
 
# Authenticate with your Automation Account
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID `
-ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

Resume-AzureRmSqlDatabase –ResourceGroupName $ResourceGroupName `
–ServerName $ServerName -DatabaseName $DatabaseName
```