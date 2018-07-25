# **dripline**
##### *Slow controls for medium scale physics experiments.*


Fundamentally, dripline defines a messaging wireprotocol, as well as conventions for using that protocol including standard behavoirs and reserved names/values.
This repo serves only to document the standard itself, with specific implementations in language-specific repositories within the driplineorg Github organization.

Dripline is developed and tested primarily using RabbitMQ 3.3.x on Debian 9

### Implementations
Implementations of the dripline standard are divided into two categories, primary and other.
Primary implementations should be feature complete with the most recent version of the dripline standard.
These provide some basic functionality (such as a CLI client), core classes to implement the standard, and semi-generic implementation classes which provide examples or generally useful patterns.
The expectation is that these may be sufficient for getting started, or may serve as dependency libraries when building more sophisticated projects.
Available primary implementations are:

* Python: [dripline-python](https://github.com/project8/dripline-python)
* C++: [dripline-cpp](https://github.com/project8/dripline-cpp)

Other implementations are either not actively kept up to the lastest standard, or are incomplete implementations.
These repos are available for reference, but do not have a primary developer with the time and/or inclination to keep maintained at the level of a primary implementation.
The other known implementations are:

* Go: [dripline-go](https://github.com/project8/dripline-go)
* LabView: [dripline-labview](https://github.com/project8/dripline-labview)

### Getting help
Code documentation is produced using sphinx via reStructuredText which is distributed with the source. A [rendered](http://www.project8.org/dripline) version is hosted using github pages and the project8.org domain.
Specific guidance for the Project 8 prototype system, as running, can be found on the [hardware wiki](http://github.com/project8/hardware/wiki/Slow_Controls_home) (login required).

