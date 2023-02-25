---
title: Framework for Computing-Aware Networking
abbrev: CAN Framework
docname: draft-xxx-can-framework-00
date:
category: info
submissionType: IETF

ipr: trust200902
area: Routing area
workgroup: can
keyword: Internet-Draft

coding: us-ascii
pi: [toc, sortrefs, symrefs]

author:
 -
  ins: X. XX
  name: XXX
  organization: XXX
  email: XX@XX.COM
  country: XXX


contributor:
 -
  name: Huijuan Yao
  org: China Mobile
  email: yaohuijuan@chinamobile.com



normative:

informative:
  I-D.liu-can-ps-usecases:

...

--- abstract

This document describes a framework for Computing-Aware Networking (CAN). Particularly, the document identifies a set of CAN components, describes their interactions, and exemplifies the workflow of the control plane and data plane.

--- middle

# Introduction

Edge computing architectures have been expanding from single edge nodes to multiple interconnected collaborating edge nodes to address issues, such as long response time reduction, suboptimal service resource utilization, and inefficient network utilization.

The underlying networking architectures for edge computing usually provide relatively static service dispatching. In such architectures, the service traffic is often directed to the closest edge from a routing perspective or to the server with the most computing resources without considering the network status, and even sometimes simply based on static configurations.

As described in {{I-D.liu-can-ps-usecases}}, traffic steering that takes into account computing resource metrics would benefit several services, such as augmented reality or augmented/virtual reality (AR/VR). This document provides an architectural framework, which will enable compute- and network-aware traffic steering decisions in edge computing.

The Computing-Aware Networking (CAN) framework assumes that there may be multiple service instances running on different edge nodes, globally providing one single service. A single edge may have limited computing resources (such as CPU or GPU) available, and different edges likely have different resources available. A single edge may host multiple instances of a service, or just one.

The CAN framework is an ingress-based overlay framework for the selection of the suitable service instance(s) from a set of instance candidates. The exact characterization of 'suitable' will be determined by a combination of networking and computing related metrics. To that aim, the CAN framework assumes that edge nodes collaborate with each other under a single administrative domain to achieve a global objective of dispatching service demands, by taking into account both service instances status and network state (e.g., forwarding path length, cost, and congestion).

Also, this document describes a workflows of the main CAN procedures in both the control and data planes.


# Terminology

This document makes use of the following terms:

- Client:An endpoint that is connected to a service provider network.

- Computing-Aware Networking (CAN): An architecture that takes into account the dynamic nature of computing resources and network state to steer service traffic to a service instance. This dynamicity is expressed by means of relevant metrics.

- Service: A function that is provided by a network operator according to a specification. 

- Service instance: A run-time environment (e.g., a server or a process on a server) that makes the functionality of a service available. One service can be exposed by multiple instances running at the same or different network locations.

- CAN Service ID (CS-ID):  An identifier representing a service, which the clients use to access said service. 

- CAN Binding ID (CB-ID): An identifier of a single service instance or site of a given service instance (CS-ID).

- Service demand: The demand for a specific service identified by a specific CS-ID.

- Service request: The request for a specific service instance.

- CAN-Router: A network device (usually at the edge of the network) that makes forwarding decisions based on CAN information to steer traffic belonging to the same service demand to the selected chosen service instance.

- Ingress CAN-Router: A network edge router that serves as a service access point for CAN clients. It steers the service packets onto an overlay path to an Egress CAN-Router linked to the most suitable edge site to access a service instance.

- Egress CAN-Router: An egress endpoint of an overlay path and which connected a CAN-serviced site.

- CAN Service Metric Agent (C-SMA):Responsible for collecting service capabilities and status, and reporting them to a CAN Path Selector (C-PS).

- CAN Network Metric Agent (C-NMA): Responsible for collecting network capabilities and status, and reporting them to a C-PS.

- CAN Path Selector (C-PS): An entity that determines the path toward the appropriate service location and service instances to meet a service demand given the service status and network status information.

- CAN Traffic Classifier (C-TC): Responsible for determining which packets belong to a traffic flow for a particular service demand, and for steering them on the path to the service instance as determined by a C-PS.

# Framework and Components {#Framework-and-concepts}

## Assumptions

