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
1. Open **Server Manager**.
2. In the **Add Roles and Features** wizard, select **Server Roles**.
3. Expand **File and Storage Services**.
4. Expand **File and iSCSI Services**.
5. Check **Data Deduplication**. Click **Next** until the **Install** button is visible and install.    
More information on installing is [here](https://docs.microsoft.com/en-us/windows-server/storage/data-deduplication/install-enable).
