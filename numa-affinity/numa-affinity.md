## Testing Machine

| CPU | GPU | Memory | NUMA Nodes |
| --- | --- | --- | --- |
| 128 | 8 | 1024Gi | 8 |

## Restricted Policy

### Test Case
| CPU | MEM | GPU | Result |
| --- | --- | --- | --- |
| 10 | 5Gi | 1 | CPU & GPU admitted to the same NUMA node |
| 10 | 5Gi | 2 | `TopologyAffinityError`|
| 10 | 200Gi | 1 | CPU & GPU admitted to the same NUMA node |
| 20 | 5Gi | 1 | `TopologyAffinityError` |
| 20 | 5Gi | 2 | CPU & GPU admitted to same NUMA nodes |
| 20 | 5Gi | 3 | `TopologyAffinityError` |

### Conclusion
- Memory is **not affective** to NUMA nodes affinity.
- Topology manager allows task to bind multi-nodes. But this has the requirement: **The number of node which GPU has affinity with == the number of node with CPU has affinity with**.

## Best-effort Policy

### Test Case
1. Single Task：

    | CPU | MEM | GPU | CPU Affinity | GPU Affinity |
    | --- | --- | --- | --- | --- |
    | 10 | 5Gi | 1 | 1-5,65-69 | 0-7,64-71 |
    | 10 | 300Gi | 1 | 1-5,65-69 | 0-7,64-71 |
    | 10 | 5Gi | 2 | 1-5,65-69 | 0-7,64-71,8-15,72-79 |
    | 20 | 5Gi | 1 | 1,2,8-15,65,66,72-79 | 8-15,72-79 |
    | 20 | 5Gi | 2 | 1,2,8-15,65,66,72-79 | 0-7,64-71,8-15,72-79 |
    | 20 | 5Gi | 3 | 1,2,8-15,65,66,72-79 | 16-23,80-87,0-7,64-71,8-15,72-79 |
    | 40 | 5Gi | 2 | 1-4,8-23,65-68,72-87 = 1-4,8-15,16-23,65-68,72-79,80-87 | 16-23,80-87,8-15,72-79 |
    | 40 | 5Gi | 3 | 1-4,8-23,65-68,72-87 = 1-4,8-15,16-23,65-68,72-79,80-87 | 16-23,80-87,0-7,64-71,8-15,72-79 |
    | 1 (Current CPU usage: 126/127) | 5Gi | 1 | 64 | 0-7,64-71 |

2. Double Tasks：

    | CPU | MEM | GPU | CPU Affinity | GPU Affinity |
    | --- | --- | --- | --- | --- |
    | 10 | 5Gi | 1 | 1-5,65-69 | 0-7,64-71 |
    | 10 | 5Gi | 1 | 8-12,72-76 | 8-15,72-79 |

    | CPU | MEM | GPU | CPU Affinity | GPU Affinity |
    | --- | --- | --- | --- | --- |
    | 10 | 5Gi | 2 | 1-5,65-69 | 0-7,64-71,8-15,72-79 |
    | 10 | 5Gi | 2 | 6-10,70-74 | 24-31,88-95,16-23,80-87 |:wq


    | CPU | MEM | GPU | CPU Affinity | GPU Affinity |
    | --- | --- | --- | --- | --- |
    | 20 | 5Gi | 1 | 1,2,8-15,65,66,72-79 | 8-15,72-79 |
    | 20 | 5Gi | 1 | 3-7,16-20,64,67-71,80-83 | 0-7,64-71 |

    | CPU | MEM | GPU | CPU Affinity | GPU Affinity |
    | --- | --- | --- | --- | --- |
    | 20 | 5Gi | 2 | 1,2,8-15,65,66,72-79 | 0-7,64-71,8-15,72-79 |
    | 20 | 5Gi | 2 | 16-25,80-89 | 24-31,88-95,16-23,80-87 |
    
    | CPU | MEM | GPU | CPU Affinity | GPU Affinity |
    | --- | --- | --- | --- | --- |
    | 20 | 5Gi | 1 | 1,2,8-15,65,66,72-79 | 0-7,64-71 |
    | 10 | 5Gi | 1 | 16-20,80-84 | 16-23,80-87 |

    | CPU | MEM | GPU | CPU Affinity | GPU Affinity |
    | --- | --- | --- | --- | --- |
    | 40 | 5Gi | 2 | 1-4,8-23,65-68,72-87 = 1-4,8-15,16-23,65-68,72-79,80-87 | 16-23,80-87,0-7,64-71 |
    | 20 | 5Gi | 3 | 5-7,24-30,64,69-71,88-93 | 8-15,72-79,48-55,112-119,32-39,96-103 |

    | CPU | MEM | GPU | CPU Affinity | GPU Affinity |
    | --- | --- | --- | --- | --- |
    | 20 | 5Gi | 2 | 1,2,8-15,65,66,72-79 | 0-7,64-71,8-15,72-79 |
    | 40 | 5Gi | 2 | 3-7,16-30,64,67-71,80-93 = 3-7,16-23,24-30,64,67-71,80-87,88-93 | 16-23,80-87,32-39,96-103 |

### Conclusion
| Condition | 1 task |  2 tasks |
| --- | --- | --- |
| CPU not across nodes, GPU not across nodes | Same nodes | Same nodes |
| CPU across nodes，GPU not across nodes | Same nodes, CPU affinize extra nodes | Mostly random |
| CPU not across nodes，GPU across nodes | Same nodes, CPU affinize extra nodes | Mostly random |
| CPU across nodes, GPU across nodes (Same number of nodes which CPU & GPU has affinity with) | Same nodes | Same nodes |
| CPU across nodes，GPU across nodes (Different number of nodes which CPU & GPU has affinity with) | Same nodes, CPU/GPU affinize extra nodes | Mostly random |

1. Memory is **not affective** to NUMA nodes affinity.
2. When the number of NUMA nodes which CPU and GPU has affinity with are different, CPU and GPU can affinize same NUMA nodes **only in the first task**. It is mostly random in other tasks.
