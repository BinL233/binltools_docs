### Kprobes (Kernel Probes)

Linux kernel feature that dynamically insert breakpoints into running kernel code without needing to recompile or reboot the kernel.

Kprobes can:

- Hook into almost any kernel function at runtime.
    - Hook: A mechanism to intercept or modify the behavior of a system or function.
- Run custom handler.

### Oracle BPFTuner

Dynamically optimize kernel-level networking performance.

| Functionality | Description |
| --- | --- |
| **TCP Stack Tuning** | Adjusts TCP buffer sizes, congestion control algorithms, memory parameters dynamically. |
| **Per-Namespace Policies** | Can tune settings **per network namespace**, useful for containers or virtualized environments. |
| **Auto-Optimization** | Detects conditions like low throughput or high latency and applies corrective tuning in real-time. |
| **Custom Policy Profiles** | Offers modes like `throughput`, `latency`, or `balanced` to optimize based on your goal. |
| **Uses eBPF** | Injects eBPF programs into the kernel to trace, monitor, and tune network behavior efficiently. |
| **Zero Downtime** | Applies kernel-level changes without needing a reboot or kernel recompilation. |

### SEC()

Low-level section annotation.

- It tells the kernel what kind of BPF program this is and where to attach it.

### BPF_FENTRY()

This is a helper macro defined by libbpf to make writing fentry programs easier and more consistent.

### Fentry programs

**`fentry`** is a type of eBPF program that hooks directly to the start (entry) of a kernel function.

- `fentry` runs at the very beginning of a kernel function.

### Congestion Window (cwnd)

cwnd is a sender-side limit ****on the amount of un-ACKed data “in flight.”

How cwnd behaves:

- **Start:** cwnd = IW (initial window, e.g. 10 MSS).
    - cwnd = 10 means 10 MSS
        - MSS: Maximum Segment Size
- **Growth:** cwnd increases (exponentially in slow start, linearly in congestion avoidance) as long as ACKs return without loss.
- **Reaction:** when loss or ECN is detected, cwnd is reduced (e.g. halved in Reno, cubic backoff in CUBIC, pacing target in BBR).
- **Balance:** cwnd stabilizes near the bandwidth-delay product (BDP) of the path:
    
    $$
    cwnd \approx \text{bandwidth} \times \text{RTT}
    $$
    
    - RTT (Round-Trip Time): The delay of a packet making a round trip between client and server.
    

### Ingress

- Incoming Traffic

### Egress

- Outgoing Trafic

### ECN (Explicit Congestion Notification)

Normally, when a router is congested and its buffers fill, it drops packets.

ECN is a way for the network to say “slow down” without dropping your packets.

- ECN has 2 flags:
    
    
    | Bits | Meaning |
    | --- | --- |
    | 00 | Not ECN-capable (Not-ECT) |
    | 10 | ECN-capable Transport (ECT(0)) |
    | 01 | ECN-capable Transport (ECT(1)) |
    | 11 | **Congestion Experienced (CE)** |

**With ECN enabled:**

- Packets carry **ECN bits** in the IP header (2 bits).
- Routers that experience congestion can **mark** (set a bit) in these packets instead of discarding them.
- The receiving endpoint sees the ECN mark and notifies the sender via the TCP header’s **ECN-Echo (ECE)** flag.
    - TCP two 1-bit flags for ECN:
        - **ECE (ECN-Echo)**: Experienced congestion on a packet which from sender
        - **CWR (Congestion Window Reduced)**: The sender got the signal and it reduced its sending rate
- The sender then reduces its sending rate.

**Benefits**

- Reduces packet loss
- Improves latency
- Works well with Active Queue Management (AQM) algorithms such as **RED (Random Early Detection)** or **CoDel**.

### DDos (Distributed Denial of Service)

A cyberattack where many systems (often compromised machines in a botnet) flood a target server or network with huge amounts of traffic, overwhelming it so legitimate users cannot access it.

### Spoofing

Forging or falsifying identifying information so that traffic appears to come from a trusted source when it does not.

**Types**

- **IP spoofing**: attacker forges the source IP address of packets.
- **Email spoofing**: fake "From" address in emails.
- **DNS spoofing**: returning fake DNS results to redirect traffic.

### CDRT (Commutative Replicated Data Type)

**Types:**

- State-based (Convergent) CDRTs (CvRDTs):
    - Each replica periodically sends its full state to others.
    - A merge function combines two states deterministically.
- Operation-based (Commutative) CDRTs (CmRDTs):
    - Each replica broadcasts operations instead of full states.
    - Operations are designed to be commutative, so order doesn’t matter.

| Name | Type | Description |
| --- | --- | --- |
| **G-Counter** | State-based | Grow-only integer counter |
| **PN-Counter** | State-based | Positive/Negative counter |
| **G-Set** | State-based | Grow-only set |
| **OR-Set** | Operation-based | Add/remove set with unique tags |
| **LWW-Register** | State-based | Last-write-wins register |
| **RGA** | Operation-based | Replicated growable array (for text editing) |
| **Map CRDT** | Mixed | Replicated key–value store built from CRDTs |