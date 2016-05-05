Chapter 1: Overview
===================

3 Layers of Distributed Systems
  1. Application Logic
  2. Shared State Layer
  3. Communication Layer

There are varying levels of coupling between (1) & (2) and (2) & (3), based on
  1. High versus low level access to state (ex. NFS blocks vs files)
  2. Strictness of consistency (can handle inconsistencies or no)
  3. Latency requirements (ex. NTP can't handle high latency communication)

## Views of Distributed Systems

### Shared State View

DS shares some state between participating nodes.

**shared state** -> state that is synchronized to the necessary extent by the system

### Object Interface View

DS is a hierarchy of objects with well-defined interfaces that communicate to share state.

Modularity, structuring, etc.

### Protocol View

As long as different nodes in the network communicate over the same protocol, then they can all participate in the
system.
