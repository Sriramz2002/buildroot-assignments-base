# Assignment 9: AESD Character Driver and Socket Server
 
## Overview
 
This project is part of the **Advanced Embedded Software Development** coursework. It extends earlier socket programming assignments and integrates a custom Linux character driver into an embedded Linux system.
 
The work connects two repositories:
 
- **Assignment 9 Buildroot/QEMU repo:**  
  https://github.com/cu-ecen-aeld/assignment-9-Sriramz2002
- **Assignment 3 and later application/driver repo:**  
  https://github.com/cu-ecen-aeld/assignments-3-and-later-Sriramz2002
The main goal was to replace the earlier file-based socket server storage with a real Linux kernel character device called `/dev/aesdchar`.
 
---
 
## What I Built
 
A complete userspace-to-kernel workflow:
 
```
TCP Client
   │
   ▼
aesdsocket server
   │
   ▼
/dev/aesdchar
   │
   ▼
Linux kernel character driver
   │
   ▼
Kernel circular buffer
```
 
The socket server receives data from a TCP client, writes it to the character driver, reads the stored data back from the driver, and sends the result back to the client.
 
---
 
## Main Features
 
- Custom Linux character driver
- `/dev/aesdchar` device interface
- Kernel circular buffer for command storage
- `read()` and `write()` support
- `llseek()` support
- Custom `ioctl()` support with `AESDCHAR_IOCSEEKTO`
- TCP socket server on port 9000
- Multithreaded client handling using POSIX threads
- Daemon mode support
- Syslog-based logging
- Buildroot and QEMU based embedded Linux testing
---
 
## How It Works
 
Earlier assignments used a normal file path:
 
```
/var/tmp/aesdsocketdata
```
 
In Assignment 9, the storage backend was changed to:
 
```
/dev/aesdchar
```
 
This means the server no longer stores data in a regular userspace file — instead, it sends data directly into a Linux kernel driver.
 
### Normal Data Flow
 
```
Client sends data
      │
aesdsocket receives it
      │
aesdsocket writes to /dev/aesdchar
      │
driver stores complete newline-terminated command
      │
aesdsocket reads stored data back
      │
client receives full stored output
```
 
### Example
 
```bash
echo "hello" | nc localhost 9000
```
 
The server writes `hello` into `/dev/aesdchar`, reads the current driver contents, and sends the result back to the client.
 
---
 
## Character Driver
 
The `aesdchar` driver implements standard Linux file operations:
 
| Operation | Description |
|-----------|-------------|
| `open()` | Opens the device file |
| `release()` | Releases the device file |
| `read()` | Reads data from the circular buffer |
| `write()` | Writes newline-terminated commands |
| `llseek()` | Repositions the file offset |
| `ioctl()` | Custom command-based seeking |
 
The driver stores complete newline-terminated write commands in a circular buffer. Example stored commands:
 
```
0: first
1: second
2: third
```
 
When the buffer becomes full, older entries are overwritten in circular-buffer order.
 
---
 
## ioctl Support
 
Assignment 9 adds a custom ioctl command:
 
```
AESDCHAR_IOCSEEKTO:write_cmd,write_cmd_offset
```
 
### Example
 
```
AESDCHAR_IOCSEEKTO:1,2
```
 
This means:
- Go to **command 1**
- Start reading at **byte offset 2** inside that command
If command 1 is:
 
```
second
```
 
Then offset 2 starts from:
 
```
cond
```
 
The socket server parses this command, calls `ioctl()` on `/dev/aesdchar`, and then reads data from the new file position.
 
---
 
## What I Achieved
 
Through this assignment, I implemented and integrated both userspace and kernel-space components:
 
- Built a working Linux character driver
- Added kernel circular-buffer based storage
- Implemented safe user-kernel copying using `copy_from_user()` and `copy_to_user()`
- Added mutex protection for shared driver data
- Implemented `llseek()` for file-position control
- Implemented custom `ioctl()` support for command-based seeking
- Updated the TCP socket server to use `/dev/aesdchar`
- Added support for `AESDCHAR_IOCSEEKTO:x,y` commands from the network client
- Tested the full system in a Buildroot/QEMU embedded Linux environment
---
 
## Skills Demonstrated
 
- Embedded Linux development
- Linux kernel module programming
- Character device driver development
- Kernel circular buffer design
- User-kernel API design
- `read`, `write`, `llseek`, and `ioctl` implementation
- TCP socket programming
- POSIX threads
- Mutex synchronization
- Buildroot integration
- QEMU-based testing
- Syslog and kernel log debugging
---
 
## Summary
 
This project shows how a userspace network application can communicate with a custom Linux kernel driver through a device file. It combines socket programming, kernel driver development, circular-buffer storage, ioctl control, and embedded Linux system integration into one working system.
