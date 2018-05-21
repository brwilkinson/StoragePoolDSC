# StoragePoolDSC
Create a storage pool DSC Resource (designed for standing up SQL/File Servers in Azure)

This is a very limited resource for deploying servers in Azure.

This creates a Storage Pool, a Virtual Disk and a Volume.

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
     
              StoragePools = @{ FriendlyName = 'DATA';DriveLetter = 'F';LUNS = (0,1,2,3);ColumnCount = 2}, 
                             @{ FriendlyName = 'LOGS'   ; DriveLetter = 'G' ; LUNS = (8)}, 
                             @{ FriendlyName = 'TEMPDB' ; DriveLetter = 'H' ; LUNS = (12)}, 
                             @{ FriendlyName = 'BACKUP' ; DriveLetter = 'I' ; LUNS = (15)} 
                             
         } 
     ) 
} 
     
MyServerConfig -ConfigurationData $CD 
````
