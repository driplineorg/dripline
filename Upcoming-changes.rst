On this page we collect features which have been discussed for upcoming versions of dripline. Once they are approved/implemented they will migrate from this page onto the proper pages in the spec itself. This could be considered similar to the discussion board for an issue tracker, but since this repo is already doing a bit too much, it seemed painful to mix in with technical issues.

In general, it is our goal that the dripline standard be stable, and that new user-space features be implemented within the constraints of the existing specs. Similarly, it is usually desirable to add new features not to the dripline library, but to derived applications (such as dragonfly or hornet) so that the dripline libraries themselves can be stable. Of course that does not always make sense, sometimes the conventions that exist fail to accommodate an arising need, or do so only in an ambiguous/messy way.

Additions to the spec
=====================

- Require that all endpoints respond to any request "quickly" (whatever that means). The expectation should be that one can send a request and get an immediate reply. If a task "takes some time" then it should be started (which can receive a positive reply indicating that the task is starting), polled (further requests with replies that indicate that the task is executing or not. It may be desirable for non-execution to include more details, such as if the most recent execution completed successfully and/or a timestamp of when the last execution completed), and then results retrieved from a cache maintained by the endpoint. It is probably also good if the process can be forcefully aborted in a way that restores a nominal state. _This should be added to the design principles section but currently dragonfly does not comply and fixing that seems non-trivial. It would also be nice to be more explicit about what "quickly" means, since network communication and instrument interactions can take measurable time but are often still "quick"_

Additions/changes to glossary
=============================

Deprecations
============

