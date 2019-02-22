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
``specifier``            string           Provides additional information about how the consumer should process the message
``timestamp``            string  All      Following the `RFC3339 <https://www.ietf.org/rfc/rfc3339.txt>`_ format (example: `2017-12-31T15:00:00.000Z`) with sub-second precision
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

The following table lists return codes. It is worth stressing that all codes fall into one of the following categories:

* <0: not defined
* 0: success
* 1-99: warnings (request fulfilled but with some caveat)
* >=100: error

The errors are further subdivided, with each multiple of 100 naming a category and other values falling within that category.
Any value not specified here may be defined within an individual mesh (client libraries are expected to be able to handle new retcode values, though obviously they may not know the description for them).
Users are encouraged to define a new category if needed and to either leave a gap within the category or start from the upper end (from x99) to avoid conflicts with values which may be defined in future dripline versions.


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
300     Generic Dripline Client Error
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
400     Generic System Resource Error
500     Generic DAQ Error
501-998 Unallocated
999     Unhandled core-language or dependency exceptions
======= ===========

