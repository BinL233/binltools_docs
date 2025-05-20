This is the note of the book: *Learning eBPF* by Liz Rice. Link: https://cilium.isovalent.com/hubfs/Learning-eBPF%20-%20Full%20book.pdf

## The eBPF Virtual Machine

- Convert eBPF bytecode instructions to native machine instructions
- Run these native machine instructions on the CPU

## eBPF Registers

- The eBPF virtual machine uses 10 general-purpose registers, numbered 0 to 9.
- Register 10 is used as a stack frame pointer (Read only)
- Registers 1 to 5 is used to store argument
- Register 0 is used to return

### Instructions

```c
struct bpf_insn {
    __u8 code;         // 8 bits: opcode (what operation to perform)
    __u8 dst_reg:4;    // 4 bits: destination register
    __u8 src_reg:4;    // 4 bits: source register
    __s16 off;         // 16 bits: offset, used for jump instructions or memory access
    __s32 imm;         // 32 bits: immediate value, a constant embedded in the instruction
};
```

## eBPF “Hello World” for a Network Interface

```c
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

int counter = 0;

/*
	Macro used in eBPF programs
	This tells the eBPF loader that the function hello 
	belongs to the xdp section
*/
SEC("xdp")

int hello(void *ctx) {
	bpf_printk("Hello World %d", counter);
	counter++;
	
	return XDP_PASS;
}

char LICENSE[] SEC("license") = "Dual BSD/GPL"; 
```

### XDP (eXpress Data Path)

A type of eBPF program designed for high-performance packet processing at the earliest possible point in the Linux network stack.

| Return Code | Meaning |
| --- | --- |
| `XDP_ABORTED` | Drop the packet and log an error. Used for debugging or invalid states. |
| `XDP_DROP` | Silently drop the packet. It won’t go to the network stack. |
| `XDP_PASS` | Allow the packet to continue as normal (into the kernel networking stack). |
| `XDP_TX` | Send the packet back out the same interface it came from. |
| `XDP_REDIRECT` | Send the packet to another interface or CPU |

### Compile

```bash
hello.bpf.o: %.o: %.c
 clang \
 -target bpf \
 -I /usr/include/$(shell uname -m)-linux-gnu \
 -g \
 -O2 -c $< -o $@
```

## Loading the Program into the Kernel

```bash
$ bpftool prog load hello.bpf.o /sys/fs/bpf/hello
 
$ ls /sys/fs/bpf
hello
```

## Inspecting the Loaded Program

```bash
$ bpftool prog list
...
540: xdp name hello tag d35b94b4c0c10efb gpl
 loaded_at 2022-08-02T17:39:47+0000 uid 0
 xlated 96B jited 148B memlock 4096B map_ids 165,166
 btf_id 254
 
$ bpftool prog show id 540 --pretty
```

## JIT Compiler

JIT compiler converts eBPF bytecode to machine code that runs natively on the target CPU.

## Attaching to an Event

```bash
$ bpftool net attach xdp id 540 dev eth0
```

- `bpftool` : CLI tool to inspect and manage eBPF objects (programs, maps, etc.)
- `net attach xdp` : Subcommand to attach an eBPF XDP program to a network device
- `id 540` : Specifies the **program ID** of the already-loaded XDP program
- `dev eth0` : Specifies the **network interface** to attach the program to (here, `eth0` )
    - `eth0` : First Ethernet network interface on a Linux system. Represents the physical network card. (Own network interface)
    - `lo` : Loopback Interface. IP address for `lo` is always `127.0.0.1` (IPv4) or `::1` (IPv6).

**What happen:**

- The kernel attaches the program to `eth0` at the XDP hook point.
- From that moment on, all packets received on `eth0` will pass through this XDP program first.
- The eBPF Program should produce trace output every time a network packet is received.
    - `cat /sys/kernel/debug/tracing/trace_pipe`

## Global Variables

### Check maps

```bash
$ bpftool map list
165: array name hello.bss flags 0x400
	key 4B value 4B max_entries 1 memlock 4096B
	btf_id 254
166: array name hello.rodata flags 0x80
	key 4B value 15B max_entries 1 memlock 4096B
	btf_id 254 frozen
```

### Check global variables

A bss section in an object file compiled from a C program typically holds global variables

```bash
$ bpftool map dump name hello.bss
[{
			"value": {
				".bss": [{
					"counter": 11127
				}
			]
		}
	}
]
```

## Detaching the Program

```bash
$ bpftool net detach xdp dev eth0
```

## Unloading the Program

```bash
$ rm /sys/fs/bpf/hello
```

**Check:**

```bash
$ bpftool prog show name hello
```