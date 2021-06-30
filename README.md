# Data Deduplication with EXPRESSCLUSTER X
A file server often will have redundant files stored across its volume. Data Deduplication can optimize free space by identifying duplicated portions of a volume and optimizing redundancies. For a good overview of Data Deduplication, why it is useful, and when it can be used, please see this [Microsoft website](https://docs.microsoft.com/en-us/windows-server/storage/data-deduplication/overview). Data Deduplication is available for Windows Server 2016 and Windows Server 2019 on NTFS volumes. This document looks at using Data Deduplication on an EXPRESSCLUSTER mirror disk and shared disk.

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
6. Click **Next** until the **Install** button is visible and install.    
More information on installing is [here](https://docs.microsoft.com/en-us/windows-server/storage/data-deduplication/install-enable).

## Enable and Configure Data Deduplication
1. Open the **Server Manager** dashboard.
2. Click **File and Storage Services**.
3. Click on **Volumes**.
4. Right-click on the volume to apply Data Deduplication and select **Configure Data Deduplication...**

5. Data Deduplication is currently disabled. Select **General purpose file server** under **Data deduplication** to enable it.
6. Set the **Deduplicate files older than (in days)** setting to **0** (for testing purposes).
7. Add any extensions to exclude from deduplication.    
   .edb and .jrs files are already excluded. It is best to exclude database files or log files which are updated frequently.
8. Add any folders to be excluded from deduplication.
9. Set a Deduplication Schedule if desired. 

## Testing
