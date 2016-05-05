# Clocks and Timing in Distributed Systems

3 Kinds of clocks:
  * Real Clocks: Advances in sync with physical time, ex. in response to hardware ticks
  * Virtual Clocks: Track the timing of events in a virtual world (ex. packet transmission in network simulation)
  * Logical Clocks: Tick with passing events in the real world

3 Key properties of distributed real-time clocks:
  1. Consistency: different nodes agree within some delta on the time
  2. Accuracy: All the nodes must believe that the time is within some epsilon of the real time
  3. Predictable local access time: Required to make sure that you can always have reasonably sync'ed view of time

## Synchronized Real Time Clocks

**Synchronized time**: useful for reconstructing order of events in a distributed system

Use of consistent shared memory (DSM) will lead to clocks sending numerous network requests, incurring a "ping-ponging"
access pattern that eats up tons of RTTs. Only works for very low-res clocks

Similarly, can't keep time synchronized with normal RPC for same reasons (RTTs)

**Good Solution**:
  * Stateful proxy works for this, asynchronously pushing updates to other nodes
  * Local extrapolation of time in between updates
