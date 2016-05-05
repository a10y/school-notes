# The Design and Implementation of a Log-Structured File System

## Intro + Impetus

There are 3 key components of file system performance
  * CPU
  * Disk speed
  * Memory size

CPU and memory have been growing exponentially, disk speed not nearly as quickly.

Buffer caches -> Main-memory write buffers, can serve reads as well, then get flushed back to disk at some interval. Due
to buffering, FS must be optimized for writing (as reads will be served from memory most often)

LFS -> Keep a log of writes in main memory, serve reads from the buffer cache. Larger your main memory, more reads can
go through the cache.

## Downsides of Current File Systems

Small-file operations (kilobytes) are common in scientific and office applications, and Unix FFS is bad for this:

Example: 5 disk IO's needed to update a single small file:
  1. Write to file data
  2. Write file attributes
  3. Write directory entry
  4. Write directory attributes

Block reads/writes in FFS are asynchronous, but *metadata* read/write is synchronized
  * Writing many small files, same directory -> large contention for the directory lock

# Sprite LFS: The Log-Structured File System

LFS has the same structures as the Unix FFS, including inodes, direct/indirect blocks, etc.

**Inode Map**: Inodes are note stored in a fixed location in LFS
