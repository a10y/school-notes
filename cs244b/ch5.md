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

Impure names better:
  1. Encodes info to facilitate name mapping (e.g. phone number area coe)
  2. Gives the holder of name info without needing to access description
    * Ex: /dev/tty, we know it's a device file by unix conventions.
  3. Facilitates distributed allocation
    * Ex. phone company can allocate new phone numbers based on the subscribers location

Disadvantages:
  * Size: need larger names to encode info
  * Cannot change entity w/o changing name (ex. move, new phone number)

## Structured Names

Flavor of impure name. eg. hierarchical file names

Scalability from hierarchy: full names are made up of several *component names*
  * Components only need to be unique relative to parent components. Think unix paths.
  * **Name Transparency**: can move object without changing name

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

__External names are necessary; internal names are an optimization.__

# Naming in Object Oriented Systems

Naming operations in OO systems include:
  1. Create object with new name/rename existing object
  2. Map name to object
  3. Invalidate a name
    * Note: deleting name usually =/= deleting object

Also: directory operations to list names

# Name Servers: Conventional Thinking

Convential ideas -> name service, a la DNS.
  * Client connects, gives server external name -> gets internal
    * Usually a UUID
    * Requires internal ID to have full proper name semantics
