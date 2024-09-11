
# ADO-EPICS Bridge

## How to install

- Everything is installed in 32-bit since currently ADO codebase is supported in 32-bits only.
- adoSrv source code: `/home/cfsa/mdavidsav/adoSrv`
- PVXS source code: `https://github.com/epics-base/pvxs`
- EPICS Base: `https://github.com/epics-base/epics-base`

**Steps**
- 1. Install ado-core
	- Install the dependencies first: `tirpc`, `xerces-c`, `freetds`, `X11`, `Xt`, `SM`, `ICE`, `event`, `rpcbind`, `nsl`.
	- Get the source code with simpleMan and required libs from CAD
  - Install ado libraries under `vobs/`.
- 2. Install epics-base in 32-bit. 
- 3. Install pvxs in 32-bit. 
- 4. Install/compile adoSrv with simpleMan. 
- 5. Setup Sybase16 or copy the libraries. Point environment to the libraries.
- 6. Install cns server and populate with ADO/manager entry. Set CNSHOST and CNSHOSTBACKUP variables. Ensure rpcbind is running.
- Directory structure:

```
.
├── adoSrv
│   ├── bin
│   ├── configure
│   ├── dbd
│   ├── include
│   ├── lib
│   ├── Makefile
│   ├── README.md
│   ├── run-md.sh
│   ├── simpleMan
│   ├── src
│   └── test
├── bin
│ 
├── ado-core
|    |-- vobs
|    |-- include
├── epics-base 
├── opi
│   └── simpleMan.bob
└── pvxs 
```

## How To Use

- **0. How to get started**
```
ssh uername@demo02.eic.bnl.gov
source /opt/ado-epics-bridge/setup.sh
```

- **1. How to register new manager and ADO on EIC machine**
```
cnsadd -h
```

- **2. Run `cnslookup` to find registered managers (`simple.eic.man`)**
```
cnslookup simple.eic.man
cnslookup simple.eic
```

- **3. How to start `simpleMan`**
```
simpleMan -manager simple.eic.man -inst simple.eic
```

- **4. Check list of ADO parameters or metadata using `adoMetaData`**
```
adoMetaData simple.eic
```

- **5. Use adoIf and adoIfA to monitor ADO parameters**
```
adoIf simple.eic sinM
adoIfA simple.eic sinM
```

- **6. How to know PV name from ADO parameter name**
```
<ADO>:<PARM>:<PROP>#<PPM>
<ADO>:<PARM>:<PROP>
<ADO>:<PARM>#<PPM>
<ADO>:<PARM>
```
Property name and/or the PPM index may be omitted for brevity when the "value" property, or PPM 0 is intended.

- **6. Use `pvget` and `pvput` to monitor/set PVs corresponding to the ADO parameters** 

```
pvget simple.eic:degM
pvput simple.eic:hexAddrS 0x1
```

- **7. Use Phoebus to monitor ADO parameters.**
	- Open the `opi/simpleMan.bob` file using Phoebus.

- **8. Change ADO parameter using Phoebus and monitor using adoIfA to confirm**
	- Open the `opi/simpleMan.bob` file using Phoebus and change any parameter while running `adoIfA` for that parameter from another terminal.

- **9. Change ADO parameter using adoIf and confirm using Phoebus**
	- Open the `opi/simpleMan.bob` file using Phoebus and change any parameter using `adoIf`.

- **10. Use clients apps installed on CAD machine to access ADO running on EIC VM**
	- Set CNSHOST and CNSHOSTBACKUP environment to the cns host on EIC machine.
	- Add eic.bnl.gov to dns-search
	- use `adoPet` to open ADO running on EIC VM: `adoPet simple.eic`

- **11. Archive ADO parameter using EPICS archiver.**
	- Go to the EPICS archiver page [here](http://demo01.eic.bnl.gov:10065/mgmt/ui/index.html)
	- Add the desired PV to archive

- **12. Demonstrate alarm functionality using EPICS alarm service**




