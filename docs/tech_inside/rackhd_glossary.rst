RackHD Glossary
=============================

=================== ===================================================================================================
RackHD Term         Definition
=================== ===================================================================================================
Bare Metal          The state of a compute node, storage node, or switch where there is no OS, Hypervisor, or Application deployed.
Bare Metal OS       An operating system that runs directly on top of the hardware/firmware, unlike an OS running in a virtual machine.
BMC                 Baseboard Management Controller. A BMC is a specialized microcontroller embedded on the motherboard of a system that manages the interface between system management software and the physical hardware on the system.
Chassis             The structural framework that accepts some number of fixed form factor nodes, containing a midplane, dedicated power, fans, and network interface. A chassis may also contain a management card that is responsible for chassis management.
Element             A generic term used to define a physical resource that can be managed or provisioned. Examples include: CPU Element, NVRAM Element, Storage Element.
Enclosure           The structural framework that contains a node. The enclosure can contain a single compute node – sometimes referred to as blades when they plug into a multi-bay chassis or a server when it is rack mountable.
Genealogy           Refers to the make-up and relational information of the hardware components of a given rack, node, or element; it also includes attributes such as port count, speed, capacity, FRU data, FW versions, etc.
IPMI                Intelligent Platform Management Interface - A standard system interface for out-of-band management of computer systems and monitoring of their operation.
KCS                 Keyboard Controller Style. A communication channel between the CPU and BMC.
Node                A generic term used to describe an enclosure that includes compute, storage, or network resources. A node can either be rack mountable, in the case of a server, or it can have a specific form factor so it only fits in a specific enclosure.
OOB                 Out of Band - refers to the use of a dedicated channel to perform management. The OOB network does not interfere with the data path, thereby minimizing any impact to system performance on the data plane.
Rack                A physical entity that provides power and accepts rack-mountable hardware. Racks can contain TOR switches, Chassis, servers, cooling, etc.
REST                Representational State Transfer - REST is an architectural style consisting of a coordinated set of architectural constraints applied to components, connectors, and data elements, within a distributed hypermedia system.
SDN                 Software Defined Networking - An approach to computer networking that allows network administrators to manage network services through abstraction of higher-level functionality. This is done by decoupling the network control plane from the data plane.
SDS                 Software-defined storage (SDS) allows for management of data storage independent of the underlying hardware. Typically this involves the use of storage virtualization to separate the storage hardware from the management software.
SLA                 As used in a Converged Infrastructure, refers to a specific set of Service-level Objective (SLO) targets that collectively define a level of service required to support an application or infrastructure.
SLO                 A set of specific targets or metrics that can be used to prescribe a level of service or  to measure the effectiveness of a Converged Infrastructure in delivering to that level of service.
VM                  Virtual Machine - the emulation of a computer system providing compute, network, and storage resources. VMs run within a hypervisor that manages the resource assignments
=================== ===================================================================================================