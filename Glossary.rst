The following terms are often used intuitively, but are given the following definitions in the context of the dripline standard. This is done to facilitate precise communication.

endpoint
    Any object to which a T_REQUEST may be sent and/or from which a T_ALERT may be sent. An endpoint has a name and its service will always be bound against that name with any or no routing key specifier (ie `<name>.#`).

mesh
    General term which refers to a particular AMQP broker and all of the services bound to it.

message
    An object conforming to the [[wire protocol|wire-protocol]]

routing key specifier
    (frequently abbreviated RKS)
    All content of an AMQP routing key following the first period. Generally, in dripline, the first term in a routing key is used to determine if it is bound to a particular queue, and the specifier is used by the service or endpoint to help determine how to process the message.

routing key target
    All characters in a routing key prior to the first period. Complement of the routing key specifier. Within implementations of dripline, this is the component commonly used with an exact match when binding a queue, with the RKS generally being wildcarded.

service
    A continuously running program which is part of a dripline mesh. A service is itself an endpoint since, for example, it must respond to a ping, but may contain any number of additional endpoints which are not services. _(is this too specific?) The service is responsible for establishing queues and bindings and for processing messages which are ultimately delivered to and fulfilled by those endpoints which are part of the service.