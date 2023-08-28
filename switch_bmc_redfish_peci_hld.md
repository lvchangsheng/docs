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

format:
| Rev |   date	 |       Author      |                       Change Description                     |
|:---:|:--------:|:-----------------:|:------------------------------------------------------------:|

### Scope  

This section describes the scope of this high-level design document in Switch OS briefly.

### Definitions/Abbreviations 

This section covers the abbreviation if any, used in this high-level design document and its definitions.

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

The proposed D-Bus interface can be found here:
https://gerrit.openbmc-project.xyz/c/openbmc/phosphor-dbus-interfaces/+/19768

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
		
	- Is it a built-in Switch OS feature or a Switch OS Application Extension?
	- What are the modules and sub-modules that are modified for this design?
	- What are the repositories that would be changed?
	- Module/sub-module interfaces and dependencies. 
	- Components changes in detail
	- DB and Schema changes
	- Sequence diagram if required.
	- Linux dependencies and interface
	- Warm reboot requirements/dependencies
	- Fastboot requirements/dependencies
	- Scalability and performance requirements/impact
	- Memory requirements
	- Docker dependency
	- Build dependency if any
	- Management interfaces - SNMP, CLI, RestAPI, etc.,
	- Serviceability and Debug (logging, counters, trace etc) related design
	- Is this change specific to any platform? Are there dependencies for platforms to implement anything to make this feature work? If yes, explain in detail and inform community in advance.
	- SAI API requirements, CLI requirements. Design is covered in following sections.

### SAI API 

This section covers the changes made or new API added in SAI API for implementing this feature. If there is no change in SAI API for HLD feature, it should be explicitly mentioned in this section.
This section should list the SAI APIs/objects used by the design so that silicon vendors can implement the required support in their SAI. Note that the SAI requirements should be discussed with SAI community during the design phase and ensure the required SAI support is implemented along with the feature/enhancement.

### Configuration and management 
This section should have sub-sections for all types of configuration and management related design. Sub-sections related to data models (YANG, REST, gNMI, etc.,) should be added as required.

#### Manifest (if the feature is an Application Extension)

Paste a preliminary manifest in a JSON format.

#### CLI/YANG model Enhancements 

This sub-section covers the addition/deletion/modification of CLI changes and YANG model changes needed for the feature in detail. If there is no change in CLI for HLD feature, it should be explicitly mentioned in this section. Note that the CLI changes should ensure downward compatibility with the previous/existing CLI. i.e. Users should be able to save and restore the CLI from previous release even after the new CLI is implemented. 
This should also explain the CLICK and/or KLISH related configuration/show in detail.

#### Config Enhancements  

This sub-section covers the addition/deletion/modification of config changes needed for the feature. If there is no change in configuration for HLD feature, it should be explicitly mentioned in this section. This section should also ensure the downward compatibility for the change. 
		
### Warmboot and Fastboot Design Impact  
Mention whether this feature/enhancement has got any requirements/dependencies/impact w.r.t. warmboot and fastboot. Ensure that existing warmboot/fastboot feature is not affected due to this design and explain the same.

### Memory Consumption
This sub-section covers the memory consumption analysis for the new feature: no memory consumption is expected when the feature is disabled via compilation and no growing memory consumption while feature is disabled by configuration. 
### Restrictions/Limitations  

### Testing Requirements/Design  
Explain what kind of unit testing, system testing, regression testing, warmboot/fastboot testing, etc.,
Ensure that the existing warmboot/fastboot requirements are met. For example, if the current warmboot feature expects maximum of 1 second or zero second data disruption, the same should be met even after the new feature/enhancement is implemented. Explain the same here.
Example sub-sections for unit test cases and system test cases are given below. 

#### Unit Test cases  

#### System Test cases

### Open/Action items - if any 

	
NOTE: All the sections and sub-sections given above are mandatory in the design document. Users can add additional sections/sub-sections if required.
