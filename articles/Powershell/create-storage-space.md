# Create Storage Space

```powershell
# Get disk IDs
Get-StoragePool -IsPrimordial $true | Get-PhysicalDisk | Where-Object CanPool -eq $True | select-object DeviceId, FriendlyName, @{Name="Size";Expression={$_.size/1GB}}, PhysicalLocation  | Sort-Object -Property PhysicalLocation

# Create Storage pool Example with 3 disk
$disks = get-physicaldisk |? {$_.DeviceId -in 2,3,4}
New-StoragePool –FriendlyName DataPool1 –StorageSubsystemFriendlyName (Get-StorageSubSystem).FriendlyName –PhysicalDisks $disks

# Create Virtual Disk Example using 3 columns and 64k Interleave
New-VirtualDisk -FriendlyName Data1 -StoragePoolFriendlyName DataPool1 -ProvisioningType Fixed -ResiliencySettingName Simple -NumberOfColumns 3 -Interleave 65536 –UseMaximumSize # 64K Interleave

# Create Volume Example using 64k Allocation Units
Get-VirtualDisk –FriendlyName DATA1 | Get-Disk | Initialize-Disk –Passthru | New-Partition –AssignDriveLetter –UseMaximumSize | Format-Volume -NewFileSystemLabel DATA1 -AllocationUnitSize 65536 # 64k Alloc
```
# Extending a Storage Pool

```powershell
# Extend Pool
$disks = get-physicaldisk |? {$_.DeviceId -in 5,6,7} # Set to the Device IDs of the disks
Add-PhysicalDisk -PhysicalDisks $disks -StoragePoolFriendlyName DataPool1

# Extend Virtual Disk
Get-VirtualDisk Data1 | Resize-VirtualDisk -Size <NEW MAX SIZE>

# Extend Volume
$Partition = Get-Volume -FileSystemLabel DATA1 | get-partition
$Partition | Resize-Partition -Size ($partition |Get-PartitionSupportedSize).sizemax
```
