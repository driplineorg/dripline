======
Design
======

There are several design principles which guide development. The descriptions here should convey our design intent and should guide future design decisions.

REST inspired
=============

The `description on wikipedia <http://en.wikipedia.org/wiki/Representational_state_transfer>`_ does a great job describing the REST standard.
In particular, dripline stresses:

1. **Client-Server Model:** Any number of "services" may be run, each acting as a server. The service can then independently respond to requests from any number of clients. Further, each resource has a specific set of responsibilities, making everything easier to maintain and new capabilities easier to deploy.  
2. **Stateless:** Every interaction is considered isolated and contains enough information to respond to the particular request. No resource should assume un-verified conditions in any other.  
3. **Layered System:** Of particular import for hardware interactions, components expose a uniform Interface without knowing or caring what happens in the background. This is what allows things like uniform interaction with LXI and GPIB instruments, despite the need for a shared bus.  
4. **Uniform Interface:** Regardless of implementing language, hardware, etc. all interactions use the same wire-protocol. This makes it easy to implement new resources in whatever environment is most convenient for that resource.

Forward Compatibility
=======================

While the value of backward compatibility is generally considered self-evident, dripline also strives to achieve forward compatibility. The main way that this is achieved is by requiring that if a message contains unexpected fields, they be ignored if possible. This allows us to add new fields (such as ``sender_info`` or ``lockout_key``) and expect running services fitting the older standard to still respond to requests. Obviously use of those new fields requires an update, but having heterogeneous versions running in parallel allows rolling updates of resources with reduced down time.
