# StoragePoolDSC
Create a storage pool DSC Resource (designed for standing up SQL/File Servers in Azure)

This is a very limited resource for deploying servers in Azure. 

There are no Ensure = Absent capabilities
    There are no capabilities to delete any Pools/VirtualDisk/Volumes (only the initial create)
There are no capabilities to extend the sizes after the initial deployment

This creates a Storage Pool, a Virtual Disk and a Volume.

You will want to select a column count equal to the number of disks that you may want to extend the pool in the future.
i.e. if the pool size has 1 disk, then you can add 1 disk
i.e. if the pool size has 6 disks, with a column count of 2, then you can add 2 extra disks to make it 8 disks.
i.e. with storage pools you are forced to add the number of disks equal to the column count.

````
Configuration MyServerConfig 
{ 
    Import-DscResource -ModuleName StoragePoolcustom 
 
    Node localhost 
    { 
        foreach ($Pool in $Node.StoragePools) 
        { 
            StoragePool $Pool.DriveLetter 
            { 
                FriendlyName = ($SQLInstanceName + '_' + $Pool.FriendlyName) 
                DriveLetter  = $Pool.DriveLetter 
                LUNS         = $Pool.LUNS 
                ColumnCount  = $(if ($Pool.ColumnCount) { $Pool.ColumnCount } else { 0 }) 
            } 
            $dependsonStoragePoolsPresent += @("[StoragePool]$($disk.DriveLetter)") 
        } 
    } 
} 
 
# 
# ConfigurationDataSQL.psd1 
# 
 
$CD = @{  
    AllNodes = @(  
        @{  
            NodeName       = "localhost"  
     
              StoragePools = @{ FriendlyName = 'DATA'   ; DriveLetter = 'F' ; LUNS = (0,1,2,3);ColumnCount = 2}, 
                             @{ FriendlyName = 'LOGS'   ; DriveLetter = 'G' ; LUNS = (8)}, 
                             @{ FriendlyName = 'TEMPDB' ; DriveLetter = 'H' ; LUNS = (12)}, 
                             @{ FriendlyName = 'BACKUP' ; DriveLetter = 'I' ; LUNS = (15)} 
                             
         } 
     ) 
} 
     
MyServerConfig -ConfigurationData $CD 
````
