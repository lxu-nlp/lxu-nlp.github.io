---
layout: post
title: "System (1): Basic Concept Review (POSIX, File System)"
date: 2019-11-12
categories: [System]
tags: [system]
math: true
---

This article is part of my review notes of "Unix System Programming" course. In this post we go over the basic concepts in system, including OS, file systems, etc.

## Operation System (OS)

OS is the bridge between physical hardware and the software/processes, managing resources (CPU, memory, storage, I/O) in an **efficient** and **convenient** manner.
* Efficient: OS encapsulates efficient operations to control resources. E.g. OS determines a good strategy to handle disk movement to achieve high throughput of read/write requests.
* Convenient: OS provides a handful of system APIs for users to interact with the system.

## POSIX

There are many popular operating systems, including Windows, MacOS, Android, ChromeOS... If each OS has its own unique standard and API, it would be hard to distribute the same software to multiple OS platforms. Therefore, we need a universal standard that works across multiple OS, which is called POSIX.

POSIX defines the API that are compatible across multiple OS, including MacOS, Solaris, etc. Other OS also largely complies with POSIX, including Linux, Android, etc.

## File System

File system is one of the fundamental concepts in OS. It needs to efficiently manage any numbers of files, supporting a series of operations including read and write.

Many OS has its own file system. Some common file systems are: FAT, EXT, NT, HFS, UFS... In this series, we focus on the Unix File System (**UFS**), which is supported by many Unix and Linux operating systems.

##### Disk

In one disk partition, the blocks are conceptually divided into multiple parts. The beginning is the boot block, then super blocks (contain parameters such as how many inodes), then inode blocks, then data blocks.

### Inode

In UFS, each file is uniquely identified by an inode number, which is an index of an array of inodes that are kept at the inode blocks in the disk partition. Each inode contains the meta-information about its associated file, including: file type, link count, user/group, permissions, timestamps, data block list, file size. In particular, the data block list contains the pointers of blocks that store the actual data of this file.

### Block List

The block list in an inode is a fixed size byte array: 15 bytes.
* First 12 bytes: pointers to direct data block
* Last 3 bytes: pointers to indirect data block, with 1, 2, 3-level recursion

Therefore, if the file is read sequentially, we don't have much overhead. However, if the file is large (with many indirect blocks), and the read is not sequential, the overhead can be quite huge because of the indirect blocks. Actually, most database systems (with many random access operations) work on the device/disk level directly, instead of using OS's file system.

### File Type

There are multiple file types in Unix serving different purposes.

##### Regular File

A regular file is conceptually a linear byte array of data. Each read/write operation takes a file offset to indicate the current position. Notice that write operation can only happen at the end; it is not allowed to write or delete in the middle of the file.

##### Directory

A directory conceptually contains many [name/(inode number)] pairs in its data blocks. Therefore, we can now identify any file by name, instead of only inode number. When we access any file by its path or name, the underlying OS looks up the inode number of this file name in the directory (can be recursive), and access by the inode number.

##### Link

We can have more than one names for the same file. Two files can have two different paths but with the same inode number. The command to create a link is:

```ln {old_path} {new_path}```

where it creates a directory entry with name {new_path} and the same inode number as {old_path}. The link count increments by one in the inode.

When we delete one of the files using `rm` command, the link count decrements by one in the inode. When the link count is $0$, and no programs are using it, the actual file will be removed by deallocating inode. When we lose the inode, we lose the way to refer to the data block, so the file is conceptually removed.

##### Symbolic Link

Inode numbers are only meaningful inside one system. What if we want a link to work across systems? Symbolic link here to save the world. The data block of symbolic link file contains the file path to be linked, without using inode numbers. Notice that to get to the linked file from symbolic link, it first needs to look up its own data block and get the linked file path, then look up the linked file path to get to its inode. Therefore, symbolic link requires more read than link.

##### Special File Types

There are two special device file types that serve as connection between programs and hardware. The inode of a special file doesn't contain data block list; instead, it contains device driver number and instance number.

##### Block device:

* Modeled like a disk
* Transfer in fixed size blocks; use buffer to speed up I/O
* Can be shared

##### Character device:

* Modeled like a terminal
* Transfer in variable size, any bytes; without buffer
* Data transferred directly from/to device to/from process
* Cannot be shared

A device can have both block device file and character device file.

##### Others

There are also some other file types, such as FIFO. We will look into them later.

### Multiple File System

We can use `mount` command to connect to an external file system into our current file system.

