# Containers From Scratch
### *A deep, hands-on journey into the core of modern container technology.*

> The Main Site is https://bocan.github.io/containers-from-scratch/ and the open
> source repo is at https://github.com/bocan/containers-from-scratch.

---

## Introduction

Containers are everywhere. They power microservices, development workflows, CI/CD pipelines, data platforms, and the entire Kubernetes ecosystem. They’re lightweight, fast, portable, and incredibly powerful — but what are they, really?

Strip away the buzzwords, YAML, and orchestration layers, and you’ll find something surprisingly simple:
a container is just a Linux process using a set of kernel features to pretend it’s running on its own machine.

That’s it.
No magic. No VM. No hidden tricks.

This project exists to show you that truth from first principles.

Instead of installing Docker or Podman and accepting that they “just work,” you will build the pieces of a container runtime yourself—one layer at a time—on a clean Linux system. Much like Linux From Scratch demystifies the userspace, Containers From Scratch demystifies the modern container ecosystem by rebuilding it with your own hands.

By the end, you’ll understand exactly how containers run, how image layers work, how processes are isolated, and where security boundaries begin and end. You’ll also write a tiny container runtime of your own.

This isn’t a cookbook.

This is a deep technical journey.

## What is a container?

A container is:

- A Linux process,
- With limited privileges,
- Running in isolated namespaces,
- Confined by cgroups,
- Using a virtualized root filesystem,
- With custom networking,
- Possibly restricted further by capabilities, seccomp, and LSMs.

In other words:

A container is a process that the Linux kernel strategically lies to.

- It thinks it has its own filesystem.
- It thinks it is PID 1.
- It thinks it has its own network stack.
- It thinks it controls the whole machine.

But none of this is true.

By the time you finish this project, you’ll know exactly how to teach the kernel to tell those lies.

## Who This Is For

This project is ideal for:

- Platform engineers
- SREs
- DevOps engineers
- Security engineers
- Linux enthusiasts
- Students of operating systems
- Anyone who wants to understand containers at a “nothing up my sleeves” level

No prior knowledge of container internals is required.
A basic familiarity with Linux is recommended

## What You Will Build

By following this project, you will:

- Build a minimal root filesystem
- Isolate a process in new PID, UTS, mount, network, and IPC namespaces
- Apply cgroup resource limits
- Create an OverlayFS-backed layered filesystem
- Wire up container networking with veth pairs and bridges
- Apply capabilities and seccomp filters
- Write your **own tiny container runtime** that:
  - Creates namespaces programmatically
  - Configures a rootfs
  - Sets up networking
  - Applies cgroups
  - Drops privileges
  - Execs into PID 1 of your container

You will end with a fully working, minimalist reimagining of `runc`.


## Why Build Containers From Scratch?

Because you gain:

### **Deep technical intuition**
You’ll understand containers at the same level as people building Docker, containerd, and Kubernetes.

### **Security insight**
You’ll be able to see which boundaries are real and which are illusions.

### **Debugging power**
When namespaces, cgroups, networking, or OverlayFS misbehave, you’ll understand *exactly* what’s wrong.

### **Confidence**
You’ll stop treating containers as black boxes and start treating them as what they are:
well-engineered *Linux tricks*.

### **Fun**
Honestly? It’s fun. There’s something wonderfully hackerish about building your own runtime.

---

## What This Course Will Teach You

Each chapter stands alone and builds on the last.
You’ll assemble raw Linux features into increasingly container-like environments until you’re launching real workloads in a runtime you created yourself.

### **1. What Is a Container**
High-level understanding and mental models.

### **2. Preparing Your Lab Environment**
Install tools, build BusyBox, and set up a Debian-based sandbox system.

### **3. Building a Minimal Root Filesystem**
Construct a tiny rootfs using BusyBox or debootstrap.

### **4. Namespaces: Teaching the Kernel to Lie**
Use `unshare`, `nsenter`, and `clone()` to isolate processes.

### **5. Cgroups: Limiting and Accounting Resources**
Confine CPU, memory, pids, and more using cgroups v2.

### **6. OverlayFS and Filesystem Layers**
Reconstruct layered images and writable containers.

### **7. Container Networking From Scratch**
veth pairs, network namespaces, routing, NAT, DHCP… all built by hand.

### **8. Container Security: Capabilities, seccomp, LSMs**
Drop privileges, block syscalls, and apply AppArmor/SELinux profiles.

### **9. Writing a Tiny Container Runtime**
Build an OCI-like runtime in Go or C that launches containers end-to-end.

### **10. Understanding OCI, runc, containerd, and Kubernetes**
Map everything you built to the real-world container ecosystem.

### **11. Optional Advanced Chapters**
Rootless containers, snapshotting, seccomp generators, and more.

---

## How to Use This Repository

This repo is structured like a book, with each chapter in `docs/`:

```
docs/
  01-what-is-a-container.md
  02-preparing-environment.md
  03-building-rootfs.md
  04-namespaces.md
  05-cgroups.md
  06-overlayfs.md
  07-networking.md
  08-security.md
  09-writing-runtime.md
  10-real-world-oci.md
  11-advanced-topics.md
```

Each chapter includes:

- Clear explanations
- Terminal commands
- Diagrams
- Exercises
- Optional deep dives
- Working artifacts

---

## Next Steps

Whenever you're ready, let's move on to:

👉 **[Chapter 1 — An Introduction to Containers](docs/01-what-is-a-container.md)
