## Deploy New Proxmox VM
****

## Background:
https://github.com/eicorg/proxmox_eval/edit/main/phase7b-api-exploration-automation.md
https://github.com/eicorg/proxmox_eval/tree/main/ansible

## Steps:

**1. Run helper script [terraform-new-vm.sh](https://github.com/eicorg/proxmox_eval/blob/main/terraform-templates/example01-terraform-new-vm.sh) to deploy the new VM:
- ssh eicuser@alarmdemo.eic.bnl.gov
- cd /home/eicuser/zaiwen/proxmox-terraform-examples
- ./terraform-new-vm.sh
  - Enter VM FQDN (e.g., eicdev01.eic.bnl.gov): 
  - Enter number of CPU cores [4]:
  - Enter memory in MB [8192] (e.g., 16384 = 16 GB):
  - Enter disk size [100G]:
  
  - Enter AWX password for admin:  
  - Enter Registered_by (NTLM user, e.g., zgong):

  - At the end:
    - ✅ Guest LVM/filesystem resize completed.
    - ✅ eicxxx.eic.bnl.gov deployment is complete.
   
  - Confirm in Proxmox GUI
    - Open the Proxmox web interface.
    - You should see the newly created VM listed
      <img width="975" height="250" alt="image" src="https://github.com/user-attachments/assets/7fa450a3-b6d7-4b42-bb3c-e4d94d7d3b5f" />

## 2. Ansible - After VM provisioning
- Logon to EIC Ansible Server: https://ansible.eic.bnl.gov/
- Run two jobs
  - Base OS Configuration
  - Userland Env Customization 
