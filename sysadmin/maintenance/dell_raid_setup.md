**Setting Up Hardware RAID on a Dell Server**  

Procedure provided by Darren 

## Purpose

Create a hardware RAID 1 virtual disk using the Dell PERC configuration utility. The controller presents the mirrored pair to the installer as one logical disk.

**Example:** Select two existing 240 GB SSDs and name the resulting RAID 1 virtual disk `OS`. Leave the other disks unallocated unless a different layout has been agreed.

> **Warning:** Creating and initializing the virtual disk can destroy existing data on the selected disks. Confirm the server service tag, disk identities, and that any required data has been preserved before proceeding.

## 1. Open the RAID Configuration Utility

1. During boot, press **F2** to enter **System Setup**.
2. Select **Device Settings**.
3. Select the **RAID Controller**.
4. Select **Main Menu**.
5. Select **Configuration Management**.

## 2. Create the Virtual Disk

1. Select **Create Virtual Disk**.
2. Set **RAID Level** to **RAID 1**.
3. Select the two intended OS physical disks by checking their boxes.
4. Select **Create Virtual Disk** and confirm **Yes** when prompted.
6. Press **Esc** as needed to return to **Main Menu**.

## 3. Initialize and Name the Virtual Disk

1. Open **Virtual Disk Management** and select the newly created virtual disk.
2. Under **Select Operation**, select **Fast Initialization**.
4. Set the virtual disk name to `OS`.
5. Select **Apply** and confirm any prompts.

**Note:** According to Darren's procedure, applying these settings starts initialization and sets the disk name. Initialization may take a while to start; allow the controller time and monitor its status. Menu labels can vary by controller firmware.

## 4. Verify

- Confirm that the virtual disk is named **OS** and uses **RAID 1**.
- Confirm that it contains only the two intended OS disks.
- Check virtual disk health and initialization status.

