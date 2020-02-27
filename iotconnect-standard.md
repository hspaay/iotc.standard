The IotConnect Standard
=======================

The IotConnect standard defines how information is exchanged between IoT publishers and consumers.


[[TOC]]

# Introduction

As connected devices become more and more prevalent, so have the problems surrounding them. These problems fall into multiple categories: 

## Interoperability
   
The use of information produced by various devices is challenging because of the plethoria of different protocol and data formats in use. This is apparent in home automation solutions such as OpenHAB and Home Assistant that each implement hundreds of bindings to talk to different devices and services. Each solution has to reimplement these bindings. This implementation then has to be adjusted to different platforms, eg Linux, Windows and MacOS, which adds even more work.

Without a common standard it is unavoidable that manufacturers of IoT devices choose their own protocols. It is in everyone's interest to provide a standard that enables an open information interchange so that bindings only have to be implemented once. 

This standard defines the messages for information exchange.

## Discovery

Discovery of connected IoT devices often depends on the technology used. There is no standard that describes what and how discovery information is made available to consumers independent of their implementation. 

Application developers often implement solutions specific to their application and the devices that are supported. To facilitate information exchange it must be possible to discover the information that is available independent of the technology used.

This standard defines the process and messaging for discovery of devices and services without the need for a central resource directory. 

Note. The IETF draft "CoRE Resource Directory (draft-ietf-core-resource-directory-20) takes the approach where a centralized service provides a directory of resources. This approach works fine but does not fit within the concept of this standard for several reasons: Mainly it requires and additional direct connection between client and directory which adds an additional protocol with its own encoding. This standard is based on only requiring connections between client and message broker, keeping the attack footprint to a minimum. The protocol is JSON based for all messages. Second, it does not support sharing of information between zones. Last, it does not support the concept of last will and testament when a publisher unexpectedly disconnects from the network. Like most things it is possible to make it work but it is not the best fitting solution.


## Configuration

Configuration of IoT devices is often done through a web portal of some sort from the device itself or a gateway. These web portals are not always as secure as they should be. They often require a login name and password and lack 2 factor authentication. Passwords are easily reused. Backdoors are sometimes left active. Overall security is lacking.
   
Configuration is not always suited for centralized management by application services. For example, to configure all temperature sensors to report in Celcius the user has to login to the device management portal(s), find the sensor and find the configuration for this. This is difficult to automate.

This standard defines the process and messaging for remote configuration of devices and services.

Nodes that can be configured contain a list of configuration records described in the node discovery. The configuration value can be updated with a configure command. This is described further in the configuration section. 

## Security
   
Security is a major concern with IoT devices. Problems exist in several areas:

  1. It is difficult to design devices for secure access from the internet. The existance of large botnets from hacked computers and devices show how severe this problem is. Good security is hard and each vendor has to reinvent this wheel. This is not likely to change any time soon.
  
  2. Commercial devices that connect to a service provider share personal information without the user understanding what this information is, and without having control on how it is used. While regulations like Europe's [GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation) attempt to address this ... somewhat, reports of data misuse and breaches remain all too frequent.

  3. There is no easy and secure way to self-serve information over the internet. Often the only option is to trust a 3rd party service provider in this, which leads to the previous two problems. In addition the monthly recurring cost might be out of reach for many users.

This standard defines the security aspects build into the specification.

# Terminology

| Terminology   | Description |
|:-----------   |:------------|
| Account       | The account used to connect a publisher to an message bus |
| Address       | Address of the node consisting of zone, publisher and node identifier. Optionally it can include the input or output type and instance.|
| Authentication| Method used to identify the publisher and subscriber with the message bus |
| Bridge        | The service that publishes subscribed information into a different zone. |
| Configuration | Configuration of the node configuration|
| Data          | The term 'data' is used for raw data collected before it is published. Once it is published it is considered information.|
| Discovery     | Description of nodes, their inputs and outputs|
| Information   | Anything that is published by a producer. This can be sensor data, images, discovery, etc|
| Message Bus   | A publish and subscribe capable transport for publication of information. Information is published by a node onto a message bus. Consumers subscribe to information they are interested in use the information address. |
| Node          | A node is a device or service that provides information and accepts control input. Information from this node can be published by the node itself or published by a (publisher) service that knows how to access the node. |
| Node Input    | Input to control the node, for example a switch.|
| Node Output   | Node Information is published using outputs. For example, the current temperature.|
| Publisher     | A service that is responsible for publishing node information on the message bus and handle configuration updates and control inputs. Publishers are nodes. Publishers sign their publications to provide source verification.|
| Retainment    | A feature of a message bus that remembers that last published message. Not all message busses support retainment. It is used in publishing the values and discovery messages so new clients receive an instant update of the latest information |
| Subscriber    | Consumer of information that uses node address to subscribe to information from that node.|
| ZBM           | Zone Bridge Manager. Manages bridges to share information with other zones.
| ZSAS          | Zone Signature Authority Service. This service manages keys and signatures of zone members |
| ZSM           | Zone Security Monitor. Monitors publications in a zone and watches for intrusions.|
| Zone          | An area in which information is shared between members of a zone. |


# Versioning

The standard uses semantic versioning in the form v\{major}.\{minor}.

