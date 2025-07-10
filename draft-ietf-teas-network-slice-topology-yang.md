---
coding: utf-8

title: IETF Network Slice Topology YANG Data Model

abbrev: Network Slice Topology Data Model
docname: draft-ietf-teas-network-slice-topology-yang-01
workgroup: TEAS Working Group
category: std
ipr: trust200902

stand_alone: yes
pi: [toc, sortrefs, symrefs, comments]

author:
  -
    name: Xufeng Liu
    org: Alef Edge
    email: xufeng.liu.ietf@gmail.com
  -
    name: Luis M. Contreras
    org: Telefonica
    email: luismiguel.contrerasmurillo@telefonica.com
  -
    name: Sergio Belotti
    org: Nokia
    email: Sergio.belotti@nokia.com
  -
    name: Aihua Guo
    org: Futurewei
    email: aihuaguo.ietf@gmail.com
  -
    name: Italo Busi
    org: Huawei
    email: italo.busi@huawei.com

contributor:
  -
    name: Reza Rokui
    org: Ciena
    email: rrokui@ciena.com
  -
    name: Jeff Tantsura
    org: Microsoft
    email: jefftant.ietf@gmail.com
  -
    name: Igor Bryskin
    org: Individual
    email: i_bryskin@yahoo.com
  -
    name: Qin Wu
    org: Huawei
    email: bill.wu@huawei.com

--- abstract

   An RFC 9543 network slice customer may utilize intent-based topologies to
   express resource reservation intentions within the provider's network.
   These customer-defined intent topologies allow customers to request
   shared resources for future connections that can be flexibly allocated
   and customized. Additionally, they provide an extensive level of control
   over underlay service paths within the network slice.
   
   This document describes a YANG data model for expressing customer intent
   topologies which can be used to enhance the RFC 9543 Network Slice Services
   in specific use cases, such as Network wholesale scenarios, where both topology
   and connectivity intents need to be expressed.
 
--- middle

# Introduction

   Network service providers utilize topologies to convey controlled information
   about their networks, such as bandwidth availability and connectivity, 
   with customers, to facilitates customer service requests. Customers can also
   define intent-based topologies to streamline their internal operations. When
   requesting provider support for such custom topologies, they are considered
   as customer intent topologies.
   
   In the context of network slicing, customer intent topologies enables customers
   to express resource reservation preferences. These topologies allow flexible
   configuration and activation of network slices on demand. By providing full
   control over resource allocation timing and methods, customer intent topologies
   ensure that resources are consistently available. Moreover, the resources
   reserved via customer intent topologies can be shared across network slices created
   at different times or between different connectivity constructs within the same slice.
   Compared to network slices with dedicated full-mesh connectivity constructs between
   endpoints, network slices utilizing customer intent topologies can reduce overall
   resource requirements, offering significant economic benefits to the customer.

   Consider a hub-and-spoke network slice scenario where multiple customer spoke
   sites dynamically connect to a central hub site, sharing available bandwidth.
   By designing a customer intent topology with two virtual nodes - one representing
   all the spoke sites and the other representing the hub site - connected via
   a shared link, we proactively reserve resources for the shared connection.
   This ensures that bandwidth is readily available whenever the customer requires
   it. In contrast, achieving equivalent bandwidth assurance through individual
   dedicated connectivity constructs would necessitate creating separate links between each
   spoke and the hub, which would lead to substantial bandwidth inefficiency.
   
   Customer intent topology complements connectivity-based network slicing by providing
   customers a mechanism to specify additional underlay service paths to gain
   extensive control over specific or all connectivity constructs within the network slice,
   as outlined in {{?RFC9543}}.

   A customer intent topology is defined within the customer's context. It can include pure customer information or may also refer to network
   resources identifiable within the provider's context. There is a minimum
   level of a-prior shared knowledge between the customer and the provider,
   and this is the same information needed to supported connectivity-based
   network slice services as described in {{?RFC9543}}.
   The provider's responsibility lies in understanding the customer intent topology request and translating that into suitable realization within their domain.

   This document introduces a YANG data model, based on {{!RFC7950}}, for
   configuring customer intent topologies. The YANG model extends the existing
   data model from {{!RFC8345}}, allowing customers to express desired
   service-level objectives (SLOs) and service-level expectations (SLEs)
   across different elements within the customer intent topology.
      
   The defined data model serves as an interface between customers and providers,
   enabling configurations and state retrievals for network slicing as a service.
   Customers can use this model to request or negotiate the creation of network
   slice instances. Additionally, they can incrementally adjust requirements for
   individual topology elements within the slice - for instance, adding or removing
   nodes or links, updating link bandwidth - and retrieve operational states.
   Leveraging other IETF mechanisms and data models, telemetry information can
   also be convey to the customer.

   The YANG model encompasses constructs that are independent of specific technologies,
   accommodating network slicing across diverse layers (including IP/MPLS, MPLS-TP,
   OTN, and WDM optical). As a result, this model serves as a foundational framework
   upon which technology-specific network slicing models - such as 
   {{!I-D.ietf-ccamp-yang-otn-slicing}} - can be developed.

   Section 3 of {{?I-D.ietf-teas-ns-controller-models}} outlines that the use
   of customer intent topologies and resource reservation control is optional within network
   slicing. These features complement the data model defined in 
   {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}}.
   
   The YANG data model in this document conforms to the Network
   Management Datastore Architecture (NMDA) {{!RFC8342}}.

