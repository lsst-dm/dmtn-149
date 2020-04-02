:tocdepth: 1
.. sectnum::

Abstract
========

.. note::
   **This technote is not yet published.**

This document describes a program which simulates Rubin's Alert Distribution
System :cite:`DMTN-093`. This simulator would be useful for development of
community brokers: it would provide a realistic target for the throughput and
data formats they would be expected to handle. In addition, it may be a useful
tool for testing changes to the configuration of Rubin's alert brokers in the
future.


High-level goals
================

A simulator will help groups who are developing community brokers which
republish Rubin's alert stream. It will provide a realistic benchmark for the
load they should expect to receive, and will act as a harness for them to run
tests.

The simulator should:

 - Provide a realistic stream of alerts, both in terms of volume and in content.
 - Be simple to install on realistic broker hardware
 - Be well-documented
 - Provide tools to make it clear whether a consumer is keeping up with the
   stream volume

Implementation
==============

The simulator will be provided as a set of Docker containers and a
``docker-compose`` file for coordinating those containers. Those containers will
be:

 - A Kafka container, which runs the broker
 - A Zookeeper container, which the broker connects to for backend state management
 - A monitoring container, which provides a web-based UI verify that the stream
   is being fully handled

.. note::
   More to come in this section.



Known Limitations
=================

The simulator will have to be an approximation of the production stream. Several
details of the implementation remain uncertain, and our estimates of the size of
the data stream might not be accurate. This section lists the known limitations
of the simulation. Of course, there may be more discrepancies between the
simulation and the eventual stream at launch; this is just what we are aware of
now.

Authentication
--------------

The production broker will require community brokers to authenticate to receive
the stream of data. We don't yet know exactly how that will work. Some of the
options like TLS-encrypting the TCP flow might cause the production stream to be
more costly to process than the simulation.

Compression
-----------

We may want to compress the stream of alerts. This could be done using gzip,
snappy, lz4, or some other algorithm. Compression would be a tradeoff; it would
impose additional CPU load on consumers of the stream, but lower the network
bandwidth requirements. It could have a complicated effect on memory pressure
which is hard to predict.

Making compression an optional feature of the simulation could help us received
feedback from community broker developers on the effects it has in practice.

Alert Format
------------

The Data Products Definition Document :cite:`LSE-163` sets some requirements on
the contents of alert packets, but the precise Avro schema we will use is still
in development. This could have an impact on the size of alert packets,
affecting the stream volume.

In addition, we have not committed on a particular mechanism for distributing
the alert format and making changes to it. One option is the Confluent Schema
Registry :cite:`confluent-schema-registry`; choosing this option may have
implications on the deserialization performance of consumers.

Alert Contents
--------------

Some community brokers plan to modify or filter the alert stream. We don't yet
have scientifically meaningful data to populate the alert stream, though. This
means that any filters may not be receiving a realistic workload.

Broker Configuration Details
----------------------------

Kafka comes with a large number of tuning and configuration details. It runs on
the :abbr:`JVM (Java Virtual Machine)`, which has yet more tuning knobs. These
could have a dramatic impact on the performance characteristics of the broker in
production. For example, garbage collection pauses could impact tail latency in
response to queries from consumers, which can have a dramatic effect on the
service's overall performance :cite:`tail-at-scale`; the production :abbr:`GC
(garbage collection)` tunings may have a dramatic impact, but we won't be
providing a fully-tuned broker at this time.

Broker Hardware
---------------

In production, the Rubin alert brokers will run on Rubin's hardware. We can't
provide that hardware to community broker developers. They will need to run the
simulator on hardware which is capable of producing the full stream without
running into bottlenecks. For example, if the simulator is run on an
underpowered laptop, it might not produce the stream at the full volume due to a
CPU bottleneck. We could provide tools to let the user know if this is
happening, or provide documented guidance on how to detect it.


Network
-------

In production, the Rubin alert brokers will deliver the alert stream over the
internet. This could result in dramatically different behavior. Packet loss and
retransmits can cause head-of-line blocking which may result in stampedes of
alerts, causing much higher observed data rates at the consumer end than at the
producer end of the stream. Networks are complex and have many failure modes
that will not be simulated with this tool.

.. .. rubric:: References

.. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
    :style: lsst_aa
