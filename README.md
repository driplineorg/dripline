#**dripline**
#####*Slow controls for medium scale physics experiments.*


Fundamentally, dripline defines a messaging wireprotocol, as well as conventions for using that protocol including standard behavoirs and reserved names/values.
Within this Repo there are subdirectories for implementing libraries in various languages. Any actual slow control system or component would be developed based on those.


Dripline is developed and tested primarily using RabbitMQ 3.3.x on Debian 8.1

###Implementations

Python: [dripline-python](https://github.com/project8/dripline-python)    
Go: [dripline-go](https://github.com/project8/dripline-go)    
C++: [dripline-cpp](https://github.com/project8/dripline-cpp)    


###Getting help
Code documentation is produced using sphinx via reStructuredText which is distributed with the source. A [rendered](http://www.project8.org/dripline) version is hosted using github pages and the project8.org domain.
Specific guidance for the Project 8 prototype system, as running, can be found on the [hardware wiki](http://github.com/project8/hardware/wiki/Slow_Controls_home) (login required).

