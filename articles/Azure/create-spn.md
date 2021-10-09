# Create SPN and Set Permissions for ADO

Create the SPN

```powershell
$DisplayName = "mySPN"
Add-Type -AssemblyName 'System.Web'
$password = "$([System.Web.Security.Membership]::GeneratePassword(20, 5))"
$credentials = New-Object -TypeName Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential -Property @{ StartDate=Get-Date; EndDate=Get-Date -Year 2024; Password=$password}
$spn = New-AzAdServicePrincipal -DisplayName $DisplayName -PasswordCredential $credentials
```

Set permissions on one more resource groups

```powershell
# Define ResourceGroup Permissions
$permissions = @{
    Reader = @('myappRG')
    Contributor = @('StorAcctRG', 'dbRG')
}
# Set Permissions
Foreach($role in $permissions.Keys){
    Foreach($resourceGroup in $permissions.$role){
        New-AzRoleAssignment -ObjectId $spn.Id -RoleDefinitionName $role -ResourceGroupName $resourceGroup
    }
}
```

Output settings for ADO 

```powershell
# Generate ADO Settings
$ctx = get-azcontext
$adoSettings = [PSCustomObject]@{
    'Subscription ID' = $ctx.Subscription.Id
    'Subscription Name' = $ctx.Subscription.Name
    'Service Principal Id' = $spn.ApplicationId
    'Service Principal Key' = $password
    'Tenant ID' = $ctx.Subscription.TenantId
}

# Output ADO Settings
$adoSettings
```