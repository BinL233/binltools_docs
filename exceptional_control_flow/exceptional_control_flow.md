## Control Flow

From startup to shutdown, each CPU core simply reads and executes (interprets) a sequence of instructions, one at a time. This sequence is control flow.

## Exceptional Control Flow

Exists at all levels of a computer system

- Low level mechanisms
    - **Exceptions**
        - Change in control flow in response to a system event (i.e., change in system state)
        - Implemented using combination of hardware and OS software
- Higher level mechanisms
    - **Process context switch**
        - Implemented by OS software and hardware timer
    - **Signals**
        - Implemented by OS software
    - **Nonlocal jumps**: `setjmp()` and `longjmp()`
        - Implemented by C runtime library

### Exceptions
![exceptional](/exceptional_control_flow/images/exceptional.png)
*From CS:APP*

### Asynchronous Exceptions (Interrupts)

Examples:

- Timer interrupt
    - To let the kernel regain control ****of the CPU at regular intervals, even if a program doesn’t voluntarily give it up.
- I/O interrupt from external device
    - ctrl-C

### Synchronous Exceptions

- **Traps**
    - **Intentional**, set program up to “trip the trap” and do something
    - Examples: system calls, gdb breakpoints
        - System calls: All most like call functions, but executed by kernel
    - Returns control to “next” instruction
- **Faults**
    - **Unintentional** **but possibly recoverable**
    - Examples: page faults (recoverable), protection faults (unrecoverable), floating point exceptions
        - Page faults: That portion (page) of user’s memory is currently on disk
    - Either re-executes faulting (“current”) instruction or aborts
- **Aborts**
    - **Unintentional and unrecoverable**
    - Examples: illegal instruction, parity error, machine check
        - Parity error: Detected data corruption in RAM
        - Machine check: hardware-detected fatal error
    - Aborts current program

## Processes

Definition: A process is an instance of a running program.

- Process provides each program with two key abstractions:
    - Logical control flow
        - Each program seems to have exclusive use of the CPU
        - Provided by kernel mechanism called context switching
    - Private address space
        - Each program seems to have exclusive use of main memory
        - Provided by (kernel mechanism called) virtual memory
- Get process id
    - `pid_t getpid(void)` : Returns PID of current process
    - `pid_t getppid(void)` : Returns PID of parent process

### Lifecycle
- Running
    - Executing, or waiting to be executed and chosen to execute by the kernel
- Stopped
    - Execution is suspended and will not be scheduled until further notice
- Terminated
    - Stopped permanently

### Terminating Processes

- Receiving a signal whose default action is to terminate
- Returning from the main routine
- Calling the exit function
    - **`void exit(int status)`**
        - Terminates with an exit status
        - Convention: normal return status is 0, nonzero on error
        - Another way to explicitly set the exit status is to return an integer valuefrom the main routine
        - Call once but never returns.

### Creating Processes

Parent process creates a new running child process by calling fork.

- **`int fork(void)`**
    - Returns 0 to the child process, child’s PID to parent process
    - Child is almost identical to parent:
        - Child get an identical (but separate) copy of the parent’s virtual address space.
        - Child gets identical copies of the parent’s open file descriptors
        - Child has a different PID than the parent
    - The new child process will start from the next line of the `fork` line.
    - Call once but returns twice (parent & child)

### Concurrent Processes

- Each process is a logical control flow.
- Two processes run concurrently if their flows overlap in time
- Otherwise, they are sequential

### Context Switching

- Processes are managed by a shared chunk of memory resident OS code called the kernel.
    - Important: the kernel is not a separate process, but rather runs as part of some existing process.
- Control flow passes from one process to another via a context switch

### Reaping Child Processes

- When process terminates, it still consumes system resources. This is a “zombie”.
- Performed by parent on terminated child (using `wait` or `waitpid`)
    - **`int wait(int *child_status**)` : Synchronizing with Children
        - Suspends current process until one of its children terminates
        - Return value is the pid of the child process that terminated
        - Argument: A pointer to an `int` where the exit status of the child will be stored
    - **`pid_t waitpid(pid_t pid, int *status, int options)`** : Waiting for a Specific Process
        - Suspends current process until specific process terminates
        - `pid` = -1: Wait for any child
        - `pid` > 0: Wait for a specific child

### Function: `execve`

- **`int execve(char *filename, char *argv[], char *envp[])`** : Loading and Running Programs
    - Loads and runs in the current process
    - **Overwrites code, data, and stack**
    - Called once and never returns
    
    ```c
    int main(void) {
    	char *args[3] = {
    		"/bin/echo", "Hi 18213!", NULL
    	};
    		
    	execve(args[0], args, environ);
    	printf("Hi 18213!\n");
    	exit(0);
    }
    ```
    
    - This code prints `Hi 18213!` . This is because `execve` overwrites all code, data, and stack. So, only `exeve` prints.
    
    ```c
    int main(void) {
    	char *args[3] = {
    		"/bin/not_exist", "Hi 18213!", NULL
    	};
    		
    	execve(args[0], args, environ);
    	printf("Hi 18613!\n");
    	exit(0);
    }
    ```
    
    - This code prints `Hi 18613!` . This is because `execve` faces the error `/bin/not_exist` not exist. So it returns an error, and does not overwrite anything.

## Error Handling

- System return -1 on failure, and set errno.
- `strerror` : Turns errno codes into printable messages
    
    ```c
    
    int main() {
        FILE *f = fopen("doesnotexist.txt", "r");
        if (!f) {
            printf("Error: %s\n", strerror(errno));
        }
        return 0;
    }
    
    output: Error: No such file or directory
    ```
    
- `perror` (print error): Quick & easy error message printer
    ```c
    int main() {
        execve("/does/not/exist", NULL, NULL);
        perror("execve failed");
        return 1;
    }

    output: execve failed: No such file or directory
    ```

## Signal Handling

### Signal

- A signal is a small message that notifies a process that an event of some type has occurred in the system
- When the parent process receives a signal, the *signal handler* for that signal is called

### Signal Examples

- `SIGINT` (ctrl-C)
- `SIGTSTP` (ctrl-Z)
- `SIGCHLD` (raised when a child process stops or terminates)

### Sending Signals

- `kill()`
    - `SIGKILL` = forcefully terminate a process
    
    ```c
    pid_t pid = ...; // get the child process id
    kill(pid, SIGKILL);
    ```
    

### Process Group

Make sure to send the signal to the whole process group

- `getpgrp()` : Return process group of current process
- `setpgid()` : Change process group of a process

### Blocking Signals

- Why do we need to block signals?
    - Prevent race conditions
    - Ensure that job states are consistent
- When do we block signals?
    - Updating job state / adding / removing jobs
    - Forking new child processes
    - I/O redirection