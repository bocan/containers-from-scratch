# 02 — Preparing Your Lab Environment
*Setting up a clean Linux environment for building containers from first principles.*

---

## 📘 Overview

Before we start building root filesystems, creating namespaces, or crafting our own container runtime, we need a stable, predictable laboratory environment.

This chapter walks you through:

- Setting up a Debian-based sandbox system
- Installing essential developer and debugging tools
- Building BusyBox from source (our minimal userspace)
- Preparing directory structures used later
- Verifying kernel support for namespaces, cgroups, and OverlayFS
- Understanding why each tool matters

When you finish this chapter, you’ll have:

- A working build environment
- A compiled BusyBox rootfs
- A stable foundation for the rest of the course

---

# 1. Choosing Your Lab Environment

You may use:

- **A VM** (recommended)
- **Bare metal**
- **WSL2** (works fine)
- **A dedicated cloud VM** (EC2, GCP, Hetzner, etc.)

**Recommended:**
- Debian 13 “Trixie”
- 2 vCPU, 4 GB RAM
- 10+ GB disk
- A non-root user with `sudo` privileges

---

# 2. Install Essential Tools

These are the tools we’ll need to:

- Compile code
- Trace syscalls
- Build rootfs
- Manipulate namespaces
- Inspect cgroups
- Build our runtime
- Debug weird kernel behaviour

### Install them now:

```bash
sudo apt update
sudo apt install -y \
    build-essential \
    git \
    curl wget \
    strace ltrace gdb \
    pkg-config \
    clang \
    linux-headers-$(uname -r) \
    libseccomp-dev \
    libcap2-bin \
    iproute2 \
    iputils-ping \
    debootstrap \
    squashfs-tools \
    golang \
    util-linux \
    bison flex \
    ninja-build cmake
```

### Why each tool matters

| Tool | Purpose |
|------|---------|
| `build-essential` | compilers + linker + libc headers |
| `strace` | tracing syscalls — essential for understanding containers |
| `ltrace` | tracing library calls |
| `gdb` | debugging userspace code |
| `iproute2` | modern networking (`ip`, `ss`, `tc`) |
| `util-linux` | `unshare`, `nsenter`, `mount` → core container plumbing |
| `debootstrap` | used later for full Debian-based rootfs |
| `golang` | language for our minimal container runtime |
| `libseccomp-dev` | building syscall filters |
| `libcap2-bin` | inspecting & dropping capabilities |
| `linux-headers-*` | required for some namespaces/cgroup experiments |

---

# 3. Validate Kernel Features

Containers depend heavily on modern Linux kernel features.

Let’s verify they’re working.

## 3.1 Namespaces

```bash
ls -1 /proc/self/ns
```

You should see:

```
cgroup
ipc
mnt
net
pid
pid_for_children
time
time_for_children
user
uts
```

## 3.2 Cgroups v2

```bash
mount | grep cgroup
```

Look for something like:

```
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime)
```

If you don’t see `cgroup2`, run:

```bash
ls /sys/fs/cgroup/cgroup.controllers
```

If this file exists, you’re good.

## 3.3 OverlayFS

```bash
grep overlay /proc/filesystems
```

Should display:

```
nodev   overlay
```

If all the above checks pass, your kernel is ready.

---

# 4. Build BusyBox From Source

BusyBox is our tiny static userspace.
It gives us:

- `/bin/sh`
- basic commands
- predictable behaviour
- a minimal, reproducible rootfs for learning

### 4.1 Download BusyBox

```bash
mkdir -p ~/src
cd ~/src
wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
tar -xjf busybox-1.36.1.tar.bz2
cd busybox-1.36.1
```

### 4.2 Configure a static build

```bash
make defconfig
```

Now enable static linking:

```bash
sed -i 's/.*CONFIG_STATIC.*/CONFIG_STATIC=y/' .config
```

Or run:

```bash
make menuconfig
# Busybox Settings → Build Options → Build Static Binary
```

### 4.3 Build it

```bash
make -j$(nproc)
```

### 4.4 Install to a rootfs directory

```bash
mkdir -p ~/rootfs
make CONFIG_PREFIX=~/rootfs install
```

Your minimal filesystem now lives in:

```
~/rootfs
```

Verify:

```bash
ls ~/rootfs/bin
```

---

# 5. Create Base Directories Needed for Containers

Even a tiny container needs a few paths:

```bash
sudo mkdir -p ~/rootfs/{proc,sys,dev,tmp}
sudo chmod 1777 ~/rootfs/tmp
```

Inside the container later, we’ll:

- mount `/proc`
- mount `/sys`
- create `/dev/null`, `/dev/zero`, etc.

But that comes later.

---

# 6. Explore BusyBox Rootfs (Optional Warm-Up)

Try running a shell inside your BusyBox rootfs *without* isolation:

```bash
sudo chroot ~/rootfs /bin/sh
```

Try commands:

```
/ # ls
/ # ps
/ # hostname
```

Notice:

- You see host processes
- You have host hostname
- You’re not in a container yet

This is intentional — next chapters fix this.

---

# 7. Diagram: What We Have Built So Far

```mermaid
flowchart TD

A[Debian Host OS] --> B[Development Tools Installed]
B --> C[BusyBox Source]
C --> D[Static BusyBox Binary]
D --> E[Minimal Root Filesystem (~/rootfs)]

style A fill:#222,color:#eee,stroke:#555
style E fill:#1e6,stroke:#0a0,color:#fff
style D fill:#444,color:#eee
```

---

# 8. Exercises

Try these now to build muscle memory.

### **Exercise 1 — Add a custom file to the rootfs**

```bash
echo 'Welcome to your handmade rootfs!' > ~/rootfs/etc/motd
sudo chroot ~/rootfs /bin/sh -c "cat /etc/motd"
```

### **Exercise 2 — Explore system calls**

```bash
sudo chroot ~/rootfs strace ls /
```

### **Exercise 3 — Verify static binary**

```bash
file ~/rootfs/bin/busybox
```

### **Exercise 4 — Create your own root directory**

```
mkdir ~/mysecondroot
cp ~/rootfs/bin/busybox ~/mysecondroot
ln -s busybox ~/mysecondroot/sh
sudo chroot ~/mysecondroot /sh
```

---

# 9. Deep Dives (Optional But Delicious)

### 🔍 Deep Dive 1 — Static vs Dynamic Binaries
### 🔍 Deep Dive 2 — The `chroot` Syscall
### 🔍 Deep Dive 3 — Testing Namespaces with `unshare`

---

# 10. Working Artifacts Produced

```
~/rootfs/
    bin/
        busybox (static)
        sh -> busybox
    proc/
    sys/
    dev/
    tmp/
```

---

# 👉 Next Chapter

**03 — Building a Minimal Root Filesystem**
