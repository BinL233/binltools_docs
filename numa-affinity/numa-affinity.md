## **`restricted` Policy**

### **Test Case**

- CPU: 10 | MEM: 5Gi | GPU: 2
    - Result: `TopologyAffinityError`
- CPU: 10 | MEM: 5Gi | GPU: 1
    - Result: CPU & GPU 共同亲和 1 个 numa 节点。
- CPU: 20 | MEM: 5Gi | GPU: 1
    - Result: `TopologyAffinityError`
- CPU: 20 | MEM: 5Gi | GPU: 2
    - Result: CPU & GPU 共同亲和 2 个 numa 节点。
- CPU: 20 | MEM: 5Gi | GPU: 3
    - Result: `TopologyAffinityError`

### **Conclusion**

restricted Policy 可以绑多 numa 节点，但需要满足：

- GPU 亲和 numa 节点的数量 == CPU 亲和 numa 节点的数量。

## **`best-effort` Policy**

### **Test Case**

- CPU: 10 | MEM: 5Gi | GPU: 2
    - `CPU: 1-5,65-69`
    - `GPU: 0-7,64-71,8-15,72-79`
    - Result: CPU & GPU 共同亲和 1 个 numa 节点，剩下一个 GPU 额外亲和 1 个 numa 节点。
- CPU: 10 | MEM: 5Gi | GPU: 1
    - `CPU: 1-5,65-69`
    - `GPU: 0-7,64-71`
    - Result: CPU & GPU 共同亲和 1 个 numa 节点。
- CPU: 20 | MEM: 5Gi | GPU: 1
    - `CPU: 1,2,8-15,65,66,72-79`
    - `GPU: 8-15,72-79`
    - Result: CPU & GPU 共同亲和 2 个 numa 节点，剩下一个 GPU 额外亲和 1 个 numa 节点。
- CPU: 20 | MEM: 5Gi | GPU: 2
    - `CPU: 1,2,8-15,65,66,72-79`
    - `GPU: 0-7,64-71,8-15,72-79`
    - Result: CPU & GPU 共同亲和 2 个 numa 节点。
- CPU: 40 | MEM: 5Gi | GPU: 2
    - `CPU: 1-4,8-23,65-68,72-87` -> `1-4,8-15,16-23,65-68,72-79,80-87`
    - `GPU: 16-23,80-87,8-15,72-79`
    - Result: CPU & GPU 共同亲和 2 个 numa 节点，剩下一个 CPU 额外亲和 1 个 numa 节点。
- CPU: 20 | MEM: 5Gi | GPU: 3
    - `CPU: 1,2,8-15,65,66,72-79`
    - `GPU: 16-23,80-87,0-7,64-71,8-15,72-79`
    - Result: CPU & GPU 共同亲和 2 个 numa 节点。剩下一个 GPU 额外亲和 1 个 numa 节点。
- CPU: 40 | MEM: 5Gi | GPU: 3
    - `CPU: 1-4,8-23,65-68,72-87` -> `1-4,8-15,16-23,65-68,72-79,80-87`
    - `GPU: 16-23,80-87,0-7,64-71,8-15,72-79`
    - Result: CPU & GPU 共同亲和 3 个 numa 节点
- CPU: 1（目前 CPU 占用 126/128） | MEM: 5Gi | GPU: 1
    - `CPU: 64`
    - `GPU: 0-7,64-71`
    - Result: CPU & GPU 共同亲和 1 个 numa 节点。

### **Conclusion**

1. 首先尝试确保亲和 1 个 numa 节点。
2. 如果不行，绑多个 numa 节点。
3. CPU / GPU 可以额外绑 numa 节点，不会报错。
4. 可以绑单个 CPU。