## Use Case Applicability

   In Traffic Engineering (TE)-enabled networks like Layer-0/1 transport (OTN, MW, DWDM), customer intent topology is useful for routing RFC 9543 network slices across varied paths with TE constraints. Thus, most of the use cases for which this model target are transport oriented. Nonetheless, it is also relevant to non-transport networks like IP/MPLS, where customers may use intent topologies to influence the realization of network slices. These intents help build the logical view of the desired RFC 9543 Network Slice service (and its constituent parts), aiding providers in fulfilling slice requests and defining the service instantiation.

### Use Case 1 : Multi-tenancy in Network Wholesaling

   A typical use case in which the customer intent topology is essential is the wholesale multi-tenant case. Here, customer C may acquire a network slice from provider P and resell sub-slices to other customers/tenants. The creation of these sub-slices within C's slice necessitates specifying a topology intent - reflecting the topology of C's purchased slice - as a key input parameter.

### Use Case 2 : Scoped Connectivity Constructs in Network Slicing

   The current expression of slice requests leveraging on {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}} allows the customer to request distinct connectivity constructs as part of the same Network Resource Partition (NRP). The topology provided by the customer could imply different NRPs, instead.

   As another example, a slice request leveraging {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}} without topology differentiation could result in all connectivity constructs being realized in the same manner on the same NRP, e.g. implementing all of them within the same VRF in a L3VPN. Using topological views can help providers infer differentiated realizations of some of the connectivity constructs, for instance, by implementing them on different VRFs. This approach can offer operational advantages, like limiting the necessary VRF reconfiguration to only those affected connectivity constructs when adding new nodes or SDPs.

   Finally, by using customer intent topology it can be easier for the slice provider to infer different technologies for sets of connectivity constructs of every topology segment (e.g., IP/MPLS, optical, microwave, etc).

## Terminologies and Notations

   The following terminologies for describing network slices are defined in 
   {{?RFC9543}} and are not redefined herein.
   
   - Network Slice (NS)
   
   - Network Slice Customer
   
   - Network Slice Service Provider
   
   - Network Slice Controller (NSC)
   
   - Network Resource Partition (NRP)
   
   The following terms are defined and used in this document.
   
   - Customer Intent Topology:
       A topology defined by the customer and provided as input to the
       network slice service provider (specifically, the Network Slice
       Controller or NSC). It represents the customer's desired network
       topology.
       
   - Abstract Topology:
       A topology exposed to the customer by the network slice service
       provider prior to the creation of network slices. The provider
       may optionally uses an abstract topology to expose useful information,
       such as available resources to the customer, which can facilitate
       the build-up of customer intent topologies by the customer.
       
   - NRP Topology:
       A topology internal to the NSC to facilitate the mapping of
       network slices to underlying network resources.

## Tree Diagram

   Tree diagrams used in this document follow the notation defined in
   {{!RFC8340}}.
   
