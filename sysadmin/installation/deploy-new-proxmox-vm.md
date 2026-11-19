# Deploy a New Proxmox VM

## Overview

The VM deployment workflow combines Terraform and Ansible:

1. The `terraform-new-vm.sh` helper script deploys a new VM from the AlmaLinux golden template using the requested FQDN, CPU, memory, and disk size.
2. The script updates the Proxmox inventory in `eicorg/ansible-sys` and triggers AWX project and inventory synchronization.
3. The script registers the VM MAC address with ITD.
4. The script reboots the VM after first boot and runs `resize-guest-lvm.sh` to expand the guest partition, LVM, and filesystems.
5. After deployment completes, run **Base OS Configuration** and **Userland Env Customization** in AWX.

## Documentation References

- [Proxmox automation workflow](https://github.com/eicorg/proxmox_production/blob/main/phase7b-api-exploration-automation.md)
- [Ansible system configuration](https://github.com/eicorg/ansible-sys)
- [Inventory synchronization scripts](https://github.com/eicorg/ansible-sys/tree/main/inventory)

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

  - **AWX admin password** — used to trigger AWX synchronization.
  - **Registered_by NTLM user**, for example: `zgong` — used to register the VM MAC address with ITD.

- Confirm that deployment ends with:

  ```text
  ✅ Guest LVM/filesystem resize completed.
  ✅ eicxxx.eic.bnl.gov deployment is complete.
  ```

- Confirm that the VM appears in the production Proxmox cluster:

  [https://prox01.eic.bnl.gov:8006](https://prox01.eic.bnl.gov:8006)

  Select the **BNL LDAP realm (`bnl-ldap`)** and log in with your BNL credentials.

## 3. Apply Ansible Post-Configuration

- Log in to [EIC Ansible / AWX](https://ansible.eic.bnl.gov/).
- Confirm that the new VM appears under **Inventories → EICInventoryProxmox → Hosts**.
- Confirm that the project and inventory source syncs completed successfully.
- Open **Templates** and run these jobs in order:

  1. **Base OS Configuration**
     - Playbook: [`base_os.yml`](https://github.com/eicorg/ansible-sys/blob/main/base_os.yml)
  2. **Userland Env Customization**
     - Playbook: [`userland_env.yml`](https://github.com/eicorg/ansible-sys/blob/main/userland_env.yml)

- For each job:

  - Confirm the project is **EicProject-ansible-sys**.
  - Confirm the inventory is **EICInventoryProxmox**.
  - Set **Limit** to the new VM's inventory hostname, for example: `eicdev01`.
  - Provide any prompted credentials or variables.
  - Confirm that the job completes successfully before proceeding.

The VM is then ready for application-specific configuration.
