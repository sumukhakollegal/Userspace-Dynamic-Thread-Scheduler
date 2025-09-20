# Userspace Dynamic Thread Scheduler

A cooperative thread scheduler library implemented in C that provides thread management capabilities similar to POSIX pthreads, enabling concurrent execution of multiple user threads within a single process.

## 🎯 Project Overview

This project implements a **userspace thread scheduler** that manages multiple threads cooperatively without relying on the operating system's kernel-level threading. The scheduler provides an API similar to POSIX pthreads but operates entirely in user space, making it an excellent educational tool for understanding thread management, context switching, and cooperative multitasking.

## 🏗️ Architecture & Components

### Core Components

#### 1. **Thread Management (`scheduler.h`, `scheduler.c`)**
- **Thread Structure**: Each thread contains:
  - `jmp_buf env`: Context for saving/restoring thread state
  - `status`: Thread state (INITIAL, RUNNING, SLEEPING, TERMINATED)
  - `stack`: Memory management for thread stack
  - `fnc`: Thread function pointer
  - `arg`: Thread arguments
  - `next`: Circular linked list pointer

#### 2. **System Utilities (`system.h`, `system.c`)**
- **Memory Management**: Page-aligned memory allocation
- **Timing Functions**: Microsecond-precision sleep
- **Safe String Operations**: Buffer-safe string handling
- **Error Handling**: Comprehensive error reporting macros

#### 3. **Main Application (`main.c`)**
- **Thread Creation**: Demonstrates creating multiple threads
- **Thread Functions**: Sample thread implementations
- **Scheduler Execution**: Entry point for the scheduler

### Key Features

- 🔄 **Cooperative Multitasking**: Threads yield control voluntarily
- ⏰ **Timer-based Preemption**: Automatic thread switching via signals
- 🧵 **Circular Thread Queue**: Round-robin scheduling algorithm
- 💾 **Stack Management**: Individual stack allocation per thread
- 🔧 **Context Switching**: Using `setjmp()`/`longjmp()` for state preservation

## 🚀 API Reference

### Thread Creation
```c
int scheduler_create(scheduler_fnc_t fnc, void *arg);
```
- Creates a new user thread
- `fnc`: Thread function to execute
- `arg`: Arguments passed to thread function
- Returns: 0 on success, -1 on error

### Scheduler Execution
```c
void scheduler_execute(void);
```
- Starts the scheduler and executes all created threads
- Blocks until all threads complete
- Not re-entrant

### Thread Yielding
```c
void scheduler_yield(int sig);
```
- Yields CPU to next thread in queue
- Called automatically by timer interrupt
- Can be called manually for cooperative yielding

## 🔧 Build & Execution

### Prerequisites
- GCC compiler
- POSIX-compliant system (Linux/macOS)
- Make build system

### Building
```bash
make clean
make
```

### Running
```bash
./cs238
```

### Cleanup
```bash
make clean
```

## 📊 Thread States

| State | Description |
|-------|-------------|
| `STATUS_` | Initial state, thread created but not started |
| `STATUS_RUNNING` | Currently executing |
| `STATUS_SLEEPING` | Yielded, waiting for next turn |
| `STATUS_TERMINATED` | Completed execution |

## 🔄 Scheduling Algorithm

### Round-Robin with Cooperative Yielding
1. **Thread Creation**: New threads added to circular queue
2. **Initialization**: First thread gets stack setup and starts execution
3. **Context Switching**: 
   - Timer interrupt triggers `scheduler_yield()`
   - Current thread state saved via `setjmp()`
   - Next thread in queue selected
   - Context restored via `longjmp()`
4. **Termination**: Completed threads marked as terminated

### Memory Layout
```
Thread Stack (1MB + Page Alignment)
├── Stack Memory (1MB)
└── Page-aligned Stack Pointer
```

## 🎮 Example Usage

```c
#include "scheduler.h"

void my_thread(void *arg) {
    const char *name = (const char *)arg;
    for (int i = 0; i < 100; i++) {
        printf("%s: iteration %d\n", name, i);
        us_sleep(40000);  // 40ms delay
    }
}

int main() {
    // Create multiple threads
    scheduler_create(my_thread, "Thread-1");
    scheduler_create(my_thread, "Thread-2");
    scheduler_create(my_thread, "Thread-3");
    
    // Start cooperative execution
    scheduler_execute();
    return 0;
}
```

## 🔍 Technical Implementation Details

### Context Switching Mechanism
- Uses `setjmp()`/`longjmp()` for lightweight context switching
- Stack pointer manipulation for thread stack management
- Signal handling for timer-based preemption

### Memory Management
- Page-aligned stack allocation (1MB per thread)
- Automatic cleanup on thread termination
- Memory leak prevention through proper deallocation

### Error Handling
- Comprehensive error checking and reporting
- Graceful failure handling with proper cleanup
- Debug macros for development assistance

## 🎓 Educational Value

This project demonstrates:
- **Thread Management**: Creating, scheduling, and terminating threads
- **Context Switching**: Manual state preservation and restoration
- **Cooperative Multitasking**: Voluntary CPU yielding
- **Memory Management**: Stack allocation and alignment
- **Signal Handling**: Timer-based preemption
- **Data Structures**: Circular linked list implementation

## 📝 Author

- **Sumukha Kollegal**

---

*This project is part of the CS 238P Operating Systems course at UC Irvine, focusing on userspace thread management and cooperative multitasking concepts.*
