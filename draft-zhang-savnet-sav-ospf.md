---
title: "Source Address Validation in OSPF Networks"
abbrev: "SAV-OSPF"
category: info

docname: draft-zhang-savnet-sav-ospf-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "SAVNET"
keyword:
 - source address validation
 - OSPF
venue:
  group: "SAVNET"
  type: "Working Group"
  mail: "savnet@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/savnet/"
  github: "wsxyer/draft-savnet-ospf-extensions"
  latest: "https://wsxyer.github.io/draft-savnet-ospf-extensions/draft-zhang-savnet-sav-ospf.html"

author:
- ins: Y. Zhang
  name: Yuanyuan Zhang
  org: Zhongguancun Laboratory
  city: Beijing
  country: China # use TLD (except UK) or country name
  email: zhangyy@zgclab.edu.cn
- ins: M. Xu
  name: Mingwei Xu
  org: Tsinghua University
  city: Beijing
  country: China # use TLD (except UK) or country name
  email: xmw@cernet.edu.cn
- ins: D. Li
  name: Dan Li
  org: Tsinghua University
  city: Beijing
  country: China # use TLD (except UK) or country name
  email: tolidan@tsinghua.edu.cn
- ins: Y. Wang
  name: Yuliang Wang
  org: Quan Cheng Laboratory
  city: Jinan
  country: China # use TLD (except UK) or country name
  email: ts-wangyl@qcl.edu.cn
- ins: L. Qin
  name: Lancheng Qin
  org: Tsinghua University
  city: Beijing
  country: China # use TLD (except UK) or country name
  email: qlc19@mails.tsinghua.edu.cn

normative:
  intra-domain-arch:
    target: https://datatracker.ietf.org/doc/draft-li-savnet-intra-domain-architecture/
    title: Intra-domain Source Address Validation (SAVNET) Architecture
    date: 2024
  RFC2827:
  RFC3704:
  RFC2328:
  RFC7684:
  RFC8362:
informative:
  RFC5250:
  RFC7770:
  intra-domain-ps:
    target: https://datatracker.ietf.org/doc/draft-ietf-savnet-intra-domain-problem-statement/
    title: Source Address Validation in Intra-domain Networks Gap Analysis, Problem Statement, and Requirements
    date: 2024
  sav-table:
    target: https://datatracker.ietf.org/doc/draft-huang-savnet-sav-table/
    title: General Source Address Validation Capabilities
    date: 2024


--- abstract

This document proposes a comprehensive intra-domain Source Address Validation (SAV) solution to deal with different source address spoofing scenarios. It can be applied to networks running OSPFv2 and OSPFv3. The Source Prefix Validation Sub-TLV is defined to support automatic SAV-specific information exchanging between area-local routers, which helps routers generate accurate SAV rules to verify the validity of data packets.

--- middle

# Introduction

In the realm of network security, the authenticity of packet sources stands as a fundamental pillar. Source address spoofing, wherein an attacker deceives a network by masquerading as a legitimate user, poses severe risks including but not limited to identity theft, unauthorized network access, and the facilitation of denial-of-service (DoS) attacks. Such activities compromise the integrity of data, disrupt service operations, and erode the trust in network communications.

Implementing robust source address validation (SAV) mechanisms within domains is crucial for mitigating these threats. Representative existing solutions for intra-domain source address validation include ACL-based ingress filtering{{RFC2827}}{{RFC3704}}, Strict uRPF{{RFC3704}}, etc. However, as analyzed in {{intra-domain-ps}}, these mechanisms often suffer from high operational overhead and limitations in their accuracy, leading to potential mismanagement and false positives in dynamic network environments.

To address these shortcomings, the Intra-domain Source Address Validation (SAVNET) Architecture{{intra-domain-arch}} proposes a framework that use both local routing information and SAV-specific information exchanged among routers to achieve accurate SAV in an intra-domain network by an automatic way.

This document follows the Intra-domain SAVNET architecture and delves into the SAV mechanisms that can be implemented in OSPF networks to address various intra-domain source address spoofing scenarios. These mechanisms leverage either routing information from the LSDB or SAV-specific information from other routers to achieve accurate and automated SAV. A new Source Prefix Validation Sub-TLV is defined to enable the exchange of SAV-specific information among OSPF routers. It equips routers with the capability to recognize all the legitimate ingress interfaces a source prefix might reach along its actual forwarding paths—paths not only defined by routing protocols but also specified by network administrators based on policies. Routers can use the accurate information they possess to prevent both the erroneous blocking of legitimate traffic and the spread of spoofed traffic within a network domain.

OSPF networks can fully or partially implement these SAV mechanisms as needed to achieve the following objectives:

