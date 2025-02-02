# Release notes

## Alpha-3

### SONiC support

Broadcom Enterprise SONiC 4.2.0 (previously 4.1.1)

### Multiple IPv4 namespaces

* Support for multiple overlapping IPv4 addresses in the Fabric
* Integrated with on-demand DHCP Service (see below)
* All IPv4 addresses within a given VPC must be unique
* Only VPCs with non-overlapping IPv4 subnets can peer within the Fabric
* An external NAT device is required for peering of VPCs with overlapping subnets

### Hedgehog Fabric DHCP and IPAM Service

* Custom DHCP server executing in the controllers
* Multiple IPv4 namespaces with overlapping subnets
* Multiple VLAN namespaces with overlapping VLAN ranges
* DHCP leases exposed through the Fabric API
* Available for VLAB as well as the Fabric

### Hedgehog Fabric NTP Service

* Custom NTP servers at the controller
* Switches automatically configured to use control node as NTP server
* NTP servers can be configured to sync to external time/NTP server

### StaticExternal connections

* Directly connect external infrastructure services (such as NTP, DHCP, DNS) to the Fabric
* No BGP is required, just automatically configured  static routes

### DHCP Relay to 3rd party DHCP service
Support for 3rd party DHCP server (DHCP Relay config) through the API


## Alpha-2

### Controller

A single controller. No controller redundancy.

### Controller connectivity

For CLOS/LEAF-SPINE fabrics, it is recommended that the controller connects to one or more leaf switches in the fabric on front-facing data ports. Connection to two or more leaf switches is recommended for redundancy and performance. No port break-out functionality is supported for controller connectivity.

Spine controller connectivity is not supported.

For Collapsed Core topology, the controller can connect on front-facing data ports, as described above, or on management ports. Note that every switch in the collapsed core topology must be connected to the controller.

Management port connectivity can also be supported for CLOS/LEAF-SPINE topology but requires all switches connected to the controllers via management ports. No chain booting is possible for this configuration.

### Controller requirements

* One  1 gig+ port per to connect to each controller attached switch
* One+ 1 gig+ ports connecting to the external management network.
* 4 Cores, 12GB RAM, 100GB SSD.

### Chain booting

Switches not directly connecting to the controllers can chain boot via the data network.

### Topology support

CLOS/LEAF-SPINE and Collapsed Core topologies are supported.

#### LEAF Roles for CLOS topology

server leaf, border leaf, and mixed leaf modes are supported.

#### Collapsed Core Topology

Two ToR/LEAF switches with MCLAG server connection.

### Server multihoming

MCLAG-only.

### Device support

#### LEAFs

* DELL:
  * S5248F-ON
  * S5232F-ON

* Edge-Core:
  * DCS204 (AS7726-32X)
  * DCS203 (AS7326-56X)
  * EPS203 (AS4630-54NPE)

#### SPINEs

* DELL:
  * S5232F-ON
* Edge-Core:
  * DCS204 (AS7726-32X)

### Underlay configuration:

Port speed, port group speed, port breakouts are configurable through the API

### VPC (overlay) Implementation

VXLAN-based BGP eVPN.

### Multi-subnet VPCs

A VPC consists of subnets, each with a user-specified VLAN for external host/server connectivity.

### Multiple IP address namespaces

Multiple IP address namespaces are supported per fabric. Each VPC belongs to the corresponding IPv4 namespace. There are no subnet overlaps within a single IPv4 namespace. IP address namespaces can mutually overlap.

### VLAN Namespace

VLAN Namespaces guarantee the uniqueness of VLANs for a set of participating devices. Each switch belongs to a list of VLAN namespaces with non-overlapping VLAN ranges. Each VPC belongs to the VLAN namespace. There are no VLAN overlaps within a single VLAN namespace.

This feature is useful when multiple VM-management domains (like separate VMware clusters connect to the fabric).

### Switch Groups

Each switch belongs to a list of switch groups used for identifying redundancy groups for things like external connectivity.

### Mutual VPC Peering

VPC peering is supported and possible between a pair of VPCs that belong to the same IPv4 and VLAN namespaces.

### External VPC Peering

VPC peering provides the means of peering with external networking devices (edge routers, firewalls, or data center interconnects). VPC egress/ingress is pinned to a specific group of the border or mixed leaf switches. Multiple “external systems” with multiple devices/links in each of them are supported.

The user controls what subnets/prefixes to import and export from/to the external system.

No NAT function is supported for external peering.

### Host connectivity

Servers can be attached as Unbundled, Bundled (LAG) and MCLAG

### DHCP Service

VPC is provided with an optional DHCP service with simple IPAM

