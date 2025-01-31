# HLD Name #

## Table of Content 
This section list all content links.

<!-- TOC -->
- [Scope](#scope)
- [Overview](#overview)
- [Others](#others)
<!-- /TOC -->

### Revision
This section list all modified revisions.


| Rev |   date	 |       Author      |                       Change Description                     |
|:---:|:--------:|:-----------------:|:------------------------------------------------------------:|
  V0.1|   202308 |     team        |            draft version                                     |

### Scope  

This document describes the redfish peic show.

### Definitions/Abbreviations 

NA.

### Overview 

Redfish has resources that describe PCIe devices and functions available
on a system. It would be useful to provide these resources to users
out-of-band over Redfish from OpenBMC.

### Requirements

This feature is intended to meet the Redfish requirements for the PCIe
resources above to provide useful system configuration information to system
administrators and operators.

### Architecture Design 

The proposed implementation will follow the standard D-Bus producer-consumer
model used in OpenBMC. The producer will provide the required PCIe values read
from hardware. The consumer will retrieve and parse the D-Bus data to provide
the Redfish PCIe resources.


The proposed producer will be a new D-Bus daemon that will be responsible for
gathering and caching PCIe hardware data and maintaining the D-Bus interfaces
and properties. The actual hardware mechanism that is used to gather the PCIe
hardware data will vary.

For example, on systems that the BMC has access to the host PCI configuration
space, it can directly read the required registers. On systems without access
to the host PCI configuration space, an entity such as the BIOS or OS can
gather the required data and send it to the PCIe daemon through IPMI, etc.

When reading hardware directly, the PCIe daemon must be aware of power state
changes and any BIOS timing requirements, so it can check for hardware
changes, update its cache, and make the necessary changes to the D-Bus
properties. This will allow a user to retrieve the latest PCIe resource data
as of the last system boot even if it is powered off.

bmcweb will be the consumer. It will be responsible for retrieving the Redfish
PCIe resource data from the D-Bus properties and providing it to the user.

### High-Level Design 

This section covers the high level design of the feature/enhancement. This section covers the following points in detail.
		
	- use systemd to restart service.

**BLOCK DIAGRAM**
```
+------------------------------------------------------------------------------------+
|                                 CLIENT                                             |
|                             EVENT LISTENER                                         |
|                                                                                    |
+-----------------------------------------------------------------^---------^--------+
CONFIGURATION|         |SSE(GET)        NETWORK        PUSH STYLE |         |SSE
SUBSCRIPTION |         |                                 EVENTS   |  POST   |
+------------------------------------------------------------------------------------+
|            |         |                                          |         |        |
|   +--------+---------+---+  +---------------------------------------------------+  |
|   |     REDFISH          |  |           EVENT SERVICE MANAGER   |         |     |  |
|   |     SCHEMA           |  |   +-------------------------------+---------+--+  |  |
|   |    RESOURCES         |  |   |          EVENT SERVICE                     |  |  |
|   |                      |  |   |            SENDER                          |  |  |
|   |                      |  |   +------^----------------------------+--------+  |  |
|   |  +-------------+     |  |          |                            |           |  |
|   |  |EVENT SERVICE|     |  |   +------+--------+            +------+--------+  |  |
|   |  +-------------+     |  |   | METRICSREPORT |            |  EVENT LOG    |  |  |
|   |                      |  |   | FORMATTER     |            |  FORMATTER    |  |  |
|   |                      |  |   +------^--------+            +------^--------+  |  |
|   |     +-------------+  |  |          |                            |           |  |
|   |   + |SUBSCRIPTIONS+--------------------+------------------------------+     |  |
|   | + | +-------------+  |  |          |   |                        |     |     |  |
|   | | +--------------+   |  |   +------+---+----+            +------+-----+--+  |  |
|   | +--------------|     |  |   | METRCISREPORT |            |  EVENT LOG    |  |  |
|   |                      |  |   | MONITOR/AGENT |            |  MONITOR      |  |  |
|   |                      |  |   +---------------+            +---------------+  |  |
|   +----+-----------------+  +---------------------------------------------------+  |
|        |                              |                             |              |
+------------------------------------------------------------------------------------+
         |                              |                             |
         |                              |                             |INOTIFY
         |                              |                             |
+--------+----------+         +---------+----------+        +-----------------------+
|                   |         |                    |        |  +-----------------+  |
|    REDFISH        |         |     TELEMETRY      |        |  |     REDFISH     |  |
|  EVENT SERVICE    |         |      SERVICE       |        |  |    EVENT LOGS   |  |
|    CONFIG         |         |                    |        |  +-----------------+  |
| PERSISTENT STORE  |         |                    |        |         |RSYSLOG      |
|                   |         |                    |        |  +-----------------+  |
|                   |         |                    |        |  |     JOURNAL     |  |
|                   |         |                    |        |  |  CONTROL LOGS   |  |
|                   |         |                    |        |  +-----------------+  |
+-------------------+         +--------------------+        +-----------------------+

```

### SAI API 

NA

### Configuration and management 
This section should have sub-sections for all types of configuration and management related design. Sub-sections related to data models (YANG, REST, gNMI, etc.,) should be added as required.

#### Manifest (if the feature is an Application Extension)

Paste a preliminary manifest in a JSON format.

#### CLI/YANG model Enhancements 

NA

#### Config Enhancements  

We have tried doing health monitoring completely within the IPMI Blob
framework. In comparison, having the metric collection part a separate daemon
is better for supporting more interfaces.

We have also tried doing the metric collection task by running an external
binary as well as a shell script. It turns out running shell script is too
slow, while running an external program might have security concerns (in that
the 3rd party program will need to be verified to be safe).
		
### Warmboot and Fastboot Design Impact  
NS

### Memory Consumption
This sub-section covers the memory consumption analysis for the new feature: no memory consumption is expected when the feature is disabled via compilation and no growing memory consumption while feature is disabled by configuration. 
### Restrictions/Limitations  

### Testing Requirements/Design  
To verify the daemon is functionally working correctly, we can monitor the
DBus traffic generated by the Daemon, and the readings on the Daemon’s DBus
objects.

This can also be tested over IPMI/Redfish using sensor command as some of
metrics data are presented as sensors like CPU and Memory are presented as
utilization sensors.

To verify the performance aspect, we can stress-test the Daemon’s DBus
interfaces to make sure the interfaces do not cause a high overhead.

#### Unit Test cases  
NA

#### System Test cases
Testing can be accomplished via automated or manual testing to verify that:
* Configuration not listed as valid results in appropriate behavior.
* Application detects and logs faults for power supply faults including input
faults, output faults, shorts, current share faults, communication failures,
etc.
* Power supply VPD data reported for present power supplies.
* Power supply removal and insertion, on a system supporting concurrent
maintenance, does not result in power loss to powered on system.
* System operates through power supply faults and power line disturbances as
appropriate.

CI testing could be impacted if a system being used for testing is in an
unsupported or faulted configuration.
