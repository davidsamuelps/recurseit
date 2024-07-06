---
title: "Cisco SD-WAN Service Side NAT"
date: "2021-03-05"
categories: 
  - "ccnp"
tags: 
  - "data"
  - "nat"
  - "policy"
  - "prefix"
  - "routing"
  - "sd-wan"
  - "sdwan"
  - "service-side"
  - "static"
  - "static-nat"
  - "traffic"
  - "viptela"
---

Among Cisco SD-WAN's tools to steer or influence traffic across the overlay, the data policies are the ones providing the highest flexibility. For a primer about SD-WAN Data Policies, please refer to this article: [Cisco SD-WAN Data Policies](https://recurseit.com/2021/01/27/cisco-sd-wan-data-policies/).

One of the SD-WAN Data Policy applications is the Service Side NAT. Compared to traditional networking where NAT deployment is rather straightforward, Service Side NAT in SD-WAN is a corner case whose deployment logic can initially be elusive. This document will describe the SD-WAN Service Side NAT principles and deployment as a practical case study.

## Technical Debt

Among several terms and well known characteristics, technical debt is an often ignored condition that stays under the rug until it becomes unmanageable and causes an outage (that's when management's alarms go up). Brownfield scenarios where infrastructure evolved over time commonly come with an accrued technical debt. This debt can manifest itself in several ways, some of which are:

- Unused configuration
    - Old firewall rules nobody would remove due to risk
    - Unused VLANs from decommissioned servers/user ranges
    - Leftovers from previous implementations like ACLs and route maps
- Complex deployments
    - Several routing protocols performing redistribution
    - IP range migration or redeployment
    - Convoluted routing policies
    - Vendor specific features
- Customized deployments
    - Specific devices deployed without proper renewal/replacement
        - EoS and EoL devices
        - Lack of proper replacement in the industry
    - Specific type of devices deployed
        - Whiteboxes/Grayboxes

While addressing (or living with) technical debt, convoluted solutions are required to solve problems caused by it. One of them is to implement NAT, commonly to avoid re-addressing certain devices in the network (like routers, servers or firewalls).

## NAT'ing the Cisco way

A common scenario can be depicted as follows: A site design covering a specific set of subnets in a clearly defined range, and a device standing out from the rest. Too cumbersome to be fixed, instead, only kept in the network. Furthermore, this device should be monitored by a set of management stations.

A common way to approach this situation is through 1:1 NAT deployed in the routers at the edge.