- Outbound traffic validation: prevent packets with spoofed source addresses generated by the stub networks from entering or moving within the domain.

- Inbound traffic validation: prevent packets spoofing the domain’s own source addresses originating from other domains from entering the domain.

## Requirements Language

{::boilerplate bcp14-tagged}

# Terminology

{:vspace}
SAV Rule
: The rule that indicates the valid incoming interfaces for a specific source prefix or source IP address.

SAV Table
: The table or data structure that implements the SAV rules and is used for source address validation on the data plane.

SAV-specific Information
: The information specialized for SAV rule generation, which is exchanged among routers.

Policy-based Routing
: Routing based on the policies defined by the network administrator.

Improper Block
: The validation results that the packets with legitimate source addresses are blocked improperly due to inaccurate SAV rules.

Improper Permit
: The validation results that the packets with spoofed source addresses are permitted improperly due to inaccurate SAV rules.


# Source Address Validation in OSPF Networks

## Overview {#overview-sec}
In networks utilizing either OSPFv2 or OSPFv3 protocols, routers positioned at different locations within the network should undertake varying responsibilities in source address validation, collectively contributing to effective outbound and inbound traffic validation. Different validation modes as described in {{sav-table}} can be used in different scenarios as explained below.

- **Edge Source Address Validation (Edge SAV)**

  Edge routers connected directly to stub networks are pivotal as starting points for traffic entering the intra-domain network. Performing SAV at these points ensures the legitimacy of outbound traffic’s source addresses right at the entry. If an edge router is aware of all prefixes of the connected stub network and is capable of implementing SAV, these prefixes should be added to an allowlist at the interface connected to the stub network. Packets originating from the stub network with source addresses matching the allowlist should be normally forwarded, while others should be blocked, thus preventing spoofed source address packets from entering the domain at their origin.

- **Transit Source Address Validation (Transit SAV)**

  Ensuring that all edge routers support Edge SAV might be challenging in a network environment composed of multi-vendor devices managed by different operations teams, due to differences in resource investment and technical capalilities. Full deployment of Edge SAV  could be a long-term process in such a situation. If not all edge routers can achieve accurate Edge SAV, packets with spoofed source addresses from stub networks might still appear within the domain. Under such circumstances, transit routers, responsible for forwarding packets, can perform source address validation during packet transit, further ensuring the authenticity of traffic forwarded in the domain.

- **Area Border Source Address Validation (Area Border SAV)**

  In large networks utilizing the OSPF protocol, an Autonomous System (AS) may be divided into multiple areas. To prevent routing loops in multi-area scenarios, traffic between non-backbone areas is typically routed through the backbone area, rather than crossing a third non-backbone area. Packets received at the Area Border Router (ABR) interfaces connected to a specific area should only have source addresses belonging to that area and not from other areas. It is practical for an ABR to add the prefixes advertised to that area in summary LSAs to a SAV blocklist at the interfaces connected to that specific area. This prevents inter-area source address spoofing by blocking packets whose source addresses matches this blocklist.

- **AS Border Source Address Validation (AS Border SAV)**

  AS Border Routers (ASBRs) are gateways through which external traffic enters the domain. ASBRs can discern the prefix of their own AS through router-LSAs and summary-LSAs. By adding these prefixes to a blocklist at interfaces connecting to other ASs, packets from other ASs with source addresses matching this blocklist can be blocked, thereby preventing spoofed traffic from entering the domain.

For Edge SAV, Area Border SAV, and AS Border SAV, routers can establish interface-based prefix allowlists or blocklists either manually or based on the local LSDB. For Transit SAV, routers should be aware of the valid incoming interfaces for a given source prefix in order to establish a prefix-based interface allowlist. Packets with source addresses matching this source prefix are normally forwarded only when their incoming interfaces are in the allowlist. Otherwise, they should be processed according to predefined security policies, which may include actions like packet dropping or rate limiting. If the source address of a packet does not match any recorded source prefix, the packet is considered valid by default.

## Use Cases
For intra-domain networks, source address spoofing primarily occurs in the following scenarios: Firstly, a stub network within the domain might spoof the source addresses of other networks, which could be those within the same area, other areas within the domain, or even other ASes. Secondly, packets from other ASes spoofing the source addresses of a domain might enter this domain. This section uses  {{sav_usecase}} as an example to comprehensively demonstrate the typical scenarios of intra-domain source address spoofing and explain how to effectively implement intra-domain SAV using the approaches proposed in {{overview-sec}}.

