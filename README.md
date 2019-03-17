## /proc/tunnel

`/proc` in kernel is how kernel send information to process that is the `/proc` file
system. `/proc` allows easy access to information about processes, used by every bit
of the kernel that can have anything interesting to report such as `/meminfo/` having
stats memory use statistics.

Everytime the file `/proc/helloworld` is read, the function `procfs_read` is called.
Buffer and offset will be returned to the application read it. If the return value
of the function isn't null, then the function is called repeatedly, if it never returns
zero the function is called endlessly.

`/proc/tunnel` is how data between kernel and user space is shared. Acting like a pipe,
having multiple 4K byte buffers - one that gets written in the kernel and consumed in
user space and another that gets written from user space and consumed in the kernel.
From user space, `/proc/tunnel` is accessed like a file. In the kernel, the access happens
through the following functions:

- `int tunnel_read(char* loc, int count)`
   This function reads up to count bytes from the user to kernel buffer into where.
	 The return value is number of characters actually read (`< count` if underflow).

	 The first call to `tunnel_read` starts reading at the beginnning of the buffer. All
	 subsequent calls begin reading at the location in the buffer where the previous
	 call stopped reading.

- `int tunnel_write(char *loc, int count)`
   This function writes up to count bytes from the user to kernel buffer into where.
	 The return value is the number of characters actually written (`< count` if underflow).
	 The initial call to `tunnel_write` starts writing at the beginning of the buffer. All
	 subsequent calls begin writing at the location in the buffer where previous	call
	 stopped writing.

- `int tunnel_user()`
   This function returns the number of used bytes in the kernel to use buffer.

- `int tunnel_avail()`
   This function returns the number of unused bytes in the kernel to user buffer.


### Enabling the tunnel

To use `/proc/tunnel` add `proc_fs.h` in `include/linux/` and `proc_misc.c` in `/fs/proc`
respectively.


*Please make sure you name the system call added kernel separately*
