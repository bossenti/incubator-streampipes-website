---
title: Modbus goes StreamPipes
author: Tim Bossenmaier
authorURL: -
AuthorImageURL: -
---
<img class="blog-image" style="..." src="/docs/blog/assets/2020-07-xx/modbus_streampipes.png">
**<div sytle="..."> x minutes to read</div>**
<br>


This blog post is about the latest supplement to the StreamPipes adapter family: the Modbus protocol.
In the subsequent lines you can read about some basics of the protocol and how you can use it in StreamPipes.

## Introduction
In recent years, IoT has developed from a vague concept into a valuable and applicable instrument that can be used in industry. 
Accordingly, for more and more companies it is not just about participating in a hype, but about the necessity to remain competitive.
<br>
But problems often arise already from gathering data on the edge level and its transfer to the cloud.
For sure, if you order a brand new machine, it will probably be IIoT-enabled, and that's presumably not going to be an issue.
However, there are many machines that are still in use, even though they have already passed their twenties or thirties.
These are especially used by small and medium-sized companies (the famous German Mittelstand).
Since the purchase of a new machine represents an enormous investment for these companies,
they usually try to use machines for a longer period of time.
Over time, they upgrade the machines step by step and adapt them to new needs (called *retrofiting*).
One approach here could be, for example, to install smart sensors in a machine and let them transfer data to the cloud.
<br>
Alternatively, IIoT can be brought directly to the PLCs. This can be achieved by using a converter
that extracts the sensor data from the PLC and makes them accessible by using special communication protocols.
One protocol supported by many of these converters is Modbus. Published in 1979 by Modicon (now Schneider Electric),
it has become one of the most popular communication protocols for electronic devices due to the fact its publicly available and royalty-free.
<br>
Hence, the integration of Modbus is an auspicious step for the use of StreamPipes in practice. 


## Basics of Modbus
In this chapter we will introduce you to Modbus, thus we the concept of how Modbus is designed,
the available object structure etc. If you are already familiar with Modbus, you can skip this chapter and 
directly continue with [using Modbus in StreamPipes](#using-modbus-in-streampipes).
<br>
###### General
Since its foundations in 1979 Modbus has become one of the most used protocols in industrial context,
which can be traced back to its simplicity.
To facilitate communication Modbus enables multiple devices connected in an Ethernet network or to the same cable
to swap messages. As wiring type, a serial interface can be used, Modbus supports RS232 and RS485, or Ethernet.
Consequently, Modbus can be classified as an application layer protocol in accordance with the [OSI model](https://en.wikipedia.org/wiki/OSI_model).
<br>
###### Architecture
Modbus uses a master-slave structure as architecture, that means we have central master which is connected to multiple slaves.
The master can send messages to the slaves on which they respond. There are two message types to be distinguished: 
First one is called broadcast, in this case the server sends a message to all slaves. Alternatively, the master sends a message to a specific slave.
A slave always responds to a message be it with the queried information or an error message in case the sent massage was not correct,
but not on broadcasts. Accordingly, broadcasts are not suitable for queries.
![](/docs/blog/assets/2020-07-xx/communication_types.gif)
<br>
###### Message structure
There exist a lot of different versions how Modbus is actually implemented with regard to the network design.
The most prominent are:
 - **Modbus RTU**: used for serial communication, compact binary data representation, only machine-readable
 - **Modbus ASCII**: used for serial communication, communication uses ASCII characters, human-readable
 - **Modbus TCP/IP**: used in TCP/IP networks, connecting on port `502`, allows other protocols on the network as well
 
Currently, StreamPipes supports only the usage of `ModbusTCP`.
<br>
As a communication protocol, the very structure of a message is crucial.
One message is referred to as *application data unit (APU)* and encompasses the *protocol data unit (PDU)*.
Whereas the PDU is independent of the underlying communication layers and therefore equal for all deifferent Modbus implementations, 
the ADU can require some additional fields dependent on the specific network or bus system.
<br>
![schematic representation of a Modbus message](/docs/blog/assets/2020-07-xx/message_structure.png)
The function code encodes the action to perform for the receiver and consists of one byte. 
Values from 1 to 127 are reserved for the specific functions. The range 128-255 is used for exception responses, 
which appear if the message is invalid or the receiver was unable to process it.

siehe weitere in Modbus Specification
Protokoll typen ascii, tcp-ip usw. TCP IP genauer, da hier implementiert

Datentypen


## Using Modbus in StreamPipes
only tcpip
with plc4x as driver

### Interesting Links
- [1] [About retrofitting [in German]](https://www.industry-of-things.de/keine-maschine-ist-zu-alt-fuers-retrofitting-a-776709/)
- [2] [Specification of Modbus](http://www.modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf)

##### Please Notice
This article and the corresponding work in StreamPipes were part of the seminar
*Data Science and Real Time Analytics* of the [FZI](https://www.fzi.de/en/home/) in summer term 2020. <br>
I really enjoyed the opportunity to work on a practical topic like this and contribute to this project.
Therefore, I would like to thank the team of StreamPipes (especially my supervisor [Philipp Zehnder](https://www.fzi.de/wir-ueber-uns/organisation/mitarbeiter/address/philipp-zehnder/)) for this chance. 