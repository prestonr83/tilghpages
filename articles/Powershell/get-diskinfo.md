# Get All Disk Info for Storage Node

```powershell
#Requires -Modules ImportExcel
#Requires -RunAsAdministrator

$node = get-storagenode -Name ([System.Net.Dns]::GetHostByName(($env:computerName))).Hostname
$reportPath = $env:TEMP



#### DONT EDIT BELOW ####

### Populate Physical Disks from Node
if($node.count -gt 1){
$phydisks = @()
foreach($i in $node){
$phydisks += Get-PhysicalDisk -StorageNode $i
}

}
else{
$phydisks = Get-PhysicalDisk -StorageNode $node
}

### Get disks from DISKPART
$dpscript = New-TemporaryFile 
Add-Content $dpscript "LIST VOLUME"
$dpvols = & diskpart /s $dpscript.FullName
$line = 8
$dpdata = @()
do{
    if(($dpvols[$line] -split " ").Where({$_ -ne ""})[0] -eq'Volume'){
        $vol = ($dpvols[$line] -split " ").Where({$_ -ne ""})
        if(!($dpvols[($line + 1)].StartsWith('  Volume')) -and $dpvols[($line + 1)].Length -gt 0 ){
            $mount = $dpvols[($line + 1)]
            $line += 2
        }
        else{
            $mount = ($vol -split " ").Where({$_ -ne ""})[2]
            $line += 1
        }
        $dpscript = New-TemporaryFile
        Add-Content $dpscript "SELECT $($vol[0]) $($vol[1])"
        Add-Content $dpscript "DETAIL VOLUME"
        $detail = & diskpart /s $dpscript.FullName
        $volType = $vol[5]
        $volSize = "$($vol[6])$($vol[7])"
        if($vol.count -eq 7){
            $volType = $vol[4]
            "$($vol[5])$($vol[6])"
        }
        foreach($disk in $detail | ? { $_ -like "*Disk [0-9]*"}){
            $disk = ($disk -split " ").Where({$_ -ne ""})
            $diskID = $disk[1]
            if($disk[0] -eq "*"){
                $diskID = $disk[2]
            }
            $dpdata += [pscustomobject]@{
                Volume = $vol[1]
                Name   = $vol[2]
                Mount  = $mount.Trim(" ")
                Type   = $vol[4]
                Size   = "$($vol[5])$($vol[6])"
                DiskId   = $diskID
            }
        }
    }
    else {
        break
    }
}While ($true)


### MAP together info with WMI

$diskInfo = @()
$wmiDisks = Get-WmiObject win32_diskdrive
Foreach($wmiDisk in $wmiDisks){
    $deviceID = $wmiDisk.DeviceID.Trim("\\.\PHYSICALDRIVE")
    $type = $wmiDisk.caption
    $sn = $wmiDisk.serialnumber
    $phDisk = $phydisks | ? { $_.deviceid -eq $deviceID }
    $lun = ""
    if($phDisk){
        $lun = ($phDisk.PhysicalLocation -split "LUN ")[1]
    }
    $virtualID = ""
    $virtualDiskName = ""
    $columns = ""
    $mount = ($dpdata | ? {$_.DiskId -eq $deviceID}).Mount
    if($sn){
        $virtualDisk = Get-VirtualDisk | ? {$_.ObjectId -like "*$($sn)*" }
        $physicalDisk = $virtualDisk | Get-PhysicalDisk
        $virtualID = $wmiDisk.DeviceID.Trim("\\.\PHYSICALDRIVE")
        $deviceID = $physicalDisk.DeviceId
        $virtualDiskName = $virtualDisk.FriendlyName
        $mount = ($dpdata | ? {$_.DiskId -eq $virtualID}).Mount
        $columns = $virtualDisk.NumberOfColumns
        foreach($pdrive in $physicalDisk){
            $deviceID = $pdrive.DeviceId
            $phDisk = $phydisks | ? { $_.deviceid -eq $deviceID }
            if($phDisk){
                $lun = ($phDisk.PhysicalLocation -split "LUN ")[1]
            }
            $diskInfo += [pscustomobject]@{
                DeviceID        = $deviceID
                LUN             = $lun
                Type            = $type
                SerialNumber    = $sn
                VirtualID       = $virtualID
                VirtualDiskName = $virtualDiskName
                Columns         = $columns
                Mount           = $mount
            }

        }

    }
    else {
        $diskInfo += [pscustomobject]@{
            DeviceID        = $deviceID
            LUN             = $lun
            Type            = $type
            SerialNumber    = $sn
            VirtualID       = $virtualID
            VirtualDiskName = $virtualDiskName
            Columns         = $columns
            Mount           = $mount
        }
    }
}


#Export Report
$fdate = (Get-date).ToString("yyyyMMdd")

$dpdata | Export-Excel "$reportPath\DiskReport_$fdate.xlsx" -AutoSize -TableName "DISKPART_INFO" -WorksheetName 'DISKPART INFO' -Title "DISKPART INFO"
$diskInfo | Export-Excel "$reportPath\DiskReport_$fdate.xlsx" -AutoSize -TableName "VOLUME_INFO" -WorksheetName 'VOLUME INFO' -Title "VOLUME INFO"

Write-host "Report saved to $reportPath\DiskReport_$fdate.xlsx"
```