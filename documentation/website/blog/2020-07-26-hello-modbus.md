---
title: Hello, Modbus!
author: Tim Bossenmaier
authorURL: ---
authorImageURL: /docs/img/bossenmaier.png
---
<br>

<img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/modbus_streampipes.png" alt="Modbus on StreamPipes">
<br>

**<div style="float: left; padding-right: 40px;"> 8 minutes to read</div>**
<br>


This blog post is about the latest supplement to the StreamPipes adapter family: the Modbus protocol.<br>
In the subsequent lines you can read about some fundamental and technical aspects of the protocol and how you can use it in StreamPipes.
<!--truncate--><br>

## Introduction
In recent years, IoT has developed from a vague concept into a valuable and applicable tool that can be used in industry.
Accordingly, for more and more companies it is not just about participating in a hype, but about the necessity to remain competitive.
Albeit, when attempting to implement IIoT applications, problems often arise already in the first steps: data gathering on the edge level and transferring into the cloud. <br>
For sure, if you order a brand-new machine, it will probably be IIoT-enabled, and these issues are unlikely to occur.
In the industry, however, there are many machines that are still in use, even though they have already passed their twenties or thirties.
These are especially used by small and medium-sized companies (e.g. the famous German Mittelstand).
Since the purchase of a new machine represents an enormous investment for these companies,
they usually try to use machines until it is no longer possible. <br>
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

