The following sections summarize various convention which all dripline-compliant implementations should strive to follow.

* amqp-exchanges_
* broadcast-requests_
* lockout_


.. _amqp-exchanges:

AMQP Exchanges
==============
All dripline messages are sent using one of two exchanges, `alerts` and `requests`. They differ both in the types of messages they handle and the types of bindings made.

Alerts
------
The alerts exchange is used for one directional information transfer, always in the form of a T_ALERT message type. These messages are not delivered to any particular consumer, and may well be of interest to zero, one, or many consumers. All messages have a routing key which begins with an indication of the type of content, and may have further details. The accepted forms are as follows:

* sensor_value.\<sensor_name\>: For values to be stored into the slow controls database tables for numeric_data and/or string_data. Currently the payload needs to include "value_raw" or "memo" and optionally "value_cal" to be inserted.  
* status_message.\<slack_channel\>.\<origin_name\>: (where \<\> indicate values which will usually be bound with a '#' and which encode particular information), a system status message from a process which does not communicate directly with slack (or other messaging service) itself. Dripline will provide a slack_relay which posts a message which is "from" <origin_name> to a channel in the project 8 slack named "#<slack_channel>" (note that '#' is reserved in routing keys in AMQP and always the start of a channel name so it is assumed) and with text == the python's str() of the message.payload.

Requests
--------
The requests exchange is used for round-trip information transfers in the form of T_Request messages and their resulting T_Reply messages. Routing keys for requests begin with the name of the endpoint being targeted (or broadcast, for that [special case described below](#broadcast-requests)), and may include a routing key specifier to indicate a particular command (for an OP_CMD) or attribute (for an OP_GET or OP_SET which configures an endpoint rather than assigning or querying the endpoint itself). The T_REPLY is then sent to the routing key specified in the properties provided along with the AMQP message.

.. _broadcast-requests:

Broadcast Requests
==================

In addition to binding against the name of the endpoint (or endpoints) which compose a service, all services shall also bind against the `broadcast.#` routing key. All services must respond to requests sent to matching binding keys (currently only ever an OP_CMD) and must not generate additional errors if the Reply is undeliverable. Since every running service is expected to respond, and there is no guaranteed order or timing, any process sending such a request should deal with this properly (ie, accept multiple replies, wait "long enough" for replies to come, cope with inconsistent reply order) if it cares. The following subsections cover the commands which must be supported.

Note that it is only ever appropriate to use broadcasts when a command is expected to apply to every running service regardless of what that set includes. It is not acceptable to use this to as a shorthand for interacting with a known subset of endpoints (in the hopes that all others will ignore it). In that use-case, one should create a single endpoint which applies the desired logic to contact the intended set of endpoints.

The following commands describe behaviors as part of the protocol. Because they are (or at least may) be sent as a broadcast, any implementation is expected to implement them so that the entire mesh behaves in a consistent and predictable way.

lock
----
See [below](#lockout). Note that a service receiving this will attempt to lock all of its endpoints.

unlock
------
See [below](#lockout). Note that a service receiving this will attempt to unlock all of its endpoints.

ping
----
No arguments. Send a Reply message with empty payload. This is meant as a useful means of discovering the full set of running/responsive services. It may not be used to trigger any other behavior.

set_condition
-------------
Single integer argument. Any unexpected value should result in return code 304 (Value Error). A particular dripline deployment can define a set of conditions as needed. It is encouraged to use large values with reasonable spacing, a la HTML or dripline error values, to facilitate intermediate values being defined later. 

It is important to note that set_condition is a bit of a panic button, the order in which services receive/respond to set_condition is not well defined and every service is expected to respond immediately (without trying to coordinate with other services). It is designed to support notions such as "abort data taking" or "danger! make everything as safe as possible" and is not suited to situations where coordination is desired or when one wants to carefully check that each service succeeded in getting to the desired state before taking further action.


.. _lockout:

Lockout
=======

A service may implement a lockout system to restrict access for certain types of request messages.  The lockout is intended to avoid stupid things happening (i.e. multiple people sending commands to the same service), and not as a security feature.  When a service is "locked," lockable messages will only be accepted if they have the right key in the ``lockout_key`` field of the message header.  Services not implementing a lockout system will ignore the ``lockout_key`` field entirely.

The following request types are lockable:

- OP_SET
- OP_CMD*
- OP_SEND
- OP_RUN

Unlock is a cmd but can bypass the lock with a force argument (see below). Broadcast commands ``ping`` and ``set_condition`` ignore lockout.

Keys
----

The lockout key is 16-bytes long. When represented as a string, it will be formatted as 16 hexidecimal characters, in one of these ways:

- ``0123456789abcdef0123456789abcdef``
- ``01234567-89ab-cdef-0123456789abcdef``

Rules
-----

A lockout system follows the following rules:

- Enabling the lock

  - The lock is enabled with an `OP_CMD` request and a `lock` instruction.
  - The key can be provided by the request, in which case it should be given as a properly formatted key in the `lockout_key` field.  Improperly formatted keys (that are non-empty strings) will result in an error (code 308).
  - If the key is not provided (i.e. the `lockout_key` field is an empty string), the key will be generated by the service.
  - If a service was unlocked, and the lock was successfully enabled, a success code 0 will be returned, and the key (whether provided or generated) will be returned in the `"lockout-key"` field of the payload of the reply.
  - If the service was already locked, an error code 307 will be returned.

- Using the lock

  - If a service is locked, any lockable request must have the valid key in the `lockout_key` field to be processed.
  - If a service is not locked (or does not implement any lockout functionality), the `lockout_key` field will be ignored.
  - When using the key provided in a request, if the key is improperly formatted, an error code 308 will be returned; if the key does not match the service's lockout key, an error code 307 will be returned.

- Disabling the lock

  - The lock is disabled with an `OP_CMD` request and an `unlock` instruction.
  - The rules for "Using the lock" above apply.
  - If a service is not locked, a warning code 1 will be returned.
  - if the service was locked, and was successfully unlocked, success code 0 will be returned.
  - The lock may be forced to disable by providing the field `"force": true` in the payload of the request. The value of the field should be a boolean.  This exception is intended to allow access to services to be regained in the event that the lockout key is lost; as mentioned above, the lockout is intended to avoid stupid mistakes, rather than as a true security feature.
