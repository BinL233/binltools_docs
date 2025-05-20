This is the note of the book: *Learning eBPF* by Liz Rice. Link: https://cilium.isovalent.com/hubfs/Learning-eBPF%20-%20Full%20book.pdf

## BPF (Berkeley Packet Filter)

Used in the tcpdump utility as an efficient way to capture the packets to be traced out.

### Filter

```c
// Simple Example

ldh [12] // Load 2 bytes
jeq #ETHERTYPE IP, L1, L2 // If is IP, jump to L1. Otherwise, L2
L1: ret #TRUE
L2: ret #0
```

## eBPF (extended BPF)

eBPF is a kernel technology

- Developer can write custom code
- The code can be dynamically load into the kernel (Don’t need to reboot machine)

We can do with eBPF include:

- Performance tracing of pretty much any aspect of a system
- High-performance networking, with built-in visibility
- Detecting and (optionally) preventing malicious activity

### BPF → eBPF

- The BPF instruction set was completely overhauled to be more efficient on 64-bit
machines, and the interpreter was entirely rewritten.
- eBPF maps were introduced, which are data structures that can be accessed by
BPF programs and by user space applications, allowing information to be shared
between them.
- The bpf() system call was added so that user space programs can interact with
eBPF programs in the kernel.
- Several BPF helper functions were added.
- The eBPF verifier was added to ensure that eBPF programs are safe to run.