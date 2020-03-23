=============
Wire Protocol
=============

This document specifies the makeup of a dripline message and how it is expressed as an AMQP message. 


.. _structure:

Structure
=========

A dripline message comprises header information that describes the message and the payload that is the message contents.

Header
------

A subset of the header fields are specified as AMQP message properties.

======================== ======= ============ ===========================================
Field                    Type    Message Type Values
======================== ======= ============ ===========================================
``content-encoding``     string  All          ``application/json``
``correlation-id``       string  All          UUID for request/reply correlation
``reply-to``             string  Requests     Routing key for replies
``message-id``           string  All          UUID or UUID/[chunk number]/[total chunks]
======================== ======= ============ ===========================================

The remaining fields are specified as `headers` in the AMQP message properties.

============================ ======= ============ ===========================================
Field                        Type    Message Type Values
============================ ======= ============ ===========================================
``message_type``             integer All          Reply (2), Request (3), Alert (4)
``message_operation``        integer Requests     Set (0), Get (1), Command (9)
``specifier``                string               Provides additional information about how the consumer should process the message
``timestamp``                string  All          Following the `RFC3339 <https://www.ietf.org/rfc/rfc3339.txt>`_ format (example: `2017-12-31T15:00:00.000Z`) with sub-second precision
``lockout_key``              string  Requests     16 hexidecimal digits (see :ref:`lockout`)
``sender_info.exe``          string  All          Full path of the executable used to send the message
``sender_info.hostname``     string  All          Name of the host computer that sends the message
``sender_info.username``     string  All          User responsible for sending the message
``sender_info.service_name`` string  All          Name of the service that sent the message
``sender_info.versions``     [below] All          Package version information for the components of the sender
``return_code``              integer Replies      Machine-interpretable status; see :ref:`return_codes`
``return_message``           string  Replies      Human-readable explanation of the return code
============================ ======= ============ ===========================================

The ``sender_info.versions`` field consists of the package version information for the components 
of the sending software.  It is a sequence of the following pieces of information for each component:

============ ======= ==================================================
Field        Type    Values
============ ======= ==================================================
``version``  string  Software version for this component of the sender
``package``  string  Software package name for this component of the sender
``commit``   string  Git commit for this component of the sender
============ ======= ==================================================

Payload
-------

The payload is the content of the message, and it comprises the AMQP message body.  It is optional in all types of dripline messages.

The payload should be formatted in `JSON <http://json.org>`_.  The encoding (``application/json``) should be specified in the ``content_encoding`` field of the AMQP message.


.. _splitmsg:

Split Messages
==============

A dripline message can be split across multiple AMQP messages to accommodate large payloads.  
The AMQP messages that comprise a single dripline message are called "chunks."  
The headers for all chunks comprising a single dripline message are identical, with the exception of ``message-id``.  
The payload, encoded as a JSON string, is split into substrings based on a maximum size, and each substring is assigned as the payload for a single chunk in order of position in the original string.

The ``message-id`` header field is used to properly reconstruct the full message.  
The structure of the ``message-id`` for a split message is ``UUID/[chunk number]/[total chunks]``.  
The ``UUID`` is a single unique identifier for the dripline message, and is duplicated in each chunk.  
The ``chunk number`` describes the position of the chunk in the full message.  
The `total chunks`` is the number of chunks into which the message was divided.

For unsplit messages the ``message-id`` field may either be ``UUID`` or ``UUID/0/1``.


.. _message_types:

Message Types
=============

There are three types of dripline messages:

:Request: messages sent from one node to another in a dripline mesh to perform a specific action.
:Reply: messages sent in reply to a request.
:Alert: messages broadcast from a node without regard to whether other nodes are listening.


.. _message_operation:

Message Operations
==================

Request messages have four possible operations:

:Set: set a value
:Get: get a value
:Command: perform a command

The exact meaning of an operation will depend on the application.  Generally `get` and `set` will get and set a value, and a command will request some application-specific command.


.. _return_codes:

Return Codes
============

The following table lists return codes. It is worth stressing that all codes fall into one of the following categories:

* <0: not defined
* 0: success
* 1-99: warnings (request fulfilled but with some caveat)
* 100-999: dripline error
* >=1000: application errors

Errors are subdivided into categories, with each multiple of 100 representing a category and values falling within that category.
Dripline errors are covered by codes in the 100-999 range.
Additional errors may be specified for a particular application of dripline.  These errors are covered by codes 1000 and above.

======= ===========
Code    Description
======= ===========
0       **Success**
1       **Generic Warning; No Action Taken**
2-99    *Unassigned, Non-Error Warnings*
100     **Generic AMQP Related Error**
101     AMQP Connection Error
102     AMQP Routing Key Error
103-199 *Unallocated AMQP Errors*
200     **Generic Application Error**
201     Application Connection Error
202     Application No Response Error
203-299 *Unallocated Hardware Errors*
300     **Generic Dripline Client Error**
301     No message encoding error
302     Decoding Failed Error
303     Payload Related Error
304     Value Error
305     Timeout
306     Method Not Supported
307     Access Denied
308     Invalid Lockout Key
309     Deprecated Feature
310-399 *Unallocated Dripline errors*
400     **Generic Client Error**
401     Invalid Request
402     Error Handling Reply
403     Unable to Send
404     Timeout
405-499 *Unallocated Client Error*
500-998 *Unallocated*
999     **Unhandled dripline or application error**
======= ===========


.. _amqp_message_use:

AMQP Message Use
================

This section lists how the different parts of an AMQP message are used in the dripline wire protocol.  It duplicates the information above, but referenced in a different way.

======================== ======= ===========================================
AMQP Field               Type    Dripline Use
======================== ======= ===========================================
``content-type``         string  Unused
``content-encoding``     string  ``application/json``
``headers``              table   Other header fields
``delivery-mode``        string  Unused
``priority``             uint8   Unused
``correlation-id``       string  UUID for message correlation
``reply-to``             string  Routing key for reply
``expiration``           string  Unused
``message-id``           string  Message UUID or UUID/[chunk number]/[total chunks]
``timestamp``            uint64  Unused (string timestamp field in headers)
``type``                 string  Unused
``user-id``              string  Unused
``app-id``               string  Unused
``cluster-id``           string  Unused
Body                     string  Payload
======================== ======= ===========================================