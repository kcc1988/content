---
layout: default
title: AllJoyn
permalink: /en-US/win10/AllJoynInfo.htm
lang: en-US
---

##AllJoyn

**What is AllJoyn?**

Driven by the AllSeen Alliance, AllJoyn is an open-source, proximity-based connectivity and services framework.  Specifically designed for the Internet of Things (IoT), it enables interoperability such that various devices can discover, connect, and communicate with each other directly, without the need for an intermediary server.  

**Why AllJoyn?**

The AllJoyn framework provides a common language interface that enables IoT devices to communicate and interact with each other, regardless of brand, platform, operating system or underlying transport technology. For developers, this translates to reduced time to market and lower cost while offering consumers a simple connectivity solution for all their devices.  

**AllJoyn Architecture**

The AllJoyn framework establishes a standard by which devices and apps can advertise and discover each other.  AllJoyn devices describe their capabilities via service interfaces on a virtual bus.  The AllJoyn Bus is composed of two types of nodes:

* *Routing Notes (RN)* - Also referred to as "Routers", they can talk to any node.
* *Leaf Nodes (LN)* - Also referred to as "Applications", they can talk to routing nodes or other leaf nodes via routing nodes.

![AllJoyn Routers & Apps]({{site.baseurl}}/images/AllJoyn/AllJoyn_Routers_Apps.png)

The below diagram shows the high-level software architecture of the AllJoyn framework:
 ![AllJoyn Architecture]({{site.baseurl}}/images/AllJoyn/AllJoyn_Architecture.png)

* *AllJoyn App Layer* - Defines the user experience
* *AllJoyn Service Frameworks* - Interoperable, cross-platform modules that define common interfaces between devices  
* *AllJoyn Core Libs* - Core libraries that interact with the AllJoyn Router and provide the ability to find and securely connect to devices  
* *AllJoyn Router* - Manages communications between devices and apps


The AllJoyn framework comes in 2 flavors:

* *Standard* - Primarily used for non-embedded devices with support for full set of core libraries
* *Thin* - Suitable for IoT devices that are resource constrained and requires an AllJoyn router in the network
 ![AllJoyn Frameworks]({{site.baseurl}}/images/AllJoyn/AllJoyn_Frameworks.png)

AllJoyn enables proximity based communication, allowing transport to occur over Ethernet, Wi-Fi, serial, and Power Line (PLC).  However, the AllJoyn framework is transport-agnostic, thus allowing for any future transport mechanisms to be added.  Additionally, bridge software can be created to link the AllJoyn framework to other systems like Zigbee, Z-wave, or the cloud.  See more details & samples below on the AllJoyn Device System Bridge contribution to the AllSeen Alliance from Microsoft
