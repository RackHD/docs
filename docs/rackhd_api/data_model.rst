Data Model Overview
=============================

Together with API, RackHD creates a set of data elements to abstract the elements and properties of the real world data center management and orchestration. To be familar with RackHD data model could help you to better understand how to use RackHD APIs.

=================== ===================================================================================================
RackHD Term         Definition
=================== ===================================================================================================
Node                Nodes are the elements that RackHD manages - compute servers, switches, etc. Nodes typically have at least one catalog, and can have Pollers and graphs assigned to or working against that node.
Catalog             Catalogs are free form data structures with information about the nodes. Catalogs are created during ‘discovery’ workflows, and present information that can be requested via API and is available to workflows to operate against.
Poller              Pollers are free form data structures which RackHD periodically collects from nodes through various source like IPMI, SNMP .etc
OBM                 A data structures that represents the Out-of-Band management settings and operations associated with the node. A node can have multiple OBMs. 
IBM                 A data structures that represents the In-Band management settings and operations associated with the node such as ssh, etc.
SKU                 Represents a specific model of hardware which can be identified through a set of rules.
Tag                 Provide a method to categorize nodes into group based on data present in node's catalog or manually assigned.
Workflow            A data strcuture specifies the order in which tasks should run and provides any context and/or option values to pass these functions.
Task                A data structure represents a unit of work with data and logic that allows it to be included and run within a workflow. 
Job                 A data structure represents a lowest entity to execute acual work passed from workflow and task.
=================== ===================================================================================================