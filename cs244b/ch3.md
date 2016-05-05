Chapter 3: Structuring of Distributed Systems
======================================================

Biggest issue with Distributed Shared Memory is that is more expensive to sync all data to every node versus just
sending RPC calls to nodes that already have the data. This chapter highlights different methods of structuring real
distributed systems historically and presently.

Message Passing
----------------

* All nodes in the system send "messages", IPC
* Messaging mechanism is transparent to both local and remote peers

Pros:

  * Works on top of existing hardware (network hardware, etc.)
  * Flexible: asynchronous messaging
  * Synchronization


Socket Programming
------------------

* Can use network connection to local network for IPC, or for remote connections
* Issue: TCP throws away the packet boundaries by providing a reliable bytestream, whereas UDP does not give
  reliability, thus you either need to implement framing over TCP or add ACKs and such to UDP


Distributed File System
-----------------------

* Clients just read and write to distributed files
* This is a bit clunky to do for every task
* Adds overhead to applications that were not already using the file interface
* Different DFSes have different consistency models:
  * NFS updates individual pages on a timeout to check for changes
  * Others detect changes only on file open

OO RPC
------

Very common in latest systems, use an __Interface Definition Language__ to define methods available for RPC.

* Object Reference: Can no longer use pointers to uniquely identify objects in the system as they cross address spaces:
  must use some unique identifier. Cheriton believes you should use a global 32- or 64-bit namespace

Example code for distributed file lookup:

```C++
String name = ...;
// Lookup object in directory by name
Ptr<NamedInterface> o = Directory::root()->interface( name );

// cast pointer to file from raw pointer if actually a file
Ptr<OpenFile> f = dynamic_cast<OpenFile*> o.rawPointer();
if (f == 0)
  // return an error
```

Implemented using __proxy objects__:
  1. You call myObject.method1(), which is a local stub compiled by the IDL
  2. Stub looks at myObject object reference, maps to a network address
  3. Stub takes the parameters, marshals them into a packet and sends the request to the node holding the object

Note the similarity to System Calls:
  * Use of stub methods to arrange parameters for another call
  * Trap handling move parameters to "system stack" (or remote stack in distributed RPC case)
  * Return values passed back, sytem privilege is relinquished

One big issue: Type information for remote objects might not be known by clients.
  * CORBA solves this by having a "type store". Hierarchical structure of modules, IFs, constants, exceptions, etc.

Next issue: Calling methods on the object type
  * Downcast the pointer to the RPC object to the more specific type
    * Issue: smart proxy must also implement the specific interface

Dynamic Invocation is silly, in Cheriton's view, as you should not be getting back objects of types not known about at
compile time. Just make sure that you have a finite set of objects available at compile time when you write the client
code and don't need to mess with Dynamic Invocation BS (type stores, etc.)

Using OORPC to Define Network Services
--------------------------------------
  1. Design the interface to the service
  2. Implement the receiver routines, register them for execution on some directory service
  3. Provide client stubs to applications to access the service remotely

Example: FS on Machine0 exports a set of routines for FS operations. Clients invoke as RPC

Need some service discovery mechanism

OORPC partitions network service implementation into 3 pieces
  1. OORPC implementation: basic call mechanism, transport binding and marshaling/demarshaling (Protobufs, etc.)
  2. Standard Library of Objects and OS facilities
  3. The application

**Biggest Benefits of OORPC**:
  1. Network Transparency
  2. Use of a familiar level of abstraction for most applications programmers
  3. Hides much of the complexity within the stub compilers

Problems with OORPC
--------------------

Performance
===========
Predictable performance is impossible, because some (local) calls take microseconds, others (remote calls) much longer
due to RTT.

Semantics
=========
The at-most-once and at-least-once are very different from the local model and can have unexpected consequences
Can be ameliorated by making the system transactional, giving client access to transaction log. This exacerbates the
performance issue, however.


Smart Proxies
-------------

Viable alternative to direct RPC: allows for caching of data locally to prevent RTT, i.e. "data shipping"

Smart proxies will keep a locally cached copy of the data. This does not work well if you have extremely large data

Stateful Proxy
--------------

Use attribute interface -> Each read is nilpotent, writes are idempotent

Benefits over OO RPC:
  1. Recovery semantics are clear -> after connection loss, proxy and master just resync state
  2. Reads/Writes are always local with async updates -> better performance, easy to reason about
  3. Restrictive set of operations exposed -> less tight coupling over traditional OO RPC, which allows any methods


Lecture
-------

REST: Representational State Transfer
-------------------------------------
1. Uniform references (URLs)
2. Well-defined operations (HTTP verbs)
3. Representation of data
4. Stateless

What does it mean to be RESTful?

* Avoiding idempotent operations
* State-oriented interface

SOAP
----

First pass before REST, standardized. Uses XML

REST is purely a request-response interface...what about notifications?

Cheriton's Thinking: Have interfaces that are state (attribute) oriented. CRUD

  * Have interfaces that only expose attributes, CRUD operations
  * Stateful proxy -> Reads of attributes are local to this process, writes are also local but propagates update
    * Gets rid of round trip times
    * Allow for batching, asynchronous replies and updates to other nodes
    * Reads -> nilpotent -> no effect at all (versus idempotent, no difference between 1 call or N)

Issues here:

  * High memory overhead of creating multiple objects on different servers
