# Operating Systems (CS3210) Final Project Fall '19: `libc`

> 👉 **To prevent plagiarism, Georgia Tech doesn't allow me to make the code for this project publicly available. If you'd like to browse the source, please [get in touch with me](mailto:jve@jahz.co) and I can get you a link to the private repo.**

## Authors

- Shannon Hwu
- Will Huang
- Jahziel Villasana

## Project Summary

For our final project, we chose to implement `libc`, the C standard library. xv6 has a small userspace library with various utility functions and system call wrappers, but it is limited in scope. Adding a `libc` implementation is a great way to extend xv6's usefulness while also using what we've learned over the semester about xv6 and operating system design.

### Headers worked on

- `stdlib.h`
- `stddef.h`
- `setjmp.h`
- `stdio.h`
- `signal.h`
- `string.h`
- `time.h`
- `errno.h`
- `stdarg.h`

### Lessons Learned

#### Buffered I/O

One of the biggest challenges we faced was the implementation of proper buffered I/O in `libc`. One of the advantages of `libc`'s design is that it buffers reads and writes. This makes I/O more efficient by avoiding system calls unless flushing buffers. Two new system calls were needed: `readv` and `writev`. While not POSIX standards, these Linux syscalls allow for writing from and reading to multiple buffers (scatter-gather reads and writes). These syscalls allow for maintaining a `FILE`'s internal as full as possible while still writing or reading to the actual file when necessary. Proper `stdio.h` implementation led to a lot of exploring of the OS's own file abstractions and some hard reasoning about how to best sync up the `libc` abstraction with the OS.

#### Signals

Perhaps the neatest addition to xv6 that we made was the `signal.h` header. Signals are a basic form of IPC, and really made xv6 feel more like a "real" operating system. Implementation of this header required some new system calls as well as other deep dives into the trap handler, scheduler, and other areas of process handling in the kernel. Once implemented, we were able to do some neat stuff, such as a `Ctrl-C` handler that lets you kill a process!

### Inspecting the code

The source for our implementation can be found in the `libc/` directory. Each header has its own directory, with a `test/` subdirectory that holds various tests for the header's functions. Note that our implementation required various system calls to be added. The source for those can be found in the usual files. (`sysproc.c`, `sysfile.c`, etc.)

### Building

Run 

```bash
make qemu-nox CPUS=1
```

### Testing

A master test file can be found in the home directory, called "libctest", which can be run in qemu shell with the command `libctest`. One of our test cases (test_sig_atomic_t()) specifies the use of 1 CPU. To do so, please run  from your linux shell to see the correct output from the libctest.

Due to the number of functions in `stdio.h`, we had to split its tests out into its own program. Run `stdiotests` to test `stdio.h`.

#### Demo

The demo uses many libc functions that we implemented. To run the demo project,

1. Run the `daemonize demo` command to daemonize the demo program; the shell should print the pid of the daemonized program to the console.
2. Run `kill -15 ` + the pid of the daemonized program. This program will use kill2 to send signal 15 to the daemonized program. The signal 15 handler for the daemonized demo program is set at the beginning of the program to print current time and signal received to the console(we tried to use stdio but the program became bigger than our fs can handle)

#### Misc
Using the command `exit` while running the simulator, you can gracefully exit the qemu-nox shell and return to your linux shell.

### Credits

We could not have made this project happen without some great help from [`musl`](http://www.musl-libc.org/). `musl` is another `libc` implementation, and was orignially going to be ported to xv6 for our project. `musl` is, however, a very robust and production grade library, and so we weren't able to port it directly. The `musl` source and community were great helps to us as we wrote our own implementation. Specifically, much of the code in `setjmp.h` and `stdlib.h` was used in our
implementation (with modifications of course). No code was blindly copy and pasted; we went to great, great pains to make sure that we understood the functionality of all the code from `musl` that we used, either directly or just as inspiration for our own. We feel that this is a fair use of the open source code that `musl`, as there are precious few resources for attempting a project of this difficulty and magnitude. 
