```bash
# Check /sys/fs/bpf pinned program
sudo ls -l /sys/fs/bpf | sed -n '1,200p'

# Show bpf program
sudo bpftool prog show

# Stream live messages
sudo cat /sys/kernel/debug/tracing/trace_pipe

# Check net
sudo bpftool net

# Delete program
sudo rm -f /sys/fs/bpf/links/...

# Shows every map currently loaded in the kernel
sudo bpftool map show

# Dump the certain map
sudo bpftool map dump id 123
```
