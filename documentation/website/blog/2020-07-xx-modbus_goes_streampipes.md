---
title: Modbus Goes StreamPipes
author: Tim Bossenmaier
authorURL: ---
authorImageURL: ---
---
<img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/modbus_streampipes.png" alt="Modbus on StreamPipes">
**<div style="float: left; padding-right: 40px;"> x minutes to read</div>**
<br>


This blog post is about the latest supplement to the StreamPipes adapter family: the Modbus protocol.<br>
In the subsequent lines you can read about some fundamental and technical aspects of the protocol and how you can use it in StreamPipes.
<!--truncate--><br>

## Introduction
In recent years, IoT has developed from a vague concept into a valuable and applicable instrument that can be used in industry.
Accordingly, for more and more companies it is not just about participating in a hype, but about the necessity to remain competitive.
Albeit, when attempting to implement IIoT applications, problems often arise already in the first steps: data gathering on the edge level and transferring into the cloud. <br>
For sure, if you order a brand new machine, it will probably be IIoT-enabled, and these issues are unlikely to occur.
In industry, however, there are many machines that are still in use, even though they have already passed their twenties or thirties.
These are especially used by small and medium-sized companies (the famous German Mittelstand).
Since the purchase of a new machine represents an enormous investment for these companies,
they usually try to use machines for a longer period of time. <br>
Over time, they upgrade the machines step by step and adapt them to new needs (called *retrofitting*).
One approach here could be, for example, to install smart sensors in a machine and let them transfer data to the cloud.
Alternatively, IIoT can be brought directly to the PLCs. This can be achieved by using a converter
that extracts the sensor data from the PLC and makes them accessible by using special communication protocols.
One protocol supported by many of these converters is Modbus. Published in 1979 by Modicon (now Schneider Electric),
it has become one of the most popular communication protocols for electronic devices due to the fact its publicly available and royalty-free.
Hence, the integration of Modbus is an auspicious step for the use of StreamPipes in practice. 
<br>
<br>