### General
Since its release in 1979, Modbus has become one of the most widely used protocols in the industrial context,
which can be traced back to its simplicity and high communication efficiency.
To facilitate communication, Modbus enables multiple devices connected in an Ethernet network or to the same cable
to swap messages. Beside Ethernet, a serial interface can be used as the wiring type, Modbus supports here RS232, RS422 and RS485.
Within the [OSI model](https://en.wikipedia.org/wiki/OSI_model) Modbus is classified as an application layer protocol.
<br>

### Architecture
A typical architecture in which Modbus can be applied consists of multiple devices connected to a single bus and communicating with a central controller.
To administrate the network, e.g. control access to communication line,
Modbus uses a [primary-replica structure](https://en.wikipedia.org/wiki/Master/slave_(technology)) as the architecture.
A central controller (primary) is connected to multiple devices (replica).
The primary always initiates communication by sending messages on the bus.
These messages can either be addressed to a single device or to all devices (a so-called broadcast). <br>
After receiving a dedicated message (no broadcast), a replica answers by sending a response on the bus to the primary. 
A response may comprise the information solicited by the primary or an error message if the original message was invalid or transmitted incorrectly.<br>
The only way to start communication is the primary sending a request.
Neither can the replica send messages to each other, nor can they send data on their own.
<br>

<div style="text-align: center">
<img style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/communication_types.gif" alt="communication types of Modbus">
</div>
<br>

### Message Structure
There exist a lot of different versions of how Modbus is actually implemented in terms of network design.
The most used are:
 - **Modbus RTU**: used for serial communication, compact binary data representation, only machine-readable
 - **Modbus ASCII**: used for serial communication, uses ASCII characters, human-readable
 - **Modbus TCP/IP**: used in TCP/IP networks, connecting on port `502`, allows to use other protocols on the network as well
 
Currently, StreamPipes supports only the usage of `ModbusTCP/IP`.
<br>

With a communication protocol the exact structure of a message is of great importance.
Modbus refers to a message as *application data unit (APU)* which encompasses the *protocol data unit (PDU)*.
While the PDU is independent of the underlying communication layers and therefore the same for all different Modbus implementations, 
the ADU may require some additional fields, depending on the specific network or bus system.

<figure style="text-align: center">
<img style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/message_structure.png" alt="message structure of Modbus"/>
<figcaption>Schematic representation of a Modbus message</figcaption>
</figure>
<br>

### Modbus TCP
Up to here, this is common for all versions of Modbus. In the following, we will present you some details on the Modbus TCP protocol,
as this is used in StreamPipes.<br>
To stay compatible with its serial version, Modbus packets are embedded into TCP frames assigned to port `502` (alternatively `5020` is used often as well).
The implementation of primary-replica architecture (as described in the [architecture](#architecture)-section) with TCP
is possible with simple devices, since most of the functionality is provided by the server.
<br>

Please be aware that the entities in TCP are modeled the other way round than in Modbus.
A Modbus primary is a *TCP client* and the replica is a *TCP server*.
Accordingly, a device that is to provide Modbus communication must have the characteristics of a TCP server.
Another aspect to keep in mind, is that the TCP client/server model is more general than the simple model used by Modbus.
If you are interested in this kind of aspects, take a look on the official [Modbus TCP implementation guidelines](https://modbus.org/docs/Modbus_Messaging_Implementation_Guide_V1_0b.pdf).
<br>

If you transmit Modbus communication via TCP, you can benefit from many advantages. First, TCP resp. TCP/IP compatible
networks are extremely widespread and commonly used in all areas. This brings economic advantages as you can 
use the existing infrastructure. Second, TCP is very flexible and it is possible to use the network for more than
just communicating via Modbus.
As always in real life, the use of TCP as the transportation layer also has downsides.
A very important aspect for the application in an industrial context is the possible vulnerabilities associated with TCP.
Also, the increasing complexity should not be spurned.
<br>

### Data Model
The data model of Modbus is very simple as it was originally designed for serial Modbus communication.
Nevertheless, it is common to all variants of Modbus.
Modbus distinguishes four different object types (commonly referred to as registers) that a replica can provide:
<br>
<div style="text-align: center">
<img style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/object_types.png" alt="object types of Modbus"/>
</div>

<br>

Thus, for practical purpose, you can use `coil` and `discrete input` for boolean values and 
`holding register` and `input register` for integers.
<br>

Up to here, we have already talked a lot about the theoretical aspects of Modbus, so let's now put this into practice.
<br>
<br>

## Using Modbus in StreamPipes
For the integration of Modbus into StreamPipes we use the driver of the awesome library [PLC4X](https://plc4x.apache.org/).
This allows us to provide easy use of Modbus in the form of a dedicated connector that can be found in the *data marketplace* of StreamPipes Connect.
As already mentioned above, the Modbus connector is currently limited to using Ethernet communication via TCP/IP.

In the following we will provide you with a step-by-step example on how to use StreamPipes for analyzing Modbus data.
To do so, we will guide you through a small use case where we read sensor data of a pump via Modbus into StreamPipes. 
These sensor data contain typical information about a pump (e.g. `mass flow`, `volume flow`, or `fluid temperature`). 
To simulate a real working system, the values change every second. <br>
In this article, we only show the steps in StreamPipes. If you would like to take a look on the Modbus replica implementation or the data, we provide it in a [GitHub repo](https://github.com/bossenti/modbus-usecase).


### Create an Modbus Adapter
The first step we need to do is to create an adapter for our Modbus device.
<div class="my-carousel">
<img class="blog-image" style="max-width:100%;" src="/docs/blog/assets/2020-07-xx/create_adapter_step_1.png" alt="create adapter">
<img class="blog-image" style="max-width:100%;" src="/docs/blog/assets/2020-07-xx/create_adapter_step_2.png" alt="create adapter">
<img class="blog-image" style="max-width:100%;" src="/docs/blog/assets/2020-07-xx/create_adapter_step_3.png" alt="create adapter">
<img class="blog-image" style="max-width:100%;" src="/docs/blog/assets/2020-07-xx/create_adapter_step_4.png" alt="create adapter">
<img class="blog-image" style="max-width:100%;" src="/docs/blog/assets/2020-07-xx/create_adapter_step_5.png" alt="create adapter">
<img class="blog-image" style="max-width:100%;" src="/docs/blog/assets/2020-07-xx/create_adapter_step_6.png" alt="create adapter">
</div>

Open the *StreamPipes Connect* by clicking it in the left toolbar and select the *PLC4X Modbus Adapter*.
A dialog opens where you need to configure the following parameters:
 - `PLC-Address`: IP-address under which the PLC can be reached
 - `PLC-Port`: The port to which the PLC listens. In most cases this is port `502` for Modbus, in some cases `5020`
 - `PLC-ID`: ID of the device in the Modbus network

So far the parameters are PLC-specific, now we will define the individual data streams. 
You must specify all three for each information you want to query.
 - `Node Address`: Address of the value in the specific register
 - `Runtime Name`: Name under which the data is to be referred later
 - `Object Type`: choose the appropriate Modbus object type (see more in the section about the [data model](#data-model))
 
To illustrate these (partly StreamPipes-specific) terms, watch the animation below:
 <div style="text-align: center">
 <img style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/detailed_view_device.gif" alt="explanation of different parameters on device">
 </div>
 <br>
 
 In the next window, you could define some data as dimensions, for example, but in our case we keep them as measurements.
 Then we can give the adapter a name and a short description.
 If everything is set properly so far, you will get a *successfully created* message in the next window.
 Now we are done with the configuration of the Modbus adapter and from now we can find it in the *All Adapters* section in *StreamPipes Connect*.
 
 <br> 
 What we have shown so far, is specifically for working with Modbus. Whereas, the following steps will be the classic StreamPipes workflow which is also applicable to many other scenarios you can imagine.
 We will also keep this very simple, if you want to get more information about the functionalities and features of StreamPipes, take the [user tour](https://streampipes.apache.org/docs/docs/user-guide-tour/).
 
 ### Create a Pipeline
 
 The next step is to build our pipeline in which the Modbus data will be processed.
 Therefore, go to the *Pipeline Editor* in the toolbar on the left side.
 Drag the `Modbus Pump` from the *Data Streams* section and drop it on the grid area below. <br>
 We want to receive a notification when the fluid temperature exceeds 10 degree. Therefore, we need to connect our data stream to a `numerical filter` from
 the *data processors* bar and configure it accordingly. Then we need the `notification` from *data sinks*. Once configured, we can connect it to the numerical filter.
 Additionally, we also link a `dashboard` from *data sinks* to our data stream to have a look on the actual values.
 Now save the pipeline, and we have set up our pipeline properly.
 
 <div class="my-carousel">
 <img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/create_pipeline_1.png"/>
 <img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/create_pipeline_2.png"/>
 <img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/create_pipeline_3.png"/>
 <img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/create_pipeline_4.png"/>
 <img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/create_pipeline_5.png"/>
 </div>
 
 To actually be able to work with our pipeline, we have to start it eventually. Open the *Pipeline* section in the toolbar on the left side and click the play button.
 <div style="text-align: center">
 <img style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/start_pipeline.png" alt="overview of pipelines"/>
 </div>
 
 ### Monitor the data stream
 The moment we start the pipeline, our configured detection tools are working.
 So from that moment on, you can observe notifications appear in the *notification* section in the toolbar on the left.
 There you can inspect the single notifications, as you can see below.
 <div style="text-align: center">
  <img style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/notifications.png" alt="notification window"/>
 </div>
 
 <br> 
 Finally, we can configure our dashboard. Go to the *Dashboard* section on the toolbar and create a new dashboard.
 Click *Edit* and add the fluid temperature as a line chart. Now we can continuously observe the incoming fluid temperature values.
 
  <div class="my-carousel">
  <img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/create_dashboard_1.png"/>
  <img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/create_dashboard_2.png"/>
  <img class="blog-image" style="max-width: 100%;" src="/docs/blog/assets/2020-07-xx/create_dashboard_3.png"/>
  </div>
  
That's our short tutorial to introduce you to our new Modbus Adapter. StreamPipes offers you a lot more tools for monitoring and analyzing your data streams.
Have fun exploring it!

<br>

#### Interesting Links
- [1] [About retrofitting [in German]](https://www.industry-of-things.de/keine-maschine-ist-zu-alt-fuers-retrofitting-a-776709/)
- [2] [Specification of Modbus](http://www.modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf)
- [3] [More on PLC4X](https://plc4x.apache.org/)

<br>
#### Please Notice
This article and the corresponding work in StreamPipes were part of the seminar
*Data Science and Real Time Analytics* of the [FZI](https://www.fzi.de/en/home/) in the summer term 2020. <br>
I really enjoyed the opportunity to work on a practical topic like this and contribute to this project.
Therefore, I would like to thank the team of StreamPipes (especially my supervisor [Philipp Zehnder](https://www.fzi.de/wir-ueber-uns/organisation/mitarbeiter/address/philipp-zehnder/)) for this chance. 