![](https://recurseit.files.wordpress.com/2021/03/2021-03-05-14_54_24-service-side-nat.vsd-compatibility-mode-visio-standard.png?w=760)

Legacy site diagram

For exemplification purposes, the addresses to establish a NAT binding will be the following:

- **Inside IP address:** 10.36.65.242.
- **Outside IP address:** 10.104.118.14.
- **Sources:** 192.0.0.0/24 and 192.0.2.0/24.

The way to perform this in the Cisco IOS way is to create a NAT statement and limit its scope through a route-map and Access List combo:

```
ip nat inside source static 10.36.65.242 10.104.118.14 route-map RM_MANAGEMENT_RANGES <<< scope limiting route-map
```

What would the ACL and Route Map look like?

```
route-map RM_MANAGEMENT_RANGES permit 10
 match ip address ACL_MANAGEMENT_RANGES <<<< scope limiting route-map matching an ACL
!
ip access-list extended ACL_MANAGEMENT_RANGES <<<< ACL containing allowed sources
 permit ip any 192.0.0.0/24 0.0.0.255 <<<<< sources
 permit ip any 192.0.2.0/24 0.0.0.255 <<<<< sources
```

Lastly, inside and outside interfaces must be identified:

```
interface GigabitEthernet0/0/0.17 <<< WAN Interface
 ip nat outside
!
interface GigabitEthernet0/0/1.920 <<< LAN Interface
 ip nat inside
```

Reiterating: The configurations above were meant to **only** create a NAT binding between inside and outside IP addresses if the source of the traffic are the management subnets. What occurs when this site and its particular setup must be migrated to SD-WAN?

## Cisco SD-WAN Service Side NAT

An equivalent of the configuration above in the SD-WAN world would require paying attention to the following caveat: SD-WAN is a fantastic tool to distribute modular, repeatable and standardized configuration in a network-wide manner. In other words, it doesn't look at devices as special/particular units, it looks at them as sites. **A site (one or two routers) is the minimal entity to which you can apply a policy from a centralized point of view (vSmart).** Conversely, it means that highly customized configurations in particular devices tend to be weaker areas, requiring more work and considerations (often called gymnastics).

![](https://recurseit.files.wordpress.com/2021/03/2021-03-05-16_50_52-service-side-nat.vsd-compatibility-mode-visio-standard.png?w=830)

SD-WAN site diagram

Note the illustration above. Unnecessary information has been removed to focus on the only important elements from the SD-WAN perspective: IP addresses, directions of the flows, and VPNs.

The equivalent configuration/policy in SD-WAN for the 1:1 NAT Cisco lingo mentioned earlier includes the following elements (note that inbound direction applies to the vEdge's perspective) to cater for both directions of the NAT'ed flows:

- Clarity about service and transport sides of the equation.
- A site list containing **only** the site in question.
- A VPN list.
- A natpool interface with the IP address to use as outside address (10.104.118.14)  to handle **inbound WAN traffic (transport to service direction)**.
    - A specific NAT statement to create the binding (under the natpool inteface's configuration) between the inside and outside IP addresses.
- A customized data policy to steer **inbound LAN traffic (service to transport direction)**.

### **NAT Pool interface configuration**

**vEdge-config**

```
configure terminal
 vpn 1
  interface natpool1
   ip address 10.104.118.14/32 <<<< IP range to be used for outside IP addresses
   nat
    static source-ip 10.36.65.242 translate-ip 10.104.118.14 inside <<<< 1:1 mapping
    no overload
  !
   no shutdown
```

Caveats to consider in the configuration above:

- The natpool interface must sit in the service VPN you are NAT'ing the traffic from.
- The number of natpool interfaces is a range between 1 and 32 (natpool1 - natpool32).
- The natpool interface must match with the mappings configured in the NAT statement.
- There can be several NAT statements, as long as the number matches the number of IP addresses contained in the natpool range (I.e. /30 would allow 4 IP addresses to be used in the pool).
- **inside** keyword—Translate the source IP address of packets that are coming from the service side of the vEdge router and that are destined to transport side of the router. This is the default.
- **outside** keyword —Translate the source IP address of packets that are coming to the vEdge router from the transport side of the vEdge router and that are destined to a service-side device.
- **overload** keyword — By default, dynamic NAT is enabled — "no overload" must be specified for 1:1 NAT.

### Data Policy

The Data Policy will be required because the natpool interface is **not** the inbound LAN interface, it is a virtual entity created for NAT'ing purposes (NAT on a stick). Without the data policy, the **inbound LAN traffic** would only traverse the LAN interface, therefore failing to NAT in the outbound direction (towards WAN/transport).

The Data Policy will effectively steer the interesting/matching traffic into the natpool interface for the required destinations (management stations).

#### Data Policy - CLI format

```
from-vsmart data-policy _VL_VPN1_HQ_EMEA
 direction from-service <<< matching traffic coming from VPN 1 - service VPN
 vpn-list VL_VPN1 <<< VPN list
  sequence 1 <<< sequences matching source (firewall's inside IP address) and destinations of the traffic
   match
    source-ip      10.36.65.242/32
    destination-ip 192.0.0.0/24
   action accept
    nat pool 1 <<< Steering traffic towards natpool interface if matched
    log <<< optional
  sequence 11 <<< sequences matching source (firewall's inside IP address) and destinations of the traffic
   match
    source-ip      10.36.65.242/32
    destination-ip 192.0.2.0/24
   action accept
    nat pool 1 <<< Steering traffic towards natpool interface if matched 
    log <<< optional
  default-action accept <<<< The default action is "reject"
```

**NAT - Data Policy - CLI**

Caveats to consider in the configuration above:

- It is a highly customized and complex configuration - all the elements have to be created from scratch, and it has to be applied on two different places. On the vEdge(s) for inbound traffic, and the vSmart controllers for outbound traffic (towards WAN).
- When compared with Cisco's configuration, it is different and confusing for many, in orders of magnitude. Especially because it does not get applied only on one device.
- The default action has to be changed explicitly to "accept".
- If the Data Policy is not applied, traffic would get NAT'ed in one direction only.
- This scenario is a corner case.
- The documentation is unclear, scattered, scarce, or confusing - that was the main motivation of this article.

#### CLI Policy - vManage

![](https://recurseit.files.wordpress.com/2021/03/2021-03-05-18_18_54-2021-03-05-16_43_29-2-cisco-vmanage-e28094-mozilla-firefox-1.png?w=1024)

Data Policy configuration

![](https://recurseit.files.wordpress.com/2021/03/2021-03-05-18_26_34-2021-03-05-16_44_42-2-cisco-vmanage-e28094-mozilla-firefox.png?w=1024)

Modifying default action of the Data Policy

![](https://recurseit.files.wordpress.com/2021/03/2021-03-05-18_27_52-2021-03-05-16_58_06-2-cisco-vmanage-e28094-mozilla-firefox.png?w=1024)

Applying Data Policy

Hope it helps!

References containing information about Service Side NAT:

- [https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/vedge/policies-book/vEdge-as-NAT-device.html](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/vedge/policies-book/vEdge-as-NAT-device.html https://www.cisco.com/c/dam/en/us/td/docs/routers/sdwan/configuration/config-18-4.pdf#page=505 https://sdwan-docs.cisco.com/Product_Documentation/Software_Features/SD-WAN_Release_16.3/07Policy_Applications/04Using_a_vEdge_Router_as_a_NAT_Device/Service-Side_NAT_Configuration_Example https://www.cisco.com/c/en/us/support/docs/routers/sd-wan/215106-service-side-destination-based-network-a.html https://sdwan-docs.cisco.com/Product_Documentation/vManage_Help/Release_18.3/Configuration/Templates/VPN_Interface_NAT_Pool https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/service-side-nat-ios-xe.html)
- [https://www.cisco.com/c/dam/en/us/td/docs/routers/sdwan/configuration/config-18-4.pdf#page=505](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/vedge/policies-book/vEdge-as-NAT-device.html https://www.cisco.com/c/dam/en/us/td/docs/routers/sdwan/configuration/config-18-4.pdf#page=505 https://sdwan-docs.cisco.com/Product_Documentation/Software_Features/SD-WAN_Release_16.3/07Policy_Applications/04Using_a_vEdge_Router_as_a_NAT_Device/Service-Side_NAT_Configuration_Example https://www.cisco.com/c/en/us/support/docs/routers/sd-wan/215106-service-side-destination-based-network-a.html https://sdwan-docs.cisco.com/Product_Documentation/vManage_Help/Release_18.3/Configuration/Templates/VPN_Interface_NAT_Pool https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/service-side-nat-ios-xe.html)
- [https://sdwan-docs.cisco.com/Product\_Documentation/Software\_Features/SD-WAN\_Release\_16.3/07Policy\_Applications/04Using\_a\_vEdge\_Router\_as\_a\_NAT\_Device/Service-Side\_NAT\_Configuration\_Example](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/vedge/policies-book/vEdge-as-NAT-device.html https://www.cisco.com/c/dam/en/us/td/docs/routers/sdwan/configuration/config-18-4.pdf#page=505 https://sdwan-docs.cisco.com/Product_Documentation/Software_Features/SD-WAN_Release_16.3/07Policy_Applications/04Using_a_vEdge_Router_as_a_NAT_Device/Service-Side_NAT_Configuration_Example https://www.cisco.com/c/en/us/support/docs/routers/sd-wan/215106-service-side-destination-based-network-a.html https://sdwan-docs.cisco.com/Product_Documentation/vManage_Help/Release_18.3/Configuration/Templates/VPN_Interface_NAT_Pool https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/service-side-nat-ios-xe.html)
- [https://www.cisco.com/c/en/us/support/docs/routers/sd-wan/215106-service-side-destination-based-network-a.html](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/vedge/policies-book/vEdge-as-NAT-device.html https://www.cisco.com/c/dam/en/us/td/docs/routers/sdwan/configuration/config-18-4.pdf#page=505 https://sdwan-docs.cisco.com/Product_Documentation/Software_Features/SD-WAN_Release_16.3/07Policy_Applications/04Using_a_vEdge_Router_as_a_NAT_Device/Service-Side_NAT_Configuration_Example https://www.cisco.com/c/en/us/support/docs/routers/sd-wan/215106-service-side-destination-based-network-a.html https://sdwan-docs.cisco.com/Product_Documentation/vManage_Help/Release_18.3/Configuration/Templates/VPN_Interface_NAT_Pool https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/service-side-nat-ios-xe.html)
- [https://sdwan-docs.cisco.com/Product\_Documentation/vManage\_Help/Release\_18.3/Configuration/Templates/VPN\_Interface\_NAT\_Pool](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/vedge/policies-book/vEdge-as-NAT-device.html https://www.cisco.com/c/dam/en/us/td/docs/routers/sdwan/configuration/config-18-4.pdf#page=505 https://sdwan-docs.cisco.com/Product_Documentation/Software_Features/SD-WAN_Release_16.3/07Policy_Applications/04Using_a_vEdge_Router_as_a_NAT_Device/Service-Side_NAT_Configuration_Example https://www.cisco.com/c/en/us/support/docs/routers/sd-wan/215106-service-side-destination-based-network-a.html https://sdwan-docs.cisco.com/Product_Documentation/vManage_Help/Release_18.3/Configuration/Templates/VPN_Interface_NAT_Pool https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/service-side-nat-ios-xe.html)
- [https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/service-side-nat-ios-xe.html](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/vedge/policies-book/vEdge-as-NAT-device.html https://www.cisco.com/c/dam/en/us/td/docs/routers/sdwan/configuration/config-18-4.pdf#page=505 https://sdwan-docs.cisco.com/Product_Documentation/Software_Features/SD-WAN_Release_16.3/07Policy_Applications/04Using_a_vEdge_Router_as_a_NAT_Device/Service-Side_NAT_Configuration_Example https://www.cisco.com/c/en/us/support/docs/routers/sd-wan/215106-service-side-destination-based-network-a.html https://sdwan-docs.cisco.com/Product_Documentation/vManage_Help/Release_18.3/Configuration/Templates/VPN_Interface_NAT_Pool https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/service-side-nat-ios-xe.html)