## Basics of Modbus
In this chapter we will introduce you to Modbus, i.e. we will talk the concept, how Modbus is set up,
the available object structure etc. If you are already familiar with Modbus, you can skip this chapter and 
directly continue with [using Modbus in StreamPipes](#using-modbus-in-streampipes).
<br>
###### General
Since its release in 1979, Modbus has become one of the most widely used protocols in the industrial context,
which can be traced back to its simplicity and high communication efficiency.
To facilitate communication, Modbus enables multiple devices connected in an Ethernet network or to the same cable
to swap messages.Beside Ethernet, a serial interface can be used as the wiring type, Modbus supports here RS232, RS422 and RS485.
Within the [OSI model](https://en.wikipedia.org/wiki/OSI_model) Modbus is classified as an application layer protocol.
<br>
###### Architecture
A typical architecture in which Modbus can be applied consists of multiple devices connected to a single bus and communicating with a central controller.
To administrate the network, e.g. control access to communication line,
Modbus uses a master-slave structure (hereinafter named client-server-architecture,
where client refers to slave and server to master) as the architecture.
A central server (controller) is connected to multiple clients (single devices).
The server always initiates communication by sending messages on the bus.
These messages can either be addressed to a single device or to all devices (a so-called broadcast). <br>
After receiving a dedicated message (no broadcast), a client answers by sending a response on the bus to the server. 
A response may comprise the information solicited by the server or an error message if the original message was invalid or transmitted incorrectly.<br>
The only way to start a communication is the server sending a request.
Neither can the clients send messages to each other, nor can they send data on their own.
<br>

![ ](/docs/blog/assets/2020-07-xx/communication_types.gif)
<br>
###### Message Structure
There exist a lot of different versions of how Modbus is actually implemented in terms of network design.
The most used are:
 - **Modbus RTU**: used for serial communication, compact binary data representation, only machine-readable
 - **Modbus ASCII**: used for serial communication, communication uses ASCII characters, human-readable
 - **Modbus TCP/IP**: used in TCP/IP networks, connecting on port `502`, allows other protocols on the network as well
 
Currently, StreamPipes supports only the usage of `ModbusTCP/IP`.
<br>

With a communication protocol the exact structure of a message is of great importance.
Modbus refers to a message as *application data unit (APU)* which encompasses the *protocol data unit (PDU)*.
While the PDU is independent of the underlying communication layers and therefore the same for all different Modbus implementations, 
the ADU may require some additional fields, depending on the specific network or bus system.
<br>

![schematic representation of a Modbus message](/docs/blog/assets/2020-07-xx/message_structure.png)
<br>
The function code encodes the action to be performed for the receiver and consists of one byte, e.g. `0x02` for *Read Discrete Inputs*.
The values from 1 to 127 are reserved for the specific functions, but only nineteen of them are actually assigned a corresponding function.
The range 128-255 is used for exception responses, which are necessary if the message is invalid or the recipient could not process it.
If the receiver of a message requires additional information to perform the specified action,
the sender specifies this in the data field. This can typically be the register address plus the number of fields.
For some function codes, the specified action does not require any additional information, therefore, the data field does not exist
(with length zero). <br>

![schematic representation of the message validation](/docs/blog/assets/2020-07-xx/message_check.png)
<br>
###### Modbus TCP
Up to here, this is common for all versions of Modbus. In the following, we will present you some details on the Modbus TCP protocol,
as this is used in StreamPipes.<br>
To stay compatible with its serial version, Modbus packets are embedded into TCP frames assigned to port `502`.
Such a TCP frame consists of the TCP components (IP and TCP headers), a Modbus-specific header and the actual Modbus message,
which coincides in length and structure with the serial one. Furthermore, several fields in the Modbus TCP header are
predominated by the serial protocol.
The implementation of client-server architecture (as described in the [architecture](#architecture)-section) with TCP
is possible with simple devices, since most of the functionality is provided by the server.
<br>
Please be aware that the entities in TCP are modeled the other way round than in Modbus.
A Modbus server (to be more clear: the master) is a *TCP client* and the devices are *TCP server*.
Accordingly, a device that is to provide Modbus communication must have the characteristics of a TCP server.
Another aspect to keep in mind, is that the TCP client/server model is more general than the simple model used by Modbus.
If you are interested in these kind of aspects, take a look on the official [Modbus TCP implementation guidelines](https://modbus.org/docs/Modbus_Messaging_Implementation_Guide_V1_0b.pdf).
<br>

If you transmit Modbus communication via TCP, you can benefit from many advantages. First, TCP resp. TCP/IP compatible
networks are extremely widespread and commonly used in all areas. This brings economic advantages as you can 
use the existing infrastructure. Second, TCP is very flexible and it is possible to use the network for more than
just communicating via Modbus.
As always in real life, the use of TCP as the communication layer also has downsides.
A very important aspect for the application in an industrial context are possible vulnerabilities assciated with TCP.
Also the increasing complexity should not be neglected.
<br>
###### Data Model
The data model of Modbus is very simple as it was originally designed for serial Modbus communication.
Nevertheless, it is common to all variants of Modbus.
Modbus distinguishes four different object types (commonly referred to as registers) that a client can provide:
<br>
![](/docs/blog/assets/2020-07-xx/object_types.png)
<br>
Usually, a specific field in the individual registers is accessible via a 16-bit address and 
Modbus does not require to keep the four address spaces separate, thus they can overlap.
Furthermore, a Modbus device can but is not required to support all types of registers and
can also offer fewer addresses than in a 16-bit space.

So for practical purpose, you can consider `coil` and `disrete input` as boolean values and 
`holding register` and `input register` as integers.
<br>

Up to here, we have already talked a lot about the theoretical aspects of Modbus, so let's now put this into practice.

## Using Modbus in StreamPipes
For the integration of Modbus into StreamPipes we use the driver of the awesome library [PLC4X](https://plc4x.apache.org/).
This allows us to provide easy use of Modbus in the form of a dedicated connector that can be found in the data marketplace of StreamPipes Connect.
As already mentioned above, the Modbus connector is currently limited to using Ethernet communication via TCP/IP.

In the following we will provide you a step-by-step example on how to use StreamPipes for analyzing Modbus data.


### Interesting Links
- [1] [About retrofitting [in German]](https://www.industry-of-things.de/keine-maschine-ist-zu-alt-fuers-retrofitting-a-776709/)
- [2] [Specification of Modbus](http://www.modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf)
- [3] [More on PLC4X](https://plc4x.apache.org/)

##### Please Notice
This article and the corresponding work in StreamPipes were part of the seminar
*Data Science and Real Time Analytics* of the [FZI](https://www.fzi.de/en/home/) in summer term 2020. <br>
I really enjoyed the opportunity to work on a practical topic like this and contribute to this project.
Therefore, I would like to thank the team of StreamPipes (especially my supervisor [Philipp Zehnder](https://www.fzi.de/wir-ueber-uns/organisation/mitarbeiter/address/philipp-zehnder/)) for this chance. 
