## Virtual Memory

**Memory** is used to hold portions of the operating system, programs and data that are currently in use or that are frequently used. The **physical memory** (also referred to as **primary**, **main** or **real memory**) consists of random access memory (**RAM**) chips that are inserted into slots on the motherboard on a computer. The times required to access different addresses (i.e., locations) in RAM are extremely short and nearly equal, in contrast to the varying delay times for accessing locations on the disk.

To increase the size of usable memory, Linux supports **virtual memory**, which is the use of a disk space as an extension of RAM. The kernel will write the contents of a currently unused block of memory to the disk so that the memory can be used for another purpose. When the original contents are needed again, they are read back into memory from disk. This is all made completely transparent to the user; programs running under Linux only see the larger amount of memory available and don't notice that parts of them actually reside on the disk from time to time. Of course, reading and writing the disk is slower (on the order of a thousand times slower!) than using RAM, so the programs don't run as fast.

The part of the disk that is used as virtual memory is called the **swap space**.

Linux can use either a normal file in the filesystem or a separate partition for swap space. A swap partition is faster, but it is easier to change the size of a swap file (there's no need to repartition the whole hard disk, and possibly install everything from scratch). When you know how much swap space you need, you should go for a swap partition, but if you are uncertain, you can use a swap file first, use the system for a while so that you can get a feel for how much swap you need, and then make a swap partition when you're confident about its size. It is generally recommended that the size of the swap partition be about twice the amount of system RAM. You can find the instructions on how to create a swap space [here](https://www.tldp.org/LDP/sag/html/swap-space.html).

The swap space is divided into segments called **pages** (4Kb areas of memory), each of which is associated with a specific address in memory. When an address is referenced, the page is swapped into memory. It is returned to the disk when no longer needed and other pages are called. This management of virtual memory is performed by a type of hardware circuitry called a **memory management unit** (**MMU**).

A note on operating system terminology: computer science usually distinguishes between **swapping** (writing the whole process out to swap space) and **paging** (writing only pages of a program). Paging is usually more efficient, and that's what Linux actually does, nevertheless you'll see the word `swapping` to be used instead of `paging` most of the times.

### Resources used to create this document:

* http://www.linux-tutorial.info/modules.php?name=MContent&obj=page&pageid=89
* http://www.linfo.org/virtual_memory.html
* https://www.tldp.org/LDP/sag/html/vm-intro.html
