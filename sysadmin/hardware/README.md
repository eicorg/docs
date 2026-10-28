# Hardware Overview

This is intended as a high-level document to cover physical machines in the EIC server room (rm. 218), their use case, and other details that are helpful to have for system administration.

## Development NAS

- OS: TrueNAS version TrueNAS-13.0-U6.2
- IP: 130.199.97.90
- DNS: devnas01.eic.bnl.gov
- Hardware: HP EliteDesk 800 G4 SFF
- CPU: Intel Core i7-8700 CPU @ 3.20GHz (6-core, 12-thread)  
  https://www.intel.com/content/www/us/en/products/sku/126686/intel-core-i78700-processor-12m-cache-up-to-4-60-ghz/specifications.html
- RAM: 16GB
- Storage: 2x Samsung SSD 990 PRO 2TB in raid1 (1.8TiB)
- BNL Barcode: 165431
- SSH: No. Use web interface.
- Web Interface: http://130.199.97.90/ui/sessions/signin

**Backup**  
There is currently no backup...

**Notes**  
This is a SFF HP desktop that is currently (as of 8/4/26) sitting in the bottom of the rack in the server room. Effective storage space is 1.82 TiB.

## VMware

- OS: VMware ESXi Host Client, version 2.14.0
- IP: 130.199.97.3
- DNS: modernbox.c-ad.bnl.gov
- Hardware: S2600WFT
- CPU: 2x Intel Xeon Silver 4214 CPU @ 2.20GHz (12-core, 24-thread)  
  https://www.intel.com/content/www/us/en/products/sku/193385/intel-xeon-silver-4214-processor-16-5m-cache-2-20-ghz/specifications.html
- RAM: 256GB
- Storage: datastore01, total capacity 21.7TB
- Has 12 slots for 3.5" hard drives in the front. All appear to be in use.
- BNL Barcode: N/A
- Red asset tag: A114100
- SSH: No. Use web interface.
- Web Interface: https://130.199.97.3/ui/#/login
- Currently VMs (by 10/28/2026):
  <img width="1594" height="643" alt="image" src="https://github.com/user-attachments/assets/3ea54495-a0d8-45b9-8d1c-35a95136e648" />


**Backup**  
There is currently no backup...

**Notes**  
This is the Supermicro server on the cart behind the server rack. It currently hosts all of the VMs for EIC except for the fpga VMs. A Western Digital external drive is plugged into this machine, presumably for backups. I cannot find the backup mechanism, and as of last communication with Kyle he is not sure this is still working.

The underlying storage setup backing datastore01 is unknown. There isn't any information that I can find on how this is configured via the VMware GUI. It appears to be on the Avago / Megaraid controller, but disk size and raid type are not known. The 12 drive bays in the front of the machine all appear to be in use. At the end of the day it gives us the 21.7TB of storage space noted above.

Note to self: When (if) the machine is ever rebooted, check and note raid setup via bios.

## Old 1-Node Proxmox Test Cluster / FPGA Development

- OS: Proxmox version Virtual Environment 7.4-3
- IP: 130.199.96.153
- DNS: containers01.c-ad.bnl.gov
- Hardware: Supermicro HS219-R16H13
- CPU: 2x AMD EPYC 9274F 24-Core Processor (24-core, 48-thread)  
  https://www.amd.com/en/products/processors/server/epyc/4th-generation-9004-and-8004-series/amd-epyc-9274f.html
- RAM: 1024GB
- Storage: Local disk pool, total storage capacity 13.06TiB
- Has 24 total slots for 2.5" hard drives in the front. 8 are currently in use.
- BNL Barcode: 182517
- SSH: Yes. But use web interface.
- Web Interface: https://130.199.96.153:8006/  or https://containers01.c-ad.bnl.gov:8006/

**Backup**  
There is currently no backup...

**Notes**  
Machine is at the top of the first server rack in the server room. This is a stand alone Proxmox node. It is not part of a cluster.

**Storage Summary**  
The Proxmox node uses local storage only—there is no shared storage such as NFS or Ceph. It has three storage definitions:
 
local: Directory storage at /var/lib/vz for ISOs, templates, and backups.
local-zfs: Local ZFS storage backed by the rpool ZFS pool, used for VM disks.
vm_disk_store: A second local ZFS storage backed by the VMpool ZFS pool, also used for VM disks.
 
