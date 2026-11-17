# Development and deployment of epicsdev-based EPICS PVAccess servers

## 1. Development

If you do not want to create a GitHub repository, skip to step 3 in this section.

If you have cloned an exiting epicsdev-based repository, then activate python environment using `uv sync` and go to **Test run** section.

1. Create a repository under the `eicorg` GitHub organization. The usual naming convention for epicsdev-based servers is:

   `epicsdev_<hardware_type>_<manufacturer>_<model>`

   Example: `epicsdev_ps_caen_fastps`

   - Enable README creation.
   - Select the BSD 3-Clause license.

2. Clone the repository to your local machine.

3. In your working directory, run:

```bash
mkdir my_server opi docs misc
touch my_server/__init__.py

uv init
uv add epicsdev
```

Copy the instrument programming manual to `misc/programming_manual.pdf`.

Copy the [server template](https://github.com/eicorg/epicsdev/blob/main/docs/template.py) to `my_server/__main__.py`.

### Check that the basic functionality works

```bash
uv run python -m my_server
```

Open a second terminal and confirm that the server publishes the template PVs:

```bash
pvmonitor template:0:cycle template:0:cycleTime
```

The output should look similar to this:

```bash
template:0:cycle     2026-10-15 11:04:08.735  61 
template:0:cycleTime 2026-10-15 11:04:08.735  1.00443 
template:0:cycle     2026-10-15 11:04:18.777  71 
template:0:cycleTime 2026-10-15 11:04:18.778  1.00423
...
```

### Use an AI agent to build the server from the instrument manual

From the project directory, prompt an AI agent such as GitHub Copilot:

```text
Modify my_server/__main__.py to provide EPICS PVAccess support for <instrument name and model>.
Use misc/programming_manual.pdf as the primary reference.
```

Add details that help the agent target the implementation, for example:

- Connection type and address: TCP/IP, USB, serial, or GPIB
- Required instrument functions and parameters
- Preferred EPICS PV naming convention
- Polling rate and performance requirements
- Required OPI screens or example configuration

Review the generated module, test it with the real instrument, and refine the logic for error handling, retry behavior, and recovery as needed.

### Test run

```bash
uv run python -m my_server
```

## 2. Deployment

To package epicsdev-based PVAccess servers as standalone Linux executables in your build pipeline, ask the AI agent to produce an executable for your repository. This usually takes around 5 minutes.

The generated executable contains the full runtime environment and is typically about 50 MB in size.

For the underlying procedure, see the [Continuous Integration Guide](https://github.com/eicorg/epicsdev/blob/main/docs/epicsdev_continuous_integration.md)
