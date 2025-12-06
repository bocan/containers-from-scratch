# 03 — Building a Minimal Root Filesystem
*A deeper, more complete rootfs to serve as the base for our hand‑rolled containers.*

---

## 📘 Overview

In Chapter 2, we built a tiny BusyBox-based filesystem.
Now it’s time to build a **more realistic minimal rootfs**, one that:

- Contains a proper directory tree
- Can run a small userland
- Supports devices
- Can be mounted cleanly inside namespaces
- Mirrors how Docker and containerd build rootfs layers internally

In this chapter, you will:

- Build a minimal Debian-based rootfs using `debootstrap`
- Add essential device nodes (e.g. `/dev/null`, `/dev/zero`)
- Mount `/proc` and `/sys` inside the rootfs
- Understand what *makes* a root filesystem valid
- Learn how container image layers derive from this structure

By the end, you’ll have a working root filesystem suitable for PID namespaces, mount namespaces, and eventually your runtime.

---

# 1. What *Is* a Root Filesystem?

A root filesystem (“rootfs”) is:

> **The directory tree that becomes `/` inside a process.**

It contains:

- `/bin`, `/sbin`, `/usr`, `/lib`
- `/proc`, `/sys`, `/dev` mountpoints
- Basic tools and libraries needed to run processes

Inside a container, your process sees *this* filesystem instead of the host’s.

---

# 2. Methods of Creating a Root Filesystem

We will use **two methods**:

### ✔ Method 1 — BusyBox (already done in Chapter 2)
A tiny static rootfs for learning.

### ✔ Method 2 — debootstrap (more realistic)
Creates a minimal Debian environment similar to a “real” container image.

This chapter focuses on **Method 2**.

---

# 3. Install debootstrap

You likely installed this in Chapter 2, but here it is again:

```bash
sudo apt update
sudo apt install -y debootstrap
```

---

# 4. Build a Minimal Debian Rootfs

Choose a directory:

```bash
mkdir -p ~/debian-rootfs
```

Run debootstrap for Debian Stable:

```bash
sudo debootstrap stable ~/debian-rootfs http://deb.debian.org/debian
```

This creates:

- A barebones Debian
- No kernel, no init system, no services
- Just enough to run a shell and basic tools

Verify:

```bash
ls ~/debian-rootfs
```

You should see:

```
bin  boot  dev  etc  lib  lib64  proc  root  run  sbin  sys  tmp  usr  var
```

This is now a **real** root filesystem.

---

# 5. Creating Essential Device Nodes

When we chroot inside, `/dev` must provide:

- `null`
- `zero`
- `tty`
- `random`
- `urandom`

Create them:

```bash
sudo mknod -m 666 ~/debian-rootfs/dev/null c 1 3
sudo mknod -m 666 ~/debian-rootfs/dev/zero c 1 5
sudo mknod -m 666 ~/debian-rootfs/dev/random c 1 8
sudo mknod -m 666 ~/debian-rootfs/dev/urandom c 1 9
sudo mknod -m 666 ~/debian-rootfs/dev/tty c 5 0
```

### Why?
BusyBox’s static binary doesn’t need glibc, but Debian rootfs requires functional device nodes to behave normally.

---

# 6. Mounting System Filesystems

Inside a container, `/proc` and `/sys` must be mounted manually.

Let’s test in our rootfs.

### Mount `/proc`:

```bash
sudo mount -t proc proc ~/debian-rootfs/proc
```

### Mount `/sys`:

```bash
sudo mount -t sysfs sysfs ~/debian-rootfs/sys
```

---

# 7. Chroot In

```bash
sudo chroot ~/debian-rootfs /bin/bash
```

Try:

```
# ls /
# ps aux
# uname -a
```

### Expected:

- `uname -a` still shows the **host kernel** (containers share kernels).
- `ps aux` shows **host processes** (no namespace isolation yet).
- File layout looks like normal Debian.

This is the baseline on which namespaces later create the illusion of isolation.

Exit:

```
exit
```

---

# 8. Unmount System Filesystems

Always clean up:

```bash
sudo umount ~/debian-rootfs/proc
sudo umount ~/debian-rootfs/sys
```

---

# 9. Directory Layout Diagram

```mermaid
flowchart TD

A[debootstrap] --> B[Minimal Debian Rootfs]

B --> C[/bin]
B --> D[/usr]
B --> E[/etc]
B --> F[/dev]
B --> G[/proc mountpoint]
B --> H[/sys mountpoint]

style B fill:#1e6,color:#fff,stroke:#0a0
```

---

# 10. Exercises

### **Exercise 1 — Add a banner**
Add a message displayed at login:

```bash
echo "Welcome to your handmade Debian rootfs!" | sudo tee ~/debian-rootfs/etc/motd
```

### **Exercise 2 — Install a package *inside* the rootfs**
Mount proc/sys first:

```bash
sudo mount -t proc proc ~/debian-rootfs/proc
sudo mount -t sysfs sysfs ~/debian-rootfs/sys
sudo chroot ~/debian-rootfs /bin/bash
```

Inside:

```bash
apt update
apt install -y vim
```

Exit and unmount:

```bash
exit
sudo umount ~/debian-rootfs/proc
sudo umount ~/debian-rootfs/sys
```

### **Exercise 3 — Check required library dependencies**
Outside:

```bash
ldd ~/debian-rootfs/bin/ls
```

Observe:

- glibc dependencies
- linux-vdso
- interpreter `/lib64/ld-linux-x86-64.so.2`

Contrast with static BusyBox.

---

# 11. Deep Dives

### 🔍 Deep Dive 1 — How Docker builds rootfs images
Dockerfile stages ultimately assemble a layered filesystem identical in structure to what you’ve created.

### 🔍 Deep Dive 2 — pivot_root vs chroot
In real containers, we don’t use `chroot()`—we use `pivot_root()` to replace the mount namespace’s root.

### 🔍 Deep Dive 3 — Why device nodes matter
Debian and most distros expect `/dev` to behave correctly.
Minimal images create symlinks and use tmpfs with `devpts`.

---

# 12. Working Artifacts From This Chapter

By the end of this chapter you should have:

```
~/debian-rootfs/
    bin/
    lib/
    lib64/
    usr/
    etc/
    dev/
        null
        zero
        tty
        random
        urandom
    proc/
    sys/
```

This will serve as your real container filesystem when we introduce **mount namespaces**, **PID namespaces**, and **OverlayFS** in later chapters.

---

# 👉 Next Chapter

**[04 — Namespaces: Teaching the Kernel to Lie](04-namespaces.md)**
This is where the magic really begins.

👉 **[Chapter 1 — An Introduction to Containers](docs/01-what-is-a-container.md)
