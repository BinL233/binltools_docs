## 1\. Shallow Copy vs. Deep Copy

  * **Shallow Copy:** Only adds a pointer pointing to existing memory (allocates memory for the pointer itself).
  * **Deep Copy:** Adds a pointer *and* allocates new memory for the data, making the new pointer point to this new memory (allocates memory for both the pointer and the value).

## 2\. Reading and Writing in Different Channel States

| Channel State | Read | Write |
| :--- | :--- | :--- |
| **`nil`** | Blocks forever | Blocks forever |
| **Closed** | Returns zero value (immediately) | **Panic** |
| **No Buffer** | Blocks until data is written | Blocks until data is read |
| **No Data** | Blocks until new data is written | Normal |
| **Full Buffer** | Normal | Blocks until data is read |

## 3\. `len` and `cap` for Channel, Array, and Slice

  * **Channel**
      * `len`: Number of elements currently in the channel buffer.
      * `cap`: Total capacity of the channel buffer.
  * **Array**
      * `len`: Length of the array.
      * `cap`: Capacity of the array (always equals `len`).
  * **Slice**
      * `len`: Length of the slice.
      * `cap`: Capacity of the slice (`cap` ≥ `len`).
      * *Note: `cap` == `len` upon initialization if not specified.*

## 4\. Channel Internal Principles

**Structure:**

```go
type hchan struct {
    qcount   uint           // Number of elements in the queue
    dataqsiz uint           // Size of the circular queue (buffer capacity)
    buf      unsafe.Pointer // Pointer to the circular queue (buffer)
    elemsize uint16         // Size of a single element
    closed   uint32         // Flag indicating if channel is closed
    elemtype *_type         // Element type
    sendx    uint           // Send index in circular queue (write position)
    recvx    uint           // Receive index in circular queue (read position)
    recvq    waitq          // Queue of goroutines waiting to receive
    sendq    waitq          // Queue of goroutines waiting to send
    lock     mutex          // Mutex lock to protect concurrent access
}
```

**How it works:**

  * **Buffered Channel:** Elements are written to the buffer and read sequentially. When `sendx` reaches the end of the buffer, it wraps around to the beginning (Ring Buffer). The same applies to `recvx`.
      * If the buffer is **full**, the sender Goroutine blocks and enters the `sendq` until a receiver reads data and frees up space.
      * If the buffer is **empty**, the receiver Goroutine blocks and enters the `recvq` until a sender writes data.
  * **Unbuffered Channel:** Sending and receiving must be synchronous.
      * When a sender Goroutine performs `<-chan`, if no receiver is waiting, it blocks in `sendq`.
      * Similarly, a receiver blocks in `recvq` if there is no sender.

## 5\. Scenarios Triggering a Panic in Channels

  * Closing a `nil` Channel.
  * Closing an already closed Channel.
  * Writing data to an already closed Channel.

## 6\. The Role of Channels in Go

  * Channels are the mechanism for communication between Goroutines (CSP Model).

## 7\. Slice Internal Principles

```go
type SliceHeader struct {
    Data uintptr // Pointer to the underlying array
    Len  int     // Current length of the slice
    Cap  int     // Current capacity of the slice
}
```

## 8\. Slice Expansion Rules

  * If original capacity \< 1024: The Slice expands to **2x** the original size.
  * If original capacity ≥ 1024: The Slice expands to **1.25x** the original size.

## 9\. Map Internal Principles

```go
type hmap struct {
    count     int     // Number of elements in the map
    flags     uint8   // State flags (e.g., writing status)
    B         uint8   // Log base 2 of bucket count (2^B buckets)
    noverflow uint16  // Number of overflow buckets
    hash0     uint32  // Hash seed to prevent hash collision attacks
    buckets   unsafe.Pointer // Pointer to the main bucket array
    oldbuckets unsafe.Pointer // Pointer to old buckets (used during expansion)
    nevacuate uintptr  // Index of buckets already evacuated during expansion
    extra     *mapextra // Optional fields (e.g., overflow buckets)
}

type bmap struct {
    tophash [8]uint8     // Stores high 8 bits of hash for up to 8 keys
    keys    [8]keytype   // Array of keys
    values  [8]valuetype // Array of values
    overflow *bmap       // Pointer to an overflow bucket
}
```

## 10\. Map Expansion Mechanism

  * **Triggers:**

    1.  Load factor \> 6.5 (Average of \> 6.5 KV pairs per bucket).
    2.  The number of overflow buckets reaches $2^{min(15, B)}$.

  * **Incremental Expansion:**

      * Creates new buckets (count is 2x the original).
      * Because there is a lot of old data, copying is triggered incrementally on every map access.
      * Each time, 2 key-value pairs are evacuated from `oldbuckets` to the new `buckets`.
      * `oldbuckets` is released after full migration.

  * **Same-Size Expansion (Compacting):**

      * Does not increase capacity (bucket count remains the same).
      * Rearranges loose key-value pairs to make bucket usage more efficient (handling deleted holes).

## 11\. Map CRUD Processes

  * **Lookup Process:**

    1.  Calculate hash of the key.
    2.  Use **low bits** of hash `mod hmap.B` to determine bucket position.
    3.  Use **high bits** of hash to search in the `tophash` array.
    4.  If `tophash[i]` matches, retrieve the key at that index.
    5.  If not found in the main bucket, search sequentially through overflow buckets.
    6.  *(If map is expanding, prioritize searching in `oldbuckets`)*.

  * **Add Process:**

    1.  Calculate hash.
    2.  Determine bucket using low bits.
    3.  Check `tophash` using high bits.
    4.  If key exists, update value directly.
    5.  If not found, find an empty slot in the bucket to insert.
    6.  *(If map is expanding, add directly to new `buckets`, but lookup still starts in `oldbuckets`)*.

  * **Delete Process:**

    1.  Calculate hash.
    2.  Determine bucket.
    3.  Check `tophash`.
    4.  If key exists, delete the key-value pair.
    5.  If not found, do nothing.

