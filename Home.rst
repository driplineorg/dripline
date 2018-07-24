Current Version: 2.1.0
======================


Summary
=======

Dripline is a mostly `REST <https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm>`_-inspired framework for control systems developed by `Project 8 <http://www.project8.org>`_.
It comprises two components:  

1. A *Wire Protocol* that defines the structure of valid messages
2. A *Use Protocol* which includes reserved keywords, values for constants, and behaviors for responding to messages.

The current version of the framework and the version history are listed on the *Version History* page, while planned future developments are listed on the *Upcoming-Changes* page.  The summary of the *Design Principles* is available for reference, and should be especially useful for those wishing to contribute.

Libraries
=========

It is possible to implement dripline in whatever language/environment is most convenient (which is part of the value). We currently have implementations available for `C++ <https://github.com/project8/dripline-cpp>`_, `golang <https://github.com/project8/dripline-go>`_, and for `python <https://github.com/project8/dripline-python>`_.  The libraries are not all equivalent, and functionality can vary between languages.
