# Data Deduplication with EXPRESSCLUSTER X
A file server often will have redundant files stored across its volume. Data Deduplication can optimize free space by identifying duplicated portions of a volume and optimizing redundancies. For a good overview of Data Deduplication, why it is useful, and when it can be used, please see this [Microsoft website](https://docs.microsoft.com/en-us/windows-server/storage/data-deduplication/overview). Data Deduplication is available for Windows Server 2016 and Windows Server 2019 on NTFS volumes. This document looks at using Data Deduplication on an EXPRESSCLUSTER mirror disk and shared disk.    
*\*Note that his is not a comprehensive document and only limited testing was performed.*

## Tested Environment
- Servers: 2 nodes with 2 additional disks each
	- 1 Disk for mirroring
	- 1 Shared disk connected via iSCSI Initiator
- OS: Windows Server 2019 Standard
- SW: EXPRESSCLUSTER X 4.3

## EXPRESSCLUSTER Setup
1. Prepare two servers with Windows Server 2019 and latest updates
2. Add an additional disk for mirroring data
3. Connect to a shared disk (optional)
4. Install EXPRESSCLUSTER X 4.3
5. Configure a cluster with a mirrored disk 
6. Add a shared disk resource (optional)

## Data Deduplication Setup
   Note: The **File Server** role is required for Data Deduplication. It can be installed at the same time if not installed yet.
1. Open **Server Manager**.
2. In the **Add Roles and Features** wizard, select **Server Roles**.
3. Expand **File and Storage Services**.
4. Expand **File and iSCSI Services**.
5. Check **Data Deduplication**.
      <p align="center">
      <img src="Data Deduplication Role Install.png">
      </p> 
6. Click **Next** until the **Install** button is visible and install.    
Note: More information on installing is [here](https://docs.microsoft.com/en-us/windows-server/storage/data-deduplication/install-enable).

## Enable and Configure Data Deduplication
1. Open the **Server Manager** dashboard.
2. Click **File and Storage Services**.
3. Click on **Volumes**.
4. Right-click on the volume to apply Data Deduplication and select **Configure Data Deduplication...**
      <p align="center">
      <img src="Data Deduplication - Configure.png">
      </p> 
5. Data Deduplication is currently disabled. Select **General purpose file server** under **Data deduplication** to enable it.
6. Set the **Deduplicate files older than (in days)** setting to **0** (for testing purposes).
7. Add any extensions to exclude from deduplication.    
   .edb and .jrs files are already excluded. It is best to exclude database files or log files which are updated frequently.
8. Add any folders to be **excluded** from deduplication.
      <p align="center">
      <img src="My Data Dedup Settings.png">
      </p> 
9. Set a **Deduplication Schedule** if desired.
      <p align="center">
      <img src="Data Dedup Schedule.png">
      </p> 

**NOTE**: I believe that the settings from the main configuration page must be saved to the disk since changes made while on the mirror disk of the Primary server are also visible from the Data Deduplication configuration page on the Standby server. You might have to refresh the view to see changes. Changes made on the Scheduling page don't seem to work the same way. Scheduling changes need to be done on each server.

   <p align="center">
   <img src="Data Deduplication - Enabled.png">
	The volume has Data Deduplication enabled but it has not run yet.
   </p>

## Testing
The obvious method of testing is to copy a large file or files multiple times to the volume and then run data deduplication. **Files must be larger than 32KB to work**.

### File Too Small
I created 1K text file on the volume, copied it two times, and manually ran a dedup job.    
_Result:_ no optimization occurred since file was too small

### File Type Excluded
I added the file type '.doc' to the list of extensions to exclude. I then created a 5MB .doc file, copied it two times on the volume, and ran a dedup job.    
_Result:_ no optimization occurred since file type was excluded.

### Different File Sizes Larger Than 32KB
I copied a 5MB .txt file, copied it two times on the volume, and ran a dedup job.    
_Result:_    
- Saved space: 0 B -> 9.53 MB    
- SavingsRate: 0 % -> 3%    
- OptimizedFilescount: 0 -> 2    
- OptimizedFilesSize: 9.54 MB    
- OptimizedFilesSavingsRate: 99%    
- InPolicyFilesCount: 2

### Failover
I failed over to the Standby server and checked the **Deduplication Rate** and **Deduplication Savings** in _File and Storage Services-\>Volumes_.    
_Result:_ the numbers matched those on the Primary server.    
    
I checked the output from _Get-DedupStatus | fl_    
_Result:_ the output was identical to the Primary server    

I did a binary file compare of the two identical files I had copied to the volume.    
_Result:_ no difference

\<To do: add more testing info\>

Manually run data deduplication job:
````
      PS C:\>Start-DedupJob -Type Optimization -Volume <Your-Volume-Here> -Memory 100 -Cores 100 -Priority High
      
      e.g. Start-DedupJob -Type Optimization -Volume "X:" -Memory 100 -Cores 100 -Priority High
````
View deduplication progress of active and queued jobs:
````
      PS C:\>Get-DedupJob
````
Check the status of the most recent job:
````
      PS C:\>Get-DedupStatus
        or for more info
      PS C:\>Get-DedupStatus | fl
````
Look at the following fields:
- For the Optimization job, look at LastOptimizationResult (0 = Success), LastOptimizationResultMessage, and LastOptimizationTime (should be recent).
- For the Garbage Collection job, look at LastGarbageCollectionResult (0 = Success), LastGarbageCollectionResultMessage, and LastGarbageCollectionTime (should be recent).
- For the Integrity Scrubbing job, look at LastScrubbingResult (0 = Success), LastScrubbingResultMessage, and LastScrubbingTime (should be recent).    
    
Also see
- OptimizedFilesSavingsRate applies only to the files that are 'in-policy' for optimization (space used by optimized files after optimization / logical size of optimized files).
- SavingsRate applies to the entire volume (space used by optimized files after optimization / total logical size of the optimization).

   <p align="center">
   <img src="Dedup Volume.png">
	Data Deduplication has occurred on the volume.
   </p>

### Sample Output
````
PS C:\> Get-DedupStatus


FreeSpace    SavedSpace   OptimizedFiles     InPolicyFiles      Volume
---------    ----------   --------------     -------------      ------
2.83 GB      1.13 GB      12                 12                 X:


PS C:\> Get-DedupStatus | fl


Volume                             : X:
VolumeId                           : \\?Volume{4ec4ab8d-0000-0000-0000-104000000000}\
Capacity                           : 4 GB
FreeSpace                          : 2.83 GB
UsedSpace                          : 1.17 GB
UnoptimizedSize                    : 2.3 GB
SavedSpace                         : 1.13 GB
SavingsRate                        : 49 %
OptimizedFilesCount                : 12
OptimizedFilesSize                 : 2.06 GB
OptimizedFilesSavingsRate          : 55 %
InPolicyFilesCount                 : 12
InPolicyFilesSize                  : 2.06 GB
LastOptimizationTime               : 6/30/2021 9:45:08 AM
LastOptimizationResult             : 0x00000000
LastOptimizationResultMessage      : The operation completed successfully.
LastGarbageCollectionTime          : 6/26/2021 9:45:08 AM
LastGarbageCollectionResult        : 0x00565302
LastGarbageCollectionResultMessage : There are no actions associated with this job.
LastScrubbingTime                  : 6/26/2021 3:45:06 AM
LastScrubbingResult                : 0x00000000
LastScrubbingResultMessage         : The operation completed successfully.

````