## 12\. Difference between `make` and `new`

  * **`make`**:
      * Used to allocate memory for `slice`, `map`, and `channel`.
      * Returns the **value** (initialized).
  * **`new`**:
      * Can allocate memory for any type.
      * Returns a **pointer** to the variable (zeroed memory).

## 13\. What is a Goroutine?

  * A lightweight thread.
  * Not scheduled by the OS.
  * The scheduler is provided by the application (Runtime).
  * The scheduler maps Goroutines to OS threads based on strategy.
  * The Goroutine scheduler is provided by the `runtime` package.
  * Mapping relationship between Goroutines and Kernel Threads: **M:N**.

## 14\. Advantages of Goroutines

  * Works in User Mode, reducing the overhead of context switching.
  * Switching Goroutines only requires saving/restoring a few registers and the stack pointer, avoiding Kernel Mode switching.

## 15\. GMP Model

**Key Entities:**

  * **G (Goroutine):** The coroutine.
  * **M (Machine):** OS Work Thread (scheduled by OS).
  * **P (Processor):** Context for running Go code and scheduling Gs.

**GMP Rules:**

1.  **M** must hold a **P** to execute code.
2.  **M** can be blocked by system calls.
3.  The count of **M** is usually slightly larger than **P** (to handle runtime tasks).
4.  The count of **P** is determined at startup (default = CPU cores), modifiable via `GOMAXPROCS`.
5.  **P** has a local runqueue. New Gs join the local queue; if full, they go to the Global Queue.
6.  **P** periodically checks the Global Queue to fetch Gs.

**Queue Rotation:**

  * Each **P** maintains a queue of **G**s.
  * **P** schedules **G**s onto **M** sequentially.
  * After a **G** finishes, **P** schedules the next **G**.
  * **P** periodically checks the Global Queue to prevent starvation.

**System Calls:**

  * If **G0** makes a system call, **M0** releases **P**.
  * **M1** acquires **P** and executes the remaining Gs in the queue.
  * **M0** continues executing **G0** (blocked).
  * After **G0** returns, it tries to acquire a generic **P**. If none available, **G0** goes to the Global Queue, and **M0** sleeps in a cache pool.

**Work Stealing:**

  * If a **P** has no Gs to schedule, it steals half the Gs from another **P**'s queue.

**Preemptive Scheduling:**

  * Prevents a single **G** from running too long.
  * The scheduler monitors execution time; if too long and others are waiting, it pauses the current **G** and schedules another.

## 16\. GC (Garbage Collection) Strategies

1.  **Reference Counting (e.g., Swift):** Maintains a count for every object. When count is 0, recycle.
      * *Pros:* Fast recycling.
      * *Cons:* Cannot handle circular references; performance cost to maintain counters.
2.  **Mark-Sweep (e.g., Go - Tricolor):** Traverses from root objects. Marks reachable as "referenced", recycles the rest.
      * *Pros:* Solves circular references.
      * *Cons:* Requires STW (Stop The World).
3.  **Generational Collection (e.g., Java):** Divides space by object lifespan (Eden, Survivor, Old).
      * *Pros:* High performance.
      * *Cons:* Complex algorithm.

## 17\. Go GC Trigger Timing

1.  **Memory Threshold:** When current memory allocation reaches the threshold (`Last GC amount * Growth Rate`).
2.  **Periodic:** Triggered at least every 2 minutes.
3.  **Manual:** `runtime.GC()`.

## 18\. Tricolor Marking (Go)

  * **Grey:** Object is in the marking queue waiting to be scanned.
  * **Black:** Marked and scanned. `gcmarkBits` is 1 (Will not be cleaned in this GC).
  * **White:** Not marked. `gcmarkBits` is 0 (Will be cleaned in this GC).

## 19\. WaitGroup

**Flow:**

1.  Before starting a goroutine, use `Add(int)` to set the number of tasks.
2.  Use `Wait()` to block the main routine until the counter is 0.
3.  Use `Done()` inside the goroutine to decrement the counter by 1.
4.  When counter reaches 0, the blocked goroutine wakes up.

**Example:**

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done() // Notify WaitGroup task is done when function exits
	fmt.Printf("Worker %d starting\n", id)

	// Simulate time-consuming operation
	time.Sleep(time.Second)

	fmt.Printf("Worker %d done\n", id)
}

func main() {
	var wg sync.WaitGroup

	// Start 5 Goroutines
	for i := 1; i <= 5; i++ {
		wg.Add(1) // Increment counter for every Goroutine started
		go worker(i, &wg)
	}

	// Wait for all Goroutines to finish
	wg.Wait()
	fmt.Println("All workers done!")
}
```

## 20\. Types of Locks in Go & Scenarios

  * **Mutex (`sync.Mutex`):** Ensures only one goroutine can access a resource at a time.
  * **Read-Write Lock (`sync.RWMutex`):** Based on Mutex. Suitable for scenarios with **many reads and few writes**.
  * **WaitGroup:** Suitable for scenarios where you need to wait for multiple concurrent operations to complete before proceeding.