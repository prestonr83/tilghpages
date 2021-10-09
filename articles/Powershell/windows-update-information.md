# Windows Update Information

## List update history

```powershell
get-wmiobject -class win32_quickfixengineering
```

To list last 50 updates including those without hotfix IDs

```powershell
function Convert-WuaResultCodeToName {
    param( [Parameter(Mandatory=$true)]
    [int] $ResultCode
    )
    $Result = $ResultCode
    switch($ResultCode){
        2{
            $Result = "Succeeded"
        }
        3{
            $Result = "Succeeded With Errors"
        }
        4{
        $Result = "Failed"
        }
    }
    return $Result
}
function Get-WuaHistory{
    # Get a WUA Session
    $session = (New-Object -ComObject 'Microsoft.Update.Session')
    # Query the latest 1000 History starting with the first recordp
    $history = $session.QueryHistory("",0,50) | ForEach-Object {
        $Result = Convert-WuaResultCodeToName -ResultCode $_.ResultCode
        # Make the properties hidden in com properties visible.
        $_ | Add-Member -MemberType NoteProperty -Value $Result -Name Result
        $Product = $_.Categories | Where-Object {$_.Type -eq 'Product'} | Select-Object -First 1 -ExpandProperty Name
        $_ | Add-Member -MemberType NoteProperty -Value $_.UpdateIdentity.UpdateId -Name UpdateId
        $_ | Add-Member -MemberType NoteProperty -Value $_.UpdateIdentity.RevisionNumber -Name RevisionNumber
        $_ | Add-Member -MemberType NoteProperty -Value $Product -Name Product -PassThru
        Write-Output $_
    }
    # Remove null records and only return the fields we want
    $history |
    Where-Object {![String]::IsNullOrWhiteSpace($_.title)} |
    Select-Object Result, Date, Title, SupportUrl, Product, UpdateId, RevisionNumber
}

Get-WuaHistory | Format-Table
```

## List Update settings

```powershell
$AutoUpdateNotificationLevels= @{
    0="Not configured"; 
    1="Disabled"; 
    2="Notify before download";
    3="Notify before installation"; 
    4="Scheduled installation"
}

$AutoUpdateDays=@{
    0="Every Day"; 
    1="Every Sunday"; 
    2="Every Monday"; 
    3="Every Tuesday"; 
    4="Every Wednesday";
    5="Every Thursday"; 
    6="Every Friday"; 
    7="Every Saturday"
}


Function Get-WindowsUpdateConfig{
    $AUSettings = (New-Object -com "Microsoft.Update.AutoUpdate").Settings
    $AUObj = New-Object -TypeName System.Object

    Add-Member -inputObject $AuObj -MemberType NoteProperty -Name "NotificationLevel"  `
               -Value $AutoUpdateNotificationLevels[$AUSettings.NotificationLevel]

    Add-Member -inputObject $AuObj -MemberType NoteProperty -Name "UpdateDays" `
               -Value $AutoUpdateDays[$AUSettings.ScheduledInstallationDay]

    Add-Member -inputObject $AuObj -MemberType NoteProperty -Name "UpdateHour"   `
               -Value $AUSettings.ScheduledInstallationTime 

    Add-Member -inputObject $AuObj -MemberType NoteProperty -Name "Recommended updates" `
               -Value $(IF ($AUSettings.IncludeRecommendedUpdates) {"Included"}  else {"Excluded"})
    $AuObj
} 

```