CAN assumes that there are multiple service instances running on different edge sites, providing one single service which is represented by the same service identifier (see {{can-ids}}.

## CAN Identifiers {#can-ids}

CAN introduces the following identifiers:

- CAN Service ID (CS-ID): An identifier representing a service, which the clients use to access said service. Such an ID identifies all of the instances of the same service, no matter on where they are actually running. The CS-ID is independent of which service instance serves the service demand. Usually multiple instances provide a (logically) single service, and service demands are dispatched to the different instance by choosing one instance among all available instances.

- CAN Binding ID (CB-ID): An identifier of a single service instance or site of a given service instance (CS-ID).

## CAN Components

The network takes forwarding decisions for a service demand received from a client according to both service instances status as well as network status. The main CAN functional elements and their interactions are shown in {{fig-can-fw}}.

~~~
      +-----+              +------+            +------+
    +------+|            +------+ |          +------+ |
    |client|+            |client|-+          |client|-+
    +------+             +------+            +------+
        |                    |                   |
        |   +------------+   |            +------------+
        +---|    C-TC    |---+     +------|    C-TC    |
            |------------|         |      |------------|
            |     | C-PS |     +------+   |CAN-Router 4|
    ........|     +------|.....| C-PS |...|            |.....
    :       |CAN-Router 2|     |      |   |            |    .
    :       +------------+     +------+   +------------+    :
    :                                                       :
    :                                            +-------+  :
    :                         Underlay           | C-NMA |  :
    :                      Infrastructure        +-------+  :
    :                                                       :
    :                                                       :
    :   +------------+                  +------------+      :
    :   |CAN-Router 1|  +-------+       |CAN-Router 3|      :
    :...|            |..| C-SMA |.......|            |......:
        +------------+  +-------+       +------------+
                |         |             |    C-SMA   |
                |         |             +------------+
                |         |                     |
                |         |                     |
              +------------+               +------------+
            +------------+ |             +------------+ |
            |  service   | |             |  service   | |
            |  instance  |-+             |  instance  |-+
            +------------+               +------------+

              edge site 1                   edge site 2
~~~
{: #fig-can-fw title="CAN Functional Components"}

Edge sites (edges for short) are the sites that provide access to edge computing resources. A compute service (e.g., a matrix computation for face recognition or a game server) is uniquely identified by a CAN Service ID (CS-ID).

Service instances can be instantiated and access through different edge sites so that a single service can have a significant number of instances running at different places in the network.

{{fig-can-fw}} shows two edge nodes (CAN-Router 1 and CAN-Router 3) that provide access to service instances. These nodes behaves as Egress CAN-Routers.

The CAN Service Metric Agent (C-SMA) is a functional component that gathers information about edge sites and server resources, as well as the status of the different service instances. The C-SMAs are located adjacent to the service instances in or next to the Egress CAN-Routers. {{fig-can-fw}} shows one C-SMA collocated with CAN-Router 3, and another C-SMA adjacent to CAN-Router 1.

The CAN Network Metric Agent (C-NMA) is a functional component that gathers information about the state of the network. The C-NMAs may be implemented as separate components or may form part of existing components such as CAN-Routers or CAN Path Selectors(C-PS). {{fig-can-fw}} shows a single C-NMA located independently within the underlay network. There may be one or more C-NMAs for an underlay network.

The C-SMAs and C-NMAs share the collected information with CAN Path Selectors (C-PSes) that use the information to select the Egress CAN-Routers (and potentially the service instances) to which to forward traffic for a service demand. They also determine the best paths (possibly tunnels) to use to forward the traffic depending on the network behavior required for the service and the current network state. The information is presented as one or more metrics and associate with the CS-ID and possible CB-ID.

There may be one or more C-PSes in the network. They can be integrated into the Ingress CAN-Routers (as with CAN-Router 2 in {{fig-can-fw}}) or they may be separate components that communicate with the Ingress CAN-Routers (as with CAN-Router 4 in {{fig-can-fw}}).

Last, CAN Traffic Classifier (C-TC) is a functional component that is responsible for selecting incoming packets that belong to the same service demand and ensuring that they are steered onto the same path toward the same service instance as instructed by a C-PS component. There is a C-TC in each Ingress CAN Router.

The Egress CAN-Routers are the egress endpoints of overlay paths. A site supporting service instances may be linked to one or more Egress CAN routers (that is, dual-homing is supported). If the C-PS has selected a specific service instance and the C-TC has marked the traffic with the CB-ID, the Egress CAN-Router simply forwards the traffic to the correct service instance. In some cases, the choice of service instance may be left open to the Egress CAN-Router (the traffic is marked only with the CS-ID) and in this case the Egress CAN-Router selects an appropriate service instance (using local knowledge of capabilities and current load), but must be sure to deliver all packets from the same service demand to the same service instance.

Note that, depending on deployment and service requirements, per-instance computing-related metrics or aggregated per-site computing-related metrics can be used by the C-PS. Using aggregated per-site computing-related metrics offers a more scalable choice, but relies on the Egress CAN-Router to consistently select a service instance.

In {{fig-can-fw}}, the "underlay infrastructure" indicates the general IP/MPLS infrastructure that is not aware of CAN. The CAN routes will be distributed among the overlay CAN-Routers assisted by the C-PS, and will not affect the underlay nodes. An implementation may use a control plane or management plane solution to distribute the server metrics, network metrics, and underlay routes - this document does not define a specific solution.

## Deployment Considerations

This document does not make an assumption about how the various CAN functional elements are implemented. Whether a CAN deployment follows a fully distributed or collocate or combine many of the functional components is deployment specific. One possibility is a centralized implementation where the computing-related metrics from the C-SMAs are collected by a centralized controller/PCE that also collects the network metrics, and which acts on service requests to produce paths and service instance selections that it programs into the relevant C-TCs.


# CAN Framework Workflow

The following subsections provide an overview of how the CAN workflow operates in a distributed functional architecture.

## Service Announcement

A service is associated with a unique identifier called a CS-ID. A CS-ID may be a network identifier, such as an IP address. The mapping of CS-ID to network identifier may be learned through a name resolution service such as DNS.

## Metrics Distribution

As described above, a C-SMA collects both service-related capabilities and metrics, and associates them with a CS-ID that identifies the service. It may aggregate the metrics for multiple service instances, or maintain them separately. The C-SMA then sends (advertises) the CS-ID along with the metrics to be received by all C-PSs in the network. The service-related metrics include computing-related metrics and potentially other service metrics, if needed.

Computing metrics may change very frequently (see {{I-D.liu-can-ps-usecases}} for a discussion). When and how frequently such information is distributed is to be determined as part of the specification of any distribution protocol. A spectrum of approaches can be considered, such as interval-based updates, threshold-triggered updates, policy-based updates, etc.

Additionally, the C-NMA collects network-related capabilities and metrics. These may be collected and distributed by existing network routing protocols, although enhancements may be needed to distribute additional information not previously available (such as link latency). The C-NMA distributes the network metrics to the C-PSes so that they can use the combination of service metrics and network metrics to determine the best Egress CAN-Router to provide access to a service instance to provide the compute function needed by a service demand.

Network metrics may also change very frequently. Routing protocols may apply elements of thresholding and policy to prevent the network being flooded with state change information. The C-NMA should apply similar processes to those used in the C-SMA to determine when and how often to distribute updates to the C-PSes.

{{fig-can-example-overlay}} shows an example to illustrate how CAN metrics can be distributed. There is a client attached to the netowrk via CAN-Router 1. There are three instances of the service with CS-ID 1: two are at Edge Site 2 attached via CAN-Router 2 and have CB-IDs 1 and 2; the third is at Edge Site 3 attached via CAN-Router 3 and with CB-ID 3. There is also a second service with CS-ID 2 with only one instance located at Edge Site 2.

In this example, the C-SMA collocated with CAN-Router 2 distributes the service metrics for both service instances. Note that these could be aggregated into a single advertisement, but in this case the metrics for each instance are advertised. Similarly, the C-SMA at Edge Site 2 advertises the service metrics for the two services offered at Edge Site 2.

The service metric advertisements are consumed by the C-PS at CAN-Router 1 that provides access for the client. The C-PS also consumes network metric advertisements from the C-NMA. All of the metrics are used by the C-PS to determine the best Egress CAN-Router to which to send traffic (and on what path) for the client's service demand given the service that is requested (CS-ID 1 or CS-ID 2), the state of the service instances reported by the metrics, and the state of the network.

~~~
            Service CS-ID 1, instance CB-ID 1 <metrics>
            Service CS-ID 1, instance CB-ID 2 <metrics>
                   :<----------------------:
                   :                       :             +-------+
                   :                       :             |CS-ID 1|
                   :                       :          +--|CB-ID 1|
                   :                +------------+    |  +-------+
                   :                |    C-SMA   |----|    Edge Site 2
                   :                +------------+    |  +-------+
                   :                |CAN-Router 2|    +--|CS-ID 1|
                   :                +------------+       |CB-ID 2|
   +--------+      :                        |            +-------+
   | Client |      :  Network +----------------------+
   +--------+      :  metrics | +-------+            |
        |          : :<---------| C-NMA |            |
        |          : :        | +-------+            |
   +-------------------+      |                      |
   |CAN-Router 1| C-PS |------|                      |
   +-------------------+      |       Underlay       |
                   :          |     Infrastructure   |     +-------+
                   :          |                      |     |CS-ID 1|
                   :          +----------------------+ +---|CB-ID 3|
                   :                    |              |   +-------+
                   :            +------------+  +-------+
                   :            |CAN-Router 3|--| C-SMA | Edge Site 3
                   :            +------------+  +-------+
                   :                                :  |   +-------+
                   :                                :  +---|CS-ID 2|
                   :                                :      +-------+
                   :<-------------------------------:
            Service CS-ID 1, instance CB-ID 3 <metrics>
            Service CS-ID 2, <metrics>

~~~
{: #fig-can-example-overlay title="Example CAN Metric Distribution"}

The above example mainly describes a per-instance computing-related metrics distribution. In the case of distributing aggregated per-site computing-related metrics, the per-instance CB-ID will not be included in the distribution. Instead, a per-site CB-ID may be used in the case of multiple sites are attached to the Egress CAN router to indicate from which site the aggregated metrics come. 

However, CB-ID is not needed in the distribution (in above example) if the edge site can support consistently service instance selection. For easy deployment, aggregated per-site computing-related metrics distribution is recommeded.


## Service Demand Handling

The C-PS determines the best choice of Egress CAN-Router and path to that router given the service and network metrics distributed as described in the previous section. The C-PS may be collocated with the Ingress CAN-Router (as shown in {{fig-can-example-overlay}}) or the selection function may be offloaded to a dedicated server.

This document does not describe any algorithms for path selection, but it is expected that a service demand or local policy may give directions to the C-PS in terms of "objective functions" that describe the desired behavior from the network and the chosen service instance.

In the example in {{fig-can-example-overlay}}, when the client sends a service demand to CAN-Router 1, the router consults the C-PS to select a service instance at a remote edge site accessed through a particular Egress CAN-Router. The C-PS also determines a path to the Egress CAN-Router. This information is provided to the Ingress CAN-Router (CAN-Router 1) so that it can correctly encapsulate and route packets.

A service transaction consists of one or more service packets sent by the client to its Ingress CAN-Router. The Ingress CAN-Router classifies the packets using the C-TC functionality, and forwards the packets to the chosen Egress CAN-Router using an overlay path with appropriate encapsulation. When the packets reach the Egress CAN-Router, the outer header of the overlay encapsulation is removed and the inner packets are sent to the service instance.

## Instance Affinity

Instance affinity is a key feature of CAN. It means that packets from the same 'flow' for a service should always be sent to the same Egress CAN-Router to be processed by the same service instance. Furthermore, in the general case, packets from the same service flow should be sent on the same path to avoid them becoming mis-ordered and to prevent the introduction of unpredictable latency variations.

The affinity is determined at the time of newly formulated service demand.

Note that different services may have different notions of what constitutes a 'flow' and may thus identify a flow differently. Typically a flow is identified by the 5-tuple value (source and destination addresses, source and destination ports, and next protocol type). However, for instance, an RTP video streaming may use different port numbers for video and audio, and so affinity may be identified as the combination of two 5-tuple flow identifiers so that both flows are sent to the same service instance.

Hence, when specifying a protocol solution for service instance affinity, a certain level of flexibility in identifying flows should be provided. Or, from a more general perspective, there should be a flexible mechanism to specify and identify the set of packets that are subject to a service instance affinity.

More importantly, the means for identifying a flow for the purpose of ensuring instance affinity must be application-independent to avoid the need for service-specific instance affinity methods.

Specifically, Instance affinity information should be configurable on a per-service basis. For each service, the information can include the flow/packets identification type and means, affinity timeout value, and etc.

This document does not define the specific mechanisms for defining or applying service instance affinity.

# Security Considerations

TBD

# IANA Considerations

This document makes no requests for IANA action.

#Acknowledgements

The authors would like to thank Joel Halpern, John Scudder, Dino Farinacci, Adrian Farrel, Cullen Jennings, Linda Dunbar, Jeffrey Zhang, Fang Gao, Aijun Wang,Cong Li, Xinxin Yi, Dirk Trossen, Luigi IANNONE, Yizhou Li, Jari Arkko, Mingyu Wu, Haibo Wang, Xia Chen, Jianwei Mao, Guofeng Qian, Zhenbin Li, Xinyue Zhang for their help.
