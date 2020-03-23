==========
Change Log
==========

This file will contain a more verbose accounting of the change for released versions, or being staged for the next release

Release: Upcoming
=================

Clarifications/Changes
----------------------
* Improved description of return code values and expected treatment.
* Change description of ret_code 300 to Dripline Client Error
* Change description of ret_code 400 to Generic System Resource Error (could be a database, or another system-daemon-type resource we may want to interact with in the future)

Additions
---------
* Messages have an optional field named ``specifier`` of type string.

Deprecations
------------
* Removing definition of DAQ specific-case exceptions (501, 502); these may be defined in the project 8 mesh
