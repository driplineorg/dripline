Encoding
========

The AMQP message body should be formatted in `JSON <http://json.org>`_.  The encoding (``application/json``) should be specified in the ``content_encoding`` field of the AMQP message.

Structure
=========

======================== ======= ======== ===========================================
Field                    Type    Required Values
======================== ======= ======== ===========================================
``msgtype``              integer All      Reply (2), Request (3), Alert (4)
``msgop``                integer Requests Set (0), Get (1), Send (7), Run (8), Command (9)
``timestamp``            string  All      Following the `RFC3339 <https://www.ietf.org/rfc/rfc3339.txt>`_ format
``lockout_key``          string  Requests 16 hexidecimal digits (see :ref:`lockout`)
``sender_info.package``  string  All      Software package used to send the message
``sender_info.exe``      string  All      Full path of the executable used to send the message
``sender_info.version``  string  All      Sender package version
``sender_info.commit``   string  All      Git commit of the sender package
``sender_info.hostname`` string  All      Name of the host computer that sends the message
``sender_info.username`` string  All      User responsible for sending the message
``payload``              any              The content of the message
``retcode``              integer Reply    Machine-interpretable status; see :ref:`retcodes`
``return_msg``           string  Reply    Human-readable explanation of the return code
======================== ======= ======== ===========================================

.. _retcodes:

Return Codes
============

The following table is direct from the current error codes as defined in dripline. I think I'd like to make some changes, but I haven't thought them through very much. Just brainstorming, I'm thinking 0-99 for success and warnings; 100-199 for errors related to AMQP (ex, errors caught from library); 200-299 for hardware related errors (interaction problems, driver exceptions, etc.); 300-399 shouldn't be 'dripline' but 'consumer', the several encoding/decoding related exceptions could probably be combined. I should discuss this with Noah w.r.t. hornet and mantis.

======= ===========
Code    Description
======= ===========
0       Success
1       No action taken warning
2-99    Unassigned, non-error warnings
100     Generic AMQP Related Error
101     AMQP Connection Error
102     AMQP Routing Key Error
103-199 Unallocated AMQP Errors
200     Generic Hardware Related Error
201     Hardware Connection Error
202     Hardware no Response Error
203-299 Unallocated Hardware Errors
300     Generic Dripline Error
301     No message encoding error
302     Decoding Failed Error
303     Payload related error
304     Value Error
305     Timeout
306     Method not supported
307     Access denied
308     Invalid key
309     Deprecated Feature
310-399 Unallocated Dripline errors
400     Generic Database Error
500     Generic DAQ Error
501     DAQ Not Enabled
502     DAQ Running
503-998 Unallocated
999     Unhandled core-language or dependency exceptions
======= ===========

