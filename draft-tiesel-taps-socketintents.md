---
title: Socket Intents
abbrev: SocketIntents
docname: draft-tiesel-taps-socketintents-latest
date: 2017-06-15
category: exp

ipr: trust200902
area: General
workgroup: TAPS Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: P. S. Tiesel
    name: Philipp S. Tiesel
    organization: Berlin Institute of Technology
    street: Marchstr. 23
    city: Berlin
    country: Germany
    email: philipp@inet.tu-berlin.de
 -
    ins: T. Enghardt
    name: Theresa Enghardt
    organization: Berlin Institute of Technology
    street: Marchstr. 23
    city: Berlin
    country: Germany
    email: theresa@inet.tu-berlin.de

normative:
  RFC5226:
  RFC2119:

informative:
  RFC4594:
  RFC4960:
  RFC6824:
  RFC7413:
  RFC7556:
  I-D.pauly-taps-guidelines:
  I-D.trammell-taps-post-sockets:
  I-D.tiesel-taps-communitgrany:
    title: "Communication Units Granularity Considerations for using Transport Diversity or Multiple Provisioning Domains"
    date: 2017-07
    author:
      -
        name: Philipp S. Tiesel
        ins: P. S. Tiesel
      -
        name: Theresa Enghardt
        ins: T. Enghardt
    seriesinfo:
      draft-tiesel-taps-communitgrany-00 (work in progress)
  DASH:
      author:
        org: International Organization for Standardization
      title: "Dynamic adaptive streaming over HTTP (DASH) - Part 1: Media presentation description and segment formats"
      date: 2011-06
      target: https://www.iso.org/standard/65274.html
      seriesinfo:
        Standard ISO/IEC 23009-1:2014
  CoNEXT2013: DOI.10.1145/2535372.2535405

--- abstract

This document outlines an API-independent concept that allows
applications to share their knowledge about upcoming communication and
express their performance preferences in a portable and abstract way:
Socket Intents.
Socket Intents express what an application knows, assumes, expects or wants to
prioritize regarding its own network communication.
The information provided by Socket Intents should be taken into account
by the network stack in a best-effort way.

Socket Intent can be used to stem against the complexity and make
use of multiple provisioning domains as well as new transport protocols
and features available to a larger user base by expressing the
applications intents in an abstract and portable way.


--- middle

Conventions and Definitions
===========================

The words "MUST", "MUST NOT", "SHALL", "SHALL NOT", "SHOULD", and
"MAY" are used in this document. It's not shouting; when these
words are capitalized, they have a special meaning as defined
in {{RFC2119}}.

Flow, Association, Stream, or Object are used as defined in {{I-D.tiesel-taps-communitgrany}}:



