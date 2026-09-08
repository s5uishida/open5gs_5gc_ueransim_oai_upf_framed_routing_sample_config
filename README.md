# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Framed Routing with OAI-CN5G-UPF(Simple Switch)
This describes a very simple configuration that uses Open5GS, OAI-CN5G-UPF and UERANSIM for Framed Routing.

This feature has been merged into Open5GS via the following pull requests by **@mitmitmitm**.

- [Framed routing](https://github.com/open5gs/open5gs/pull/2009)
- [Framed routes udr](https://github.com/open5gs/open5gs/pull/2022)
- [[SMF/PFCP] Send framed routes in both UL and DL pdrs](https://github.com/open5gs/open5gs/pull/2356)

The related documents can be found below.
- https://github.com/gonalobastos/5G-Framed-Routing

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC, OAI-CN5G-UPF and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of OAI-CN5G-UPF](#changes_up)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN](#changes_ran)
    - [Changes in configuration files of UE0 (IMSI-001010000000000)](#changes_ue0)
    - [Changes in configuration files of UE1 (IMSI-001010000000001)](#changes_ue1)
- [Network settings of Open5GS 5GC, OAI-CN5G-UPF and UERANSIM UE / RAN](#network_settings)
  - [Network settings of OAI-CN5G-UPF](#network_settings_up)
  - [Network settings of External Node](#network_settings_ext)
  - [Network settings of VM3](#network_settings_vm3)
    - [Add netns](#add_netns)
    - [Setup veth pair for UE0 and PC1/PC4](#setup_ue0)
    - [Setup veth pair for UE1 and PC2/PC3](#setup_ue1)
- [Add Framed Routes to Subscriber information](#add_framed_routes)
  - [Add Framed Routes to UE0](#add_framed_routes_ue0)
  - [Add Framed Routes to UE1](#add_framed_routes_ue1)
- [Build Open5GS, OAI-CN5G-UPF and UERANSIM](#build)
- [Run Open5GS 5GC, OAI-CN5G-UPF and UERANSIM UE / RAN](#run)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run OAI-CN5G-UPF](#run_up)
  - [Run UERANSIM](#run_ueran)
    - [Start gNodeB](#start_gnb)
    - [Start UE0](#start_ue0)
    - [Start UE1](#start_ue1)
  - [Run tcpdump on PC1](#run_pc1)
  - [Run tcpdump on PC2](#run_pc2)
  - [Run tcpdump on PC3](#run_pc3)
  - [Run tcpdump on PC4](#run_pc4)
- [Ping Framed Routes](#ping)
  - [Ping IP address (192.168.20.100/24) of Framed Routes of UE0 on PC1](#ping_pc1)
  - [Ping IP address (192.168.21.100/24) of Framed Routes of UE1 on PC2](#ping_pc2)
  - [Ping IP address (192.168.22.100/24) of Framed Routes of UE1 on PC3](#ping_pc3)
  - [Ping IP address (192.168.23.100/24) not configured for Framed Routes](#ping_pc4)
- [Changelog (summary)](#changelog)
---
<a id="overview"></a>

## Overview of Open5GS 5GC Simulation Mobile Network

I created a 5GC simulation mobile network for  the purpose of using  the IP routes (Framed Routes) behind the UE.

The following minimum configuration was set as a condition.
- Two UEs have the same DNN and connect to the same DN.
- Two UEs have different Framed Routes. On the UPF VM, make sure to be able to ping the Framed Routes via the IP address (Tunnel GW/uesimtun0) assigned to each UE.
- Confirm not to be able to ping to a network that is not configured in the Framed Routes.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The following figure shows the netns and veth pairs within VM3.

<img src="./images/netns-overview.png" title="./images/netns-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.7.7 (2026.04.14) - https://github.com/open5gs/open5gs
- UPF - OAI-CN5G-UPF v2.2.0 (2025.12.13) - https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-upf
- UE / RAN - UERANSIM v3.2.8(+[patch](https://github.com/aligungr/UERANSIM/pull/785)) (2026.04.15) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | CPU<br>(Min) | Mem<br>(Min) | HDD<br>(Min) |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24<br>192.168.14.111/24 | Ubuntu 24.04 | 1 | 2GB | 20GB |
| VM2 | OAI-CN5G-UPF U-Plane  | 192.168.0.151/24<br>192.168.13.151/24<br>192.168.14.151/24<br>**192.168.16.151/24** | Ubuntu 24.04 | 1 | 6GB | 20GB |
| EXT | External Node | 192.168.0.152/24<br>**192.168.16.152/24** | Ubuntu 24.04 | 1 | 1GB | 10GB |
| VM3 | UERANSIM RAN (gNodeB) | 192.168.0.131/24<br>192.168.13.131/24 | Ubuntu 24.04 | 1 | 1GB | 10GB |
|| UERANSIM UE0 | **192.168.20.1/24<br>192.168.23.1/24** | -- | -- | -- | -- |
|| UERANSIM UE1 | **192.168.21.1/24<br>192.168.22.1/24** | -- | -- | -- | -- |
|| PC1 Internal Node | **192.168.20.100/24** | -- | -- | -- | -- |
|| PC2 Internal Node | **192.168.21.100/24** | -- | -- | -- | -- |
|| PC3 Internal Node | **192.168.22.100/24** | -- | -- | -- | -- |
|| PC4 Internal Node | **192.168.23.100/24** | -- | -- | -- | -- |

Pairs of network namespaces and virtual network interfaces are follows.
| Role | netns | veth | veth | netns | Role |
| --- | --- | --- | --- | --- | --- |
| UE0 | ueransim-001010000000000-internet | veth-ue0-pc1<br>**192.168.20.1/24** | veth-pc1<br>**192.168.20.100/24** | pc1 | PC1 |
||| veth-ue0-pc4<br>**192.168.23.1/24** | veth-pc4<br>**192.168.23.100/24** | pc4 | PC4 |
| UE1 | ueransim-001010000000001-internet | veth-ue1-pc2<br>**192.168.21.1/24** | veth-pc2<br>**192.168.21.100/24** | pc2 | PC2 |
||| veth-ue1-pc3<br>**192.168.22.1/24** | veth-pc3<br>**192.168.22.100/24** | pc3 | PC3 |

Subscriber Information (other information is the same) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files. As of 2023.01.29, Framed Routes cannot be set with the WebUI. Also, if you change the `open5gs-dbctl` script, it seems that you can register these with this script, but I could not register.**
| UE # | IMSI | DNN | OP/OPc | Framed Routes | Internal IP address |
| --- | --- | --- | --- | --- | --- |
| UE0 | 001010000000000 | internet | OPc | **192.168.20.0/24** | **192.168.20.1** |
| UE1 | 001010000000001 | internet | OPc | **192.168.21.0/24<br>192.168.22.0/24** | **192.168.21.1<br>192.168.22.1** |

**Note. <ins>192.168.23.0/24</ins> is not configured for Framed Routes.**

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

The DN is as follows.
| DN | TUNnel interface of DN | DNN | TUNnel interface of UE |
| --- | --- | --- | --- |
| 10.45.0.0/16 | ogstun | internet | uesimtun0 |

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC, OAI-CN5G-UPF and UERANSIM UE / RAN

Please refer to the following for building Open5GS, OAI-CN5G-UPF and UERANSIM respectively.
- Open5GS v2.7.7 (2026.04.14) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- OAI-CN5G-UPF v2.2.0 (2025.12.13) - https://github.com/s5uishida/install_oai_upf
- UERANSIM v3.2.8(+[patch](https://github.com/aligungr/UERANSIM/pull/785)) (2026.04.15) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2025-04-27 11:38:05.000000000 +0900
+++ amf.yaml    2025-11-20 07:31:25.166453278 +0900
@@ -20,27 +20,27 @@
         - uri: http://127.0.0.200:7777
   ngap:
     server:
-      - address: 127.0.0.5
+      - address: 192.168.0.111
   metrics:
     server:
       - address: 127.0.0.5
         port: 9090
   guami:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       amf_id:
         region: 2
         set: 1
   tai:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       tac: 1
   plmn_support:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       s_nssai:
         - sst: 1
   security:
```
- `open5gs/install/etc/open5gs/nrf.yaml`
```diff
--- nrf.yaml.orig       2025-04-27 11:38:05.000000000 +0900
+++ nrf.yaml    2025-05-04 08:13:05.973154453 +0900
@@ -11,8 +11,8 @@
 nrf:
   serving:  # 5G roaming requires PLMN in NRF
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
   sbi:
     server:
       - address: 127.0.0.10
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2025-01-15 04:12:06.000000000 +0900
+++ smf.yaml    2025-01-15 04:26:52.000000000 +0900
@@ -7,6 +7,8 @@
   max:
     ue: 1024  # The number of UE can be increased depending on memory size.
 #    peer: 64
+  parameter:
+    use_upg_vpp: true
 
 smf:
   sbi:
@@ -20,16 +22,14 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.14.111
     client:
       upf:
-        - address: 127.0.0.7
-  gtpc:
-    server:
-      - address: 127.0.0.4
+        - address: 192.168.14.151
+          dnn: internet
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.14.111
   metrics:
     server:
       - address: 127.0.0.4
@@ -37,20 +37,17 @@
   session:
     - subnet: 10.45.0.0/16
       gateway: 10.45.0.1
-    - subnet: 2001:db8:cafe::/48
-      gateway: 2001:db8:cafe::1
+      dnn: internet
   dns:
     - 8.8.8.8
     - 8.8.4.4
-    - 2001:4860:4860::8888
-    - 2001:4860:4860::8844
   mtu: 1400
 #  p-cscf:
 #    - 127.0.0.1
 #    - ::1
 #  ctf:
 #    enabled: auto   # auto(default)|yes|no
-  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+#  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
 
 ################################################################################
 # SMF Info
```

<a id="changes_up"></a>

### Changes in configuration files of OAI-CN5G-UPF

See [here](https://github.com/s5uishida/install_oai_upf#conf) for the original file.
And change this `config.yaml` to apply [Simple Switch mode](https://github.com/s5uishida/install_oai_upf#ss_conf) and [Framed Routing](https://github.com/s5uishida/install_oai_upf#fr).

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2024-10-15 20:27:30.453513592 +0900
+++ open5gs-gnb.yaml    2026-04-18 22:24:06.658606032 +0900
@@ -1,17 +1,17 @@
-mcc: '999'          # Mobile Country Code value
-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
 linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.13.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.0.111
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<a id="changes_ue0"></a>

#### Changes in configuration files of UE0 (IMSI-001010000000000)

First, copy `open5gs-ue0.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue0.yaml
```
Next, edit `open5gs-ue0.yaml`.
- `UERANSIM/config/open5gs-ue0.yaml`
```diff
--- open5gs-ue.yaml.orig        2025-03-16 15:49:12.000000000 +0900
+++ open5gs-ue0.yaml    2026-04-18 22:26:12.961649228 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -29,6 +29,12 @@
 # Network mask used for the UE's TUN interface to define the subnet size  
 tunNetmask: '255.255.255.0'
 
+# Create the UE TUN interface inside a dedicated Linux network namespace.
+useNamespace: true
+
+# Optional prefix used when deriving the namespace name.
+nsNamePrefix: 'ueransim'
+
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
   - 127.0.0.1
```

<a id="changes_ue1"></a>

#### Changes in configuration files of UE1 (IMSI-001010000000001)

First, copy `open5gs-ue1.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue1.yaml
```
Next, edit `open5gs-ue1.yaml`.
- `UERANSIM/config/open5gs-ue1.yaml`
```diff
--- open5gs-ue.yaml.orig        2025-03-16 15:49:12.000000000 +0900
+++ open5gs-ue1.yaml    2026-04-18 22:26:38.256058413 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000001'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -29,6 +29,12 @@
 # Network mask used for the UE's TUN interface to define the subnet size  
 tunNetmask: '255.255.255.0'
 
+# Create the UE TUN interface inside a dedicated Linux network namespace.
+useNamespace: true
+
+# Optional prefix used when deriving the namespace name.
+nsNamePrefix: 'ueransim'
+
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
   - 127.0.0.1
```

<a id="network_settings"></a>

## Network settings of Open5GS 5GC, OAI-CN5G-UPF and UERANSIM UE / RAN

<a id="network_settings_up"></a>

### Network settings of OAI-CN5G-UPF

First, see [this](https://github.com/s5uishida/install_oai_upf#network_settings).  
Next, configure the TUNnel interface and set the routings towards Framed Routes.
```
ip route add 192.168.20.0/24 dev tun0
ip route add 192.168.21.0/24 dev tun0
ip route add 192.168.22.0/24 dev tun0
ip route add 192.168.23.0/24 dev tun0
```

<a id="network_settings_ext"></a>

### Network settings of External Node

Set the routings towards UEs and Framed Routes.
```
ip route add 10.45.0.0/16 via 192.168.16.151
ip route add 192.168.20.0/24 via 192.168.16.151
ip route add 192.168.21.0/24 via 192.168.16.151
ip route add 192.168.22.0/24 via 192.168.16.151
ip route add 192.168.23.0/24 via 192.168.16.151
```

<a id="network_settings_vm3"></a>

### Network settings of VM3

Delete default GW.
```
# ip route del default
```

<a id="add_netns"></a>

#### Add netns

First, create 3 netns for the terminals.
```
ip netns add pc1
ip netns add pc2
ip netns add pc3
ip netns add pc4
```
From here on, I will explain how to setup netns and veth, but please note that these settings will be applied to the netns created by running UE0 and UE1.
In other words, run UE0 and UE1 before performing these operations.

<a id="setup_ue0"></a>

#### Setup veth pair for UE0 and PC1/PC4

This explanation assumes that running UE0 will create `ueransim-001010000000000-internet` as netns.

First, move to netns:`ueransim-001010000000000-internet`.
```
ip netns exec ueransim-001010000000000-internet bash
```
Enable IP forwarding.
```
sysctl -w net.ipv4.ip_forward=1
```
Create `veth-ue0-pc1` and `veth-pc1`, then move `veth-pc1` to netns:`pc1`. Assign `192.168.20.1/24` to `veth-ue0-pc1` and enable `veth-ue0-pc1`.
```
ip link add veth-ue0-pc1 type veth peer name veth-pc1
ip link set veth-pc1 netns pc1
ip addr add 192.168.20.1/24 dev veth-ue0-pc1
ip link set veth-ue0-pc1 up
```
Similarly, create `veth-ue0-pc4` and `veth-pc4`, then move `veth-pc4` to netns:`pc4`. Assign `192.168.23.1/24` to `veth-ue0-pc4` and enable `veth-ue0-pc4`.
```
ip link add veth-ue0-pc4 type veth peer name veth-pc4
ip link set veth-pc4 netns pc4
ip addr add 192.168.23.1/24 dev veth-ue0-pc4
ip link set veth-ue0-pc4 up
```
Next, move to netns:`pc1`.
```
ip netns exec pc1 bash
```
Assign `192.168.20.100/24` ​​to `veth-pc1` and enable `veth-pc1`. Then, set `192.168.20.1` as the default route. Finally, enable interface `lo`.
```
ip addr add 192.168.20.100/24 dev veth-pc1
ip link set veth-pc1 up
ip route add default via 192.168.20.1 dev veth-pc1
ip link set lo up
```
Similarly, move to netns:`pc4`.
```
ip netns exec pc4 bash
```
Assign `192.168.23.100/24` ​​to `veth-pc4` and enable `veth-pc4`. Then, set `192.168.23.1` as the default route. Finally, enable interface `lo`.
```
ip addr add 192.168.23.100/24 dev veth-pc4
ip link set veth-pc4 up
ip route add default via 192.168.23.1 dev veth-pc4
ip link set lo up
```

<a id="setup_ue1"></a>

#### Setup veth pair for UE1 and PC2/PC3

This explanation assumes that running UE1 will create `ueransim-001010000000001-internet` as netns.

First, move to netns:`ueransim-001010000000001-internet`.
```
ip netns exec ueransim-001010000000001-internet bash
```
Enable IP forwarding.
```
sysctl -w net.ipv4.ip_forward=1
```
Create `veth-ue1-pc2` and `veth-pc2`, then move `veth-pc2` to netns:`pc2`. Assign `192.168.21.1/24` to `veth-ue1-pc2` and enable `veth-ue1-pc2`.
```
ip link add veth-ue1-pc2 type veth peer name veth-pc2
ip link set veth-pc2 netns pc2
ip addr add 192.168.21.1/24 dev veth-ue1-pc2
ip link set veth-ue1-pc2 up
```
Similarly, create `veth-ue1-pc3` and `veth-pc3`, then move `veth-pc3` to netns:`pc3`. Assign `192.168.22.1/24` to `veth-ue1-pc3` and enable `veth-ue1-pc3`.
```
ip link add veth-ue1-pc3 type veth peer name veth-pc3
ip link set veth-pc3 netns pc3
ip addr add 192.168.22.1/24 dev veth-ue1-pc3
ip link set veth-ue1-pc3 up
```
Next, move to netns:`pc2`.
```
ip netns exec pc2 bash
```
Assign `192.168.21.100/24` ​​to `veth-pc2` and enable `veth-pc2`. Then, set `192.168.21.1` as the default route. Finally, enable interface `lo`.
```
ip addr add 192.168.21.100/24 dev veth-pc2
ip link set veth-pc2 up
ip route add default via 192.168.21.1 dev veth-pc2
ip link set lo up
```
Similarly, move to netns:`pc3`.
```
ip netns exec pc3 bash
```
Assign `192.168.22.100/24` ​​to `veth-pc3` and enable `veth-pc3`. Then, set `192.168.22.1` as the default route. Finally, enable interface `lo`.
```
ip addr add 192.168.22.100/24 dev veth-pc3
ip link set veth-pc3 up
ip route add default via 192.168.22.1 dev veth-pc3
ip link set lo up
```

<a id="add_framed_routes"></a>

## Add Framed Routes to Subscriber information

[MongoDB Compass](https://www.mongodb.com/products/compass) is a useful GUI tool for working with MongoDB data.
I used this tool to add Framed Routes in the following operations.

<a id="add_framed_routes_ue0"></a>

### Add Framed Routes to UE0

The UE0's sample subscriber information registered in MongoDB is as follows in JSON format.
Among these, the items indicated by the arrows are Framed Routes to be added.
```json
{
  "_id": {
    "$oid": "672e210b2a5baf13e3c51a26"
  },
  "ambr": {
    "downlink": {
      "value": 1,
      "unit": 3
    },
    "uplink": {
      "value": 1,
      "unit": 3
    }
  },
  "schema_version": 1,
  "msisdn": [],
  "imeisv": "4370816125816151",
  "mme_host": [],
  "mme_realm": [],
  "purge_flag": [],
  "access_restriction_data": 32,
  "subscriber_status": 0,
  "operator_determined_barring": 0,
  "network_access_mode": 0,
  "subscribed_rau_tau_timer": 12,
  "imsi": "001010000000000",
  "security": {
    "k": "465B5CE8 B199B49F AA5F0A2E E238A6BC",
    "amf": "8000",
    "op": null,
    "opc": "E8ED289D EBA952E4 283B54E8 8E6183CA",
    "sqn": {
      "$numberLong": "1344"
    }
  },
  "slice": [
    {
      "_id": {
        "$oid": "672e210b2a5baf13e3c51a27"
      },
      "sst": 1,
      "default_indicator": true,
      "session": [
        {
          "qos": {
            "arp": {
              "priority_level": 8,
              "pre_emption_capability": 1,
              "pre_emption_vulnerability": 1
            },
            "index": 9
          },
          "ambr": {
            "downlink": {
              "value": 1,
              "unit": 3
            },
            "uplink": {
              "value": 1,
              "unit": 3
            }
          },
          "_id": {
            "$oid": "672e210b2a5baf13e3c51a28"
          },
-->       "ipv4_framed_routes": [
-->         "192.168.20.0/24"
-->       ],
          "name": "internet",
          "type": 1,
          "pcc_rule": []
        }
      ]
    }
  ],
  "__v": 0
}
```

<a id="add_framed_routes_ue1"></a>

### Add Framed Routes to UE1

The UE1's sample subscriber information registered in MongoDB is as follows in JSON format.
Among these, the items indicated by the arrows are Framed Routes to be added.
```json
{
  "_id": {
    "$oid": "691e43a812ac4d03469f1cff"
  },
  "ambr": {
    "downlink": {
      "value": 1,
      "unit": 3
    },
    "uplink": {
      "value": 1,
      "unit": 3
    }
  },
  "schema_version": 1,
  "msisdn": [],
  "imeisv": "4370816125816151",
  "mme_host": [],
  "mme_realm": [],
  "purge_flag": [],
  "access_restriction_data": 32,
  "subscriber_status": 0,
  "operator_determined_barring": 0,
  "network_access_mode": 0,
  "subscribed_rau_tau_timer": 12,
  "imsi": "001010000000001",
  "security": {
    "k": "465B5CE8 B199B49F AA5F0A2E E238A6BC",
    "amf": "8000",
    "op": null,
    "opc": "E8ED289D EBA952E4 283B54E8 8E6183CA",
    "sqn": {
      "$numberLong": "385"
    }
  },
  "slice": [
    {
      "_id": {
        "$oid": "691e43a812ac4d03469f1d00"
      },
      "sst": 1,
      "default_indicator": true,
      "session": [
        {
          "qos": {
            "arp": {
              "priority_level": 8,
              "pre_emption_capability": 1,
              "pre_emption_vulnerability": 1
            },
            "index": 9
          },
          "ambr": {
            "downlink": {
              "value": 1,
              "unit": 3
            },
            "uplink": {
              "value": 1,
              "unit": 3
            }
          },
          "_id": {
            "$oid": "691e43a812ac4d03469f1d01"
          },
-->       "ipv4_framed_routes": [
-->         "192.168.21.0/24",
-->         "192.168.22.0/24"
-->       ],
          "name": "internet",
          "type": 1,
          "pcc_rule": []
        }
      ]
    }
  ],
  "__v": 0
}
```

<a id="build"></a>

## Build Open5GS, OAI-CN5G-UPF and UERANSIM

Please refer to the following for building Open5GS, OAI-CN5G-UPF and UERANSIM respectively.
- Open5GS v2.7.7 (2026.04.14) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- OAI-CN5G-UPF v2.2.0 (2025.12.13) - https://github.com/s5uishida/install_oai_upf
- UERANSIM v3.2.8(+[patch](https://github.com/aligungr/UERANSIM/pull/785)) (2026.04.15) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on Open5GS 5GC C-Plane machine.

<a id="run"></a>

## Run Open5GS 5GC, OAI-CN5G-UPF and UERANSIM UE / RAN

First run the 5GC, then UERANSIM (UE & RAN implementation).

<a id="run_cp"></a>

### Run Open5GS 5GC C-Plane

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 2
./install/bin/open5gs-scpd &
sleep 2
./install/bin/open5gs-amfd &
sleep 2
./install/bin/open5gs-smfd &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
```

<a id="run_up"></a>

### Run OAI-CN5G-UPF

See [this](https://github.com/s5uishida/install_oai_upf#run).
**Don't forget [Network settings of OAI-CN5G-UPF](#network_settings_up).**

<a id="run_ueran"></a>

### Run UERANSIM

First, do an NG Setup between gNodeB and 5GC, then register the UE with 5GC and establish a PDU session.

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<a id="start_gnb"></a>

#### Start gNodeB

Start gNodeB as follows.
```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.8
[2026-04-26 02:26:37.327] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2026-04-26 02:26:37.353] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2026-04-26 02:26:37.353] [sctp] [debug] SCTP association setup ascId[3]
[2026-04-26 02:26:37.354] [ngap] [debug] Sending NG Setup Request
[2026-04-26 02:26:37.360] [ngap] [debug] NG Setup Response received
[2026-04-26 02:26:37.360] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
04/26 02:26:37.345: [amf] INFO: gNB-N2 accepted[192.168.0.131]:36247 in ng-path module (../src/amf/ngap-sctp.c:113)
04/26 02:26:37.345: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:953)
04/26 02:26:37.352: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1277)
04/26 02:26:37.352: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:1000)
```

<a id="start_ue0"></a>

#### Start UE0

Start UE0 as follows. This will register the UE with 5GC and establish a PDU session.
Also, UE0 moves to netns:`ueransim-001010000000000-internet` and runs there.
```
# ./nr-ue -c ../config/open5gs-ue0.yaml
UERANSIM v3.2.8
[2026-04-26 02:26:47.880] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2026-04-26 02:26:47.881] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2026-04-26 02:26:47.881] [nas] [info] Selected plmn[001/01]
[2026-04-26 02:26:47.881] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2026-04-26 02:26:47.882] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2026-04-26 02:26:47.882] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2026-04-26 02:26:47.882] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2026-04-26 02:26:47.882] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2026-04-26 02:26:47.882] [nas] [debug] Sending Initial Registration
[2026-04-26 02:26:47.882] [rrc] [debug] Sending RRC Setup Request
[2026-04-26 02:26:47.883] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2026-04-26 02:26:47.883] [rrc] [info] RRC connection established
[2026-04-26 02:26:47.883] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2026-04-26 02:26:47.883] [nas] [info] UE switches to state [CM-CONNECTED]
[2026-04-26 02:26:47.889] [nas] [debug] Authentication Request received
[2026-04-26 02:26:47.889] [nas] [debug] Received SQN [000000000FC1]
[2026-04-26 02:26:47.889] [nas] [debug] SQN-MS [000000000000]
[2026-04-26 02:26:47.893] [nas] [debug] Security Mode Command received
[2026-04-26 02:26:47.893] [nas] [debug] Selected integrity[2] ciphering[0]
[2026-04-26 02:26:47.905] [nas] [debug] Registration accept received
[2026-04-26 02:26:47.905] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2026-04-26 02:26:47.905] [nas] [debug] Sending Registration Complete
[2026-04-26 02:26:47.905] [nas] [info] Initial Registration is successful
[2026-04-26 02:26:47.906] [nas] [debug] Sending PDU Session Establishment Request
[2026-04-26 02:26:47.906] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2026-04-26 02:26:48.110] [nas] [debug] Configuration Update Command received
[2026-04-26 02:26:48.161] [nas] [debug] PDU Session Establishment Accept received
[2026-04-26 02:26:48.161] [nas] [info] PDU Session establishment is successful PSI[1]
[2026-04-26 02:26:48.209] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up in namespace[ueransim-001010000000000-internet].
```
The Open5GS C-Plane log when executed is as follows.
```
04/26 02:26:47.876: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:461)
04/26 02:26:47.876: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2777)
04/26 02:26:47.876: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:622)
04/26 02:26:47.876: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1912)
04/26 02:26:47.876: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1688)
04/26 02:26:47.876: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1670)
04/26 02:26:47.876: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:183)
04/26 02:26:47.876: [sbi] INFO: [d9fbf9ae-40cb-41f1-aaec-65d2e8781875] Setup NF Instance [type:AUSF] (../lib/sbi/path.c:307)
04/26 02:26:47.877: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.877: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:47.877: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.878: [sbi] INFO: [d9fcd856-40cb-41f1-9d6c-25711c5cc5c2] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
04/26 02:26:47.878: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.881: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
04/26 02:26:47.882: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.882: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.883: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.885: [ausf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/ausf/nudm-handler.c:337)
04/26 02:26:47.886: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:47.886: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.887: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.888: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:47.888: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.889: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.890: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.891: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.892: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.893: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.893: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:361)
04/26 02:26:47.894: [sbi] INFO: [d9fcc5c8-40cb-41f1-b534-cd6a98411397] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
04/26 02:26:47.894: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.894: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/pcf/npcf-handler.c:114)
04/26 02:26:47.895: [sbi] INFO: [d9fcd856-40cb-41f1-9d6c-25711c5cc5c2] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
04/26 02:26:47.895: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:47.897: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
04/26 02:26:48.102: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:3146)
04/26 02:26:48.102: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:609)
04/26 02:26:48.102: [gmm] INFO:     UTC [2026-04-25T17:26:48] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
04/26 02:26:48.102: [gmm] INFO:     LOCAL [2026-04-26T02:26:48] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:556)
04/26 02:26:48.102: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2798)
04/26 02:26:48.102: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] LBO[0] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1419)
04/26 02:26:48.102: [gmm] INFO: V-SMF Instance [da0d4d30-40cb-41f1-8f8f-495f513f9186](LIST) (../src/amf/gmm-handler.c:1496)
04/26 02:26:48.102: [gmm] INFO: [da0d4d30-40cb-41f1-8f8f-495f513f9186] Setup NF Instance [type:SMF] (../src/amf/gmm-handler.c:1498)
04/26 02:26:48.102: [gmm] INFO: V-SMF Instance [da0d4d30-40cb-41f1-8f8f-495f513f9186] (../src/amf/gmm-handler.c:1508)
04/26 02:26:48.102: [gmm] INFO: V-SMF discovered in Non-Roaming or LBO-Roaming[0] (../src/amf/gmm-handler.c:1577)
04/26 02:26:48.102: [gmm] INFO: nsmf_pdusession [1:0x5ef12fe2fc70:(nil)] (../src/amf/gmm-handler.c:1617)
04/26 02:26:48.102: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.103: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1069)
04/26 02:26:48.103: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3393)
04/26 02:26:48.103: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:331)
04/26 02:26:48.103: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:48.104: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.104: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.106: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.107: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:456)
04/26 02:26:48.107: [sbi] INFO: [d9fcc5c8-40cb-41f1-b534-cd6a98411397] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
04/26 02:26:48.107: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.108: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:140)
04/26 02:26:48.108: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/pcf/npcf-handler.c:448)
04/26 02:26:48.108: [sbi] INFO: [d9fcd856-40cb-41f1-9d6c-25711c5cc5c2] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
04/26 02:26:48.109: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.110: [sbi] INFO: [d9fb9ba8-40cb-41f1-879d-2b474d40280a] Setup NF Instance [type:BSF] (../lib/sbi/path.c:307)
04/26 02:26:48.110: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.111: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/pcf/nbsf-handler.c:121)
04/26 02:26:48.112: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:373)
04/26 02:26:48.112: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:594)
04/26 02:26:48.150: [gtp] INFO: gtp_connect() [192.168.13.151]:2152 (../lib/gtp/path.c:60)
04/26 02:26:48.150: [sbi] INFO: [d8c3b9a0-40cb-41f1-a42d-95705edfd14d] Setup NF Instance [type:AMF] (../lib/sbi/path.c:307)
04/26 02:26:48.150: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.153: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.154: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:48.154: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.155: [sbi] INFO: [d9fcd856-40cb-41f1-9d6c-25711c5cc5c2] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
04/26 02:26:48.155: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:48.156: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:954)
```
The OAI-CN5G-UPF log when executed is as follows.
```
[2026-04-26 02:26:48.120] [upf_n4 ] [info] handle_receive(669 bytes)
[2026-04-26 02:26:48.120] [upf_app] [info] 
[2026-04-26 02:26:48.120] [upf_app] [info] ╔═════════════════════════════════════════════════════════════════════════════╗
[2026-04-26 02:26:48.120] [upf_app] [info] │             Received N4_SESSION_ESTABLISHMENT_REQUEST seid 0x0              │
[2026-04-26 02:26:48.120] [upf_app] [info] ╚═════════════════════════════════════════════════════════════════════════════╝
[2026-04-26 02:26:48.120] [upf_n4 ] [info] pfcp_session::add(far) seid 0x1 FAR=1
[2026-04-26 02:26:48.120] [upf_n4 ] [info]   └─ Adding new FAR 1 to session 0x1
[2026-04-26 02:26:48.120] [upf_n4 ] [info] pfcp_session::add(far) seid 0x1 FAR=2
[2026-04-26 02:26:48.120] [upf_n4 ] [info]   └─ Adding new FAR 2 to session 0x1
[2026-04-26 02:26:48.120] [upf_n4 ] [info] pfcp_session::add(far) seid 0x1 FAR=3
[2026-04-26 02:26:48.120] [upf_n4 ] [info]   └─ Adding new FAR 3 to session 0x1
[2026-04-26 02:26:48.123] [pfcp_switch] [info] Route created
[2026-04-26 02:26:48.157] [pfcp_switch] [info] Source NAT added
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x1 PDR=1
[2026-04-26 02:26:48.157] [upf_n4 ] [info]   └─ Adding new PDR 1 to session 0x1
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::set(fteid) seid 0x1 
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::get(fteid) seid 0x1 
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x1 PDR=2
[2026-04-26 02:26:48.157] [upf_n4 ] [info]   └─ Adding new PDR 2 to session 0x1
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::set(fteid) seid 0x1 
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::get(fteid) seid 0x1 
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x1 PDR=3
[2026-04-26 02:26:48.157] [upf_n4 ] [info]   └─ Adding new PDR 3 to session 0x1
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::set(fteid) seid 0x1 
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::get(fteid) seid 0x1 
[2026-04-26 02:26:48.157] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x1 PDR=4
[2026-04-26 02:26:48.157] [upf_n4 ] [info]   └─ Adding new PDR 4 to session 0x1
[2026-04-26 02:26:48.161] [upf_n4 ] [info] handle_receive(75 bytes)
[2026-04-26 02:26:48.161] [upf_app] [info] 
[2026-04-26 02:26:48.161] [upf_app] [info] ╔═════════════════════════════════════════════════════════════════════════════╗
[2026-04-26 02:26:48.161] [upf_app] [info] │             Received N4_SESSION_MODIFICATION_REQUEST seid 0x1               │
[2026-04-26 02:26:48.161] [upf_app] [info] ╚═════════════════════════════════════════════════════════════════════════════╝
[2026-04-26 02:26:48.161] [upf_n4 ] [info] pfcp_session::update(far) seid 0x1 FAR=1
[2026-04-26 02:26:48.161] [upf_n4 ] [info]   └─ Updating FAR 1 in session 0x1
```
Looking at the console log of the `nr-ue` command, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2026-04-26 02:26:48.209] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up in namespace[ueransim-001010000000000-internet].
```
Just in case, after logging in VM3 from another terminal, move to netns:`ueransim-001010000000000-internet` and make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip netns exec ueransim-001010000000000-internet ip addr show
...
5: uesimtun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/24 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::69b:b5bd:c43b:5bfc/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```
**Don't forget [Setup veth pair for UE0 and PC1/PC4](#setup_ue0).**

<a id="start_ue1"></a>

#### Start UE1

Start UE1 as follows. This will register the UE with 5GC and establish a PDU session.
Also, UE1 moves to netns:`ueransim-001010000000001-internet` and runs there.
```
# ./nr-ue -c ../config/open5gs-ue1.yaml 
UERANSIM v3.2.8
[2026-04-26 02:26:59.278] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2026-04-26 02:26:59.279] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2026-04-26 02:26:59.279] [nas] [info] Selected plmn[001/01]
[2026-04-26 02:26:59.279] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2026-04-26 02:26:59.280] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2026-04-26 02:26:59.280] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2026-04-26 02:26:59.280] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2026-04-26 02:26:59.280] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2026-04-26 02:26:59.280] [nas] [debug] Sending Initial Registration
[2026-04-26 02:26:59.280] [rrc] [debug] Sending RRC Setup Request
[2026-04-26 02:26:59.280] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2026-04-26 02:26:59.281] [rrc] [info] RRC connection established
[2026-04-26 02:26:59.281] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2026-04-26 02:26:59.281] [nas] [info] UE switches to state [CM-CONNECTED]
[2026-04-26 02:26:59.285] [nas] [debug] Authentication Request received
[2026-04-26 02:26:59.285] [nas] [debug] Received SQN [0000000005A1]
[2026-04-26 02:26:59.286] [nas] [debug] SQN-MS [000000000000]
[2026-04-26 02:26:59.289] [nas] [debug] Security Mode Command received
[2026-04-26 02:26:59.289] [nas] [debug] Selected integrity[2] ciphering[0]
[2026-04-26 02:26:59.300] [nas] [debug] Registration accept received
[2026-04-26 02:26:59.300] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2026-04-26 02:26:59.300] [nas] [debug] Sending Registration Complete
[2026-04-26 02:26:59.300] [nas] [info] Initial Registration is successful
[2026-04-26 02:26:59.300] [nas] [debug] Sending PDU Session Establishment Request
[2026-04-26 02:26:59.300] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2026-04-26 02:26:59.502] [nas] [debug] Configuration Update Command received
[2026-04-26 02:26:59.521] [nas] [debug] PDU Session Establishment Accept received
[2026-04-26 02:26:59.523] [nas] [info] PDU Session establishment is successful PSI[1]
[2026-04-26 02:26:59.566] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up in namespace[ueransim-001010000000001-internet].
```
The Open5GS C-Plane log when executed is as follows.
```
04/26 02:26:59.273: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:461)
04/26 02:26:59.273: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2777)
04/26 02:26:59.273: [amf] INFO:     RAN_UE_NGAP_ID[2] AMF_UE_NGAP_ID[2] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:622)
04/26 02:26:59.273: [amf] INFO: [suci-0-001-01-0000-0-0-0000000001] Unknown UE by SUCI (../src/amf/context.c:1912)
04/26 02:26:59.273: [amf] INFO: [Added] Number of AMF-UEs is now 2 (../src/amf/context.c:1688)
04/26 02:26:59.273: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1670)
04/26 02:26:59.273: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:183)
04/26 02:26:59.274: [sbi] INFO: [d9fbf9ae-40cb-41f1-aaec-65d2e8781875] Setup NF Instance [type:AUSF] (../lib/sbi/path.c:307)
04/26 02:26:59.274: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.274: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:59.274: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.275: [sbi] INFO: [d9fcd856-40cb-41f1-9d6c-25711c5cc5c2] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
04/26 02:26:59.275: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.277: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
04/26 02:26:59.278: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.278: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.279: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.280: [ausf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/ausf/nudm-handler.c:337)
04/26 02:26:59.282: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:59.282: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.282: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.284: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:59.284: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.284: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.286: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.286: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.287: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.288: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.288: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:361)
04/26 02:26:59.288: [sbi] INFO: [d9fcc5c8-40cb-41f1-b534-cd6a98411397] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
04/26 02:26:59.289: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.289: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/pcf/npcf-handler.c:114)
04/26 02:26:59.289: [sbi] INFO: [d9fcd856-40cb-41f1-9d6c-25711c5cc5c2] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
04/26 02:26:59.290: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.291: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
04/26 02:26:59.494: [gmm] INFO: [imsi-001010000000001] Registration complete (../src/amf/gmm-sm.c:3146)
04/26 02:26:59.494: [amf] INFO: [imsi-001010000000001] Configuration update command (../src/amf/nas-path.c:609)
04/26 02:26:59.494: [gmm] INFO:     UTC [2026-04-25T17:26:59] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
04/26 02:26:59.494: [gmm] INFO:     LOCAL [2026-04-26T02:26:59] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:556)
04/26 02:26:59.494: [amf] INFO: [Added] Number of AMF-Sessions is now 2 (../src/amf/context.c:2798)
04/26 02:26:59.494: [gmm] INFO: UE SUPI[imsi-001010000000001] DNN[internet] LBO[0] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1419)
04/26 02:26:59.494: [gmm] INFO: V-SMF Instance [da0d4d30-40cb-41f1-8f8f-495f513f9186](LIST) (../src/amf/gmm-handler.c:1496)
04/26 02:26:59.494: [gmm] INFO: [da0d4d30-40cb-41f1-8f8f-495f513f9186] Setup NF Instance [type:SMF] (../src/amf/gmm-handler.c:1498)
04/26 02:26:59.494: [gmm] INFO: V-SMF Instance [da0d4d30-40cb-41f1-8f8f-495f513f9186] (../src/amf/gmm-handler.c:1508)
04/26 02:26:59.494: [gmm] INFO: V-SMF discovered in Non-Roaming or LBO-Roaming[0] (../src/amf/gmm-handler.c:1577)
04/26 02:26:59.494: [gmm] INFO: nsmf_pdusession [1:0x5ef12fe2fc70:(nil)] (../src/amf/gmm-handler.c:1617)
04/26 02:26:59.494: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.495: [smf] INFO: [Added] Number of SMF-UEs is now 2 (../src/smf/context.c:1069)
04/26 02:26:59.495: [smf] INFO: [Added] Number of SMF-Sessions is now 2 (../src/smf/context.c:3393)
04/26 02:26:59.495: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:331)
04/26 02:26:59.495: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:59.495: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.496: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.498: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.498: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:456)
04/26 02:26:59.499: [sbi] INFO: [d9fcc5c8-40cb-41f1-b534-cd6a98411397] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
04/26 02:26:59.499: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.499: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/pcf/npcf-handler.c:448)
04/26 02:26:59.500: [sbi] INFO: [d9fcd856-40cb-41f1-9d6c-25711c5cc5c2] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
04/26 02:26:59.499: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:140)
04/26 02:26:59.500: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.501: [sbi] INFO: [d9fb9ba8-40cb-41f1-879d-2b474d40280a] Setup NF Instance [type:BSF] (../lib/sbi/path.c:307)
04/26 02:26:59.501: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.502: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/pcf/nbsf-handler.c:121)
04/26 02:26:59.502: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:373)
04/26 02:26:59.503: [smf] INFO: UE SUPI[imsi-001010000000001] DNN[internet] IPv4[10.45.0.3] IPv6[] (../src/smf/npcf-handler.c:594)
04/26 02:26:59.511: [sbi] INFO: [d8c3b9a0-40cb-41f1-a42d-95705edfd14d] Setup NF Instance [type:AMF] (../lib/sbi/path.c:307)
04/26 02:26:59.511: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.513: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.514: [sbi] INFO: [d9fc8e00-40cb-41f1-98a9-bbdc447dca91] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
04/26 02:26:59.514: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.515: [sbi] INFO: [d9fcd856-40cb-41f1-9d6c-25711c5cc5c2] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
04/26 02:26:59.515: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
04/26 02:26:59.516: [amf] INFO: [imsi-001010000000001:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:954)
```
The OAI-CN5G-UPF log when executed is as follows.
```
[2026-04-26 02:26:59.511] [upf_n4 ] [info] handle_receive(707 bytes)
[2026-04-26 02:26:59.511] [upf_app] [info] 
[2026-04-26 02:26:59.511] [upf_app] [info] ╔═════════════════════════════════════════════════════════════════════════════╗
[2026-04-26 02:26:59.511] [upf_app] [info] │             Received N4_SESSION_ESTABLISHMENT_REQUEST seid 0x0              │
[2026-04-26 02:26:59.511] [upf_app] [info] ╚═════════════════════════════════════════════════════════════════════════════╝
[2026-04-26 02:26:59.511] [upf_n4 ] [info] pfcp_session::add(far) seid 0x2 FAR=1
[2026-04-26 02:26:59.511] [upf_n4 ] [info]   └─ Adding new FAR 1 to session 0x2
[2026-04-26 02:26:59.511] [upf_n4 ] [info] pfcp_session::add(far) seid 0x2 FAR=2
[2026-04-26 02:26:59.511] [upf_n4 ] [info]   └─ Adding new FAR 2 to session 0x2
[2026-04-26 02:26:59.511] [upf_n4 ] [info] pfcp_session::add(far) seid 0x2 FAR=3
[2026-04-26 02:26:59.511] [upf_n4 ] [info]   └─ Adding new FAR 3 to session 0x2
[2026-04-26 02:26:59.513] [pfcp_switch] [info] Route created
[2026-04-26 02:26:59.515] [pfcp_switch] [info] Source NAT added
[2026-04-26 02:26:59.516] [pfcp_switch] [info] Route created
[2026-04-26 02:26:59.518] [pfcp_switch] [info] Source NAT added
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x2 PDR=1
[2026-04-26 02:26:59.518] [upf_n4 ] [info]   └─ Adding new PDR 1 to session 0x2
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::set(fteid) seid 0x2 
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::get(fteid) seid 0x2 
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x2 PDR=2
[2026-04-26 02:26:59.518] [upf_n4 ] [info]   └─ Adding new PDR 2 to session 0x2
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::set(fteid) seid 0x2 
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::get(fteid) seid 0x2 
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x2 PDR=3
[2026-04-26 02:26:59.518] [upf_n4 ] [info]   └─ Adding new PDR 3 to session 0x2
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::set(fteid) seid 0x2 
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::get(fteid) seid 0x2 
[2026-04-26 02:26:59.518] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x2 PDR=4
[2026-04-26 02:26:59.518] [upf_n4 ] [info]   └─ Adding new PDR 4 to session 0x2
[2026-04-26 02:26:59.521] [upf_n4 ] [info] handle_receive(75 bytes)
[2026-04-26 02:26:59.522] [upf_app] [info] 
[2026-04-26 02:26:59.522] [upf_app] [info] ╔═════════════════════════════════════════════════════════════════════════════╗
[2026-04-26 02:26:59.522] [upf_app] [info] │             Received N4_SESSION_MODIFICATION_REQUEST seid 0x2               │
[2026-04-26 02:26:59.522] [upf_app] [info] ╚═════════════════════════════════════════════════════════════════════════════╝
[2026-04-26 02:26:59.522] [upf_n4 ] [info] pfcp_session::update(far) seid 0x2 FAR=1
[2026-04-26 02:26:59.522] [upf_n4 ] [info]   └─ Updating FAR 1 in session 0x2
```
Looking at the console log of the `nr-ue` command, UE1 has been assigned the IP address `10.45.0.3` from Open5GS 5GC.
```
[2026-04-26 02:26:59.566] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up in namespace[ueransim-001010000000001-internet].
```
Just in case, after logging in VM3 from another terminal, move to netns:`ueransim-001010000000001-internet` and make sure it matches the IP address of the UE1's TUNnel interface.
```
# ip netns exec ueransim-001010000000001-internet ip addr show
...
6: uesimtun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.3/24 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::95a7:4d75:48c5:6900/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```
**Don't forget [Setup veth pair for UE1 and PC2/PC3](#setup_ue1).**

<a id="run_pc1"></a>

### Run tcpdump on PC1

On PC1, run `tcpdump` on `veth-pc1` to check Frame Routing of UE0 (`192.168.20.0/24`).
```
# ip netns exec pc1 tcpdump -l -i veth-pc1 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on veth-pc1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="run_pc2"></a>

### Run tcpdump on PC2

On PC2, run `tcpdump` on `veth-pc2` to check Frame Routing of UE1 (`192.168.21.0/24`).
```
# ip netns exec pc2 tcpdump -l -i veth-pc2 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on veth-pc2, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="run_pc3"></a>

### Run tcpdump on PC3

On PC1, run `tcpdump` on `veth-pc3` to check Frame Routing of UE1 (`192.168.22.0/24`).
```
# ip netns exec pc3 tcpdump -l -i veth-pc3 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on veth-pc3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="run_pc4"></a>

### Run tcpdump on PC4

On PC4, run `tcpdump` on `veth-pc4` and confirm that no frame routing is configured for UE0 (`192.168.23.0/24`).
```
# ip netns exec pc4 tcpdump -l -i veth-pc4 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on veth-pc4, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="ping"></a>

## Ping Framed Routes

<a id="ping_pc1"></a>

### Ping IP address (192.168.20.100/24) of Framed Routes of UE0 on PC1

On EXT (External Node), ping IP address (`192.168.20.100/24`) of Framed Routes of UE0 and confirm with `tcpdump` running on PC1.
```
# ping 192.168.20.100
PING 192.168.20.100 (192.168.20.100) 56(84) bytes of data.
64 bytes from 192.168.20.100: icmp_seq=1 ttl=62 time=0.978 ms
64 bytes from 192.168.20.100: icmp_seq=2 ttl=62 time=0.916 ms
64 bytes from 192.168.20.100: icmp_seq=3 ttl=62 time=0.987 ms
```
The `tcpdump` log on PC1 is as follows.
```
02:55:37.840517 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 2294, seq 1, length 64
02:55:37.840528 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 2294, seq 1, length 64
02:55:38.841623 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 2294, seq 2, length 64
02:55:38.841633 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 2294, seq 2, length 64
02:55:39.851988 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 2294, seq 3, length 64
02:55:39.851998 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 2294, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC2 and PC3.**

<a id="ping_pc2"></a>

### Ping IP address (192.168.21.100/24) of Framed Routes of UE1 on PC2

On EXT (External Node), ping IP address (`192.168.21.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on PC2.
```
# ping 192.168.21.100
PING 192.168.21.100 (192.168.21.100) 56(84) bytes of data.
64 bytes from 192.168.21.100: icmp_seq=1 ttl=62 time=0.855 ms
64 bytes from 192.168.21.100: icmp_seq=2 ttl=62 time=1.00 ms
64 bytes from 192.168.21.100: icmp_seq=3 ttl=62 time=0.949 ms
```
The `tcpdump` log on PC2 is as follows.
```
02:56:20.488198 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 2295, seq 1, length 64
02:56:20.488209 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 2295, seq 1, length 64
02:56:21.516394 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 2295, seq 2, length 64
02:56:21.516405 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 2295, seq 2, length 64
02:56:22.517610 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 2295, seq 3, length 64
02:56:22.517620 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 2295, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC1 and PC3.**

<a id="ping_pc3"></a>

### Ping IP address (192.168.22.100/24) of Framed Routes of UE1 on PC3

On EXT (External Node), ping IP address (`192.168.22.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on PC3.
```
# ping 192.168.22.100
PING 192.168.22.100 (192.168.22.100) 56(84) bytes of data.
64 bytes from 192.168.22.100: icmp_seq=1 ttl=62 time=0.901 ms
64 bytes from 192.168.22.100: icmp_seq=2 ttl=62 time=0.902 ms
64 bytes from 192.168.22.100: icmp_seq=3 ttl=62 time=0.954 ms
```
The `tcpdump` log on PC3 is as follows.
```
02:56:59.657354 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 2296, seq 1, length 64
02:56:59.657365 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 2296, seq 1, length 64
02:57:00.684699 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 2296, seq 2, length 64
02:57:00.684709 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 2296, seq 2, length 64
02:57:01.708760 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 2296, seq 3, length 64
02:57:01.708769 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 2296, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC1 and PC2.**

<a id="ping_pc4"></a>

### Ping IP address (192.168.23.100/24) not configured for Framed Routes

On EXT (External Node), ping IP address (`192.168.23.100/24`) that is not configured in Framed Routes, and confirm no packets with `tcpdump` running on PC4.
```
# ping 192.168.23.100
PING 192.168.23.100 (192.168.23.100) 56(84) bytes of data.
```
**Also make sure there are no tcpdump logs on PC1, PC2 and PC3.**

---
I was able to confirm the very simple configuration for Framed Routing.
In practice, I think that PSA-UPF and UE will require more complex network routing configuration.
In this article, I kept the minimum settings necessary to check Framed Routing.
Also in this scenario, UE0 and UE1 only serve routing and not NAT. You may run `ping` and `iperf3` commands bidirectionally between PC1, PC2, PC3 and EXT.

I would like to thank the excellent developers and all the contributors of Open5GS, OAI-CN5G-UPF and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2026.04.25] Changed to the method that uses network namespaces for UERANSIM gNodeB and UE.
- [2026.02.11] Initial release.
