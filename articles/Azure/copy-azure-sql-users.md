# Copy Users in Azure SQL DB

Copying users between Azure SQL DBs isn't supported by DBA Tools and often when copying databases between environments users need to be copied. This script will copy the users or just generate the scripts. There is also a clean up option to additional users not in the source database.

```powershell
#Requires -Modules dbatools

$DeleteExtraAccounts = $true # Set to True if you want to remove user accounts on the destination database that are not in the source database
$generateScripts = $true # Set to True to only generate the scripts needed for changes. This is for issues with 2FA and AAD accounts which is not supported with DBATools.
$DestinationServer = '' # Connection String for the destination server
$SourceServer = '' # Connection String for the source server


$SourceNonPooledConnection = Connect-DbaInstance -SqlInstance $SourceServer -NonPooledConnection
$DestinationNonPooledConnection = Connect-DbaInstance -SqlInstance $DestinationServer -NonPooledConnection

$userQuery = @"
SELECT
    [UserType] = CASE membprinc.[type]
                     WHEN 'S' THEN 'SQL User'
                     WHEN 'U' THEN 'Windows User'
                     WHEN 'G' THEN 'Windows Group'
                     WHEN 'X' THEN 'Azure AD'
					 WHEN 'E' THEN 'Azure AD User'
                 END,
    [DatabaseUserName] = membprinc.[name],
	[AuthType]         = membprinc.[authentication_type_desc],
    [Role]             = roleprinc.[name]
FROM
    --Role/member associations
    sys.database_role_members          AS members
    --Roles
    JOIN      sys.database_principals  AS roleprinc ON roleprinc.[principal_id] = members.[role_principal_id]
    --Role members (database users)
    JOIN      sys.database_principals  AS membprinc ON membprinc.[principal_id] = members.[member_principal_id]
    --Login accounts
    LEFT JOIN sys.server_principals    AS ulogin    ON ulogin.[sid] = membprinc.[sid]
WHERE
    membprinc.[type] IN ('S','U','G','X','E')
    -- No need for these system accounts
    AND membprinc.[name] NOT IN ('sys', 'INFORMATION_SCHEMA')
	AND membprinc.[authentication_type_desc] != 'DATABASE'
"@


$srcUsers = Invoke-DbaQuery -Query $userQuery -SqlInstance $SourceNonPooledConnection -EnableException
$dstUsers = Invoke-DbaQuery -Query $userQuery -SqlInstance $DestinationNonPooledConnection -EnableException
$dstRoles = Invoke-DbaQuery -Query "select name from sys.database_principals" -SqlInstance $DestinationNonPooledConnection -EnableException
$missingRole = $srcUsers.Role | ? {$_ -notin $dstRoles.name}
if($missingRole){
    Write-Warning "Missing Roles on [$($DestinationNonPooledConnection.ComputerName)]$($DestinationNonPooledConnection.ConnectionContext.CurrentDatabase). Users in those roles will not have the permissions needed!"
}
$extraUsers = $dstUsers.DatabaseUserName | select -Unique | ? {$_ -notin $srcUsers.DatabaseUserName}
$newUsers = $srcUsers.DatabaseUserName | select -Unique | ? {$_ -notin $dstUsers.DatabaseUserName}

$scripts = ''
Foreach($user in $newUsers){
    $props = $srcUsers | ? {$_.DatabaseUserName -eq $user}
    if($props[0].AuthType -eq 'EXTERNAL'){
        $newUserQuery = "CREATE USER [$user] FROM  EXTERNAL PROVIDER  WITH DEFAULT_SCHEMA=[dbo];"
    } else {
        $newUserQuery = "CREATE USER [$user] FOR LOGIN [$user] WITH DEFAULT_SCHEMA=[dbo];"
    }
    Write-host "Creating $user on [$($DestinationNonPooledConnection.ComputerName)]$($DestinationNonPooledConnection.ConnectionContext.CurrentDatabase)"
    if($generateScripts){
        $scripts += "$newUserQuery `n"
    } else {
        Invoke-DbaQuery -Query $newUserQuery -SqlInstance $DestinationNonPooledConnection
    }
    foreach($role in $props.Role){
        if($role -in $missingRole){
            Write-Warning "Role $role for user $user is missing on [$($DestinationNonPooledConnection.ComputerName)]$($DestinationNonPooledConnection.ConnectionContext.CurrentDatabase). Skipping role assignment!"
            continue
        }
        $roleQuery = "ALTER ROLE $role ADD MEMBER [$user];"
        Write-host "Adding $user to role $role on [$($DestinationNonPooledConnection.ComputerName)]$($DestinationNonPooledConnection.ConnectionContext.CurrentDatabase)"
        if($generateScripts){
            $scripts += "$roleQuery `n"
        } else {
            Invoke-DbaQuery -Query $roleQuery -SqlInstance $DestinationNonPooledConnection
        }
    }
}

If($DeleteExtraAccounts){
    Foreach($user in $extraUsers){
        $dropUserQuery = "DROP USER [$user];"
        Write-host "Dropping $user on [$($DestinationNonPooledConnection.ComputerName)]$($DestinationNonPooledConnection.ConnectionContext.CurrentDatabase)"
        if($generateScripts){
            $scripts += "$dropUserQuery `n"
        } else {
            Invoke-DbaQuery -Query $dropUserQuery -SqlInstance $DestinationNonPooledConnection
        }
    }
}

if($scripts){
    Write-host "`n`n`n *****Script Generated*****"
    $scripts
}
```