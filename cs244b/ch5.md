Directories and Naming
----------------------

In DS, want some idea of system-wide naming
  * Necessary in a distributed environment

Why do we have names vs. eg. descriptions?
  * Names more correct
    * What if the description/properties change?
    * Proper names should never change
  * Efficient
    * Shorter than descriptions
  * Flexible
    * Extra level of indirection allows you to change underlying entity later
      but preserve name
    * Ex: Name = "Phil", descr = "Man with green hat". What happens if Phil takes
          off his hat?

What is a name?
  * Can you have names for things that don't exist?
    * Undecidable to tell if something exists, so we just say yes
  * It's a "binding"
    * tuple(x,y) -> "y is definition of x"
    * Context -> collection of binding tuples
      * can check if names are invalid, unique, etc.
      * aliases -> secondary names
      * Usually a "name mapping": name -> object

## Proper Names vs. Common Names vs. Descriptions

Proper Name: Denotes something but lacks connotation
  * eg. human given names: "David" or "Andrew" doesn't say anything about the person
  * Usually requires a "baptism" step to give specific thing a name
  * Example: declared variable name in programming languages

Descriptions: List of attributes/properties
  * Identifies all objects matching these properties
    * "Definite Descriptions" -> identify only one object (basically proper name)
  * Usually implicitly assigned, no "baptism" needed

Common Names: Reference to a description
  * eg. "four legged herbivore" -> "cow" is the common name for this descr.
  * basically a dictionary
  * Example: names of types in programming languages
  * Unlike proper names:
    * Synonymy - multiple names for same obj
    * ambiguous -

# Pure and Impure Names

Pure names: encodes no info about the underlying object.

Impure names are opposite: ex. telephone numbers -> location

Impure names often better:
  1. Encodes info to facilitate name mapping (e.g. phone number area coe)
  2. Gives the holder of name info without needing to access description
    * Ex: /dev/tty, we know it's a device file by unix conventions.
  3. Facilitates distributed allocation
    * Ex. phone company can allocate new phone numbers based on the subscribers location

Disadvantages:
  * Size: need larger names to encode info
  * Cannot change entity w/o changing name (ex. move, new phone number)

There is a spectrum between __pure names__ and __pure descriptions__

## Structured Names

Flavor of impure name. eg. hierarchical file names

Scalability from hierarchy: full names are made up of several *component names*
  * Components only need to be unique relative to parent components. Think unix paths.
  * **Name Transparency**: can move object without changing name
  * **Location Transparency**: can use the same name in multiple locations with
    same meaning

## Problems with UUIDs

  1. Too large (128-256 bits to actually be universally unique)
  2. Clients must go through a Name -> UUID server to get UUIDs. Adds overhead/security 
     burden (on trusting the server)
  3. Assigning UUIDs to existing objects is not an easy problem to solve
  4. Hard to determine when an object gets a new UUID

## Proper use of Internal Names

* External names required, internal names simply an optimization
* Internal names must be kept short
  * Can be accomplished by keeping local to the communication channel where used
  * Then internal names are only usable Client <-> Server, not Client <-> Client
    * But latter case less common anyway
  * Make the names transient
* Ideally, just have per-connection internal names, ensure no wrapping

# External Naming

* Since internal naming is local to connections, biggest choice for system is
  external naming sceme.

Requirements:
  1. Stable storage & management of user-assigned names
  2. Preventing duplicates
  3. Uniform global namespace
  4. Efficient map & query
  5. Locality support (i.e. current working directory)
  6. User-customized aliases

All objects in one large namespace, otherwise interop is hard

## Centralized vs Decentralized Naming Authority

**Centralized**: One authority to consult for name mapping
  * e.g. DNS
  * More efficient in comm. cost
  * Difficult to extend/scale

**Decentralized**: Broadcast a query, get multiple replies, pick one
  * Inefficient broadcasting cost, but easy to scale

**Best Solution**: Combination. Cache info, make it so only has to access 1 name server in
common case, resort to multiple broadcast on lookup failure.

## Global Directory Architecture

Basic design:
  * Hierarchical dirs, replicated for availability + perf

# External Names vs. Internal Names

External Names: Chosen by the user, gives human meaningful names to things
  * e.g. hostname, file names, etc.
  * Reasons/how they happen:
    1. Referring to objects from interfaces (CLI, GUI, etc.)
    2. Meaningful error reporting
    3. Address-space indep. identifier. Passing objs between machines
  * W/o system-wide support for external names, get **ad-hoc names**

Internal Names: More compact and machine efficient
  * e.g. MAC/IP address, inode numbers, fhandle for NFS, etc.

# Name Servers: Conventional Thinking

Convential ideas -> name service, a la DNS.
  * Client connects, gives server external name -> gets internal
    * Usually a UUID
    * Requires internal ID to have full proper name semantics

## Names vs Types

Name: an object's __proper name__.

Type: an object's __common name__. Class descriptor.

Info often provided by __directory system__:
  * representation
  * protocols to access object
  * operations supported by objects of different types

# Naming in Object Oriented Systems

Naming operations in OO systems include:
  1. Create object with new name/rename existing object
  2. Map name to object
  3. Invalidate a name
    * Note: deleting name usually =/= deleting object

Also: directory operations to list names


# Location-Based Naming and Identification

Use of physical location of entity for identification/authenticity
  * e.g. check if the CEO's executive dashboard is only being accessed by him in his office.

Seems silly, but better than secrecy.
  1. Secrets generally single point of failure. Discovery of key is game over.
  2. Proving something is secret is impossible, only know when you've been broken.
  3. No way to tell when the system has been compromised. Attackers can own the system for months without you knowing.
