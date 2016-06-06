# Transactions, Agreement and Reconciliation

Atomic transaction: either __commit__ or __abort__
  * Useful for distributed sys, where all nodes can either be forced to succeed
    or fail together

### Advantages of Transactions

  * Easy failure semantics -> consistency easier to maintain
  * Simplifies concurrency control: single point of locking/unlocking
    * Resolve deadlock by aborting conflicting txns
  * **Batching** of lock release and flushing of log records

## Commit Protocols

### 1-phase

  * Coordinator tracks when transaction is finished
  * Sends out a decision to all nodes
  * Nodes send ACK back to coordinator, returns after all ACKs received

  Problems:
    * Transparent to failures: does not maintain consistency if >=1 nodes crash
    * Coordinator may crash after sending decision

### 2-phase

  * Coordinator sends P2C (prepare to commit)
  * Nodes say YES/NO
  * If any replied NO, coordinator sends ABORT
  * Else, coordinator sends COMMIT

Safe if node crashes after sending YES
  * Requires a complicated recovery protocol

  Problems:
    * Blocking: if coord crashes after P2C before sending COMMIT/ABORT, all nodes hang
      * Solution: non-blocking commit protocol

### Non-blocking Commit Protocol

Solution: add extra phase (3-phase commit). Now protocol is:
  * Coordinator sends P2C
  * Nodes send votes back
  * Coordinator disseminates P2C result
  * Coordinator then sends COMMIT or ABORT

Now, coordinator can return after receiving ACKs from the dissemination, can fire off
the COMMIT/ABORT and quit without awaiting results. Nodes can then do the commit
asynchronously.

### Byzantine Commit Protocols

Assume that some subset of coordinator, participants is malicious, i.e. "lies"
  * Digital signatures on messages for authenticity
  * Requires O(n^2)
  * Impossible to figure out how many "malicious" nodes there are in reality
    * Though in theory you can by heuristics?

### OOTP

Distributed txn protocol, using txns as distributed objects

Steps:
  1. txn created, given unique ID
  2. Operations performed, txn records all participants
  3. Transaction completes, txn object destructs and either commits or aborts
