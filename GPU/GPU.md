## Memory Copy

Memory will copied from CPU to the off-chip memory in GPU first.

## Streaming Multiprocessor (SM)

- SMs in GPU are like cores in CPU
    - Compare to CPU cores, SMs are simple, weak processors, but can execute more threads in parallel.
- SM contains multiple identical processing units (single-precision cores, math cores, tensor core…).
- Units execute same instruction at the same time.
- Multiple islands in an SM.

![gpu-image](/GPU/images/image.png)

## Threads & Thread Blocks

- GPU threads are SW threads.
- Threads organized into a 3D grid (x, y, z)

## Warps

A warp is the basic execution unit in CUDA, consisting of threads that execute SIMULTANEOUSLY on a Streaming Multiprocessor (SM).

- Warp is a HW grouping of GPU SW threads (**32** threads for Nvidia, 32/64 for AMD).
    - All 32 threads in a warp execute the same instruction at the same time on a SM.
- Multiple Warps per SM.
- Multiple Warps can be maped to an island.
- Pipelining can be achieved with multiple warps.

## **CUDA Execution Model: Grid, Blocks, and Threads**

1. **Grid** → Made up of **blocks**.
2. **Block** → Made up of **threads**.
    - Block will be assigned to SM. 1 block can only be assigned to 1 SM.
3. **Thread** → Executes the actual computations.

## Banks

A bank refers to a memory unit in shared memory, where multiple banks allow parallel access to shared memory by different threads.

They are physically stored in the Shared Memory of each Streaming Multiprocessor (SM). (32 banks per SM)

- Size: 4 bytes wide (means is can contain many 4 bytes data. It can span the capacity.)
- **Bank Conflict:**
    - When two or more threads in a warp **access different addresses in the same bank**, the requests are **serialized**, reducing performance.
    
    | **Scenario** | **Bank Conflict?** | **Performance Impact** |
    | --- | --- | --- |
    | Each thread accesses a different bank | No | Fast (Parallel Access) |
    | All threads access the same address | No (Broadcast) | Fast (Optimized for CUDA) |
    | Multiple threads access different addresses in the same bank | Yes | Slow (Serialization) |

## CUDA Variables

- `threadIdx.x` `threadIdx.y` `threadIdx.z` : Thread index within a block (from `0` to `blockDim.x/y/z - 1`)
- `blockIdx.x` `blockIdx.y` `blockIdx.z` : Block index within the grid (from `0` to `gridDim.x/y/z - 1`)
- `blockDim.x` `blockDim.y` `blockDim.z`: Number of threads per x/y/z dimension of block
- `gridDim.x` `gridDim.y` `gridDim.z` : Number of blocks per x/y/z dimension of grid

Global Thread ID = `blockIdx.x * blockDim.x + threadIdx.x` 

## Stream

A GPU stream is a series of kernels.

Multiple streams can compute on multiple kernels simultaneously.

- Kernels need to launch on different streams.

## Cuda Functions

```cpp
// Allocate memory on the GPU (device memory)
// devPtr: Pointer to the allocated GPU memory. (buffer)
// size: Number of bytes to allocate.
cudaError_t cudaMalloc(void **devPtr, size_t size);

// Transfers data between host (CPU) and device (GPU) memory.
// dst: Destination pointer.
// src: Source pointer.
// count: Number of bytes to copy.
// kind: Direction of the copy (see below).
// cudaMemcpyHostToDevice → Copies from CPU (host) to GPU (device).
// cudaMemcpyDeviceToHost → Copies from GPU (device) to CPU (host).
// cudaMemcpyDeviceToDevice → Copies from one GPU memory location to another.
// cudaMemcpyHostToHost → Copies between two CPU memory locations (rarely needed in CUDA)
cudaError_t cudaMemcpy(void *dst, const void *src, 
	size_t count, cudaMemcpyKind kind);
	
// CUDA function that blocks the CPU until all previously issued GPU tasks 
// (kernels, memory copies, etc.) are completed.
cudaDeviceSynchronize();  

// Blocks the CPU only for a specific stream, 
// ensuring that all previously launched operations in 
// that stream have completed before continuing.
// Other streams can continue execution in parallel while 
// the CPU waits for this specific stream.
cudaStreamSynchronize(cudaStream_t stream);

// Create Stream. 
cudaStreamCreateWithFlags(&streams[i], cudaStreamNonBlocking);
```