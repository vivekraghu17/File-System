Table of Contents:
1.	Description 
2.	FS layout 
3.	Features
4.	Installing Dependencies  
5.	Using the FS
6.	 Acknowledgement


Description:
	A filesystem in user space built using FUSE, a software interface for Unix-like computer operating systems that lets non-privileged users create their own file systems without editing kernel code. This project was created as a part of the "Unix System Programming" course at PES University, 2018.
	FUSE or File System in User Space module provides a "bridge" to the actual kernel interfaces by running file system code in user space. Creating a file system can be achieved by writing a Virtual File System which is nothing but an abstraction layer above a more concrete file system to provide custom access to users.

Layout:

	The first thing we’ll need to do is divide the disk into blocks; simple file systems use just one block size, and that’s exactly what we’ll do here. Let’s choose a commonly-used size of 4 KB. Thus, our view of the disk partition where we’re building our file system is simple: a series of blocks, each of size 4 KB. The blocks are addressed from 0 to N − 1, in a partition of size N 4-KB blocks. The region of the disk we use for strong user data is the data region .
the file system has to track information about each file. This information is a key piece of metadata, and tracks things like which data blocks (in the data region) comprise a file, the size of the file, its owner and access rights, access and modify times, and other similar kinds of information. To store this information, file systems usually have a structure called an inode
Our file system thus far has data blocks , and inodes, Many allocation-tracking methods are possible, of course. For example, we could use a free list that points to the first free block, which then points to the next free block, and so forth. We instead choose a simple and popular structure known as a bitmap, one for the data region (the data bitmap), and one for the inode table (the inode bitmap).

You may notice that it is a bit of overkill to use an entire 4-KB block for these bitmaps such a bitmap can track whether 32K objects are allocated, and yet we only have 80 inodes and 56 data blocks. However, we just use an entire 4-KB block for each of these bitmaps for simplicity. The careful reader (i.e., the reader who is still awake) may have noticed there is one block left in the design of the on-disk structure of our very simple file system. We reserve this for the superblock, denoted by an S in the diagram below. The superblock contains information about this particular file system, including, for example, how many inodes and data blocks are in the file system (80 and 56, respectively in this instance), where the inode table begins (block 3), and so forth. It will likely also include a magic number of some kind to identify the file system type
Thus, when mounting a file system, the operating system will read the superblock first, to initialize various parameters, and then attach the volume to the file-system tree. When files within the volume are accessed, the system will thus know exactly where to look for the needed on-disk structures.

One common idea is to have a special pointer known as an indirect pointer. Instead of pointing to a block that contains user data, it points to a block that contains more pointers, each of which point to user data. Thus, an inode may have some fixed number of direct pointers (e.g., 12), and a single indirect pointer. If a file grows large enough, an indirect block is allocated (from the data-block region of the disk), and the inode’s slot for an indirect pointer is set to point to it.
Assuming 4-KB blocks and 4-byte disk addresses, that adds another 1024 pointers; the file can grow to be (12 + 1024) · 4K or 4144KB.

User defined Stat :

/*
	Structure of a metadata tree node
	*/
	struct FStree{
	    char * path;                    // Path upto node
	    char * name;                    // Name of the file / directory
	    char * type;                    // Type : "directory" or "file"
	    mode_t permissions;		        // Permissions 
	    uid_t user_id;		            // userid
	    gid_t group_id;		            // groupid
	    int num_children;               // Number of children nodes
	    int num_files;		            // Number of files
	    time_t a_time;                  // Access time
	    time_t m_time;                  // Modified time
	    time_t c_time;                  // Status change time
	    time_t b_time;                  // Creation time
	    off_t size;                     // Size of the node
	    unsigned long int inode_number; // Inode number of the node in disk    
	    struct FStree * parent;         // Pointer to parent node
	    struct FStree ** children;      // Pointers to children nodes
	    struct FSfile ** fchildren;     // Pointers to files in the directory
	};

The following operations are supported in this file system
1.	Create a directory.
2.	Remove a directory - both empty and non-empty.
3.	Create a file using touch, nano, gedit etc.
4.	Delete an existing file.
5.	Appending to and truncating a file.
6.	Open and close a file.
7.	Read and write to files.
8.	Persistence of all aspects of the file system.

Installing Dependencies :
	The file system was built over the Distro, Ubuntu 16.04 Xenial Xerus. The fuse version used for this project was FUSE 2.9.4. To install libfuse-dev package on Ubuntu 16.04 (Xenial Xerus), run the following commands.
$ sudo apt-get update
$ sudo apt-get install libfuse-dev

Using this File System

Clone this repository

$ git clone https://github.com/vivekraghu17/MyFS
Cd into the folder 'MyFS' and mount the filesystem using the makefile

$ cd MyFS
$ make
To view the error logs of fuse and check for memory leaks / errors, run the following commands

$ cd MyFS
$ make debugrun
cd into the mountpoint which is present at ~/Desktop/mountpoint and use the filesystem!

$ cd ~/Desktop/mountpoint
To unmount the filesystem run the following commands

$ cd ~/Desktop
$ sudo umount mountpoint


