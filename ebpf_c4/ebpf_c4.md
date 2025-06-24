This is the note of the book: *Learning eBPF* by Liz Rice. Link: https://cilium.isovalent.com/hubfs/Learning-eBPF%20-%20Full%20book.pdf

bpf() is used to “perform a command on an extended BPF map or program.”

```c
int bpf(int cmd, union bpf_attr *attr, unsigned int size);
```

![Screenshot 2025-05-12 at 10.51.07 AM.png](/ebpf_c4/images/Screenshot%202025-05-12%20at%2010.51.07 AM.png)

### Dynamically control behavior per-user

```c
struct user_msg_t {
	char message[12];
};

BPF_HASH(config, u32, struct user_msg_t);
BPF_PERF_OUTPUT(output);

struct data_t {
	int pid;
	int uid; // User ID
	char command[16];
	char message[12];
};

int hello(void *ctx) {
	struct data_t data = {};
	struct user_msg_t *p;
	
	char message[12] = "Hello World";
	
	data.pid = bpf_get_current_pid_tgid() >> 32;
	data.uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;
	
	bpf_get_current_comm(&data.command, sizeof(data.command));
	
	p = config.lookup(&data.uid); // Find map
	if (p != 0) {
		// Safely copies data from kernel memory into a destination buffer
		bpf_probe_read_kernel(&data.message, sizeof(data.message), p->message);
	} else {
		bpf_probe_read_kernel(&data.message, sizeof(data.message), message);
	}
	
	output.perf_submit(ctx, &data, sizeof(data)); // Put map
	
	return 0;
}
```

## Loading BTF Data

```bash
bpf(BPF_BTF_LOAD, {btf="\237\353\1\0...}, 128) = 3
```

- Loading a blob of BTF data into the kernel
- Return the file descriptor that refers to that data (3 in this example)

### BTF (BPF Type Format)

BTF allows eBPF programs to be portable across different kernel versions

- You can compile a program on one machine and use it on another

## Creating Maps

```bash
bpf(BPF_MAP_CREATE, {map_type=BPF_MAP_TYPE_PERF_EVENT_ARRAY, , key_size=4,
value_size=4, max_entries=4, ... map_name="output", ...}, 128) = 4
```

- `BPF_MAP_CREATE` : Create a map
- `{map_type=BPF_MAP_TYPE_PERF_EVENT_ARRAY, , key_size=4, value_size=4, max_entries=4, ... map_name="output", ...}` :
    - This is a struct: passed to the syscall, specifically `struct bpf_attr` when creating a map
        - `map_type` : The type of map
        - …
- `128` : The size of the struct
- `= 4` : Return value

```bash
bpf(BPF_MAP_CREATE, {map_type=BPF_MAP_TYPE_HASH, key_size=4, value_size=12,
max_entries=10240... map_name="config", ...btf_fd=3,...}, 128) = 5
```

- This create a Hash Table Map

## Loading a Program

```bash
bpf(BPF_PROG_LOAD, {prog_type=BPF_PROG_TYPE_KPROBE, insn_cnt=44,
insns=0xffffa836abe8, license="GPL", ... prog_name="hello", ...
expected_attach_type=BPF_CGROUP_INET_INGRESS, prog_btf_fd=3,...}, 128) = 6
```

- `BPF_PROG_LOAD` : This tells the kernel to load an eBPF program into the kernel verifier
- `{prog_type=BPF_PROG_TYPE_KPROBE, insn_cnt=44, insns=0xffffa836abe8, license="GPL", ... prog_name="hello", ... expected_attach_type=BPF_CGROUP_INET_INGRESS, prog_btf_fd=3,...}` :
    - struct bpf_attr fields:
        - `insn_cnt=44` : The program contains 44 eBPF instructions.
        - `insns=0xffffa836abe8` : Kernel memory address (pointer) to the eBPF bytecode instructions
        - `license="GPL"` : The license of your program
        - `expected_attach_type=BPF_CGROUP_INET_INGRESS` : Declares intended attach point type
        - `prog_btf_fd=3` : File descriptor 3, referencing BTF metadata you previously loaded.
- `128` : Size of struct

## Modifying a Map from User Space

- Python BBC
    
    ```python
    b["config"][ct.c_int(0)] = ct.create_string_buffer(b"Hey root!")
    b["config"][ct.c_int(501)] = ct.create_string_buffer(b"Hi user 501!")
    ```
    
- Then we can get systemcall:
    
    ```bash
    bpf(BPF_MAP_UPDATE_ELEM, {map_fd=5, key=0xffffa7842490, value=0xffffa7a2b410,
    flags=BPF_ANY}, 128) = 0
    ```
    

## BPF Program and Map References

### Pinning

- Pinning makes the object "named and persistent”.
- There is an additional reference to the program, so the program remains loaded after the command completes.
- BPF objects are pinned to the BPF virtual filesystem, usually mounted at `/sys/fs/bpf/` .