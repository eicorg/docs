# Deploy a New Proxmox VM

## Overview

The new VM deployment workflow combines Terraform and Ansible:

1. The `terraform-new-vm.sh` helper script deploys a new VM from the AlmaLinux Gold template using the requested FQDN, CPU, memory, and disk size.
2. The script synchronizes the new VM into the AWX/Ansible inventory.
3. The script registers the VM MAC address with ITD.
4. The script reboots the VM after first boot and automatically expands the guest GPT partition and LVM physical volume to match the requested virtual-disk size.
5. After the helper script completes, run the **Base OS Configuration** and **Userland Env Customization** jobs in AWX to apply standard post-provisioning configuration.

## Documentation References

- [Proxmox automation workflow](https://github.com/eicorg/proxmox_eval/edit/main/phase7b-api-exploration-automation.md)
- [Ansible configuration](https://github.com/eicorg/proxmox_eval/tree/main/ansible)

## 1. Request Hostname and IP Address

Before deployment, request the VM hostname and IP address through the [ITD IP/DNS Registration page](https://info.itd.bnl.gov/cgi-bin/ipdns/ipreg/single).

Confirm that the requested FQDN resolves in DNS before proceeding.

## 2. Deploy the VM

- Log in to the deployment host:

   ```bash
   ssh eicuser@alarmdemo.eic.bnl.gov
   cd /home/eicuser/zaiwen/proxmox-terraform-examples
   ```

- Run the helper script:

   ```bash
   ./terraform-new-vm.sh
   ```

- Enter the initial VM settings:

   - **VM FQDN**, for example: `eicdev01.eic.bnl.gov`
   - **CPU cores** — default: `4`
   - **Memory in MB** — default: `8192`
   - **Disk size** — default: `100G`

- During deployment, the script will also prompt for:

   - **AWX admin password** — used to synchronize the new VM into the AWX/Ansible inventory.
   - **Registered_by** NTLM user, for example `zgong` — used to register the VM MAC address with ITD.

- Confirm that the deployment ends with:

  ```text
  ✅ Guest LVM/filesystem resize completed.
  ✅ eicxxx.eic.bnl.gov deployment is complete.
  ```

- Confirm that the VM appears in the [Proxmox web interface](https://containers01.c-ad.bnl.gov:8006).

  <img width="975" height="250" alt="New VM listed in Proxmox" src="https://github.com/user-attachments/assets/7fa450a3-b6d7-4b42-bb3c-e4d94d7d3b5f" />

## 3. Apply Ansible Post-Configuration

- Log in to [EIC Ansible](https://ansible.eic.bnl.gov/).
- Confirm that the new VM appears in the Proxmox inventory.
- Run these jobs in order:

   1. **[Base OS Configuration](https://github.com/eicorg/proxmox_eval/blob/main/ansible/base_os.yml)**
   2. **[Userland Env Customization](https://github.com/eicorg/proxmox_eval/blob/main/ansible/userland_env.yml)**

The VM is then ready for application-specific configuration.
