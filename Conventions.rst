===========
Conventions
===========

Dripline sets specific requirements on the structure and content of messages, as well as dictating that certain behaviors be supported.
There is, by design, no strict instruction as to how those requirements are implemented and satisfied.
While this provides the flexibility to implement in any platform, it does not help one gain an intuition for how libraries will work.

Within implementation libraries within this organization, we will strive to provide a greater consistency in implementation patterns; these are described here.
This description will hopefully help in learning to use the libraries, and the consistency should make transitions between them go more smoothly.

Navigation
==========

* `User Interface & Configuration <cli-and-config>`_  

  * `Command Line Interface <ui-cli>`_  
  * `Configuration Files <config-files>`_  

* `AMQP Bindings and Consuming Messages <amqp-bindings>`_  


.. _cli-and-config:

User Interface & Configuration
==============================

.. _ui-cli

Command Line Interface (CLI)
++++++++++++++++++++++++++++
Command line options will generally follow the conventions of common Unix/(GNU/Linux)/BSD tools.
You can find more information information in the standards for `POSIX <http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html>`_ and `GNU <https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html>`_.
The following description is adopted from `the scarab library <https://github.com/project8/scarab/blob/develop/documentation/application_building.rst>`_ (which is used by dripline-cpp):

A command will consist of four parts::
  executable [subcommand] [options] [non-option arguments]
  
executable
  the (path to the) executable program itself
subcommand
  Larger commands with multiple modes of operation typically group those under subcommands.
  This must come immediately after the executable itself and may determine what options and non-option arguments are available.
options
  Options are indicated by a hyphen and a single character (e.g. `-c`) or two hyphens and a multi-character name (e.g. `--config`).
  They may be optional or required and may be passed in any order.
  All options must be explicitly defined by the (sub)command.
  Options may be "flags" (which indicate a behavior by their presence alone), 
non-option arguments
  A non-option argument does not start with a hyphen and have more-specific treatment in dripline's applications. The available arguments is not an explicit list but is instead processed during execution. These areguemnts are divided into two subgroups: keyword arguments consist of a key/value pair, separated by a single equals `=` while ordered arguments have no equals. Keyword arguments are collected in a map-like object and accessible by the keys, ordered arguments are collected in an ordered array-like object.

Several options are generically useful and therefore always available:

`-h` or `--help`
  prints usage information for the executable or subcommand (if given), then exits without processing other options
`-V` or `--version`
  prints the version information and then exits
`-c` or `--config`
  (requires a value which is a path to a local file).
  Gives a configuration file describing the desired action.
  Config files are discussed in more detail in the next section.
  The content of the config file are collected in a map-like object and any non-option arguments (see above) provided will be added to it or override values if they already exist.

.. _config-files:

Configuration Files
+++++++++++++++++++
Configuration files are a convenient way to specify configuration information in a saved way, which can be reused and/or version controlled.
It can also be useful simply to save keystrokes for commands which are otherwise long.
In general, the content a dripline config file is expected to translate to a map-like object at the top level, with values equivalent to those which could be passed as non-option arguments.
For dripline-cpp and dripline-python, the supported file formats are `YAML <http://yaml.org>`_ and `JSON <https://www.json.org>`_ (the former being a superset of the latter).

.. _amqp-binding:

AMQP Bindings & Consuming Messages
==================================

Alerts Exchange Messages
++++++++++++++++++++++++

On the alerts exchange, messages may be sent with various routing keys, per the requirements of the alerts exchange.
The message content follows predictable payload formats which allow the data to be sensibly processed and used. Messages with the following first words in their routing key are described in further detail.

sensor_value.\<from\>
  These messages include status information related to a particular endpoint. The payload will have the key `memo` or the key `value_raw` (or may have both). If it has the key `value_raw`, it may also have the key `value_cal`. The expectation is that `memo` is a human-parsible annotation about the state of the endpoint at a particular time. The `value_raw` is the raw data rececieved by the dripline service from the relevant hardware or other service, and the `value_cal` processes the `value_raw` into something sensible. A typical use case would be a temperature sensor, read out as a resistance, but calibrated to units of Kelvins.
  
status_message.\<from\>.\<severity\>
  These messages have two variable routing key terms. The `from` word indicates what service the message is produced by, and `severity` indicates how git of a concern the message is expected to be. Typical values of severity are the set \{notice, alert, critical\}, though others may be defined. The payload is expected to be a human-understandable string. These are typically consumed and printed to a terminal, logging, or messaging service (email, sms, twitter, slack, etc.).