Introduction        {#intro}
============

Despite recent advances in the transport area, the adaption of new
transport protocols and transport protocol features is slow and only
happens in limited domains (primarily in the Web browser and within
datacenters).
The same problem occurs for taking advantage of multiple available access
networks or provisioning domains (PvDs).
In both cases, the benefits of the new transport diversity comes at
the cost of an increased complexity that has to be mastered by the
application programmer.

Enabling features like TCP fast open {{RFC7413}} or controlling how
MPTCP {{RFC6824}} creates subflows requires specialized APIs
that are not part of the standard socket API, often require deep
knowledge of the transport protocol internals, and are not portable
across different implementations.

Applications that want to use multiple network interfaces usually have to use
their own heuristics to select which access network to use.
Choosing the right interface is difficult as their characteristics
differ, e.g. in terms of performance, and obtaining the necessary
information is often not easy since it may require special privileges
and differs heavily by implementation.

In all cases mentioned above, an application that wants to take
advantage of the available transport diversity is faced with
substantially higher complexity regarding network APIs and
networking code.



Problem Statement
=================

Application programmers opening a communication channel typically know
how this channel will be used. Beside the hard requirements already
necessary for establishing the communication channels, e.g., reliable
in-order stream transport, there is more information available: An
application developer has an intuition about optimization preferences,
e.g., optimize for bandwidth, latency, or cost, about expectations, e.g.
towards data loss, and possibly also about specifics, such as how many bytes
will be sent or received.

This information does not directly map to the choice of a transport
protocol, to certain protocol parameters, nor to which PvD to use, but the
information can imply that the application can benefit from certain
transport options or help to choose between multiple PvD as described
in {{RFC7556}}, Section 6.2, and therefore enable the OS to adjust its
defaults for this communication channel accordingly.

The preferences, expectations and other information known about the
upcoming communication MAY be expressible in an intuitive, abstract
way independent of the network- and transport protocol.
Its representation SHOULD be independent of the actual API used for
network communication, e.g., these SHOULD be expressible
in whatever API available, e.g., as "socketopts" for BSD sockets or
as part of the address resolution configuration for
[Post Sockets](#I-D.trammell-taps-post-sockets).
Finally, given the expectations and external constraints known, the OS
SHOULD use the information provided via Socket Intents in an best-effort
fashion and therefore try to choose the best transport protocol, default
parameters and PvDs available and MAY try to further optimize based on them.




General Concept {#concept}
===============

With Socket Intents, applications MAY express their communication
preferences in order to take advantage of the available transfer
diversity.
Depending on the API used, Socket Intents can be used on a per
Flow, Association, Stream, or Object level.
Communication preference refers to desired transport characteristics,
e.g., low delay or high throughput, stable transport or minimal cost,
and is optional information.


Socket Intent Types {#typespec}
-------------------

The following sections contain a list or Socket Intent types and
their possible values.

Socket Intents are structured as key / value pair. The key is expressed by a short name, the value has a fixed data type (Enum, Int or Float).

The namespace for the short names is portioned as follows:
- Experimental Socket Intent type MUST start with "x-".
- Private or vendor specific Socket Intent type MUST start with "y-\[vendor\]-".
- The remming Socket Intent type namespace SHOULD be managed by
  an IANA registry. The assignment of new types requires an RFC or expert review.

For Enum data types, a list of valid values MUST be provides by the document specifying that intent.

An implementation faced with unknown intent types or invalid or
unknown values MAY ignore that Intent but SHOULD return an error code to the application.


Interactions between Socket Intents and QoS
-------------------------------------------
Socket Intents are not QoS labels, but have an orthogonal meaning.

 - Socket Intents SHALL be purely advisory.
 - Socket Intents MUST NOT be used to derive IntServ / RSVP style guarantees.
 - Socket Intents SHOULD be taken into account on a best-effort basis and MAY be used to derive DiffServ Service Classes as described in {{RFC4594}}.



Initial Socket Intent Types {#types}
===========================

Note:
: Recommended default values for Enum types are marked with an asterisk (*) behind the level name.

Traffic Category
----------------

The Traffic Category describes the dominating traffic pattern of the
respective communication unit expected by the application.

Short name:
: category

Applicability:
: Flow, Association, Stream

Data type:
: Enum

| Level         | Description                                          |
|---------------|------------------------------------------------------|
| query         | Single request / response style workload, latency bound |
| control       | Long lasting low bandwidth control channel, not bandwidth bound |
| stream        | Stream of bytes/objects with steady data rate |
| bulk          | Bulk transfer of large objects, presumably bandwidth bound |
| mixed*        | Don't know or none of the above |

Note:
: Most categories suggest the use of other intents to further describe the traffic pattern anticipated, e.g., the bulk category suggesting the use of the Object Size intents or the stream category suggesting the
Stream Bitrate and Duration intents.


Object Size to be Sent / Received
---------------------------------

This Intent is used to communicate the expected size of a transfer.

Short name:
: sndobjsz / recvobjsz

Applicability:
: Flow, Association, Stream, Object

Data type:
: Int (bytes)



Duration
--------

This Intent is used to communicate the expected lifetime of the
respective communication unit.

Short name:
: duration

Applicability:
: Flow, Association, Stream

Data type:
: Int (msec)



Stream Bitrate Sent / Received
------------------------------

This Intent is used to communicate the bitrate of the respective
communication unit.

Short name:
: sndrate / recvrate

Applicability:
: Flow, Association, Stream

Data type:
: Int (bytes/sec)



Burstiness
-----------

This Intent describes the anticipated sender-side burst characteristics of the
traffic for this communication unit. It expresses how the traffic sent by the
application is expected to vary over time, and, consequently, how long
sequences of consecutively sent packets will be.
Note that the actual burst characteristics of the traffic at the receiver
side will depend on the network.

This Intent can provide hints to the application on what the resource usage
pattern for this communication unit will look like, which can be useful for
balancing the requirements of different application.


Short name:
: burst

Applicability:
: Association,  Connection, Stream

Data type:
: Enum



| Level         | Description                                          |
|---------------|------------------------------------------------------|
| no_bursts     | Application sends traffic at a constant rate |
| regular_bursts | Application sends bursts of traffic periodically |
| random_bursts | Application sends bursts of traffic irregularly |
| bulk          | Application sends a bulk of traffic |
| mixed*        | Don't know or none of the above |



Timeliness
-----------

This Intent describes the desired delay characteristics for this communication unit.
It provides hints for the OS whether to optimize for low delay or for other criteria.
There are no hard requirements or implied guarantees on whether these requirements can
actually be satisfied.


Short name:
: timeliness

Applicability:
: Association,  Connection, Stream, Object

Data type:
: Enum


| Level         | Description                                          |
|---------------|------------------------------------------------------|
| stream        | Delay and packet delay variation should be kept as low as possible |
| interactive   | Delay should be kept as low as possible, but some variation is tolerable |
| transfer*     | Delay and packet delay variation should be reasonable, but are not critical |
| background    | Delay and packet delay variation is no concern  |




Application Resilience
----------------------

This Intent describes how an application deals with disruption of its communication,
e.g. connection loss. It communicates how well the application can recover from such
disturbance and can have implications on how many resources the OS should allocate to
failover techniques for this particular communication unit.


Short name:
: resilience

Applicability:
: Association,  Connection, Stream, Object

Data type:
: Enum


| Level         | Description                                          |
|---------------|------------------------------------------------------|
| sensitive*    | Disruptions result in application failure, disrupting user experience |
| recoverable   | Disruptions are inconvenient for the application, but can be recovered from |
| resilient     | Disruptions have minimal impact for the application |



Cost Preferences
----------------

This describes the Intents of an Application towards costs cased by
the respective communication unit. It should guide the OS how to
handle cost vs. performance and reliability tradeoffs.

Short name:
: cost

Applicability:
: Association,  Connection, Stream, Object

Data type:
: Enum

| Level         | Description                                          |
|---------------|------------------------------------------------------|
| no_expense    | Avoid expensive transports and consider failing otherwise |
| optimize_cost | Prefer inexpensive transports and accept service degradation |
| balance_cost* | Use system policy to balance cost and other criteria |
| ignore_cost   | Ignore cost, choose transport solely based on other criteria |


Usage examples
==============

Example 1
---------

Consider a cellphone performing an OS upgrade.
This process usually implies downloading a large file.
This is a bulk transfer for which the application may already know
the file size.
Timing is typically noncritical and the data can be downloaded as
background traffic with minimal cost and power overhead.
It would not hurt if the TCP connection was closed during the
transfer as the download can be continued.

For this case, the application should set the "Traffic Category"
to "bulk", "Timeliness" to "background", and "Application Resilience"
to "resilient".
In addition, "Object Size to be Received" can be provided.
Finally, the application may set the the "Cost Preferences" to "no_expense".

The OS can use this information and therefore may schedule this
transfer on a flaky but not traffic-billed WiFi link and may reject
the connection attempt if no cheap access link is available.


Example 2
---------

Consider a user watching non-live video content using MPEG-DASH {{DASH}}.
This usually means fetching a stream of video chunks.
The application should know the size of each chunk and may know
the bitrate and the duration of each chunk and the whole video.
Disconnection of the TCP connection should be avoided
because that might have an effect that is visible to the user.

For this case, the application should set the "Traffic Category" to 
"stream", the "Timeliness" to "stream", and "Application Resilience"
to "sensitive". It may also provide the "Stream Bitrate Received"
and "Duration" expected.
Finally, the application may set the the "Cost Preferences" to 
"balance_cost".

The OS can use this information and, e.g, use MPTCP {{RFC6824}} if
available to schedule the traffic on the cheaper link (e.g, WiFi)
while establishing an additional subflow over an expensive link
(e.g., LTE).
If the desired bandwidth cannot be matched by the cheaper link, the
more expensive link can be added to satisfy the desired bandwidth.

If the application would set the "Cost Preferences" to
"optimize_cost", the OS would not schedule traffic on the second
subflow and the application would reduce the video quality to
adapt to the available data rate.


Example 3
---------
Consider a user managing a remote machine via SSH.
This usually involves at least one long-lived console session and 
possibly file transfers using SCP or rsync multiplexed on the same 
association (e.g. TCP connection).

For the console session, the application can set the "Traffic Category"
to "control", the "Burstiness" to "random bursts", the timeliness to 
"interactive" and the resilience to "sensitive".

For the file transfers, SSH may set both, the "Traffic Category" and
"Burstiness" to "bulk". It may also know the size of the transfer
and therefore sets "Object Size to be Sent" or "Object Size to be
Received".

Assuming there are transport opportunities supporting multiple
streams in a single association (e.g. SCPT {{RFC4960}}),
the OS can use this information to schedule the streams
over different links to meet their requirements (latency vs. bandwidth).
In case the OS has to use TCP, it can still optimize by disabling 
TCP Nagle Algorithm for console session related transmissions.


Implementation Guidelines
=========================

TBD



Security Considerations {#sec}
=======================

Performance Degradation Attacks
-------------------------------

We assume that applications specify their preferences in a selfish, but
not malicious way and that it is up to the OS to find a compromise
between demands. 

A malicious application could confuse the OS in a way that leads to scheduling traffic with certain Intents on amore expensive interface, 
penalizing this traffic, or even rejecting it.
The attack vector added by this is negligible:
As the malicious application could also generate the traffic it claims 
to intent, it already has a much more powerful attack vector.

As a mitigation, the OS could monitor and compare the intents specified 
with the traffic actually generated and notify the user if the usage
of Socket Intents is unusual or defective.


Information Leakage
-------------------

Varying the transport or IP layer parameters of packets belonging to 
different Streams or Objects multiplexed in the same encrypted 
association might enable an attacker to gain some ground truth
about the shares of different kinds of traffic. As this might also
be implied by packet timings, application developers might weight the
small additional information disclosure against the possible 
performance gains. Using Socket Intents on Association level can be 
considered safe.



IANA Considerations {#iana}
===================

The Socket Intents type namespace SHOULD be managed by the IANA
registry. Details conforming to {{RFC5226}} are laid out in
{{typespec}}, the initial types for the registry are described in
{{types}}.



Publications History
====================

- The original idea of Socket Intents was published in {{CoNEXT2013}}.
- A performance study "Socket Intents: OS Support for Using Multiple Access Networks and its Benefits for Web Browsing" is under submission.


--- back



