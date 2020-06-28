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


## Using Modbus in StreamPipes

### Interesting Links
- [1] [About retrofitting [in German]](https://www.industry-of-things.de/keine-maschine-ist-zu-alt-fuers-retrofitting-a-776709/)
- [2] [Specification of Modbus](http://www.modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf)

##### Please Notice
This article and the corresponding work in StreamPipes were part of the seminar
*Data Science and Real Time Analytics* of the [FZI](https://www.fzi.de/en/home/) in summer term 2020. <br>
I really enjoyed the opportunity to work on a practical topic like this and contribute to this project.
Therefore, I would like to thank the team of StreamPipes (especially my supervisor [Philipp Zehnder](https://www.fzi.de/wir-ueber-uns/organisation/mitarbeiter/address/philipp-zehnder/)) for this chance. 