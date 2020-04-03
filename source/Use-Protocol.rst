============
Use Protocol
============

The following sections summarize various conventions that all dripline-compliant implementations should strive to follow.

* amqp-exchanges_
* amqp-queues_
* broadcast-requests_
* lockout_

For information on the AMQP concepts such as exchanges, queues, and routing keys, please consult `this explanation <https://www.rabbitmq.com/tutorials/amqp-concepts.html>`_ of AMQP concepts.  It's useful to recall that in the AMQP model:

- messages are published to a particular exchange, 
- the routing key binds an exchange to a queue and indicates which messages on a particular exchange go to which queue,
- and messages are consumed from the queue.

Services and asynchronous child endpoints consume messages from queues; in the former case the service distributes messages to its synchronous child endpoints according to the routing key.


.. _amqp-exchanges:

AMQP Exchanges
==============

All dripline messages are sent using one of two exchanges, ``alerts`` and ``requests``. They differ both in the types of messages they handle and the types of bindings made.

Alerts
------

The alerts exchange is used for one directional information transfer, always in the form of an alert message. These messages are not delivered to any particular consumer, and may well be of interest to zero, one, or many consumers. All messages have a routing key which begins with an indication of the type of content, and may have further details. See :ref:`binding coventions<amqp-binding> for examples of acceptable uses of the alerts exchange.

Requests
--------

The requests exchange is used for round-trip information transfers in the form of request messages and their resulting reply messages. Routing keys for requests begin with the name of the endpoint being targeted (or broadcast, for that :ref:`special case described below<broadcast-requests>`), and may include a :ref:`specifier<structure>` to indicate a particular command (for a command) or attribute (for a get or set that configures an endpoint rather than assigning or querying the endpoint itself). The reply is then sent to the routing key specified in the header properties provided along with the original request.

Properties
----------

AMQP exchanges can be configured in a number of ways, and in the official dripline implementations we use these properties:

* Type: ``EXCHANGE_TYPE_DIRECT``
* Passive: ``false``
* Durable: ``false``
* Auto Delete: ``false``


.. _amqp-queues:

AMQP Queues
===========

Within a dripline mesh each message consumer, either a service or an asynchronous endpoint, has its own queue from which it consumes messages.

Properties
----------

AMQP queues can be configured in a number of ways, and in the official dripline implementations we use these properties:

* Passive: ``false``
* Durable: ``false``
* Exclusive: ``true``
* Auto Delete: ``true``


.. _broadcast-requests:

Broadcast Requests
==================

In addition to binding the name of the endpoint (or endpoints) that compose a service, all services shall also bind the ``broadcast.#`` routing key on the requests exchange. All services must respond to broadcast requests, which is consistent with the above rules of the requests exchange, and they must not generate additional errors if thereply is undeliverable. Since every running service is expected to respond, and there is no guaranteed order or timing, any process sending such a request should deal with this properly (ie, accept multiple replies, wait "long enough" for replies to come, cope with inconsistent reply order) if it cares. The following subsections cover the commands that must be supported.

Note that it is only ever appropriate to use broadcasts when a request is expected to apply to every running service regardless of what that set includes. It is not acceptable to use this to as a shorthand for interacting with a known subset of endpoints (in the hopes that all others will ignore it). In that use-case, one should create a single endpoint that applies the desired logic to contact the intended set of endpoints.

The following commands describe behaviors as part of the protocol. Because they are (or at least may) be sent as a broadcast, any implementation is expected to implement them so that the entire mesh behaves in a consistent and predictable way.

lock
----
See :ref:`below <lockout>`. Note that a service receiving this will attempt to lock all of its endpoints.

unlock
------
See :ref:`below <lockout>`. Note that a service receiving this will attempt to unlock all of its endpoints.

ping
----
No arguments. Send a Reply message with empty payload. This is meant as a useful means of discovering the full set of running/responsive services. It may not be used to trigger any other behavior.

set_condition
-------------
Single integer argument. Any unexpected value should result in return code 304 (Value Error). A particular dripline deployment can define a set of conditions as needed. It is encouraged to use large values with reasonable spacing, a la HTML or dripline error values, to facilitate intermediate values being defined later. 

It is important to note that ``set_condition`` is a bit of a panic button, the order in which services receive/respond to set_condition is not well defined and every service is expected to respond immediately (without trying to coordinate with other services). It is designed to support notions such as "abort data taking" or "danger! make everything as safe as possible" and is not suited to situations where coordination is desired or when one wants to carefully check that each service succeeded in getting to the desired state before taking further action.


.. _lockout:

Lockout
=======

An endpoint may implement a lockout system to restrict access for certain types of request messages.  The lockout is intended to avoid stupid things happening (i.e. multiple people sending commands to the same service), and not as a security feature.  When a service is "locked," lockable messages will only be accepted if they have the right key in the ``lockout_key`` field of the message header.  Endpoints not implementing a lockout system will ignore the ``lockout_key`` field entirely.  However, in the official implementation all endpoints include lockout capabilities.

The following request types are lockable:

- set
- command*

\* Unlock is a command but can bypass the lock with a force argument (see below). Broadcast commands ``ping`` and ``set_condition`` ignore lockout.

Keys
----

The lockout key is 16-bytes long. When represented as a string, it will be formatted as 16 hexidecimal characters, in one of these ways:

- ``0123456789abcdef0123456789abcdef``
- ``01234567-89ab-cdef-0123456789abcdef``

Rules
-----

A lockout system follows the following rules:

- Enabling the lock

  - The lock is enabled with a command request and a ``lock`` instruction.
  - The key can be provided by the request, in which case it should be given as a properly formatted key in the ``lockout_key`` field.  Improperly formatted keys (that are non-empty strings) will result in an error (code 308).
  - If the key is not provided (i.e. the ``lockout_key`` field is an empty string), the key will be generated by the endpoint.
  - If an endpoint was unlocked, and the lock was successfully enabled, a success code 0 will be returned, and the key (whether provided or generated) will be returned in the ``lockout-key`` field of the payload of the reply.
  - If the endpoint was already locked, an error code 307 will be returned.

- Using the lock

  - If an endpoint is locked, any lockable request must have the valid key in the ``lockout_key`` field to be processed.
  - If an endpoint is not locked (or does not implement any lockout functionality), the ``lockout_key`` field will be ignored.
  - When using the key provided in a request, if the key is improperly formatted, an error code 308 will be returned; if the key does not match the endpoint's lockout key, an error code 307 will be returned.

- Disabling the lock

  - The lock is disabled with a command request and an ``unlock`` instruction.
  - The rules for "Using the lock" above apply.
  - If an endpoint is not locked, a warning code 1 will be returned.
  - if the endpoint was locked, and was successfully unlocked, success code 0 will be returned.
  - The lock may be forced to disable by providing the field ``force: true` in the payload of the request. The value of the field should be a boolean.  This exception is intended to allow access to services to be regained in the event that the lockout key is lost; as mentioned above, the lockout is intended to avoid stupid mistakes, rather than as a true security feature.