## Prefixes in Data Node Names

   In this document, names of data nodes and other data model objects
   are prefixed using the standard prefix associated with the
   corresponding YANG imported modules, as shown in {{tab-prefixes}}.

   | Prefix   | YANG Module                  | Reference         |
   |----------|------------------------------|-------------------|
   | yang     | ietf-yang-types              | {{!RFC6991}}      |
   | inet     | ietf-inet-types              | {{!RFC6991}}      |
   | nt       | ietf-network-topology        | {{!RFC8345}}      |
   | nw       | ietf-network-topology        | {{!RFC8345}}      |
   | tet      | ietf-te-topology             | {{!RFC8795}}      |
   | ns-path  | ietf-ns-underlay-path        | RFC XXXX          |
   | ns-topo  | ietf-ns-topo                 | RFC XXXX          |
   | ietf-nss | ietf-network-slice-service   | RFC YYYY          |
{: #tab-prefixes title="Prefixes and Corresponding YANG Modules"}

RFC Editor Note:
Please replace XXXX with the RFC number assigned to this document.
Please replace YYYY with the RFC number assigned to {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}}.
Please remove this note.

# Modeling Considerations

   A network slice topology is a customer intent topology 
   modeled as network topology defined in {{!RFC8345}}, with augmentations. 
   A new network type "network-slice" is defined in this document.  
   When a network topology data instance contains the network-slice 
   network type, it represents an instance of a network slice 
   topology.
   
   This data model augments the network topology model by incorporating 
   intent-based Service-Level Objectives (SLOs) and Service-Level
   Expectations (SLEs). These apply to various components within the
   customer intent topology, including nodes, links, and termination points (TPs).

## Relationship with Traffic Engineering (TE)-based Topology
   
   The model defined in this document can be combined through multi-inheritance
   with other topology data models, such as Traffic Engineering (TE) topologies
   described in {{!RFC8795}} or Optical Transport Network (OTN) topologies
   described in {{!I-D.ietf-ccamp-otn-topo-yang}}. This flexibility allows
   the creation of technology-specific customer intent topologies tailored to
   specific network requirements.

## Relationship with ACTN Virtual Network (VN)
   
   The ACTN VN model, defined in {{!RFC9731}}, provides a self-consistent set of methods for expressing connectivity intents (Type 1 VN),
   optional path constraints and topology intents (Type 2 VN), using TE metrics and TE objective functions defined in
   {{!RFC8795}}. Type 2 VN path constraints rely on Type 1 VN for expressing connectivity intents. See {{vn-intro}} for more details.
   
   On the other hand, RFC9543 network slice services provide connectivity intents equivalent to Type 1 VN, using SLO and SLE attributes in a technology-agnostic manner not tied to TE technologies. This
   distinction is detailed in {{Appendix D of !I-D.ietf-teas-ietf-network-slice-nbi-yang}}.
   
   The proposed models in this draft aim to deliver a solution equivalent to Type 2 VN to provide optional path constraints and topology intent within the
   context of RFC 9543 network slicing. These models complement the existing solution outlined in
   {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}}, while ensuring consistent use of SLO and SLE attributes in a technology-agnostic manner to express customer intent.

In a nutshell:

- the data models, defined in this draft, are intended to be used when there is a need to extend, with more control over network resources allocation by the customer, the connectivity service intent, expressed using the Network Slice Service data model, defined in {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}};
- the VN type 2 data models, defined in {{!RFC9731}}, are intended to be used when there is a need to extend, with more control over network resources allocation by the customer, the connectivity service intent expressed using the VN type 1 data models, defined in {{!RFC9731}}.

{{Appendix D of !I-D.ietf-teas-ietf-network-slice-nbi-yang}} provides guidance to decide when to use the Network Slice Service data model, defined in {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}}, or the VN type 1 data models, defined in {{!RFC9731}}, to express the connectivity intent.

## Relationship with Service Attachment Point (SAP) Topology
 
   {{!RFC9408}} introduces a YANG data model that represents an abstract
   view of the provider network topology. This model includes a list of
   Service Attachment Points (SAPs), where customer services can be
   connected. The SAP topology is made visible to customers by the provider
   before configuring network slice services. In contrast, the customer
   intent topology described in this document captures a customer's
   intentions, while the provider acts as the recipient of these intents.
   As a result, these two models serve distinct purposes.
   
   In certain scenarios, customers can leverage the SAP topology to
   construct customer intent topologies to aid in the realization
   of their intended network configurations. For instance, within a node
   of a customer intent topology, the Link Termination Point (LTP)
   identifiers may explicitly reference their supporting Termination
   Points (TPs), which correspond to the SAPs exposed in the provider's
   SAP model. However, the specifics of this mechanism fall beyond the
   scope of this document.

## Data Model Relationship

   The data model presented in this document builds upon the generic network
   topology model defined in {{!RFC8345}}. Other data models, including OTN
   Slicing (as defined in {{!I-D.ietf-ccamp-yang-otn-slicing}}), can leverage
   this extended model.
   
   The relationship of the related data models is illustrated in {{fig-model-relationship}}. 
   Within this diagram, the box outlined with dotted lines specifically represents
   the data model defined in this document.
  