Future minor version upgrades of this standard must remain backwards compatible. New fields can be added but MUST be optional. Implementations MUST accept and ignore unknown fields and in general follow the [robustness principle](https://engineering.klarna.com/why-you-should-follow-the-robustness-principle-in-your-apis-b77bd9393e4b)

A major version upgrade of this standard is not required to be backwards compatible but **MUST** be able to co-exists on the same bus. Implementations must ignore messages with a higher major version.

Publishers include their version of the standard when publishing their node. See 'discovery' for more information.

# Technology Agnostic

This standard is technology agnostic. It is a standard that describes the information format and exchange for discovery, configuration, inputs and outputs, irrespective of the technology used to implement it. Use of different technologies will actually serve to further improve interoperability with other information sources.

A reference implementation of a publisher is provided for the golang and python languages using the MQTT service bus.


# System Overview

![System Overview](./system-overview.png)


## Zone

A zone defines the area in which information is shared amongst its members. A zone can be a home, a street, a city, or a virtual area like an industrial sensor network or even a game world. Each zone has a globally unique identifier, except for the local zone called '\$local'. Local zones cannot share information with other zones.

A zone has members which are publishers or subscribers (consumers). All members of a zone have access to information published in that zone. The information is not available outside the zone unless intentionally shared. Publication in the zone is limited to members that have the publish permissions. Not surprisingly these are called 'publishers'.

A zone can be closed or open to consumers. An open zone allows any consumer to subscribe to publications in that zone without providing credentials. A closed zone requires consumers to provide valid credentials to connect to the message bus of that zone. Whether a zone is open or closed is determined by the configuration of the message bus for that zone.

A zone has its own topology separate from the underlying TCP/IP network used. It can operate on a local area network or use the internet. The only requirement is that each member can connect to the message bus.

## Message Bus

The use of publish/subscribe message bus has a key role in exchange and security of information in a zone. It not only routes all communications for the zone but also secures publishers and consumers by allowing them to reside behind a firewall, isolated from internet access.

A message bus carries only publications for the zone it is intended for. Multi-zone or multi-tenant message busses can be used but each zone must be fully isolated from other zones. Note that a bridge can publish messages from one zone into another. More on this below.

As the network topology is separate from the zone topology, publishers and subscribers in a zone can be on different networks and behind firewalls. This reduces the attack footprint as none of the publishers or subscribers need to be accessible from the internet. The message bus is the only directly exposed part of the system. It is key to make sure the message bus is properly secured and hardened. For more on securing communication see the security section.

The message bus must be configured to require proper credentials of publishers. Open zones can allow subscribers to omit credentials.

### Message Bus Protocols

This standard is agnostic to the message bus implementation and protocol. The minimum requirement is support for publishing and subscribing using addresses. 

It is highly recommended however that publishers implementation support the MQTT transport protocol. Support for additional transports such as AMQP or HTTP with websockets is optional. 

The reason to choose MQTT as the defacto default is because a common standard is needed for interoperability. MQTT is low overhead, well supported, supports LWT (Last Will & Testament), has QOS, and clients can operate on constrained devices. It is by no means the ideal choice as explained in [this article by Clemens Vasters](https://vasters.com/archive/MQTT-An-Implementers-Perspective.html)

If in future a better protocol becomes the defacto standard, the MQTT protocol will remain supported as a fallback option until this changes in a future version of this standard.

### Guaranteed Delivery (or lack thereof)

The use of a simple message bus, like MQTT, brings with it certain limitations, the main one being the lack of guaranteed delivery. The role of the message bus is to deliver a message to subscribers **that are connected**. While this simplifies the implementation, it pushes the problem of guaranteed delivery to the application. It is effectively a lossy transport between publishers and subscribers.

The standard mitigates this to some extend by supporting a 'history' output that contains recent values. It is possible to catch up to missed messages by checking the history after a reconnect. The penalty is higher bandwidth, and this is only useful in cases of low message rate or high bandwidth capabilities.

Secondly, MQTT supports 'retainment' messages where the last value of a publication is retained. When a consumer connects, it receives the most recent message for all addresses it subscribes to, bringing it instantly up to date (with potentially gaps). Note that not all MQTT implementations support retainment. 

This usage of the message bus will do fine in cases where the goal is to get an up to date recent most value. The loss of a occasional output value in this case is not critical. The use of the history publication can be used to fill in any gaps if needed, but this is only effective for low update frequencies. It well suited for monitoring environmental sensors.

In cases of critical messages such as emergency alerts, some kind of handshake or failover mechanism is strongly adviced. In these cases the transport is merely a step in a longer chain. What matters is that the message is guaranteed to be processed. This requires application level support.

Information exchange handshake over the message bus must be applied at the application level. A service that sends out a critical output message will repeat it until it has received an acknowledgement on its input that it has been processed.

Based on these considerations the use of simple message bus, like MQTT, should be sufficient for most use-cases. 

### Severely Constrained Clients

For severely constrained devices such as micro-controller, a message bus client might simply be too complicated to implement. While the JSON message format is easy to generate, it is not as easy to parse. In these cases it might be better to use an adapter/publisher as the connection point for these devices that translates between to the native protocol and this standard.

### Severely Constraint Bandwidth

In the IoT space, bandwidth can be quite limited. The use of LTE Cat M1, NB-IoT, or LPWAN LTE restrictrs the bandwidth due to cost of the data plan. For example, some plans are limited to 10MB per month. If a sensor reports every minute then a single message is limited to approx 1KB per message including handshake. This almost certainly requires some form of compression or other optimization. Just establishing a TLS connection can take up this much.

The objective of this standard is to support interoperability, not low bandwidth. These are two different concerns that are addressed separately. The use of adapters make it very easy to work with low bandwidth devices using their native protocol.

## Nodes

A zone is populated by nodes that produce or consume information. A node can be a hardware device, a service, or a combination of both.

Nodes can but do not have to be compatible with this standard. For nodes that are not compatible, so-called 'adapter node' provides interoperability between the node native protocol and this standard. 

Nodes that publish information according to this standard are called **publishers**. They publish their own output information or publish information from incompatible nodes.

A node has inputs and/or outputs through which information passes. A node can have many as inputs and outputs that are connected to the node. Inputs and outputs are part of their node and cannot exist without it. 

## Publishers

Publishers are nodes that send and receive messages as per this standard. 

Nodes that are not compatible with this standard require an 'adapter' that publishes on its behalf. In this case there are two nodes, the adapter that is the publisher, and the node whose information is being published. If a gateway is involved then there are three or more nodes. The publisher, the gateway node and each of the nodes that is connected to the gateway.

For example, a ZWave adapter can obtain sensor data from ZWave nodes via a ZWave controller (gateway), and publish information of the ZWave nodes that are connected to this controller. The adapter, the gateway and each zwave device is represented as a node. 

Publishers must use credentials to connect to a zone's message bus before they can publish. To publish securely, a publisher must also have to joined the zone through the Zone Signature Authority Service (ZSAS). More on that later.

Publishers are responsible for:

1. Publishing output information 
2. Handling requests to update inputs
3. Publish node discovery information
4. Publish input and output discovery information
5. Update node configuration
6. Update security keys and certificates 

These tasks are discussed in more detail in following sections.

### Addressing

Information is published using an address on the message bus. This address consists of the node address to whom the information relates, a command keyword related to that node, and optionally a suffix if the command relates to the node input or output.

Address segments can only contain of alphanumeric, hyphen (-), and underscore (\_) characters. Reserved words start with a dollar ($) character. The separator is the '/' character. 

> An address has the form: **\{zone}/\{publisher}/\{node}/\{command}\[/\{iotype}/\{instance}]**

Where:
* {zone} is the zone in which the node lives
* {publisher} is the ID of the publisher that publishes the node information
* {node} is the ID of the node that is being addressed
* {command} the node command, be it node discovery, publishing its outputs, or setting its input.
* {iotype} and {instance} refers to a particular input or output of the node

For message bus systems that do not support the '/' character as address separator, the separator character of the message bus implementation can be used. However, the message itself must contain the original address using the '/' character as the separator.

**Reserved commands:**

| command      | purpose |
|:--------     |:--------|
| \$batch      | Publication of a batch of events |
| \$configure  | Command to update the node configuration |
| \$event      | Publication of a single event with multiple values |
| \$history    | publication of a list of values of a single output. Must be followed by an output type and instance |
| \$input      | Publication of a node input discovery. Must be followed by an input type and instance |
| \$latest     | Publication of a single output value including metadata. Must be followed by an output type and instance|
| \$node       | Publication of a node discovery |
| \$output     | Publication of a node output discovery. Must be followed by an output type and instance |
| \$set        | Control the input value of a node, must be followed by an input type and instance |
| \$status     | Publication of the node status |
| \$value      | Publication of an output sensor value without metadata. Must be followed by an output type and instance |
| \$keys       | Set the keys and signature of a publisher (encrypted) |


When an output is directly controlled by an input, the iotype and instance are the same. To control the input a message is sent to the node with the $set command followed by the iotype and instance. The node processes the command and if accepted updates the output by publishing the result with the $value and $latest commands.

### Node Aliases

When devices are replaced then the node identifier of the replacement can differ from the original. ZWave for example generates a new node ID each time a node is added to the network. This leads to the problem that when replacing a node, all consumers must be updated to use the replacement node ID instead, which can be quite a bit of effort. 

To address this, nodes can be configured with an 'alias' ID. When a node alias is set, all input and output publications use the alias instead of the node ID in the address for the message bus. The address inside the input and output messages remain unchanged.

The node alias can be set through its configure command. Support for node aliases is optional and implemented in the publisher. If the publisher configuration does not have an 'alias' configuration option then it is not supported.

## Subscribers

Anyone with permission to connect to the message bus can subscribe to messages published in the zone. Publishers subscribe as well in order to handle key updates, configuration updates and input updates.

Consumers like user interfaces and services that do not publish are merely subscribers and do not classify as nodes. Open zones can allow anyone to subscribe without credentials.


## Zone Signature Authority Service - ZSAS

The Zone Signature Authority Service issues keys to publishers that have joined the security zone. Publisher use these keys to create a [digital signature](https://en.wikipedia.org/wiki/Digital_signature) for each message they publish so consumers can verify they come from the publisher and haven't been tampered with. The publisher identity information is signed by the ZSAS when the publisher joined the zone and verifies that the publisher is who it claims to be. For local zones the ZSAS is the highest authority and is protected by the message bus only allowing that service to publish on its address. For public zones the ZSAS itself includes a certificate for its domain in its identity.

For more detail, see the security section


# Discovery

Support for discovery lets consumers find the information they are interested in without the need for a central directory. The objective is for the node to be sufficiently described so consumers can identify and configure it without further information.

Publishers are responsible for publishing discovery messages for nodes, their inputs and outputs. The discovery data describes the nodes in detail, including their type, attributes and configurations.

Just like publications of the various values, the discovery publications consist of a JSON object with two fields: "message" and "signature". Creation and verification of the base64 encoded signature is described in the 'signing' section.

**Retainment:**

Where supported, discovery messages are published with retainment. When connection to the message bus was lost and is re-established, the discovery messsages are re-published in case the retainment cache was cleared.

On message busses without retainment and without a central directory, discovery messages are re-published periodically. The default interval is once a day but can be changed through publisher configuration. Subscribers must do their own caching of discovery information in case they disconnect. The downside of this approach is that it can take up to a day to discover new publishers. 

## Discover Nodes

Node discovery messages contain a detailed description of the node. It does not contain information on inputs and outputs as these are published separately to allow bridges to bridge only specified inputs or outputs.

Node discovery address:

  >  **\{zone}/\{publisher}/\{node}/\$node**

If the node is the publisher itself then the reserved '\$publisher' node identifier *MUST* be used.

  >  **\{zone}/\{publisher}/\$publisher/\$node**

Where:
* {zone} is the zone in which the node lives
* {publisher} is the ID of the publisher of the information. The publisher Id is unique within its zone
* {node} is the node that is discovered. This is a device or a service identifier and unique within a publisher. A special ID of “$publisher” is reserved for nodes that are publishes.
* $node suffix for node discovery

Node discovery message structure:

| Field        | Data Type | Required     | Description
|:-----------  |:--------- |:----------   |:------------
| address      | string    | **required** | The address of the node discovery (zone/publisher/node/\$node)|
| attr         | map       | **required** | Attributes describing the node. Collection of key-value string pairs that describe the node. The list of predefined attribute keys are part of the standard. See appendix B: Predefined Node Attributes. |
| config       | List of **Configuration Records** | optional | Node configuration, if any exist. Set of configuration objects that describe the configuration options. These can be modified with a ‘$configure’ message.|
| sender       | string    | **required** | Address of the publisher node of the message (zone/publisher/\$publisher/\$node) |
| timestamp    | string    | **required** | Time the record is created |
| identity     | Identity  | publishers   | Publisher identity used to identify the publisher in secure zones. Includes public keys for verifying and encryption. Only included with publishers |
| identitySignature  | string    | optional     | Base64 encoded signature of the identity signed by the ZSAS. Empty for publishers that have not joined the secure zone.

**Configuration Record**

The configuration record is used in both node configuration and input/output configuration. Each configuration attribute is described in a record as follows:

| Field    | Data Type | Required  | Description |
|:-------- |:--------- |:--------- |:----------- |
| name     | string    | **required** | Name of the configuration. This has to be unique within the list of configuration records. See also Appendix C: Predefined Configuration Names |
| datatype | enum      | optional| Type of value. Used to determine the editor to use for the value. One of: bool, enum, float, int, string. Default is ‘string’ |
| default  | string    | optional| Default value for this configuration in string format |
| description| string  | optional | Description of the configuration for human use |
| enum     | \[strings] | optional* | List of valid enum values as strings. Required when datatype is enum |
| max      | float     | optional | Optional maximum value for numeric data |
| min      | float     | optional | Optional minimum value for numeric data | 
| secret   | bool      | optional | Optional flag that the configuration value is secret and will be left empty. When a secret configuration is set in \$configure, the value is encrypted with the publisher node public key. |
| value    | string    | **required**| The current configuration value in string format. If empty, the default value is used if provided. |

**Identity Record**

The identity record is included with nodes that are publishers and is intended to verify the identity of the publisher. It is signed by the ZSAS when the publisher joins a secure zone and renewed periodically. Consumers must verify the signature using the ZSAS public key when the publisher node is updated.

| Field           | type     | required     | Description |
|:--------------- |:-------- | :----------  |:----------- |
| expires         | string   | **required** | ISO8601 Date this identity expires |
| location        | string   | optional     | Location of the publisher, city, province/state, country |
| organization    | string   | optional     | Organization the publisher belongs to |
| publicKeyCrypto | string   | **required** | Base64 encoded public key for encrypting messages to the publisher | 
| publicKeySigning| string   | **required** | Base64 encoded public key for verifying publisher signatures |
| publisher       | string   | **required** | The publisher node ID |
| timestamp       | string   | **required** | Time the identity was re-signed |
| url             | string   | optional     | URL of the publisher information page |
| zone            | string   | **required** | The zone ID in which the publisher lives |


Example payload for node discovery

~~~json
zone-2/openzwave/5/\$node:
{
  "message": {
    "address": "zone-2/openzwave/5/$node",
   
    "attr": {
      "make": "AeoTec",
      "type": "multisensor"
    },
    "config": {
      "name": {
        "datatype": "string",
        "description": "Friendly name",
        "value": "barn multisensor"
      }, 
    },
    "timestamp": "2020-01-20T23:33:44.999PST",
    "sender": "zone-2/openzwave/$publisher/$node"
  },
  "signature": "..."
}
~~~

## Discover Available Inputs and Outputs

Inputs and outputs discovery are published separately from the node. The discovery of each output and each input is published separately. This facilitates asynchroneous discovery of inputs and outputs and allows control over which outputs are shared with other zones. 

Address of input discovery:

> **\{zone}/\{publisher}/\{node}/\$input/\{inputType}/\{instance}**

Address of output discovery:

> **\{zone}/\{publisher}/\{node}/\$output/\{outputType}/\{instance}**

| Address segment | Description |
| :-------------- | :---------- |
| {zone}       | The zone in which the node lives |
| {publisher}  | The service that is publishing the information. A publisher provides its identity when publishing a node discovery. The publisher Id is unique within its zone.  |
| {node}       | The node whose input or output is discovered. This is a device or a service identifier and unique within a publisher. |
| \$input      | Command for input discovery. |
| {inputType}  | Type identifier of the input. For a list of predefined types see Appendix D |
| \$output     | Command for output discovery. |
| {outputType} | Type identifier of the output. For a list of predefined types see Appendix D |
| {instance}   | The instance of the input or output on the node. If only a single instance exists the standard is to use 0 unless a name is used to provide more meaning. |

For example, the discovery of a temperature sensor on node '5', published by a service named 'openzwave', is published on address:

  > **$local/openzwave/5/\$output/temperature/0**

The message structure:

| Field       | Data Type | Required     | Description |
|:----------- |:--------- |:---------    |:----------- |
| address     | string    | **required** | The address on which the message is published |
| config      | List of **Configuration Records**|optional|List of Configuration Records that describe in/output configuration. Only used when an input or output has their own configuration. See Node configuration record above for the definition |
| datatype    | string    | optional     | Value datatype. One of boolean, enum, float, integer, jpeg, png, string, raw. Default is "string". |
| default     | string    | optional     | Default output value |
| description | string    | optional     | Description of the in/output for humans |
| enum        | list      | optional*    | List of possible values. Required when datatype is enum |
| max         | number    | optional     | Maximum possible in/output value |
| min         | number    | optional     | Minimum possible in/output value |
| sender      | string    | **required** | Address of the publisher node of the message |
| timestamp   | string    | **required** | Time the record is created |
| unit        | string    | optional     | The unit of the data type |

Example payload for output discovery:

~~~json
$local/openzwave/5/\$output/temperature/0:
{
  "message": {
    "address": "$local/openzwave/5/$output/temperature/0",
    "datatype": "float",
    "sender": "$local/openzwave/$publisher/$node",
    "timestamp": "2020-01-20T23:33:44.999PST",
    "unit": "C",
    "value": "20.5",
  },
  "signature": "...",
}
~~~   

# Publishing Output Values

Publishers monitor the outputs of the node(s) they manage and publish a new value when there is a change. 

Output values can be published on several addresses, depending on the need and circumstances. 

Address format:  **\{zone}/\{publisher}/\{node}/\{command}/\[\{type}/\{instance}/]**

| Address segment | Description|
|:--------------- |:-----------|
| {zone}          | The zone in which publishing takes place. |
| {publisher}     | The service that is publishing the information. The publisher Id is unique within its zone.|
| {node}          | The node that owns the input or output. This is a device identifier or a service identifier and unique within a publisher.|
| {command}       | Command that indicates the purpose of the publication. Each command is described in more detail in the following paragraphs.|
| {type}          | The type of  input or output, eg temperature. This standard includes a list of output types. |
| {instance}      | The instance of the type on the node. For example, a node can have two temperature sensors. The combination type + instance is unique for the node. The instance can be a name or number. If only a single instance exists the instance can be shortened to “_” |

 With exception of the \$value command, all publications contain a payload consisting of a JSON object with a message and signature:

~~~json
{
  "message": {},
  "signature": "..."
}
~~~

The signature is created by creating the hash of the message content and encrypting it using the private key of the publisher of the message and encode the result in a base64 string. See more in the 'signing' section.


## \$value: Publish Single 'no frills' Output Value

The payload used with the '\$value' command is the straight information without metadata such as timestamp and signature.

The \$value publication is the fallback that every publisher *MUST* publish. It is intended for interoperability with highly constrained devices or 3rd party software that do not support JSON parsing. The payload is therefore the straight value.

Address:  **\{zone}/\{publisher}/\{node}/\$value/\{type}/\{instance}**

Payload: Output value, converted to string. There is no message signature.

Example:
~~~
zone-1/openzwave/6/\$value/temperature/0: 20.6
~~~

## \$latest: Publish Latest Output With Metadata

The \$latest publication indicates the publisher provides the latest known value of the output including metadata such as the timestamp. The value is represented as a string. Binary data is converted to base64.

This is the recommended publication publishing updates to single value sensors. See also the \$event publication for multiple values that are related.

Address:  **\{zone}/\{publisher}/\{node}/\$latest/\{type}/\{instance}**

The message structure is as follows:

| Field        | Data Type | Required     | Description |
|:-------------|:----------|:------------ |:----------- |
| address      | string    | **required** | The address on which the message is published |
| sender       | string    | **required** | Address of the publisher node of the message |
| timestamp    | string    | **required** | timestamp of the value ISO8601 "YYYY-MM-DDTHH:MM:SS.sssTZ" |
| unit         | string    | optional     | unit of value type, if applicable |
| value        | string    | **required** | value in string format |

Example of a publication on zone-1/openzwave/6/\$latest/temperature/0:

~~~json
{
  "message": {
    "address": "zone-1/openzwave/6/$latest/temperature/0",
    "sender": "zone-1/openzwave/$publisher/$node",
    "timestamp": "2020-01-16T15:00:01.000PST",
    "unit": "C",
    "value": "20.6",
  },
  "signature": "..."
}
~~~

## \$history: Publish History of Recent Output Values

The payload for the '\$history' command contains an ordered list of the recent values along with address information and signature. The history is published each time a value changes. 

This is intended to be able to determine a 24 hour trend. It can also be used to check for missing values in case transport reliability is untrusted. The content is not required to persist between publisher restarts.

Address:  **\{zone}/\{publisher}/\{node}/\$history/\{type}/\{instance}**

The message structure:

| Field        | Data Type | Required     | Description |
|:----------   |:--------  |:-----------  |:------------ |
| address      | string    | **required** | The address on which the message is published |
| duration     | integer   | optional     | Nr of seconds of history. Default is 24 hours (24*3600 seconds)
| history      | list      | **required** | eg: \[\{"timestamp": "YYYY-MM-DDTHH:MM:SS.sssTZ","value": string}, ...] |
|| timestamp   | string    | ISO8601 "YYYY-MM-DDTHH:MM:SS.sssTZ" |
|| value       | string    | Value in string format using the node's unit |
| sender       | string    | **required** | Address of the publisher node of the message |
| timestamp    | string    | **required** | timestamp of the message |
| unit         | string    | optional     | unit of value type |

For example:

~~~json
zone-1/openzwave/6/temperature/0/\$history:
{
  "message": {
    "address" : "zone-1/openzwave/6/$history/temperature/0",
    "duration": "86400",
    "history" : [
      {"timestamp": "2020-01-16T15:20:01.000PST", "value" : "20.4" },
      {"timestamp": "2020-01-16T15:00:01.000PST", "value" : "20.6" },
      ...
    ],
    "sender": "zone-1/openzwave/$publisher/$node",
    "unit": "C",
  },
  "signature": "...",
}
~~~

## \$event: Publish Event With Multiple Output Values

The optional \$event publication indicates the publisher provides multiple output values with the same timestamp as a single event.

Address:  **\{zone}/\{publisher}/\{node}/\$event**

The message structure:

| Field        | Data Type | Required     | Description |
|:----------   |:--------  |:-----------  |:------------ |
| address      | string    | **required** | The address on which the message is published, zone/publisher/node/\$outputs |
| event        | map       | **required** | Map with {output type/instance : value} 
| sender       | string    | **required** | Address of the publisher node of the message |
| timestamp    | string    | **required** | timestamp of the event in ISO8601 format |

For Example:

~~~json
zone-1/vehicle-1/\$publisher/\$event:
{
  "message" : {
    "address" : "zone-1/vehicle-1/$publisher/$event",
    "event" : [
      {"speed/0": "30.2" },
      {"heading/0": "165" },
      {"rpm/0": "2000" },
      {"odometer/ecu": "2514333222" },
      ...
    ],
    "sender": "zone-1/vehicle-1/$publisher/$node",
    "timestamp": "2020-01-16T15:00:01.000PST",
  },
  "signature": "...",
}
~~~   

## \$batch: Publish Batch With Multiple Events

The optional \$batch publication indicates the publisher provides multiple events. This is intended to reduce bandwidth in case for high frequency sampling of multiple values with a reduced publication rate. Consumers must process the events in the provided order, as if they were sent one at a time.

Address:  **\{zone}/\{publisher}/\{node}/\$batch**

The message structure:

| Field        | Data Type | Required     | Description |
|:----------   |:--------  |:-----------  |:------------ |
| address      | string    | **required** | The address on which the message is published, eg zone/publisher/node/\$batch |
| batch        | list      | **required** | Time ordered list of events with their timestamp, oldest first and newest last.
|| timestamp   | string    | timestamp of the event in ISO8601 format "YYYY-MM-DDTHH:MM:SS.sssTZ" |
|| event       | map       | Map with {output type/instance : value} |
| sender       | string    | **required** | Address of the publisher node of the message |
| timestamp    | string    | **required** | ISO8601 timestamp this message was created |

# Updating Inputs

## \$set: Control Input Value
Publishers subscribe to receive commands to update the inputs of the node they manage.

Address:  **\{zone}/\{publisher}/\{node}/\$set/\{type}/\{instance}**

The message structure:

| Field        | Data Type | Required      | Description
|:------------ |:--------- |:----------    |:-----------
| address      | string    | **required**  | The address on which the message is published |
| timestamp    | string    | **required**  | Time this request was created, in ISO8601 format, eg: YYYY-MM-DDTHH:MM:SS.sssTZ. The timezone is the local timezone where the value was published. If a request was received with a newer timestamp, up to the current time, then this request is ignored. |
| sender       | string    | **required** | Address of the publisher node of the message |
| value        | string    | **required** | The input value to set |

For Example:

~~~json
zone-1/openzwave/6/\$set/switch/0:
{
  "message": {
    "address" : "zone-1/openzwave/6/$set/switch/0",
    "sender": "zone-1/mrbob/$publisher/$node",
    "timestamp": "2020-01-02T22:03:03.000PST",
    "value": "true",
  },
  "signature": "...",
}
~~~

# Configuring A Node  

Support for remote configuration lets administrators manage the devices and services sources without having to login to each device and service throught their web portals. The standard defines the messages for obtaining and updating the configuration of nodes by authorized users only.

Publishers of discovery information provide the existing node and/or input and output configuration, and can also accept commands to update this configuration.

Changing configuration and controlling inputs can be limited to specific senders.


Address:  **\{zone}/{publisher}/{node}/\$configure**


Configuration Message structure:

| Field       | type     | required     | Description
|:----------- |:-------- |:------------ |:----------
| address     | string   | **required** | The address on which the message is published |
| config      | map      | **required** | key-value pairs for configuration to update { key: value, …}. **Only fields that require change should be included**. Existing fields remain unchanged.
| sender      | string   | **required** | Address of the sender submitting the request. This is the zone/publisher/node of the consumer.
| timestamp   | string   | **required**    | Time this request was created, in ISO8601 format


Example payload for node configuration:

~~~json
zone1/openzwave/5/\$configure:
{
  "message": {
    "address": "zone1/openzwave/5/$configure",
    "sender": "zone1/openzwave/$publisher/$node",
    "timestamp": "2020-01-20T23:33:44.999PST",
    "config": {
      "name": "My new name"
    }
  },
  "signature": "...",
}
~~~   

# Publish Node Status

The availability status of a node is published by its publisher when the availability changes or errors are encountered. 

Address:  **\{zone}/\{publisher}/\{node}/\$status**

| Address segment | Description
|:--------------- |:------------
| {zone}          | The zone in which discovery takes place. 
| {publisher}     | The publisher of the node discovery which is handling the configuration update for that node.
| {node}          | The node whose configuration is updated. 
| \$status         | Keyword for node status. Published when the availability of a node changes or new errors are reported. It is published by the publisher of the node.


Message Structure:

| Field        | type     | required     | Description |
|:-----------  |:-------- |:-----------  |:----------- |
| address      | string   | **required** | The address on which the message is published |
| errorCount   | integer  | optional     | Nr of errors since startup |
| errorMessage | string   | optional     | Last reported error message |
| errorTime    | string   | optional   | timestamp of last error message in ISO8601 format |
| interval     | integer  | **required** | Maximum interval of status updates in seconds. If no updated status is received after this interval, the node is considered to be lost. |
| lastSeen     | string   | **required** | timestamp in ISO8601 format that the publisher communicated with the node. |
| sender      | string   | **required** | Address of the sender submitting the request. This is the zone/publisher/node/$node of the consumer. |
| status       | enum     | **required** | The node availability status. See below for valid values |
| timestamp    | string   | **required** | Time the status was last updated, in ISO8601 format |

Status values:
* ready: node is ready to perform a task
* asleep: node is a periodically listening and updates to the node can take a while before being processed.
* error: node is in an error state and needs servicing
* lost: communication with the node is lost

# Security

Bus communication should be considered unsafe and must be protected against man-in-the-middle and spoofing attacks. All publications must be considered untrusted unless it is correctly signed.

Message signing is the second line of defence against attackers. (the first line is connecting to the message bus)

In order to sign a message, a publisher must have a set of public/private keys. Initially these keys are created by the publisher on first use. These keys are intended to join the zone. If they are used to sign a message, the signature verification will succeed but without certificate the publisher cannot be verified.

To obtain a certificate the publisher must join a secure zone as described below.

## Joining A Secure Zone

A publisher joins the zone in order to receive keys and signature that can be verified by subscribers.

Keys and signature are issued by the Zone Signature Authority Service - ZSAS. The ZSAS can work with a global Trusted Certificate Authority. This is required to verify signatures when sharing messages between zones.

The steps to join the zone:
1. Generate a temporary keyset
2. Publish the publiser node discovery message containing the public key
3. Wait for the administator to mark the public key as trusted.
4. Continue with the regular key renewal process

**Generating a temporary keyset**

Initially the publisher must create their own private and public keyset. The publisher must publish its own node discovery message that includes this public key and an empty signature field.

**Administrator Marks The Publisher As Trusted**

The **ZSAS** is the Zone Signature Authority Service. Its sole purpose is to generate and sign keys for publishers.

The ZSAS needs to be told that the publisher with public key X is indeed the publisher with the ID it claims to have. Optionally additional identity can be required such as location, contact email, phone, administrator name and address. 

The method to establish trust can vary based on the situation. The following method is used in the ZSAS reference implementation. 

1. On installation of a new publisher, the administrator notes the publisher ID and the public key generated by that publisher. The publisher publishes its discovery using the temporary key.

2. Next, the administrator logs in to the ZSAS service. The service shows a list of untrusted publishers. The administrator verifies if the publisher and the public key match. If there is a match, the administrator informs the ZSAS that the publisher can be trusted. 

3. When a publisher status changes from untrusted to trusted, the ZSAS starts the cycle of key renewal as described below.
   

## Renewing Publisher Keys And Signature - ZSAS

A publisher can not request a new set of keys. Instead, it is issued new keys and signature by the ZSAS automatically when its certificate has less than half its life left. This is triggered when a publisher publishes its own node information using a valid signature. Once a publisher uses the newly issued key and certificate, ZSAS removes the old key from its records. This key can no longer be used to obtain a new key and signature. It is therefore important that the publisher persists the new key and signature before publishing using the new keys. Note that the ZSAS does not store the private key it generated for the publisher. Once send to the publishers, only the public key is retained. The lifetime of a signed identity is relatively short. The default is 30 days. After half this time the identity is renewed by the ZSAS service. If no trusted public key is on record for the publisher, the publisher is marked as untrusted and the identity key is stored for review by administrator. 

The ZSAS listens for publisher discovery messages on address:

  **\{zone}/\{publisher}/\{node}/\$publisher**

The new keys and signature is published on the publisher's zsas address. The publisher must subscribe to this address if it wishes to join the secure zone:

  **\{zone}/\{publisher}/\{node}/\$keys**

The payload is encrypted using the last known publisher's public encryption key. The publisher must decrypt it using its encryption private key. A separate keyset is used for encryption messages to the publisher. The zsas includes the encryption keys, which are renewed at the same time as the signature keys. Since encryption uses a different algorithm than signing and the public key might not be interchangeable, a separate keyset is used to prevent any dependencies. 

Message Structure:

| Field            | type     | required     | Description |
|:------------     |:-------- | :----------  |:----------- |
| address          | string   | **required** | The address on which the message is published |
| identity         | Identity | **required** | The publisher identity information (see node publisher) |
| identitySignature| string   | **required** | The signature of the identity, signed by the ZSAS
| sender           | string   | **required** | Address of the sender, eg the zsas {zone}/\$zsas/\$publisher/\$node |
| privateKeyCrypto | string   | **required** | The publisher new private key for decrypting messages received by the publisher | 
| privateKeySigning   | string   | **required** | The publisher new private key used to sign messages |
| signature        | string   | **required** | The base64 encoded message signature, created by the sender (zsas) |
| timestamp        | string   | **required** | Time the certificate was issued |
| zoneKey          | string   | optional     | The zone shared secure for encrypting/decrypting zone wide messages|


~~~json
{
  "message": {
    "address": "...",
    "cryptoPrivateKey": "...",
    "identity": {
      "address": "zone/publisher/$publisher",
      "cryptoPublicKey": "...",
      "expires": "2019-02-12T15:55:09.000Z",
      "location": "Your City, Province/State, Country",
      "organization": "Your organization",
      "signPublicKey": "...",
      "timestamp": "2019-01-12T14:55:09.000Z",
      "url": "https://wwww.example.org",
    },
    "identitySignature": "...",
    "signature": "...",
    "signPrivateKey": "...",
    "timestamp": "...",
    "zoneKey": "...",
  },
  "signature": "..."
}
~~~   

## Expiring Identity Keys

Depending on policy settings, the ZSAS can choose not to auto renew the identity keys after they have expired. This is more secure but it requires that the publisher is connected before the expiration date.  Once the identity has expired, the administrator must again go through the procecss of joining the publisher to the zone. 

## Signing Messages

After all that work to issue identity and keys, it is now time to use them to make sure that published message are signed by the publisher. 

Unless stated otherwise, a publisher MUST sign messages that it publishes and include it base64 encoded in the payload. In cases where the signature cannot be verified the consumer has the option to ignore unverified publishers.

For example, an alarm company receives a security alert. The alert signature is verified to be from a registered client and not from a bad actor creating a distraction. The node location and alert timestamp are used to identify where and when the alert was given.

### Creating A Signature

The method of signing messages uses [ECDSA Elliptic Curve Cryptography](https://blog.cloudflare.com/ecdsa-the-digital-signature-algorithm-of-a-better-internet). Its keys are shorter than RSA, it has not (yet - Feb 2020) been broken and it is claimed to be [more secure than RSA](https://blog.cloudflare.com/a-relatively-easy-to-understand-primer-on-elliptic-curve-cryptography/).

Publishers sign their messages by:
1. Create a hash of the message using SHA256
2. Encrypt the hash using the private key using ECDSA
3. base64 encode the signature for transportation

See the example code:
* golang: github: iotconnect.standard/examples/example.go
* python: github: examples/example.py
* javascript: github: examples/example.js


## Verifying A Message Signature

Consumers verify a signature by:

1. Determine the publisher of the message from the address
2. Determine the public key of the publisher from its discovery information
3. Determine the hash of the message
4. Verify the signature using the hash and public key


## Verifying Publisher Identity

When a consumer receives a publisher discovery message it needs to verify that the publisher is indeed who it claims to be using the identity signed by the ZSAS service.

Note that the use of a ZSAS service is optional. It is valid to manually install identities on a publisher as long as they can be verified. The process below assumes the presence of a ZSAS.

The ZSAS has the reserved publisher ID of '\$zsas' with a reserved node ID of '\$zsas'. It publishes its own identity (just like any other publisher) with its node on address '{zone}/\$zsas/\$zsas'. In order to verify signatures the consumer has to use the node public key. When the node updates, its identity record must be signed by ZSAS, which is the same process using the ZSAS public key.

A ZSAS can be registered with a global Trusted Certificate Authority (TCA) and creates certificates that are chained to the TCA. By default this uses 'Lets Encrypt' but this can be replaced by other public CAs. Use of a TCA is optional for local-only zones but required when briding between zones. The domain name used for registering a zone with the TCA is '{zone}.iotconnect.zone', where zone has to be globally unique. 

In order to verify the ZSAS all consumers must obtain the TCA certificate.

See the example code:
* golang: examples/example.go
* python: examples/example.py
* javascript: examples/example.js

Todo: how to ensure global uniqueness? UUID?, hash?, registration

The zone '$local' is reserved for local-only zones. In this case the ZSAS generates its own certificate and is considered the highest authority.

## Security Monitor - Zone Security Monitor (ZSM)

The goal of the Zone Security Service is to detect invalid publications and alert the administrator.

The ZSM subscribes to published messages and validates that the messages carry a valid signature. If the signature is not valid then the administrator is notified.


# Sharing Information With Other Zones - Zone Bridge Manager (ZBM)

While it is useful to share information within a zone, it can be even more useful if some of this information is shared with other zones.

This is the task of a Zone Bridge. Zone Bridges are managed by the Zone Bridge Manager (ZBM) publisher. This service is responsible for creating and deleting bridge nodes.

A 'bridge node' connects to a remote zone's message bus and publishes information from the local zone message bus onto the remote zone's message bus. It is configured with connection attributes to access to the remote zone message bus. A Zone bridge has a node in both zones, joined both zones, and can be discovered like any other node.

When a node output is bridged, it is published under its own address. The zone part of the address does not change. It merely becomes available on the message bus of the bridged zone. The signature and content remain unchanged.

## Managing Bridges

TODO: These crud operations should not be command keywords. Or should they?

To create a bridge the ZBM service must be active in a zone. Publish the following command to create a new bridge:
>  **\{zone}/\$zbm/\$zbm/\$set/action/\$createBridge**

The payload is the new bridge node ID. The new bridge has address: {zone}/\$zbm/{bridge id}

To remove a bridge:
>  **\{zone}/\$zbm/\$zbm/\$set/action/\$removeBridge**

The payload is the bridge node ID.

A bridge publishes itself in both zones with its address. 

## Bridge Configuration

Using the standard node configuration mechanism, the bridge is configured with the zone it is bridging to.

Bridge configuration variables can be set on address: {zone} / \$zbm / \{bridge} / \$configure:

Bridges support the following configuration:

| Field        | value type   | value        | Description
|:------------ |:---------    |:-----------  |:----------
| host         | string       | **required** | IP address or hostname of the remote bus
| port         | integer      | optional     | port to connect to. Default is determined by protocol
| protocol     | enum         | optional     | Protocol to use: "MQTT" (default), "REST"
| clientId     | string       | **required** | ID of the client to connect as
| loginId      | string       | **required** | Login identifier obtained from the administrator
| credentials  | string       | **required** | Password to connect

## Forward Nodes, Inputs or Outputs

To forward nodes through the bridge, use the following input command
* To forward an entire node:  
> **\{zone}/\$zbm/\{bridge}/\$set/action/forwardNode**

* To forward a node input:  
> **\{zone}/\$zbm/\{bridge}/\$set/action/forwardInput**

* To forward a node output:  
> **\{zone}/\$zbm/\{bridge}/\$set/action/forwardOutput**
 
The payload is a JSON document containing the message and signature:

~~~json
{
  "message": {},
  "signature": "..."         (base64 encoded signature)
}
~~~   
Message structure:

| Field            | type     | required     | Description |
|:------------     |:-------- |:------------ |:----------- |
| address          | string   | **required** | The address on which the message is published |
| forward          | string   | **required** | The node, input or output address to forward |
| forwardDiscovery | boolean  | optional     | Forward the node/output \$discovery publication, default=true |
| forwardBatch     | boolean  | optional     | Forward the output \$batch publication(s), default=true |
| forwardEvent     | boolean  | optional     | Forward the output \$event publication(s), default=true |
| forwardHistory   | boolean  | optional     | Forward the output \$history publication(s), default=true |
| forwardLatest    | boolean  | optional     | Forward the output \$latest publication(s), default=true |
| forwardStatus    | boolean  | optional     | Forward the node \$status publication, default=true |
| forwardValue     | boolean  | optional     | Forward the output \$value publication(s), default=true |
| sender       | string   | **required** | Address of the sender, eg: zone/publisher/node/\$node of the user that configures the bridge. |
| timestamp   | string    | **required** | Time the record is created |


## Remove Bridged Nodes, Inputs or Outputs 

To remove a forward use the following command:

 **\{zone} / \$zbm / \{bridge} / \$set / action / removeNode**
 **\{zone} / \$zbm / \{bridge} / \$set / action / removeInput**
 **\{zone} / \$zbm / \{bridge} / \$set / action / removeOutput**


The payload is a JSON document containing a message and signature field.

Message structure:

| Field        | type     | required     | Description |
|:------------ |:-------- |:-----------  |:----------- |
| address      | string   | **required** | The address on which the message is published |
| remove       | string   | **required** | The node, input or output address to remove. |
| sender       | string   | **required** | Address of the sender, eg: zone/publisher/node/\$node of the user that configures the bridge. |
| timestamp   | string    | **required** | Time the record is created |


# Appendix A: Node Types

Nodes represent hardware or software services. The node types standardizes on the names of predefined devices or services.

| Key              | Value Description |
|:-------------    |:------------      |
| alarm            | Node is an alarm emitter |
| adapter          | Software adapter, eg virtual device |
| avcontrol        | Audio/video controller, eg remote control |
| beacon           | Location beacon|
| button           | Device with one or more buttons |
| camera           | Web or traffic camera |
| clock            | Time clock |
| computer         | General purpose computer |
| dimmer           | Light dimmer |
| ecu              | Engine control unit with loads of sensors |
| gateway          | Gateway for other nodes (onewire, zwave, etc) |
| gps              | GPS location receiver |
| keypad           | Entry key pad |
| lightbulb        | Light bulb or LED light, eg philips hue |
| lightswitch      | Light switch |
| lock             | Electronic door lock |
| multisensor      | NodDevicee with multiple sensors |
| networkrouter    | Network router |
| networkswitch    | Network switch |
| onoffswitch      | General purpose on/off switch |
| powermeter       | Power or KW meter |
| repeater         | Zwave or other signal repeater |
| receiver         | A (not so) smart radio/receiver/amp (eg, denon) |
| scale            | Physical weight scale |
| sensor           | Device with one sensor. See also multisensor. |
| tv               | A (not so) smart TV |
| unknown          | Unknown device or service |
| wallpaper        | Wallpaper montage of multiple images |
| wap              | Wireless access point |
| watervalve       | Water valve control unit |
| weatherstation   | Weather station with multiple sensors and controls |

# Appendix B: Predefined Node Attributes

Node attributes provide a description of the device or service. These are read-only and usually hard coded into the device or service. 

| Key              | Value Description |
|:-------------    |:------------      |
| version          | Publishers include the version of the standard. Eg v1.0 |
| firmware         | Firmware identifier or version |
| localip          | IP address of the node, for nodes that are publishers themselves |
| location         | String with "latitude, longitude" of device location  |
| mac              | Node MAC address for nodes that have an IP interface |
| make             | Node make or manufacturer |
| model            | Node model |
| type             | Type of node. Eg, multisensor, binary switch, See the Node Types list for predefined values |


# Appendix C: Predefined Configuration Names

Standard configuration names

| Name          | Value Description |
|:------------- |:------------      |
| ip4           | Device static IP-4 address
| ip6           | Device static IP-6 address
| location      | Device location name
| name          | Device friendly name
| netmask       | Network netmask


# Appendix D: Input and Output Types

When available, units used in publication follow the SI standard 

| input/output type| Units  | Value Datatype   | description |
|:--------------  |:--------|:-----------------|:------------|
| acceleration    | m/s2    | List of floats   | coordinates: x,y,z
| action          |         | json      | perform an action; instance has the action name; message has parameters
| airquality      |         | integer   | Number representing the air quality
| alarm           |         | boolean   | Indicator of alarm status. True is alarm, False is no alarm
| atmosphericpressure | kpa, mbar, Psi, hg | float|  
| avchannel       |         | integer   |
| avmute          |         | boolean   |
| avpause         |         | boolean   |
| avplay          |         | boolean   |
| avvolume        | %       | integer   |
| battery         | %       | integer   |
| co2level        | ppm     | float     |
| colevel         | ppm     | float     |
| color           | rgb     | string    |
| colortemperature| K       | float     |
| compass         | degrees | float     | 0-359 degree compass reading |
| contact         |         | boolean   |
| cpulevel        | %       | integer   | 
| current         | A       | float     | Electrical current in Amps
| dewpoint        | C, F    | float     | Dewpoint in degrees Celcius
| distance        | m, yd, ft, in | float | distance in meters
| dimmer          | %       | integer   |
| doorwindowsensor|         | boolean   |
| duration        | sec     | float     |
| electricfield   | V/m     | float     | Static electric field in volt per meter
| elevation       | m, ft   | float     | elevation in meters or feet
| energy          | KWh     | float     |
| errors          |         | integer   |
| freememory      | %       | integer   | Relative memory available in percent
| fuel            | L, G    | float     | Amount of fuel in Liters or Gallons
| heatindex       | C, F    | float     | Apparent temperature (humiture) based on air temperature and  relative humidity. Typically used when higher than the air temperature. At 20% relative humidity the heatindex is equal to the temperature.
| heading         | Degrees | float     | Compass heading in 0-360 degrees 
| hue             |         |           |
| humidex         | C       | float     | Humidity temperature index (feels like temperature) derived from    dewpoint, in degrees Celcius 
| humidity        | %       | float     | Relative humidity in %
| image           | jpeg, png | bytes   | Image in jpeg or png format
| latency         | sec     | float     | 
| location        |         | List of 3 floats | \[latitude, longitude, elevation]
| locked          |         | boolean   | Door lock status 
| luminance       | cd/m2, lux | float  | Amount of light in candela/m2 or in lux
| magneticfield   | T, mT, uT, nT, G(auss) | float | Static magnetic field in Tesla or Gauss
| motion          |         | boolean   | Motion detection status |
| power           | W       | float     | Power consumption in watts |
| pressure        | kpa, Psi| float     | gas or liquid pressure
| pushbutton      |         | boolean   | Momentary pushbutton |
| signalstrength  | dBm     | float     | Wireless signal strength
| speed           | m/s, Km/h, mph | float |
| switch          |         | boolean   |
| temperature     | C, F    | float     | Celcius or Fahrenheit. The default is Celcius if available.
| ultraviolet     | UV      | float     | Radiation index with wavelength from 10 nm to 400 nm, range 0-11+
| voltage         | V       | float     | Volts
| volume          | L, G    | float     | Volume in liters or gallons
| waterlevel      | m(eters), ft, in | float |
| wavelength      | m       | float     |
| weight          | Kg, Lbs | float     | 
| windchill       | C, F    | float     | Apparent temperature based on the air temperature and wind speed,    when lower than the air temperature.
| windspeed       | m/s, km/h, mph | float | 
