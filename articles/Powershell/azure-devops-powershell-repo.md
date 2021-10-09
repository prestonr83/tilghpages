# Publish and Connect to Azure Devops Package Repo for Powershell

## Use Nuget to package and publish your module

```powershell
nuget sources Add -Name "PowershellModules" -Source "https://pkgs.dev.azure.com/<ADO ORG NAME>/<ADO PROJECT NAME>/_packaging/<FEED NAME>/nuget/v3/index.json" -username "ADO USERNAME" -password "ADO PAT"
nuget pack <NAME OF NUSPEC FILE>.nuspec
nuget push -Source "PowershellModules" -ApiKey AzureDevOpsServices "<NAME OF NUPKG>.nupkg"
```

## Add Ado Artifact feed as powershell repo

```powershell
$patToken = "PUT TOKEN HERE" | ConvertTo-SecureString -AsPlainText -Force

$credsAzureDevopsServices = New-Object System.Management.Automation.PSCredential("<ADO USERNAME>", $patToken)

Register-PSRepository -Name "PowershellAzureDevopsServices" -SourceLocation "https://pkgs.dev.azure.com/<ADO ORG NAME>/<ADO PROJECT NAME>/_packaging/<FEED NAME>/nuget/v2" -PublishLocation "https://pkgs.dev.azure.com/<ADO ORG NAME>/<ADO PROJECT NAME>/_packaging/<FEED NAME>/nuget/v2" -InstallationPolicy Trusted -Credential $credsAzureDevopsServices

Install-Module -Name <MODULE NAME IN REPO> -Repository PowershellAzureDevopsServices -Credential $credsAzureDevopsServices
```