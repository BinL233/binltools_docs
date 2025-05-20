This is the note of the book: *Learning eBPF* by Liz Rice. Link: https://cilium.isovalent.com/hubfs/Learning-eBPF%20-%20Full%20book.pdf

## BCC example

### Code

```python
#!/usr/bin/python
from bcc import BPF

# eBPF uses C language code
program = r"""
int hello(void *ctx) {
 bpf_trace_printk("Hello World!");
 return 0;
}
"""

# Object
b = BPF(text=program)

# use the syscall used to execute a program
syscall = b.get_syscall_fnname("execve")

# Attaches a kprobe (kernel probe) to kernel function
# event=syscall specifies the kernel function to probe
b.attach_kprobe(event=syscall, fn_name="hello")

# Print
b.trace_print()
```

### Output

```bash
$ hello.py
b' bash-5412 [001] .... 90432.904952: 0: bpf_trace_printk: Hello World'
```

### Operation
![operation](/ebpf_c2/images/operations.png)

- `bpf_trace_printk()` helper function in the kernel always sends output to /sys/kernel/debug/tracing/trace_pipe

## BPF Maps

Maps can be used to share data among multiple eBPF programs or to communicate between a user space application and eBPF code running in the kernel. 

Typical uses include the following:

- User space writing configuration information to be retrieved by an eBPF program
- An eBPF program storing state, for later retrieval by another eBPF program (or a future run of the same program)
- An eBPF program writing results or metrics into a map, for retrieval by the user space app that will present results

### Hash Table Map

```c
// BPF_HASH() is a BCC macro that defines a hash table map
BPF_HASH(counter_table);

int hello(void *ctx) {
	u64 uid;
	u64 counter = 0;
	u64 *p;
	
	/*
		The helper function used to obtain the user ID that
	  is running the process that triggered this kprobe event
	  Also, we want to get lower 32 bits
  */
	uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;
	
	/*
		Use hash map find value of uid
		Returns the pointer to the value
	*/
	p = counter_table.lookup(&uid);
	
	if (p != 0) {
		counter = *p;
	}
 
	counter++;
	
	// Update hash map
	counter_table.update(&uid, &counter);
	return 0;
}
 
```

### Perf and Ring Buffer Maps

**Ring Buffers**

![buffer](/ebpf_c2/images/buffer.png)

- Ring buffer as a piece of memory logically organized in a ring
- Use “write” and “read” pointers for writing and reading
    - Write pointer moves to after the end of that data, ready for the next write operation
    - Data gets read from wherever the read pointer is
        - using the header to determine how much data to read
    - The read pointer moves along in the same direction as the write pointer
    - If the read pointer catches up with the write pointer, it simply means there’s no data to
    read
    - If a write operation would make the write pointer overtake the read pointer, the data doesn’t get written and a drop counter gets incremented
- Data has header with information (e.g. length)

**Code**:

```c
/*
	Macro BPF_PERF_OUTPUT for creating a map 
	that will be used to pass messages from the kernel to user space
*/
BPF_PERF_OUTPUT(output);

struct data_t {
	int pid;
	int uid;
	char command[16];
	char message[12];
};

int hello(void *ctx) {
	struct data_t data = {};
	char message[12] = "Hello World";
	
	/*
		Returns a u64:
		Upper 32 bits: TGID (Thread Group ID = PID of the process)
		Lower 32 bits: PID (Thread ID)
	*/
	data.pid = bpf_get_current_pid_tgid() >> 32;
	
	/*
		Returns a u64:
		Upper 32 bits: GID (Group ID)
		Lower 32 bits: UID (User ID)
  */
	data.uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;
	
	/*
		helper function for getting the name of the executable (or “command”) 
		that’s running in the process that made the execve syscall
	*/
	bpf_get_current_comm(&data.command, sizeof(data.command));
	
	// Copies it into the right place in the data structure
	bpf_probe_read_kernel(&data.message, sizeof(data.message), message);
	
	// Puts that data into the map
	output.perf_submit(ctx, &data, sizeof(data));
	
	return 0;
}
```

## Tail Calls

Tail calls can call and execute another eBPF program and replace the execution context, similar to how the execve() system call operates for regular processes.

```c
long bpf_tail_call(void *ctx, struct bpf_map *prog_array_map, u32 index)
```