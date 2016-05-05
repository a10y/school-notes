Chapter 3: Distributed Shared Memory
====================================

Shared Memory -> `mmap`ed file.

Distributed Shared Memory -> consistent memory across physical nodes

## Consistency

Total ordering of updates consistent with the Happens-before ordering seen at each node

**Happens Before Relation**: Any write of value V to a variable must occur before reading V from the variable. Partial ordering, observable by a single process.

Caching of writes can lead to inconsistencies

## Basic DSM Consistency Protocols

Discuss consistency in terms of a shared object
  * network files
  * shared memory page

Readers/Writers protocol
  * <= 1 writable copy of an object among the nodes
  * OR >= 0 readable copies

State transition diagram, with the following states
  * Invalid
  * WaitForExcl
  * WaitForShared
  * Excl
  * Shared
  * SharedWaitExcl

Last exclusive owner grants read access to each reader, keeps them in a *copy set* for the page

## DSM vs SMMP

Shared Memory multiprocessors, similar to DSMs in that they share memory across multiple physical cores

Differences
  1. DSM shares a set of pages, SMMP shares all memory
    * 1 node failure in DSM is recoverable, in SMMP brings whole system down
  2. DSM deals with out-of-order delivery, SMMP relies on bus to serialize updates
  3. DSM has unreliable broadcast, SMMP is reliable (bus)

## DSM Performance

Higher consistency cost than SMMP
  * DSM transmits pages across network, SMMP moves cache lines (64 bytes on Intel)
  * Due to RTT for sending page, contention will make the DSM unusably slow

**Example:** Variables A and B both stored in a page. Node1 writes A, Node2 writes B, Node1 reads A again.
  How many packets were sent? 2 to ask for excl/get ack, 2 again, 2 again to request shared access to read

Under contention, DSM can be 10,000x slower than SMMP

**False sharing:** Two different logical units map to the same physical unit

Possible "solutions:"
  * Careful management of sharing
  * Align data to page boundaries to prevent the above "false sharing" -> fragmentation

Producer/Consumer has terrible perf on DSM, thrashing between Excl/Shared state

Implement DSM with finer-grained units of consistency, remove contention while keeping same consistency, or both

## Fine Grain Consistency

3 units in cache:
  1. Consistency Unit: unit of update/ownership
  2. Mapping Unit: Size of the thing that is stored in cache slot
  3. Transfer Unit: Amount moved into/out of cache on miss/replacement

Using cache line as unit of consistency over the page reduces false sharing (but only slightly, no?)
  * Also reduces the transfer time to send cache lines over the network as opposed to pages

Can adjust the consistency granularity for different regions of memory, as tracked by the DSM system

## Fault Tolerance + Tight Coupling

Difficult to handle failures: Imagine node dies while holding Excl access to page. What to do?
  - Hang indefinitely -> bad
  - Pre-empt the owner, sending them a failure signal -> better, complicated. How do you understand what the failing
    address points to?
  - If the owner of a page dies before anyone else has cached it, the page is gone.

## Transactional Memory

Three fatal flaws:
  1. Aborts on read/write conflicts so long-running reads will fail unless writes suspended
  2. Likelihood of aborting the transaction increases as the running time of txn increases
  3. Hardware limitations -> a txn may *never* commit.

## Munin + Treadmarks

**Release Consistency**: Synchronize on release.

**Merge Consistency**: Multiple concurrent writers -> merge into single final write

## Spectrum of State Sharing

### Direct Communication

A source of updates directly communicates updates to interested receivers directly (TCP, multicast, etc.)

Issues:
  1. Sender must be aware (i.e. keep track of) all other members of the network who wish to receive updates
  2. Sender/receiver must co-exist at same time (retransmissions, what if one goes down, etc.)
  3. Uncertain how frequently to dispatch updates, or who wants which updates, etc.

This is "Push" model. Point (3) can be fixed by adding a pull model, where you subscribe to updates you want

### Consistent Shared Memory

Consolidate into a Shared State server, which sends updates and pushes to nodes in reply to their requests to pull.

Now, central server is responsible for keeping up-to-date copy of each datum.

However, caching needed to get competitive performance, so now you need consistency proto to keep caches in sync with
the central shared mem.

### Application Specific Shared Memory

Relaxed consistency constraints for sharing of data

Case Study: Realtime Video
--------------------------

Push-based model, where the server continuously sends the stream

Client proxy object has a `playbackBuffer` attribute that is read by the display class, abstracts away the network
communication details.
