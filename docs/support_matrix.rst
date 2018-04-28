RackHD Support Matrix
========================

.. contents:: Table of Contents

Sever Compatibility List (Qualified by RackHD team)
---------------------------------------------------
=============== =========================== =============== =================== =============== ======================= ================
Vendor          Type                        T1: Discovery…  T2: OS Installation T3: FW Update   T4: RAID Configuration  T4: Secure Erase
=============== =========================== =============== =================== =============== ======================= ================
Dell            DSS 900	                    Yes             Yes                 No              No                      No
...             PowerEdge R640 (14 gen)     Yes             Yes                 Yes             Yes                     Yes
...             PowerEdge R630 (13 gen)     Yes             Yes                 Yes             Yes                     Yes
...             PowerEdge R730 (13 gen)     Yes             Yes                 Yes             Yes                     Yes
...             PowerEdge R730xd (13 gen)   Yes             Yes                 Yes             Yes                     Yes
...             PowerEdge C6320 (13 gen)    Yes             Yes                 Yes             Yes                     Yes
Cisco           UCS C220 M3                 Yes             Yes                 No              No                      No
White Box	    Quanta D51-1U               Yes             Yes                 Yes             Yes                     Yes
...             Quanta D51-2U               Yes             Yes                 Yes             Yes                     Yes
...             Quanta T41                  Yes             Yes                 Yes             Yes                     Yes
...             Intel Rinjin                Yes             Yes                 Yes             Yes                     Yes
Virtual Node    InfraSIM vNode              Yes             Yes                 No              No                      No
=============== =========================== =============== =================== =============== ======================= ================

.. important::
    1. RackHD classified main server node features into four tiers as below:
        - Tier 1: Discovery, Catalog, Telemetry, Power Management and UID LED control
        - Tier 2: OS Installation
        - Tier 3: Firmware Update
        - Tier 4: RAID Configuration, Secure Erase
    2. RackHD utilizes industry standard protocols to talk with server such as IPMI, PXE, etc. So in theory, any server that supports those protocols can be supported by RackHD at T1 & T2 feature level easily. Many community users have been using RackHD to support various severs from HP, Lenovo, Inspur etc. 
    3. Specific for Cisco server, RackHD supports UCS Manager solution provided by Cisco to manage server nodes behind UCS manager. So user could use "RackHD + UCS service" combination to support big range of Cisco servers.
    4. Specific for Dell server, we provided extended services "smi_service" to support additional Dell server advanced features such as WSMAN. So user also could use "RackHD + smi_service" combination to support big range of Dell servers (ex: 14 Gen server, Dell FX2) and more features.
    5. The RAID Configuration and Secure Erease Feature rely on underlying hardware support. Currently RackHD supports LSI Megaraid RAID card series, so any server that uses this card could support these features. 
    6. InfraSIM vNode is a virtualized server which could simulate most features in a physical server. It is widely used by RackHD team in feature development and testing. (see more at https://github.com/InfraSIM/)

Switch Compatibility List (Qualified by RackHD team)
----------------------------------------------------
=========== =================== =============== =================== 
Vendor      Type                T1: Discovery…  T2: Configuration   
=========== =================== =============== =================== 
Arista      Arista 7124         Yes             Yes                 
Brocade     VDX-6740            Yes             Yes                 
...         VDX-6740T           Yes             Yes                 
Cisco       Nexus 3048          Yes             Yes                 
...         Nexus 3172T         Yes             Yes                 
...         Nexus C3164PQ       Yes             Yes                 
...         Nexus C9332PQ       Yes             Yes                 
...         Nexus C9392PX-E     Yes             Yes                 
Dell        S4048-ON            Yes             Yes                 
...         S6100-ON            Yes             Yes                 
...         Z9100-ON            Yes             Yes                 
=========== =================== =============== =================== 

.. important::
    RackHD classified main switch node features into two tiers as below:
     - Tier 1: Discovery, Catalog, Telemetry
     - Tier 2: Configuration

iPDU/SmartPDU Compatibility List (Qualified by RackHD team)
-----------------------------------------------------------
=========== ======================= =============== =================== =============
Vendor      Type                    T1: Discovery…  T2: Control Outlet  T3: FW Update
=========== ======================= =============== =================== =============
APC         AP8941                  Yes             Yes                 No
...         AP7998                  Yes             Yes                 No
ServerTech  STV4101C                Yes             Yes                 No
...         STV4102C                Yes             Yes                 No
...         VDX-6740T               Yes             Yes                 No
...         CS-18VYY8132A2          Yes             Yes                 Yes
Panduit     IPI Smart PDU Gateway   Yes             Yes                 No
=========== ======================= =============== =================== =============

.. important::
    RackHD classified main iPDU node features into three tiers as below:
     - Tier 1: Discovery, Catalog, Telemetry
     - Tier 2: Control Outlet
     - Tier 3: Firmware Update

RackHD OS Installation Support List (Qualified by RackHD team)
--------------------------------------------------------------
============= =================================================
OS              Version
============= ================================================= 
ESXi          5.5/6.0
RHEL          7.0/7.1/7.2
CentOS        6.5/7
Ubuntu        trusty(14.04)/xenial(16.04)/artful(17.10)
Debian        wheezy(7)/jessie(8)/stretch(9)
SUSE          openSUSE: leap/42.1, SLES: 11/12
CoreOS        899.17.0
Windows       Server 2012
PhotonOS      1.0
============= ================================================= 