~~~ aasvg
{::include art/sav_usecase.ascii-art}
~~~
{: #sav_usecase title="Intra-domain SAVNET Use Cases"}

{{sav_usecase}} shows a domain AS1 with two areas, namely the backbone area Area0 and non-backbone area Area1. ABR R6 connects Area0 and Area1, and ASBR R9 connects AS1 and AS2. First, let’s consider the scenarios where stub networks of R3 and R5 in Area1 spoof the source addresses of other networks. We will show how routers in Area1 can use source address verification to identify the spoofed traffic.

**Use Case 1: Edge SAV.** Suppose R3 knows its connected subnet's address range is 10.3.0.0/16. R3 can add the prefix 10.3.0.0/16 to the SAV whitelist at the interface connecting this subnet. Then packets originating from the subnet will only be forwarded by R3 if their source addresses belong to 10.3.0.0/16. Packets spoofing the source addresses of other networks will be prevented from entering the network at R3.

**Use Case 2: Transit SAV.** Assume R5 does not support SAV due to hardware/software capability limitations or performance considerations. Routers R1-R4 and R6 in Area1 can check whether the packets from R5 spoof the source address of another network by learning the valid incoming interfaces for packets from that network. Specifically, three types of spoofed source address packets might enter the network through R5:

1. Packets from R5 spoofing other network prefixes within the same area (e.g., 10.1.0.0/16 of R1). If R2, R3, R4 and R6 learn that the legitimate incoming interfaces for 10.1.0.0/16 are the interfaces corresponding to links R1-R2, R1-R3, R2-R4 and R4-R6 separately, then spoofed traffic from R5 entering R2, R3, and R6 through links R5-R3, R5-R2, and R5-R6 can be blocked by these transit routers.

2. Packets from R5 spoofing prefixes of Area0 (e.g., 10.8.0.0/16 of R8). Assume R4, R2 and R1 learn that the legitimate incoming interfaces for 10.8.0.0/16 are the interfaces corresponding to links R6-R4, R4-R2 and R2-R1 separately. Then spoofed traffic from R5 entering R4, R2 and R1 through paths R5-R3-R4, R5-R2, and R5-R3-R1 can be blocked.

3. Packets from R5 spoofing prefixes of AS2 (e.g., 20.0.0.0/8). Similar to the second scenario, if R4, R2 and R1 learn that the legitimate incoming interfaces for 20.0.0.0/8 are the interfaces corresponding to links R6-R4, R4-R2 and R2-R1 separately, then spoofed traffic from R5 entering R4, R2, and R1 through paths R5-R3-R4, R5-R2, R5-R3-R1 can be blocked.

**Use Case 3: Area Border SAV.** ABR R6 advertises Area0's routing prefixes to Area1 through Type 3 LSAs, including the prefix 10.8.0.0/16. R6 can establish a SAV blacklist for these prefixes at its interfaces connecting to Area1. If packets received at these interfaces have source addresses that hit the SAV blacklist, such as packets from R5 to R6 spoofing 10.8.0.0/16, they can be blocked by R6.

Next, we look at how ASBR R9 identifies spoofed traffic from AS2 spoofing AS1's source addresses when it reaches R9.

**Use Case 4: AS Border SAV.** R9 can obtain AS1's routing prefixes through Type 1 and Type 3 LSAs and establish a SAV blacklist at its interfaces connecting to AS2. If packets received at these interfaces have source addresses that hit the SAV blacklist, such as packets from R20 to R9 spoofing 10.1.0.0/16, they can be blocked by R9.

In summary, an intra-domain network can customize its SAV solutions based on specific network environments and security requirements. It can flexibly activate various SAV modes on different routers, such as Edge SAV only, a combination of Edge SAV, Area Border SAV, and AS Border SAV, or Edge SAV and Transit SAV. This ensures that the solution is not only effective but also tailored to the actual circumstances of the network.

# Solution to Transit SAV
For Transit SAV, a router must be aware of all the valid incoming interfaces for a source prefix to avoid improper blocking of packets with legitimate source addresses. In OSPF networks, packets are typically forwarded along the paths specified by the Shortest Path Tree (SPT). However, if a router has enabled Policy-Based Routing (PBR), the packets will be preferentially forwarded to the next hop as designated by the PBR rules. Thus, an effective way to implement Transit SAV is for the router owning that source prefix to proactively send SAV messages to other routers. The messages should inform the routers about which interfaces packets matching that source prefix will reach along the actual forwarding paths, including those determined by the routing protocol and those specified by PBR. To accomplish this, a router can generate a SAV message for a protected source prefix, specifying the neighboring router to receive the message and the destination routers to which the message will be ultimately advertised. The SAV message propagates along the actual forwarding path of the source prefix. Routers receiving a SAV message through an interface recognize that interface as valid for the source prefix conveyed in the message. Based on the received SAV messages, routers can establish a prefix-based interface allowlist (see {{sav-table}}) for data-plane SAV filtering.

A SAV message should at least include the following fields:

{:vspace}
Message Type (MT)
: Distinguishes SAV messages based on the kind of routing information used to generate the message. 'S' means the message is generated based on SPT. 'P' means the message is generated based on PBR rules.

Source Router ID (SR)
: The identifier of the router that generated the message.

Source Prefix (SP)
: The source prefix that the message seeks to protect.

Neighbor Router ID (NR)
: The identifier of the neighbor router to which the message is sent.

Destination Router ID (DR)
: The destination router's identifier, to which the message is ultimately advertised.

Destination Prefix (DP)
: The destination prefix to which the message is ultimately advertised.

{{spt-sec}} and {{pbr-sec}} respectively elaborate on the generation and propagation of SAV messages in scenarios only involving conventional routing and those with PBR enabled. The format of each field in a SAV message is specific to the OSPF version used, which will be explained in {{ospf-extension-sec}}.

Transit SAV is applicable to both single-area and multi-area scenarios within a domain. In multi-area scenarios, the SAV messages generated by routers are propagated only within their respective areas.

## SAV Message Propagation based on SPT {#spt-sec}

This section describes how intra-domain routers generate and propagate SAV messages based on the local Shortest Path Tree (SPT) and establish SAV rules. {{intra_topo}} shows an intra-domain network running the OSPF protocol. The SPT of Router R1 is shown in {{intra_spt}}. Packets originating from R1, with a source address within the subnet 10.1.0.0/16, are forwarded along this SPT path. The SAV message generated by R1 should be propagated along the SPT to all leaf node routers, carrying the prefix 10.1.0.0/16. All routers along the path can then receive the message and establish local SAV rules for this prefix. Similarly, ABRs and ASBRs can send SAV messages along the SPT to other routers within the area. These messages carry prefixes from other areas or ASes, enabling the routers to establish SAV rules for these prefixes.

Specifically, the SPT of R1 includes leaf nodes R3, R5, and R6, with R3 directly connected to R1, while R5 and R6 are in the subtree of neighbor R2. R1 needs to generate SAV messages for neighbor R2 and R3. The MT field is set to 'S' (SPT-based), SR is R1, and SP is 10.1.0.0/16. The message to R2 includes NR as R2 and DR as R5 and R6. The message to R3 includes both NR and DR as R3. Upon receiving R1’s SAV message from link R1-R2, R2 establishes the SAV rule <10.1.0.0/16, int.2.1>, indicating that interface 2.1 is the valid incoming interface for the prefix 10.1.0.0/16. Similarly, R3 establishes the SAV rule <10.1.0.0/16, int.3.1>, indicating that interface 3.1 is the valid incoming interface for the prefix. Since the message received by R3 indicates DR as R3 itself, it does not need to propagate the message further. R2, however, needs to propagate the message to R5 and R6 according to its local SPT. R5 is directly connected to R2, while R6 is in the subtree of neighbor R4. Therefore, R2 generates messages for neighbor R4 and R5, keeping the MT, SR, and SP fields unchanged. R2’s message to R4 includes NR as R4 and DR as R6. R2’s message to R5 includes NR as R5 and DR as R5. Upon receiving the message from R2, R4 and R5 establish local SAV rules <10.1.0.0/16, int.4.1> and <10.1.0.0/16, int.5.2>, respectively. Similarly, R4 continues to propagate the message to R6 based on the local SPT, and R6 establishes the SAV rule <10.1.0.0/16, int.6.1>. Thus, all routers on the SPT of R1 can learn the valid incoming interfaces for R1’s source prefix. If a packet with a source address within the prefix 10.1.0.0/16 is received on a invalid interface, such as R5 receiving a packet at interface 5.1, predefined security measures like packet dropping, rate limiting, or traffic monitoring may be taken.

~~~~~~~~~~
                             int.4.1
                    +----+        +----+
             int.2.1| R2 +--------+ R4 |
                   /+----+\      /+--+-+\
                  /        \    /        \
                 /          \  /          \int.6.1
          +----+/            \/            \+----+
          | R1 |             /\             | R6 |
         /+----+\           /  \           /+----+
  ------/        \         /    \int.5.2  /int.6.2
 (subnet)         \ +----+/      \+--+-+ /
  ------           \| R3 +--------+ R5 |/
10.1.0.0/16         +----+        +----+
              int.3.1        int.5.1
~~~~~~~~~~
{: #intra_topo title="An example of intra-domain network."}

~~~~~~~~~~
          R2 SAV rules                  R4 SAV rules
          <10.1.0.0/16,int.2.1>         <10.1.0.0/16,int.4.1>
                    +----+        +----+
                    | R2 +-------*| R4 |
                   *+----+\       +----+\
                  /        \             \
                 /          \             \
          +----+/            \             *+----+
          | R1 |              \             | R6 |
         /+----+\              \            +----+
  ------/        \              \           R6 SAV rules
 (subnet)         \ +----+       *+----+    <10.1.0.0/16,int.6.1>
  ------           *| R3 |        | R5 |
10.1.0.0/16         +----+        +----+
             R3 SAV rules            R5 SAV rules
             <10.1.0.0/16,int.3.1>   <10.1.0.0/16,int.5.2>

* the interface that receives SAV message of 10.1.0.0/16 from R1
~~~~~~~~~~
{: #intra_spt title="SAV message propagation along the SPT of R1."}

SAV messages are propagated within an area. SAV messages generated by different routers carry different source prefix information based on their role.

If a router is connected to a stub network, it should generate SAV messages to protect the prefixes of this stub network from being spoofed. To reduce communication overhead, the router can send a SAV message with the SP field set to a specific value. Routers receiving a message with SP set to this specific value use the prefixes of SR's stub network stored in the LSDB as source prefixes to be validated. They are included in the Router-LSA advertised by SR with link type "stub".

If a router is an ABR or ASBR, it can also generate SAV messages to protect the prefixes outside the area from being spoofed. For an ABR, the source prefix announced in a SAV message can be prefixes contained in summary-LSAs advertised within the area and prefixes contained in AS-external-LSAs received from other areas. For an ASBR, the source prefix announced in a SAV message can be prefixes contained in AS-external-LSAs advertised within the area. To reduce communication overhead, an ABR or ASBR can send a SAV message with the SP field set to a specific value. Routers receiving a message with SP set to a specific value use prefixes included in the summary-LSAs advertised by this ABR or prefixes included in the AS-external-LSAs advertised by this ASBR as source prefixes to be validated.

## SAV Message Propagation considering Policy-based Routing {#pbr-sec}
Policy-based routing (PBR) is a widely supported feature on modern routers. It enables network administrators to make routing decisions based on policies that reflect specific network requirements, rather than solely depending on routing tables generated by traditional routing protocols. This capability allows for finer control over traffic flows, facilitating decision-making based on criteria such as source and destination addresses, specific types of traffic, or even application data. By establishing PBR rules, administrators can direct different types of traffic along designated paths, thus addressing specific security or Quality of Service (QoS) needs effectively.

In networks where routers are configured with PBR, it is crucial to propagate SAV messages not only along the paths determined by the SPT but also along paths specified by PBR rules. This ensures that all interfaces a source prefix might reach according to both traditional routing protocols and PBR are considered legitimate entry points, thereby minimizing the risk of improper blocking of valid traffic.

Take {{intra_topo}} as an example. Assume Router R1 has a local PBR rule defined as <10.1.1.0/24, 10.5.0.0/16, R3>. This rule redirects traffic from the source IP addresses within 10.1.1.0/24 heading towards the stub network connected to R5, to go via R3 instead. R1 should generate a SAV message based on this rule and propagate this message along link R1-R3 and R3-R5. This action designates interfaces int.3.1 at R3 and int.5.1 at R5 as vaid incoming interfaces for the prefix 10.1.1.0/24.

Similarly, assume Router R2 has a PBR rule which redirects all traffic on port 80 going to the subnet 10.6.0.0/16 of R6 towards R5. Besides forwarding the SAV message originated from R1 to R6 along the SPT, R2 should also send the SAV message through R5 to reach R6 base on this PBR rule. This ensures that int.5.2 at R5 and int.6.2 at R6 are recognized as valid incoming interfaces for the source prefixes of R1.

### SAV Message Generation
When a router, such as Router R1, is configured with PBR rules that direct specific types of traffic to a neighbor (e.g., R2), it is essential for R1 to generate SAV messages based on these PBR rules and send them to R2. This enables R2 to permit the corresponding traffic through and to continue propagating the SAV information to other relevant routers within the domain. The fields for SAV messages generated based on PBR rules are set as follows:

- **MT**: Set to 'P' (PBR-based).
- **SR**: R1.
- **NR**: R2.
- **SP**, **DR**, and **DP**: Depend on the type of the PBR rule. Given that SAV messages are primarily concerned with sending which source prefixes to which destinations, PBR rules are divided into four categories based on the specification of source and destination prefixes. Other parameters in the rules, such as port numbers, are represented as 'other' and are not considered in the generation of SAV messages.

  1. **<srcIP, *, other, nexthop>**: This implies that packets with a source address belonging to srcIP may reach any router in the domain through nexthop R2. Thus, SP is set to srcIP, while DR and DP are not specifically defined.

  2. **<*, dstIP, other, nexthop>**: This implies that any packet from R1 might reach the router associated with dstIP through nexthop R2. Hence, SP is set to a default value representing all source prefixes of R1, DP is set to dstIP, and DR is not specifically defined.

  3. **<srcIP, dstIP, other, nexthop>**: This implies that packets with a source address belonging to srcIP may reach the router associated with dstIP through nexthop R2. Therefore, SP is set to srcIP, DP is set to dstIP, and DR is not specifically defined.

  4. **<*, *, other, nexthop>**: This implies that any packet from R1 may reach any router in the domain through nexthop R2. Thus, SP is set to a default value representing all source prefixes of R1, while DR and DP are not specifically defined.

  For cases where DR and DP are not specifically defined, nexthop R2 determines the endpoints of the SAV message propagation based on local routing policies (see {{pbr-forwarding-sec}}).

### SAV Message Forwarding {#pbr-forwarding-sec}
When Router R2, configured with PBR rules, receives a SAV message from Router R1, it must forward this message along both the SPT paths and the paths specified by its PBR rules. R2 uses the DR and DP fields to determine whether it is the endpoint of the message propagation. If not, and the intended neighbor for message forwarding is in the same area, the message is forwarded accordingly. SR and SP remain unchanged, while NR is updated to the intended neighbor. Other fields are handled as follows:

**MT:**

- If the received message’s MT is 'P' (PBR-based), it remains unchanged during forwarding.
- If the received message’s MT is 'S' (SPT-based), and the message is forwarded based on local PBR rules, MT is changed to 'P'.

**DR & DP:**

- If the received message specifies DP as dstIP:
  1. Forward the message to the next-hop of dstIP in the Forwarding Information Base (FIB), keeping DR and DP unchanged.
  2. If there is a local PBR rule of type `<*, dstIP, other, nexthop>` or `<*, *, other, nexthop>`, forward the message to the nexthops in these rules, keeping DR and DP unchanged.

- If the received message specifies DR:
  - **Case 1:** If the received message type is 'S':
    1. Forward the message based on local SPT(see {{spt-sec}}).
    2. If there is a local PBR rule `<*, dstIP, other, nexthop>` and dstIP belongs to the nodes along the SPT path to DR, forward the message to nexthop, updating DP to dstIP, leaving DR unspecified.
    3. If there is a local PBR rule `<*, *, other, nexthop>`, forward the message to nexthop, updating DR to include all Router IDs of the nodes along the SPT path to DR, leaving DP unspecified.
  - **Case 2:** If the received message type is 'P':
    1. Forward the message based on local SPT(see {{spt-sec}}).
    2. If there is a local PBR rule `<*, dstIP, other, nexthop>` and dstIP belongs to DR, forward the message to nexthop, updating DP to dstIP, leaving DR unspecified.
    3. If there is a local PBR rule `<*, *, other, nexthop>`, forward the message to nexthop, keeping DR and DP unchanged.

- If the received message does not specify DR and DP:
  1. Forward the message to all the other neighbors, updating DR to include all reachable nodes through these neighbors on SPT, leaving DP unspecified.
  2. If there is a local PBR rule `<*, dstIP, other, nexthop>`, forward the message to nexthop, setting DP to dstIP, leaving DR unspecified.
  3. If there is a local PBR rule `<*, *, other, nexthop>`, forward the message to nexthop, keeping DR and DP unchanged.

The primary difference between SPT-based and PBR-based SAV messages lies in the content of the DR field. SPT-based messages only record SPT leaf nodes since SAV messages are sent along the SPT path to these leaf nodes, and intermediate nodes on this path do not need to be explicitly listed to receive the message. This approach reduces message length and communication overhead. In contrast, PBR-based messages must record all nodes reachable by the message to ensure that it reaches all intended destinations.

It is important to note that the description provided above outlines only the basic logic for the coummunication of SAV messages. In practice, to optimize communication, messages that share the same MT and SP fields and are directed to the same NR can be consolidated before being sent.

# Protocol Extension {#ospf-extension-sec}

## Extensions to OSPFv2 for SAV Message Propagation
This document proposes extensions to OSPFv2 to support the dissemination of SAV messages via the OSPF protocol. The extension utilizes OSPFv2 Extended Prefix Opaque LSA as defined in {{RFC7684}}. The Advertising Router field is utilized to represent the SR within a SAV message, which denotes the origin of the SAV message. The LS Type is 10, which is scoped to an area-local visibility.

The OSPFv2 Extended Prefix Opaque LSA may contain one or more Extended Prefix TLVs. Each TLV is used to carry specific source prefix (SP) information pertinent to SAV. The Address Prefix field specifies the IP address of the source prefix. The Prefix Length field indicates the length of the source prefix.

A new sub-TLV, termed the "Source Prefix Validation Sub-TLV," is introduced within the Extended Prefix TLV. This sub-TLV is designed to encapsulate additional fields required for SAV, excluding SR and SP, which are already detailed above. It has the following format:

~~~~~~~~~~
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |           Type (TBD)          |            Length             |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |         Message Type          |           Reserved            |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                     Neighbor Router (NR)                      |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |            DR Count            |         DP Count             |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                   Destination Router (DR) 1                   |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                            ......                             |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                   Destination Router (DR) n                   |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  DP 1 Length  |      Destination Prefix (DP) 1 (variable)     |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                            ......                             |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  DP n Length  |      Destination Prefix (DP) n (variable)     |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~
{: #sav_ospfv2 title="OSPFv2 Source Prefix Validation Sub-TLV"}

{:vspace}
Type
: The type of the Sub-TLV. Value TBD.

Length
: The total length of the Sub-TLV in bytes, excluding the Type and Length fields.

Message Type
: 0 - the SAV message is based on SPT
: 1 - the SAV message is based on PBR

Reserved
: SHOULD be set to 0 on transmission and MUST be ignored on reception.

NR
:   The Neighbor Router ID to which the SAV message is sent.

DR Count
:  The number of Destination Router IDs included in this Sub-TLV.

DP Count
: The number of Destination Prefixes included in this Sub-TLV.

DR
: Each DR field represents a Destination Router ID to which the SAV message is ultimately advertised. The number of DR fields corresponds to the value in the DR Count field.

DP Length
: Length of a Destination Prefix in bits.

DP
: Each DP field represents a Destination Prefix to which the message is ultimately advertised. The number of DP fields corresponds to the value in the DP Count field.

An OSPFv2 Extended Prefix TLV can contain multiple OSPFv2 Source Prefix Validation Sub-TLVs, each of which can specify only one NR. Routers that receive this TLV process only the Sub-TLV where the NR field matches their own Router ID and ignore others.

## Extensions to OSPFv3 for SAV Message Propagation
In networks utilizing the OSPFv3 protocol, SAV messages are advertised through OSPFv3 Extended LSAs, including E-Intra-Area-Prefix-LSA, E-Inter-Area-Prefix-LSA and E-AS-External-LSA {{RFC8362}}. The Advertising Router field is used to represent the SR within a SAV message. The Address Prefix and PrefixLength fields of the Intra-Area Prefix TLV, Inter-Area Prefix TLV, and External-Prefix TLV are used to represent a specific SP. Additionally, A new OSPFv3 Source Prefix Validation sub-TLV is defined to be included in these TLVs to carry SAV message fields excluding SR and SP. The format of this new sub-TLV is as follows:

~~~~~~~~~~
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |           Type (TBD)          |            Length             |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |         Message Type          |           Reserved            |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                     Neighbor Router (NR)                      |
 |                             ...                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |            DR Count            |         DP Count             |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                   Destination Router (DR) 1                   |
 |                             ...                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                            ......                             |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                   Destination Router (DR) n                   |
 |                             ...                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  DP 1 Length  |      Destination Prefix (DP) 1 (variable)     |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                            ......                             |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  DP n Length  |      Destination Prefix (DP) n (variable)     |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~
{: #sav_ospfv3 title="OSPFv3 Source Prefix Validation Sub-TLV"}

{:vspace}
Type
: The type of the Sub-TLV. Value TBD.

Length
: The total length of the Sub-TLV in bytes, excluding the Type and Length fields.

Message Type
: 0 - the SAV message is based on SPT
: 1 - the SAV message is based on PBR

Reserved
: SHOULD be set to 0 on transmission and MUST be ignored on reception.

NR
: The Neighbor Router ID to which the SAV message is sent.

DR Count
: The number of Destination Router IDs included in this Sub-TLV.

DP Count
: The number of Destination Prefixes included in this Sub-TLV.

DR
: Each DR field represents a Destination Router ID to which the SAV message is ultimately advertised. The number of DR fields corresponds to the value in the DR Count field.

DP Length
: Length of a Destination Prefix in bits.

DP
: Each DP field represents a Destination Prefix to which the message is ultimately advertised. The number of DP fields corresponds to the value in the DP Count field.

An Intra-Area Prefix TLV, Inter-Area Prefix TLV, or External-Prefix TLV can contain multiple OSPFv3 Source Prefix Validation Sub-TLVs, each specifying only one NR. Routers receiving the TLV only process the Sub-TLV where the NR field matches their own Router ID and ignore others.

# Automatic Update Considerations
When network changes occur, routers need to promptly update their SAV rules to ensure the accuracy of source address validation. For Edge SAV, Area Border SAV, and AS Border SAV, the SAV rules derived from the LSDB can be updated almost in real-time as the LSDB is refreshed. For Transit SAV, propagating SAV messages via OSPF flooding ensures that all routers in the network can timely receive SAV message updates and refresh their SAV rules correspondingly.

For SAV messages generated based on the SPT, when a router's source prefix to be protected changes or SPT changes, the router should generate a new SAV message to inform other routers in the network to update their SAV rules. A new SAV rule about prefix p based on this message should replace the existing SAV rule about prefix p that was generated based on an earlier SPT-type SAV message.

For SAV messages generated based on PBR rules, if a router adds a new PBR rule which redirects traffic with a specified source prefix to a next-hop N, it should send a SAV message about this prefix to N. If a router adds a new PBR rule which redirects traffic without a specified source prefix to a next-hop N, it should send a SAV message about its own prefixes to N, and also forward the SAV messages it received from other routers to N based on this rule. Then N will further forward the received messages based on the DR and DP fields accordingly, allowing other routers in the network to timely learn the new forwarding paths added by PBR.

It is recommended to set an age field for SAV rules built on PBR-type SAV messages. The age refers to the amount of time that has elapsed since the SAV rule was originally generated and gradually increases over time. Periodic sending of SAV messages based on a certain PBR rule refreshes the corresponding SAV rules and reset the age field to 0. If a PBR-type SAV rule is not refreshed before it reaches Max Age, it is considered expired and should be removed from the SAV table.

# Incremental/Partial Deployment Considerations
In terms of deployment strategy, it is recommended to deploy Edge SAV and AS Border SAV first to block spoofed traffic at the edges of a domain. If Edge SAV cannot be fully deployed within the domain, Area Border SAV is recommended to be deployed in multi-area scenarios, allowing ABRs to prevent inter-area source address spoofing. Transit SAV is recommended to be deployed in each area to validate spoofed source address packets during their forwarding process. The deployment strategy can be adjusted based on the actual performance and specific needs of each area. Start by deploying in critical areas ensures that the most sensitive or vulnerable parts of the network gain protection early in the deployment process.

In scenarios where Transit SAV is partially deployed within an area, it is suggested to extend the Router Information (RI) LSA {{RFC7770}} to exchange information about each router’s support for Transit SAV. Routers that support Transit SAV can proactively advertise SAV messages to neighbors that also support Transit SAV, to protect their own source prefixes from being spoofed. Routers receiving SAVNET messages can establish SAV rules to block packets coming from unauthorized interfaces. The source prefixes can be better protected with more routers within the domain supporting Transit SAV.

In scenarios where Transit SAV is partially deployed within an area, if routers that do not support Transit SAV have enabled PBR, this may prevent other routers from getting all the valid incoming interfaces for a given source prefix. In such cases, a flexible traffic handling strategy is recommended. Packets coming from unauthorized interfaces are dropped or rate limited only when the traffic is clearly abnormal.

# Security Considerations
This document does not introduce any new security considerations beyond those inherent in the existing OSPF protocol implementations. The security features already available in OSPF, such as authentication mechanisms, can be leveraged to ensure the secure transmission of SAV messages. Utilizing OSPF’s built-in capabilities for securing routing information ensures that the integrity and authenticity of SAV messages are maintained across the network.

# IANA Considerations

## OSPFv2
Under "OSPFv2 Extended Prefix TLV Sub-TLVs registry" as defined in [RFC7684], IANA is requested to assign a registry value for OSPFv2 Source Prefix Validation Sub-TLV as follows:

~~~~~~~~~~
+===========+==========================+==================+
|   Value   |        Description       |     Reference    |
+===========+==========================+==================+
|   TBD1    | Source Prefix Validation |   This document  |
+-----------+--------------------------+------------------+
~~~~~~~~~~

## OSPFv3
Under "OSPFv3 Extended-LSA Sub-TLVs registry" as defined in [RFC8362], IANA is requested to assign a registry value for OSPFv3 Source Prefix Validation Sub-TLV as follows:

~~~~~~~~~~
+===========+==========================+==================+
|   Value   |        Description       |     Reference    |
+===========+==========================+==================+
|   TBD2    | Source Prefix Validation |   This document  |
+-----------+--------------------------+------------------+
~~~~~~~~~~


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
