# Week 1: Container Internals & The Syscall View

## 1. Questions & Answers

### 1. Why can a security tool on the host see what happens inside every container?
A container is not a virtual machine with its own kernel; it is simply an ordinary Linux process running directly on the host kernel with namespaces and cgroups attached. Because every container on the node shares the exact same host Linux kernel, any process action inside a container must execute via system calls on the host kernel, where host-level monitoring tools can intercept them.

### 2. What is the difference between a namespace and a cgroup? Give one example of each.
- **Namespace (Visibility):** Controls what a process can see. Example: A PID namespace isolates the process table so that the entrypoint process sees itself as PID 1 while hiding host processes.
- **cgroup (Resource Allocation):** Controls what a process can consume. Example: `memory.max` sets a hard ceiling on RAM allocation (e.g., 64 MiB / 67108864 bytes).

### 3. What does `--privileged` change, and why is it a risk?
`--privileged` disables the standard Linux capability drop and grants all capabilities (`CapEff: 000001ffffffffff`), including `CAP_SYS_ADMIN`. This eliminates the container isolation boundary and allows straightforward host breakouts, such as mounting host block devices directly into the container filesystem.

### 4. Name three system calls and the real-world action each one represents.
- `execve`: Executes a program or binary (e.g., running `/usr/bin/ls`).
- `openat`: Opens or references a file on the filesystem (e.g., reading `/etc/shadow`).
- `connect`: Initiates an outbound network connection (e.g., establishing a TCP connection to an external C2 server).

### 5. What is added on top of raw syscall events to make them useful for Kubernetes security?
Contextual enrichment (mapping kernel PIDs to container IDs, pod names, namespaces, and image tags) combined with a detection/policy engine that evaluates the enriched telemetry against security rules.

## 2. Lab Outputs

### Experiment A: Process & Namespace Inodes
- Host PID: 2933
- Container PID namespace: pid:[4026532713]
- Host Shell PID namespace: pid:[4026531836]
- User namespace (Shared): user:[4026531837]

### Experiment C: cgroup Limits
- `memory.max`: 67108864 (64 MiB)
- `cpu.max`: 50000 100000 (0.5 CPU core)

### Experiment D: Capabilities
- Default: `00000000a80425fb` (14 capabilities)
- `--cap-drop=ALL`: `0000000000000000` (0 capabilities)
- `--privileged`: `000001ffffffffff` (All capabilities enabled)
