---
title: "Adding an arm64 syscall entry"
date: 2023-07-27T17:53:17-03:00
categories: ["linux kernel"]
---

<img style="display: block; margin: auto;" src="/blog/tux.webp"/>

For some architectures, kernel has changed the generation of some inner headers for syscall tables. Now at kernel 6.1.y (Pi fork) there are some utility scripts like *syscallnr.sh* which acts over a table file called *syscall.tbl*. Apparently, for arm64, this is not the case since we still have to add the entries manually like the old days.

At linux-rpi-6.1.y folder, change the following headers:

```c
// include/linux/syscalls.h
asmlinkage long sys_my_pi_syscall(void);
```

```c
// include/uapi/asm-generic/unistd.h
#define __NR_my_pi_syscall 451
__SYSCALL(__NR_my_pi_syscall, sys_my_pi_syscall)
```

```c
// add a 'plus one' at total number...
#define __NR_syscalls 452
```

```c
// usr/include/asm-generic/unistd.h
#define __NR_my_pi_syscall 451
__SYSCALL(__NR_my_pi_syscall, sys_my_pi_syscall)
```

```c
// add a 'plus one' at total number...
#define __NR_syscalls 452
```

```c
// arch/arm64/include/asm/unistd32.h
#define __NR_my_pi_syscall 451
__SYSCALL(__NR_my_pi_syscall, sys_my_pi_syscall)
```

```c
// for syscall implementation: use an existing file or create a new one.
// i've used the kernel/sys.c...
SYSCALL_DEFINE0(my_pi_syscall)
{
    printk(KERN_ERR "my pi syscall called for arm64!!!");
    return 0;
}
```

There is a document about this called *adding-syscalls.rst* inside the tree, this info is also there. After updating the kernel image and installing the new headers:

```
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_HDR_PATH=/mnt/rootfs_sdcard/usr/ headers_install
```

You can call the *my_pi_syscall* with syscall function from libc.