vid Gudeman

March 19, 2015

CIS33 Operating Systems

#Linux Unix file system:

#####1.1 What is a file?

>"A file is any source, with a name, from which data can be read; or any target, with a name, to which data can be written”. [1]

"On a UNIX system, everything is a file; if something is not a file, it is a process." [2]

In Unix the term “file” refers not only to an abstracted concept of a memory location to hold data but also any physical device. A keyboard (input device) and a monitor (output device) are both considered files. Files are other structures that provide services (like daemons) that have no physical aspect but provide services to the computer system. This provides the system with great flexibility because it allows the programmer a standard I/O interface to be expected whether the data is coming from a hard disk or a keyboard, or if the data is going to memory or the monitor.  

There are three kinds of files [1]:
ORDINARY FILES
DIRECTORIES 
PSEUDO FILES

#####1.1.1 ordinary files 
 The typical concept of a file, an abstraction for a memory schema  that contains data. Common examples include: executable programs, object files, images, word processing documents, spreadsheets  etc. This include binary files e.g. text editor program and text files e.g. the file that is edited. 

#####1.1.2 directories
This resides in memory somewhere.  Directories hold files in that they organize, and provide access to other files.  They can also organize and provide access to other directories.

#####1.1.3 pseudofiles
Pseudofiles do not hold data and as such do not take any room in memory.  They are organized into directories. Mostly they are to provide access to a service provided by the operating system kernel. These are known as Proc Files. Pseudofiles also include special files that provide for I/O and include: 

Device files are pseudofiles that provide internal representation of a physical device.



(Domain) Sockets: similar to TCP/IP sockets, provide inter-process networking protected bty the file system’s access control.

Named pipes are also pseudo files. Pipes connect the output of one file to the input of another file. These are similar to sockets without the network issues.

The Linux Virtual File System
The Linux system utilizes two layers of abstraction. [3] The lowest level, called ext3, is the standard file system with the various types of files. Again, for Linux anything that can handle input and output is a file.  In order to facilitate processing these types the Linux kernel provides a software layer above the standard file system called the Virtual File System. [2]

#####2.1.1 The Virtual File system described
The Linux VFS is designed around object-oriented principles.  There are two components:

A SET OF DEFINITIONS that provided a standard interface specification
A SOFTWARE  LAYER  to manipulate the objects.

There are 4 VFS object types [4]:
Each of the 4 objects has a set of functions provided to the VFS file. 

|   Object   	|               Description              	|      Operation      	|
|:----------	|:--------------------------------------	|:-------------------	|
| Superblock 	| specific filesystem                    	| read_inode, syn_fs  	|
| Dentry     	| directory entry, single component path 	| create, link        	|
| I-node     	| specific file                          	| d_compare, d_delete 	|
| File       	| open file associated with a process    	| read, write         	|
**Figure 1**: The four Linux VFS object types

For each of these object types VFS defines a set of operations. EVERY VFS object contains a POINTER to a FUNCTION TABLE. The function table in turn lists the addresses of the actual functions that implement the defined operations for those objects. 

Silbershatz [3] gives the example of a file operation:

  - int open(. . .)  // opens the file
  - ssize_t read(. . .)  //reads form a file
  - ssize_t write(. . .) // writes to file
  - int mmap ( . . .) // memory map the file

The entire definition of the file object is specified in the **struct file_operations** located in the  **/usr/include/linux/fs.h**. A typical entry looks like **Figure 2**.


**Figure 2** – An entry of the code that the Linux VFS uses to define a file. This demonstrates that it is written in the C language. 

#####2.1.2 The VFS inode  and file objects

The VFS layer can call any of the functions on the ohject that the inode represents without know what type of file it is.  An inode object is a data structure containing pointers to the disk blocks that contain the actual file contents.  The file object represents a point of access to the data in an open file. A process cannot access the inode’s contents withouft first obtaining a file object point to the inode. The file object keeps track of where in the file the process in currently reading or writing, to keep track of the sequential file I/O. The file object also keep track of the PERMISSIONS  and also reads ahead of the process to fetch data into memory to improve performance. 

There is one inode object but there can be many file objects.  There is one file object for each open instance of the file. 

#####2.1.2.1 The contents of an inode
1. At creation the inode is endowed with this information:
2. Owner and group owner of the file
3. File type (regular, directory or pseudofile)
4. Permissions for the file
5. Date and time of creation, last read and change
6. Number of links to this file
7. File size
8. An address defining the actual location of the file data

What the inode does NOT have:
1. file name
2. directory

The file name and directory are stored in special directory files. [1]

For Directories, since there is no reading or writing of data, the VFS programming interface is defined in the inode itself which contrasts to how regular files are handled.

#####2.1.3 The Linux VFS superblock object
The superblock object represents a connected set of files which  form a self-contained file system. The operating system kernel maintains a single superblock for each disk device mounted as a file system and for each networked file system currently connected.  The superblock provided inodes through the use of a VFS derived unique file-system/inode number pair.

#####2.1.4 The Linux VFS dentry object
The dentry object represents a directory entry, which may include the name of a directory in the path name of a file.  For example  /usr/include/linux/stdio.h actually contains 4 dentry objects: (1) /, (2) usr, (3) include and (4) stdio.h.  The Linux kernel maintains a cache of dentry objects to speed up path-name translation. 

