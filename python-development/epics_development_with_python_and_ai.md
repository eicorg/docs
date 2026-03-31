# EPICS Device Development With Python, p4p and AI
## Why Python-based servers are essential for large EPICS installations.
1. **Simplicity**. It is often simpler to generate server from scratch, than to deploy available IOC.
- no dbRecords,
- no iocShell,
- AI automation: week work done in a day.
2. **Easy deployment**.
3. **Easy maintainace, modularity**: one server per device, separate servers for data processing, fewer nunmer of PCs.

[p4p library](https://github.com/epics-base/p4p) (PVAccess for Python) is a Python wrapper that uses the C++ PVXS library to implement the EPICS PVAccess (PVA) client and server protocols. 

It is the recommended Python interface for the modern PVAccess protocol.

[epicsdev](https://github.com/ASukhanov/epicsdev) is a lightweight, p4p-based, framework for building EPICS PVAccess servers in Python.
It provides IOC-like services (autosave, stats, put logging), plus a compact API for
defining PVs and publishing values. This makes it practical to develop device support
 quickly and iterate with AI assistance.

## Why `epicsdev` simplifies development

With `epicsdev`, most repetitive IOC plumbing is already implemented:

- Common server PVs are created automatically.
- Limits and metadata are attached from PV definitions.
- Writable PVs automatically get a validated `put` handler.
- Autosave and value recall are built in.
- Optional put logging (`caPutLog`-style) is built in.
- Heartbeat and uptime are maintained by a background thread.

In practice, device code usually focuses on:

1. Defining PVs (`myPVDefs()` style).
2. Writing setters for command/control PVs.
3. Implementing polling/acquisition logic in the main loop.

## Mandatory PVs for IOC-style statistics

The following PVs are created by default for every server:

- `HOSTNAME`: host name of the server.
- `VERSION`: program/module version string.
- `HEARTBEAT`: increments every second.
- `UPTIME`: elapsed time since startup.
- `STARTTOD`: startup timestamp.
- `CPU_LOAD`: CPU usage in percent.
- `CA_CONN_COUNT`: active TCP connection count.

Additional framework PVs are also added for control and monitoring:

- `status`
- `server`
- `verbose`
- `sleep`
- `cycle`
- `cycleTime`

## How PVs are defined

`epicsdev` expects each PV definition as:

```python
[pvName, description, initialValue, extraParameters]
```

`extraParameters` is optional and may include:

- `features`: string flags (`W` writable, `D` discrete enum, etc.).
- `type`: explicit EPICS scalar type (`f32`, `u32`, `i32`, ...).
- `units`, `limitLow`, `limitHigh`, `valueAlarm`, `setter`, and others.

### Automatic NT type deduction from initial value

If `type` is not provided, the runtime infers the Nominal Type from `initialValue`:

1. Scalar `int` -> `i32`.
2. Scalar `float` -> `f32`.
3. Scalar `str` -> string scalar.
4. Python list/tuple/1D sequence -> `NTScalarArray` of inferred element type.
5. `numpy.ndarray` with 2D or higher shape -> `NTNDArray`.
6. If `features` includes `D`, value is created as `NTEnum`.

This allows simple PV definitions while still supporting explicit override when
exact value type is important.

## Main loop pattern

Typical server startup flow:

1. Build application PV definitions.
2. Call `init_epicsdev(prefix, pvDefs, ...)`.
3. Optionally run device-specific initialization.
4. Set server state to running with `set_server('Start')`.
5. Enter `while True` loop:
```python
    while True:
        state = serverState()
        if state.startswith('Exit'):
            break
        if not state.startswith('Stop'):
            poll()# Do your routine work here
        if not sleep():# Sleep and update performance metrics periodically
            periodic_update()
    printi('Server is exited')
```
`sleep()` performs more than sleeping:

- Sleeps for current `sleep` PV value.
- Updates cycle counters and average cycle time.
- Periodically updates stats PVs (`cycle`, `cycleTime`, `CPU_LOAD`, `CA_CONN_COUNT`).
- Periodically triggers autosave when writable PVs were changed.
- Returns `False` when a periodic-update boundary is reached.

# AI-generated PVAccess servers.
Fully AI-generated:
- [epicsdev.putlog](https://github.com/ASukhanov/epicsdev/blob/main/epicsdev/putlog.py): 
Fully functional caPutLog logger.
- [epicsdev.examples.ai_imagegen.py](https://github.com/ASukhanov/epicsdev/blob/main/examples/ai_imagegen.py): 
Simple 2D multi-gaussian generator. 

AI-generated, human-assisted:
- [epicsdev.imagegen](https://github.com/ASukhanov/epicsdev/blob/main/epicsdev/imagegen.py):
Capable to host and update 10,000 of 100-point waveform PVs per second.
- [epicsdev.multiadc](https://github.com/ASukhanov/epicsdev/blob/main/epicsdev/multiadc.py):
- [epicsdev.rigol_scope](https://github.com/ASukhanov/epicsdev_rigol_scope):
Fully-functional support of RIGOL oscilloscopes.
- [epicsdev.tektronix](https://github.com/ASukhanov/epicsdev_tektronix):
Fully-functional support of TEKTRONIX oscilloscopes.
- [epicsdev.lecroy](https://github.com/ASukhanov/epicsdev_lecroy):
Support of LeCroy oscilloscopes.
- [epicsdev.rhode](): 
Support of Rohde&Schwartz oscilloscopes.

# Differences in EPICS and ADO paradigms.
1. EPICS tends to combine and compile many plugins into a single executable. In ADO a device usually publishes only device-specific parameters, other ADOs are processing them.
2. PVs have have very limited set of standard features. There are no features like WRITABLE, ARCHIVABLE, DIAGNOSTIC and more.
3. EPICS tends to expose tons of internal (non-essential) PVs.The ratio of non-essential/essetntial could be 10 or higher
4. GETCODE functionality is not supported (prohibited) in EPICS.
5. EPICS data transfer is faster.
 