VM disks are distributed between the two local ZFS pools (rpool and VMpool), while /var/lib/vz is used only for Proxmox content such as ISOs, templates, and backups.
 
To verify the storage configuration:
 
zpool list
pvesm list local-zfs
pvesm list vm_disk_store

Currently VMs (by 10/28/2026):
  - fpggdev01/02/03.eic.bnl.gov
  - cpiocdev001.eic.bnl.gov
  - docs.eic.bnl.gov
  - hdb.eic.bnl.gov
  - kevindev01 
  

## Retired 3-Node Proxmox Evaluation Cluster

Retired October 28, 2026: All three hosts have been shut down. Their former hostnames, IP addresses, and web URLs are being reassigned to production servers. 

The retired Proxmox Evaluation Cluster is a cluster of 3 machines. Below is a quick overview of basic hardware/configuration.  
Detailed setup documentation is here: https://github.com/eicorg/proxmox_eval

- OS: Proxmox version Virtual Environment 9.0.3
- IP : 130.199.97.157, 130.199.97.115, 130.199.97.138
- DNS: prox01.eic.bnl.gov, prox02.eic.bnl.gov, prox03.eic.bnl.gov 
- Hardware: 3x HP Z2 G5 SFF Desktop machines (https://github.com/eicorg/proxmox_eval/blob/main/z-appendix-hardware-inventory.md)
  - Barcode: 00168732
  - Barcode: 00168727
  - Barcode: 00168728
- CPU: Intel Core i7-10700 CPU @ 2.90GHz (8-core, 16-thread)  
  https://www.intel.com/content/www/us/en/products/sku/199316/intel-core-i710700-processor-16m-cache-up-to-4-80-ghz/specifications.html
- RAM: 16GB
- Storage: See: https://github.com/eicorg/proxmox_eval/blob/main/phase3-ceph-storage-setup.md
- Administration: web interface and SSH
- Web Interface : https://prox01.eic.bnl.gov:8006, https://prox02.eic.bnl.gov:8006, https://prox03.eic.bnl.gov:8006


## 5-Node Proxmox Production Deployment — In Progress

The planned deployment consists of five Proxmox VE nodes and one separate backup server. 

**Proxmox Cluster**
- OS: Proxmox version Virtual Environment 9.2.2
- IP : 130.199.97.157, 130.199.97.115, 130.199.97.138, 130.199.97.169, 130.199.97.170  
- DNS: prox01.eic.bnl.gov, prox02.eic.bnl.gov, prox03.eic.bnl.gov, prox04.eic.bnl.gov, prox05.eic.bnl.gov
- Storage: https://github.com/eicorg/proxmox_production/blob/main/phase3-ceph-storage-setup.md  
- Hardware: 1 HPE ProLiant DL380 Gen11 + 4x Dell PowerEdge Servers (Details: https://github.com/eicorg/proxmox_production/blob/main/z-appendix-hardware-inventory.md)
  - 1 HPE ProLiant DL380 Gen11: CPU 112 cores, RAM 512 GB, Storage 4× 3.84 TB NVMe, Network 2× dual-port 100 GbE adapters — four 100 GbE ports total  
    - Barcode: 00170373
  - 4x Dell PowerEdge Servers: CPU: 24 cores, RAM 256GB, Storage 4× 240 GB SATA SSDs — 960 GB raw per server, Network 4x 1GbE, and 4x 25GbE
    - Barcode: 00179371, 00180864, 00179372, 00180860
- Administration: web interface and SSH
  - https://prox01.eic.bnl.gov:8006
  - https://prox02.eic.bnl.gov:8006
  - https://prox03.eic.bnl.gov:8006
  - https://prox04.eic.bnl.gov:8006
  - https://prox05.eic.bnl.gov:8006  

**Backup Server**  
- Details: https://github.com/eicorg/proxmox_eval/blob/main/phase8-proxmox-backup-server.md
- IP: 130.199.97.168
- DNS: proxbackup.eic.bnl.gov
- Hardware: 1 Dell PowerEdge Servers: CPU: 24 cores, RAM 256GB, Storage 4x 2.5" 250 GB Intel SSD drives, Network 4x 1GbE, and 4x 25GbE
- Barcode: 00179373
- Web Interface:
  - https://proxbackup.eic.bnl.gov:8007

**Notes**  
These machines are currently located in the first rack in the server room.
