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

- iocBoot can have template IOC configuration files and prototype OPI files.

- A later version of iocBoot directory inside the IOC repository (on GitHub) can contain a version of cookiecutter (Jinja) compatible `st.cmd` and `config` files (for manage-iocs) so that it can be deployed using web/GUI for each user.

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

- For the deployment, we created a central directory (called `modules`) to have all IOC apps or modules that are not part of EPICS installation.

- For each deployed IOC instance, we need to set TOP from `envPaths` and add `cd ${TOP}` at the beginning of the st.cmd file and `cd ${IOC}` at the end of the st.cmd file (before init call). We also need a `config` file for the `manage-iocs` utility.

- On EIC VM, the central NFS directory for deployment is under `/eic/release/epics`. Under this NFS-mounted central directory, we have

```
release
 └── epics
     ├── modules
     ├── iocs (aka iocBoot)
     └── opi
```

- Inside the `iocs` and `opi` directories, new directories can be created that reflect the tree structure of the users or group. `iocs` is same as `iocBoot`. Here we just put each iocBoot instances. Both `iocs` and `opi` are hosted on GitHub at `https://github.com/eicorg/iocs` and `https://github.com/eicorg/opi` under the team controls/epics. The idea is individual IOC module or apps repository can have template iocBoot and OPI files. But all production iocBoot instances and OPI files are under version control from separate mono repositories. 

- If non-NFS deployment is preferred, use the `/epics/iocs` directory.

- While we are using monorepo for `iocs` (production IOC instances) and OPI directories, all homegrown IOC apps have their own individual repository. Thus `modules` is a polyrepo.

- The IOCs are started/stopped/monitored using the `manage-iocs` utility or an user interface (on top of `manage-iocs`).

- `manage-iocs` utility is configured to control IOCs under two locations `/epics/iocs` (local to the host) and `/eic/release/epics/iocs` (NFS mounted).

- Check the deployed IOC examples under `/eic/release/epics/` and also the corresponding eicorg GitHub repositories for further details.

