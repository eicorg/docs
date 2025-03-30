# Naming convention
- EPICS: Flat.
Disadvantage: The flat naming for large systems requires strict policy for naming convention and additional tools like ChannelFinder.
- ADO: Device:Parameter.
Advantage1: Naming convention is intrinsically device-centric. No additional naming convention required.
Advantage2: ADO parameters have aliases, they can be addressed with system or generic name.

# Introspection.
Ability to find information about device features.
- EPICS:
Channel Access lacks introspection completely. InPVAccess supports limited introspection, it is based on server IP address, not device name. For large system ChannelFinder is needed.
- ADO:
Introspection is supported intrinsically. Very useful tool provided: adoPet.

# Name service
- EPICS:
An External tool ChannelFinder, requires many pre-requisites. To support ChannelFinder, device need to host a RecSync client to expose its properties. Thi complicates the structure of the IOC. It is not clear if ChannellFinder could be in sync with Python-based IOCs (e.g. p4p).
- ADO:
The cnslookup is integral part of the ADO framework. There is no need for tools like ChannelFinder. The cnslookup requires central database which is core part of the ADO infrastructure.

# Communication protocol.
- EPICS:
Both Channel Access and PVAccess protocols are custom. The PVAccess is very complicated. Due to its complexity, the pure-Python implementation of the PVAccess still does not exist.
PVaccess supports RPC-type requests, this functionality does not seem to be very demanding as it can be implemented using standard requests.
- ADO:
The communication protocol is based on RPC standard, and it is using very small subset of the RPC features. The protocol is less complex as EPICS. There exist pure-Python and Java implementations of the protocol.

# Precursors for secure access control.
- EPICS:
Protocol does not propagate user ID and program ID to server.
- ADO:
User ID and Program ID of the client are available to server as part of client request.
THis simplifies the implemetation of secure access control.

# Parameter properties
- EPICS:
The concept of DB records is counter-intuitive and unnecessary complicated. Some of its features (e.g. communication between DB records) are never used in modern applications.
- ADO:
ADO parameters follows the simple rules: parameter has properties, the properties are exposed for inquiry and modifications.

# Server executable
- EPICS:
The IOC is build with an internal shell and a state machine. They seems to be unnecessary complication.

# Data logging
- EPICS:
Archiver Apliances. Set of custom tools.
- ADO:
Set of custom tools. Provides similar functionality as Archiver Apliances. The data extraction tool is Java-based DataServer.
Data ingestion tool is based on oudated SDDS file format.
- Alternative:
Consider to employ off-the-shelf time series database like TDengine or InfluxDB.

# GUI
- EPICS:
Advantage: The CS Studio make it possible to create impressive GUI. 
Disadvantage: There is no simple tools like Gpm.
- ADO:
Disadvantage. The Pet interface is very limited.
The impressive GUI can be created using Python and QT, but this requires basic Qt expertise.
 
# Central Database.
- EPICS:
There is no central database in EPICS.
- ADO:
The database is core component of the ADO infrastructure. Interface between devices and database is simple, it is part of the communication protocol.
The database is urrently SYBASE, which is planned to be replaced by PostgreSQL or MySQL.
 