~~~~
     +----------+                 +----------+
     | Network  |                 | Network  |
     | Slice    |                 | Topology +
     | NBI YANG +------+          | Model    |
     | Model    |      |          | RFC 8345 |
     +----+-----+      |          +-----+----+
          |            |                |
          |augments    |augments        |augments
          |            |                |
     +----^-----+      |          ......^.....
     | OTN      |      +----------< Network  :
     | Slicing  | augments        : Slice    :
     | Model    >-----------------: Topology :
     |          |                 : Model    :
     +----------+                 :..........:
~~~~
{: #fig-model-relationship title="Model Relationship"}
  
# Model Applicability

   Network slicing can be achieved through various technologies. The data
   model defined in this document serves as a means for configuring
   resource reservation-based network slices. In this approach, resources for
   network slices are reserved and represented using a customer intent topology.
   This topology can then be mapped to a network resource partition (NRP)
   and realized based on the scenarios outlined in {{?RFC9543}}.
   
   Network slices can be abstracted in various ways, depending on the specific
   requirements of the network slice customer. For instance, a customer might
   request a network slice with direct connectivity between pairs of Service
   Demarcation Points (SDPs). Within this network slice, each connectivity construct
   could be further supported by an end-to-end tunnel that follows a specific path
   defined in a customer intent topology, which the customer provides. The 
   resources associated with each link are immediately commissioned during
   the network slice configuration process.
   
   Alternatively, a customer can request resources to be reserved for potential
   network slices through a customer intent topology. These reserved resources
   are not immediately commissioned at the time of the request. Instead, they
   serve as a pool of allocated resources that the customer can utilize to build
   network slices in the future. By adopting this approach, customers gain the
   flexibility to share resources across multiple endpoints and activate them
   on demand.

   In the example shown in {{fig-ns-topo-example}}, two topology intents named as
   Network Slice Blue and Network Slice Red, are created
   by separate customers and delivered to the network slice service provider.
   The provider maps the two intents to corresponding network resource partitions (NRPs)
   internally. In realizing the network resource partitions, node virtualization
   is used to separate and allocate resources in physical devices.  Two virtual
   routers VR1 and VR2 are created over physical router R1, and two virtual
   routers VR3 and VR4 are created over physical router R2, respectively.  Each of the
   virtual routers,as a partition of the physical router, takes a portion 
   of the resources such as ports and memory in the physical router.  
   Depending on the requirements and the implementations, they may share 
   certain resources such as processors, ASICs, and switch fabric.

   A network slice customer has the capability to configure customer intent topologies
   without needing any prior knowledge of the provider's network or resource
   availability. However, this approach could potentially create challenges for
   the provider in understanding and realizing the intended topology.

   Alternatively, the provider can choose to describe the available resources and
   capabilities in the form of an abstract topology, which is then exposed to the
   customer before network slice requests. By doing so, the provider empowers the
   customer to build their customized intent topologies based on this pre-exposed
   information. This approach streamlines the process, minimizing unnecessary
   negotiations between the customer and the provider. The process and the data models
   for the provider to expose abstract topologies are outside the scope of this document. 

   The provider communicates the operational state of the customer intent topology, reflecting the allocated resources that result from negotiations between the customer and the provider. Subsequently, customers can process the requested customer intent topology and seamlessly integrate it into their own network topology. Importantly, this relationship between the customer and provider can be recursive. For instance, a customer who requests network slices can also serve as a provider, offering network slice services to its own customers further up the hierarchy.

   As an example, Appendix B. shows the JSON encoded data instances of
   the customer topology intent for Network Slice Blue.

~~~~
       Customer Topology (Merged)          Customer Topology (Merged)
       Network Slice Blue                  Network Slice Red
                            +---+         +---+      +---+
                       -----|R3 |---   ---|R1 |------|R2 |
                      /     +---+         +---+      +---+                  
      +---+      +---+        ^             ^          ^  \     +---+
   ---|R1 |------|R2 |        |             |          |   -----|R4 |---
      +---+      +---+        |             |          |        +---+
        ^          ^          v             v          v          ^
        |          |        +---+         +---+      +---+        |
        |          |   -----|VR5|---   ---|VR2|------|VR4|        |
        v          v  /     +---+         +---+      +---+        v         
      +---+      +---+                                    \     +---+
   ---|VR1|------|VR3|                                     -----|VR6|---
      +---+      +---+                                          +---+
       Customer Topology (Intended)        Customer Topology (Intended)
       Network Slice Blue                  Network Slice Red

                                 Customers
   ---------------------------------------------------------------------
                                 Provider

        Customized Topology (Network Resouce Partition)
        Provider Network with Virtual Devices

        Network Slice Blue: VR1, VR3, VR5         +---+             
                                        ----------|VR5|------
                                       /          +---+
                    +---+         +---+
              ------|VR1|---------|VR3|                              
                    +---+         +---+
              ------|VR2|---------|VR4|
                    +---+         +---+
                                       \          +---+
                                        ----------|VR6|------
        Network Slice Red: VR2, VR4, VR6          +---+

                                 Virtual Devices 
   ---------------------------------------------------------------------
                                 Physical Devices

        Native Topology
        Provider Network with Physical Devices
                                                  +---+
                                        ----------|R3 |------
                                       /          +---+
                    +---+         +---+
              ======|R1 |=========|R2 |                             
                    +---+         +---+
                                       \          +---+
                                        ----------|R4 |------
                                                  +---+
~~~~
{: #fig-ns-topo-example title="Network Slicing Topologies for Virtualization"} 

# YANG Model Overview

   The YANG data model in this draft consists of two modules for flexible use
   and augmentation:
   - The first YANG module defines a customer intent topology, with SLO and SLE
   associated with the topological constructs.
   - The second YANG module extends the YANG model defined in 
   {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}} by adding underlay paths to
   the connectivity constructs.

   Within the YANG model, the following constructs and attributes are defined:
   - Network Topology: This represents a set of shared and reserved resources,
   organized as a virtual topology connecting all endpoints. Customers can utilize
   this network topology to define detailed connectivity paths traversing the
   topology. Additionally, it enables resource sharing between different endpoints.

   - Service-Level Objectives (SLOs): These objectives are associated with
   various objects within the topology, including nodes, links, and termination
   points. SLOs provide guidelines for achieving specific performance or quality
   targets.
   
# Model Tree Structure

## Network Slice Topology Model Tree Structure

~~~~
{::include ./ietf-ns-topo.tree}
~~~~
{: #fig-ietf-ns-topo-tree title="Tree diagram for network slice topology"}

## Network Slice Underlay Path Model Tree Structure

~~~~
{::include ./ietf-ns-underlay-path.tree}
~~~~
{: #fig-ietf-ns-underlay-path-tree title="Tree diagram for underlay path"}
   
# YANG Modules

## YANG Module for Network Slice Topology

~~~~
   <CODE BEGINS> file "ietf-ns-topo@2025-07-03.yang"
{::include ./ietf-ns-topo.yang}
   <CODE ENDS>
~~~~
{: #fig-ietf-ns-topo-yang title="YANG model for network slice topology"}   

## YANG Module for Network Slice Underlay Path

~~~~
   <CODE BEGINS> file "ietf-ns-underlay-path@2025-07-03.yang"
{::include ./ietf-ns-underlay-path.yang}
   <CODE ENDS>
~~~~
{: #fig-ietf-ns-underlay-path title="YANG model for underlay path"}   
  
# Manageability Considerations

   To ensure the security and controllability of physical resource
   isolation, slice-based independent operation and management are
   required to achieve management isolation.
   Each network slice typically requires dedicated accounts,
   permissions, and resources for independent access and O&M. This
   mechanism is to guarantee the information isolation among slice
   tenants and to avoid resource conflicts. The access to slice
   management functions will only be permitted after successful security
   checks.

# Security Considerations

   The YANG module specified in this document defines a schema for data
   that is designed to be accessed via network management protocols such
   as NETCONF {{!RFC6241}} or RESTCONF {{!RFC8040}}.  The lowest NETCONF layer
   is the secure transport layer, and the mandatory-to-implement secure
   transport is Secure Shell (SSH) {{!RFC6242}}.  The lowest RESTCONF layer
   is HTTPS, and the mandatory-to-implement secure transport is TLS
   {{!RFC8446}}.

   The NETCONF access control model {{!RFC8341}} provides the means to
   restrict access for particular NETCONF or RESTCONF users to a
   preconfigured subset of all available NETCONF or RESTCONF protocol
   operations and content.

   There are a number of data nodes defined in this YANG module that are
   writable/creatable/deletable (i.e., config true, which is the
   default).  These data nodes may be considered sensitive or vulnerable
   in some network environments.  Write operations (e.g., edit-config)
   to these data nodes without proper protection can have a negative
   effect on network operations.  Considerations in Section 8 of
   {{!RFC8795}} are also applicable to their subtrees in the module defined
   in this document.

   Some of the readable data nodes in this YANG module may be considered
   sensitive or vulnerable in some network environments.  It is thus
   important to control read access (e.g., via get, get-config, or
   notification) to these data nodes.  Considerations in Section 8 of
   {{!RFC8795}} are also applicable to their subtrees in the module defined
   in this document.

# IANA Considerations

   It is proposed to IANA to assign new URIs from the "IETF XML
   Registry" {{!RFC3688}} as follows:

~~~~
   URI: urn:ietf:params:xml:ns:yang:ietf-ns-topo
   Registrant Contact: The IESG
   XML: N/A; the requested URI is an XML namespace.
~~~~

~~~~
   URI: urn:ietf:params:xml:ns:yang:ietf-ns-underlay-path
   Registrant Contact: The IESG
   XML: N/A; the requested URI is an XML namespace.
~~~~

   This document registers two YANG modules in the YANG Module Names
   registry {{!RFC6020}}.

~~~~
   name: ietf-ns-topo
   namespace: urn:ietf:params:xml:ns:yang:ietf-ns-topo
   prefix: ns-topo
   reference: RFC XXXX
~~~~

~~~~
   name: ietf-ns-underlay-path
   namespace: urn:ietf:params:xml:ns:yang:ietf-ns-underlay-path
   prefix: ns-path
   reference: RFC XXXX
~~~~

# Acknowledgments

   The authors would like to thank Danielle Ceccarelli, Bo Wu,
   Mohamed Boucadair, Vishnu Beeram, Dhruv Dhody, Oscar G. De Dios,
   Adrian Farrel, and many others for their valuable comments and
   suggestions.

--- back

# Relationship with ACTN Virtual Network (VN) {#vn-intro}

   {{?RFC8453}} and {{!RFC9731}} introduce the concept of a Virtual
   Network (VN), which can be presented to customers. These VNs are constructed from
   abstractions of the underlying networks, specifically those that are 
   traffic-engineering (TE) capable. While VNs share similarities with RFC 9543 network slicing,
   they operate under the assumption of TE-capable networks.

   Two distinct types of VNs are defined:

   - Type 1 VN: Modeled as a single abstract node with edge-to-edge connectivity 
     between customer endpoints.
   - Type 2 VN: Modeled as a single abstract node with an underlay topology, allowing
     configuration of intended underlay paths for connections within the single abstract
     node.

   The topologies for VNs, including both the single-node abstract topology and the
   underlay topology, can either be mutually agreed upon between the Customer Network
   Controller (CNC) and the Multi-Domain Service Coordinator (MDSC) prior to VN creation,
   or they can be created as part of VN instantiation by the customer.   
   
   In the context of network slicing, {{?RFC9543}} defines
   a network slice service as a collection of connectivity constructs between pairs of
   Service Demarcation Points (SDPs). This concept closely resembles the Type 1 VN,
   which is implemented as a single abstract node.

   {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}} further elaborates on network slices
   by incorporating references to a customer intent topology based on {{!RFC8345}}. 
   This approach aligns with the ACTN Type 2 VN, although without specifying the 
   explicit use of such a topology.

   Consequently, the data model defined in this document serves as a complementary
   option to the data model outlined in {{!I-D.ietf-teas-ietf-network-slice-nbi-yang}}.
   It empowers customers to define a customized intent topology specifically tailored
   for their network slices.

   Reusing the Type 2 VN for defining customer intent topologies alongside the RFC9543 network slice service model would result in duplicated information for connectivity intents (SDPs and connectivity-constructs vs. LTPs and connectivity matrices), and additionally, would bind the network slice solution to TE technologies (as discussed in {{Appendix D of !I-D.ietf-teas-ietf-network-slice-nbi-yang}} for VN Type 1). 
   
# Data Tree for the Example in Section 3

## Native Topology

   This section contains an example of an instance data tree in the JSON
   encoding {{!RFC7951}}.  The example instantiates "ietf-network" for the
   topology of Network Slice Blue depicted in {{fig-ns-topo-example}}.
   
~~~~
{::include json-examples/ietf-ns-topo-blue.txt}
~~~~
 
{: numbered="false"}
