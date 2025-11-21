This page collects quick eBPF inspection commands for reference.

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

# Check whether the function in your libbpf
nm -D /usr/lib64/libbpf.so | grep bpf_link_create

```

Completely delete tc programs
```bash
# List current tc filters before trying to remove anything
sudo tc -d filter show dev lo ingress || true

# Try deleting filters by common preference numbers
for pref in 1 49151 49152; do
    if sudo tc filter del dev lo ingress pref "$pref" 2>/dev/null; then
        echo "  Deleted filter with pref $pref"
    fi
done
echo

# Flush the ingress qdisc (forces filter detach)
sudo tc qdisc del dev lo ingress 2>/dev/null || true
sudo tc qdisc add dev lo ingress handle ffff: ingress 2>/dev/null || true

# Re-list filters after flush
sudo tc -d filter show dev lo ingress || true

# Check if a specific BPF program is still loaded
PROG_ID=3319
sudo bpftool prog show id "$PROG_ID" || echo "Program $PROG_ID no longer present"

# Kill loader processes that may still hold BPF file descriptors open
for pname in tracer_loader target_loader; do
    pids=$(pgrep -f "$pname" || true)
    if [ -n "$pids" ]; then
        sudo kill $pids
        echo "  Killed $pname (PIDs: $pids)"
    fi
done
echo

# Wait a moment for cleanup
sleep 1

# Final check
sudo bpftool prog show id "$PROG_ID" || echo "Program $PROG_ID removed"
```
