Welcome to Dripline
===================

Slow controls for medium scale physics experiments
--------------------------------------------------

Fundamentally, dripline defines a messaging wireprotocol, as well as conventions for using that protocol including standard behavoirs and reserved names/values.
The design is largely inspired by `REST <https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm>`_ and is originally developed by the `Project 8 Collaboration <https://www.project8.org>`_, though with the move to this Github organization, others are now invited to contribute.
This repo serves only to document the standard itself, with specific implementations in language-specific repositories within the driplineorg Github organization.


Implementations
+++++++++++++++

It is possible to implement dripline in whatever language or environment is most convenient.
There are a number of implementations available here, categorized by how complete and maintained the are.
The following primary implementations should be feature complete with the most recent version of the dripline standard.
These provide some basic functionality (such as a CLI client), core classes to implement the standard, and semi-generic implementation classes which provide examples or generally useful patterns.
The expectation is that these may be sufficient for getting started, or may serve as dependency libraries when building more sophisticated projects.
Available primary implementations are:

* Python: `dripline-python <https://github.com/driplineorg/dripline-python>`_
* C++: `dripline-cpp <https://github.com/driplineorg/dripline-cpp>`_

Other implementations are either not actively kept up to the lastest standard, or are incomplete implementations.
These repos are available for reference, but do not have a primary developer with the time and/or inclination to keep maintained at the level of a primary implementation.
The other known implementations are:

* Go: `dripline-go <https://github.com/project8/dripline-go>`_
* LabView: `dripline-labview <https://github.com/project8/dripline-labview>`_
* Web: `dripline-web <https://github.com/project8/dripline-web>`_


Getting help
++++++++++++

Dripline documentation can be `here <https://driplineorg.github.io>`_.

The `dripline <https://github.com/driplineorg/dripline>`_ repo has an issue tracker for questions about or problems with the dripline standard.  The `dripline-cpp <https://github.com/driplineorg/dripline-cpp>`_ and `dripline-python <https://github.com/driplineorg/dripline-python>`_ repositories each have their own issue trackers for questions about or problems with those implementations.


Other Notes
+++++++++++

Dripline is developed and tested primarily using RabbitMQ 3.3.x on Debian 9 and run in docker containers.
Other workflows are certainly valid, but a managed-container framework is recommended for most cases.


Name
++++

The origin of the name "dripline" is lost to history, but it is *always* written lowercase (except when it's not).
