## GitHub IOC Repository Structure

- On GitHub, under the `eicorg` account, the IOCs are hosted under the controls/epics team. Here is the path: https://github.com/orgs/eicorg/teams/epics/repositories.
- Any IOC repository name has the pattern: `ioc-<system name>`.

- Before creating/pushing any IOC repository, ensure that you have the following directory structure:
```
.
└──ioc-<system name>
    ├── configure
    ├── <system name>App
    ├── iocBoot
    ├── Makefile
    └── .gitignore
```

- Put all source code for the IOC inside `<system name>App` and startup files with OPIs under `iocBoot`.

- iocBoot can have multiple sub-directories each corresponding to each user group.

- A later version of iocBoot directory inside the IOC repository (on GitHub) will contain a version of cookiecutter (Jinja) compatible `st.cmd` and `config` files (for manage-iocs) so that it can be deployed using web/GUI for each user.


- The content of the `.gitignore` will look like the following one, so that all auto-generated files are excluded from the repository:
```
# Install directories
/bin/
/cfg/
/db/
/dbd/
/html/
/include/
/lib/
/templates/

# Local configuration files
/configure/*.local

# iocBoot generated files
/iocBoot/*ioc*/cdCommands
/iocBoot/*ioc*/dllPath.bat
/iocBoot/*ioc*/envPaths
/iocBoot/*ioc*/relPaths.sh

# Build directories
O.*/

# Common files created by other tools
/QtC-*
/.vscode/
*.orig
*.log
.*.swp
.DS_Store
```

## EPICS IOC Deployment Plan

- EPICS IOC apps, `iocBoot` and `OPIs` are hosted on GitHub. They are released and deployed through GitHub CI/CD or web interface only, so that we have all releases synchronized with GitHub. Until GitHub CI/CD or web interface is set up, we need to do it manually.

- For local deployment, create another central directory (say called `modules`) to have all `db` and `dbd` files (under each individual module) that are not part of EPICS installation. Now set TOP from `envPaths` and add `cd ${TOP}` at the beginning of the st.cmd file and `cd ${IOC}` at the end of the st.cmd file (before init call). This way you do not need to copy the `bd` and `dbd` file for each `iocBoot` instance.

- On EIC VM, the central NFS directory for deployment is under `/eic/release/epics`. Under this NFS mounted central directory, we have

```
release
 └── epics
     ├── modules
     ├── iocs (aka iocBoot)
     └── opi
```

- Inside, the `iocs` and `opi` directories, new directories can be created that reflect the tree structure of the users or group. `iocs` is same as `iocBoot`. Here we just put each iocBoot instances.

- The IOCs are started/stopped/monitored using the `manage-iocs` utility or an user interface (on top of `manage-iocs`).

- Check the deployed IOC examples under `/eic/release/epics/` and also the corresponding eicorg GitHub repositories for further details.