### Local VPC peering loopbacks

To enable local inter-vpc peering that allows routing of traffic between VPCs, local loopbacks are required to overcome silicon limitations.

### Scale

* Maximum fabric size: 20 LEAF/ToR switches.
* Routes per switch: 64k
  * [ silicon platform limitation in Trident 3; limits te number of endpoints in the fabric  ]
* Total VPCs per switch: up to 1000
  * [ Including VPCs attached at the given switch and VPCs peered with ]
* Total VPCs per VLAN namespace: up to 3000
  * [ assuming 1 subnet per VPC ]
* Total VPCs per fabric:  unlimited
  * [ if using multiple VLAN namespaces ]
* VPC subnets per switch: up to 3000
* VPC subnets per VLAN namespace up to 3000
* Subnets per VPC: up to 20
  * [ just a validation; the current design allows up to 100, but it could be increased even more in the future ]
* VPC Slots per remote peering @ switch: 2
* Max VPC loopbacks per switch: 500
  * [ VPC loopback workarounds per switch are needed for local peering when both VPCs are attached to the switch or for external peering with VPC attached on the same switch that is peering with external ]

### Software versions

* Fabric: v0.23.0
* Das-boot: v0.11.4
* Fabricator: v0.8.0
* K3s: v1.27.4-k3s1
* Zot: v1.4.3
* SONiC
  * Broadcom Enterprise Base 4.1.1
  * Broadcom Enterprise Campus 4.1.1

### Known Limitations

* MTU setting inflexibility:
  * Fabric MTU is 9100 and not configurable right now (A3 planned)
  * Server-facing MTU is 9136 and not configurable right now (A3+)
* no support for Access VLANs for attaching servers (A3 planned)
* VPC peering is enabled on all subnets of the participating VPCs. No subnet selection for peering. (A3 planned)
* peering with external is only possible with a VLAN (by design)
* If you have VPCs with remote peering on a switch group, you can’t attach those VPCs on that switch group (by definition of remote peering)
* if a group of VPCs has remote peering on a switch group, any other VPC that will peer with those VPCs remotely will need to use the same switch group (by design)
* if VPC peers with external, it can only be remotely peered with on the same switches that have a connection to that external (by design)
* the server-facing connection object is immutable as it’s very easy to get into a deadlock, re-create to change it (A3+)


## Alpha-1

* Controller:
    * A single controller connecting to each switch management port. No redundancy.

* Controller requirements:
    * One 1 gig port per switch
    * One+ 1 gig+ ports connecting to the external management network.
    * 4 Cores, 12GB RAM, 100GB SSD.

* Seeder:
    * Seeder and Controller functions co-resident on the control node. Switch booting and ZTP on management ports directly connected to the controller.

* HHFab - the fabricator:
    * An operational tool to generate, initiate, and maintain the fabric software appliance.  Allows fabrication of the environment-specific image with all of the required underlay and security configuration baked in.

* DHCP Service:
    * A simple DHCP server for assigning IP addresses to hosts connecting to the fabric, optimized for use with VPC overlay.

* Topology:
    * Support for a Collapsed Core topology with 2 switch nodes.

* Underlay:
    * A simple single-VRF network with a BGP control plane.  IPv4 support only.

* External connectivity:
    * An edge router must be connected to selected ports of one or both switches.  IPv4 support only.

* Dual-homing:
    * L2 Dual homing with MCLAG is implemented to connect servers, storage, and other devices in the data center.  NIC bonding and LACP configuration at the host are required.

* VPC overlay implementation:
    * VPC is implemented as a set of ACLs within the underlay VRF. External connectivity to the VRF is performed via internally managed VLANs.  IPv4 support only.

* VPC Peering:
    * VPC peering is performed via ACLs with no fine-grained control.

* NAT
    * DNAT + SNAT are supported per VPC. SNAT and DNAT can’t be enabled per VPC simultaneously.

* Hardware support:
    * Please see the supported hardware list.

* Virtual Lab:
    * A simulation of the two-node Collapsed Core Topology as a virtual environment. Designed for use as a network simulation, a configuration scratchpad, or a training/demonstration tool.  Minimum requirements: 8 cores, 24GB RAM, 100GB SSD

* Limitations:
    * 40 VPCs max
    * 50 VPC peerings
    * [ 768 ACL entry platform limitation from Broadcom ]

* Software versions:
    * Fabricator: v0.5.2
    * Fabric: v0.18.6
    * Das-boot: v0.8.2
    * K3s: v1.27.4-k3s1
    * Zot: v1.4.3
    * SONiC: Broadcom Enterprise Base 4.1.1