#####3.0 The Linux ext3 File System
The standard on-disk file system used by Linux is ext3. The first two iterations transitions form the Minix operating system and progressive increased.  There is now a ext4. There are multiple other types of files systems including GFS, hfs, hpfs, nfs, vfat among others. [The workhorse of the standard linux data file is ext.[5]

#####3.1.1
The ext3 structure
Files and directories for Linux are stored on disk but their contents are interpreted differently. 

Each block in a directory files consists of a linked list of entries.  Each entry contains the length of the entry, the name of a file and the information of which inode the entry refers to. Linux supports 4 different block sizes to accommodate different sizes of files. The block sizes are 1,2,4,and 8KB.

In order to achieve performance it is good to have file-related blocks located adjacent to each other on the disk. So the Linux kernel clusters these. The ext3 block allocation policy partitions the file system into segments called block groups. Each block group corresponds to a cylinder on the disk. 

#####3.1.2 File allocation policies
During file allocation ext3 selects the block group for that file. For data blocks ext3 looks for the group  in which file’s inode has been allocated. During inode allocation for non directory files, ext3 selects the block group in which the file’s parent directory resides. Directory files are dispersed throughout available block groups. The goal is to keep related information physically close but disperse the disk load among groups to keep fragmentation to a minimum. 

Ext3 maintains a bitmap of all free blocks in a block group.  
When starting a file:
Starting at the beginning of a block group ext3 searches for a free block

When extending a file:
Ext starts at the most recent allocated block for that file. 

The search is executed in two stages. First, in the bitmap, it looks for a free byte (which would represent an 8 byte block). If no bytes are available then it looks for free bits. It tries to find at least one 8 byte block.

Once a free block is found the search is extended backwards until an allocated block is encountered. This prevents holes from being left in the storage allocation as well as the bitmap that represents the allocations. 

Once a block is allocated it attempts to pre allocate the next 8 blocks in order to reduce fragmentation in the future.

#####3.1.3 Journaling
Ext3 lists all modifications to the files system in sequential order in a Journal. A set of  operations that completes a task is defined as Transaction. A pointer is kept to determine where in the task the operations are occurring. Once a transaction is written to the journal it is “committed” and followed by the pointer. Once it is completed it is removed from the journal. In a system crash, upon restarting the operating system checks with the journal and is pointed to where it left off and it completes the transaction. 

The Linux Process File system
The VFS abstraction allows the implementation of a file system to store information about processes.

#####4.1.2 Files in /proc
The process files can be found in /proc. Most operating systems  usually do not store data files in this file system. By contrast, Linux adds text files here in order to make information about the processes available. The operating system uses a mapping function that splits the 32 bit inode number into a 16 bit PID number and the other 16 bits is used to define what type of information is being requested. 

#####4.1.3 PID of zero and global variables
If the PID in the inode is zero it means that the inode is a global variable, not process specific. Global files are in /proc. These files contain information concerning such things as kernel version, free memory, performance statistics and actively running drivers. As an example Figure 3 shows the output of the file  version.

```
Linux version 3.13.0-32-generic (build@kissel)
(gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) )
#57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014
```
**Figure 3**: The information in **/src/version** file

#####4.1.4 Linux Kernel and /proc file
The kernel dynamically allocates inode numbers and maintains a bit map of these inodes. It also maintains a tree data structure of registered global files in **/proc**. Each of these files contains the file’s inode number, file name, and access permissions. It also contains special functions to generate the file’s contents. 

Drivers use **/proc/sys** directory which is reserved for kernel variables. There is a special system call, **sysctl()**, which provides easy access for processes.  **Figure 4** shows the relationships of VFS with ext3 and **/proc** system.


**Figure 4**:  Schema of the VFS in relation to the ext3, device drivers and the processes file systems [6]. The reiser filesystem is another type of file system that is used in Linux formatted disks. 


Functioning of the file system

#####5.1 Locking 
Linux adheres to POSIX standard providing a facile way for a process to lock as little as a single byte and as much as an entire file in one indivisible operation. It is required for the caller to specify: file to be locked, the starting byte and the number of bytes. If the lock is successful the operating system makes a table entry. There are shared locks and exclusive locks. In shared locks if the file is locked and another caller tries to lock it on top of the first lock it will be successful. With exclusive locks only one caller at a time can lock a file. 

#####5.2 File system calls in Linux
A file is created with the creat system call.  The file descriptor is given a name  and a protection mode to determine who can access this program.  For example:

fd = creat (“foo”, mode);

fd represents a  file descriptor and this is given a small non negative integer. So this call creates the file foo with certain permissions.  A second system call called open the file.  When a file is created it is assigned the file descriptors 0, 1 and 2. This makes the file ready for standard input, standard output and standard error respectively.

Each file has a pointer that keeps track of the next byte to read or write. 

The most heacily used calls are the read and write calls. The have three parameters :
1. a file descriptor – telling which open file to manipulate
2. a buffer address – telling where to get or put the data
3. a count – telling how many bytes to transfer

This call exemplifies the simple and elegant design of the Linux file system

#####6.0 Bibliography:
[1] Guide to Unix and Linux, by Harley Hahn, 2009, McGraw-Hill

[2] Introduction to Linux: Chapter 3. About files and the file system, http://tldp.org/LDP/intro-linux/html/sect_03_01.html

[3] Operating System Concepts, 9th Edition, 2013, Silberschatz et al, Wiley

[4] Modern Operating Systems, 3rd Edition, 2008, Andrew Tanenbaum, Pearson-Prentice Hall

[5] A practical Guide to Ubuntu Linux, 3rd Edition, Mark G. Sobell, 2010, Prentice Hall

[6] www.ibm.com

