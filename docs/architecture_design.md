# 1. Architecture Overview

The system follows a modular, multi-process architecture. Each sensor is implemented as a separate process, while the collector is the central coordinator responsible for receiving and distributing sensor data. Logging is performed by a dedicated logger component which may be a thread or a process depending on the implementation. All inter-process communication is handled through a configurable IPC abstraction layer. This architecture enables high scalability, testability, and fault isolation between components.

# 2. Process/Thread Design

- **Sensor Process**: Each sensor runs as an independent process. It periodically generates data and sends it to the collector through an IPC mechanism.
- **Collector Process**: A central process responsible for receiving data from sensors and writing it to shared memory.
- **Logger Thread/Process**: Either a thread within the collector or a separate process that reads data from shared memory and writes it to a log file.
- **Signal Handler**: Integrated into the main process to capture SIGINT (shutdown) and SIGHUP (reload configuration), and coordinate cleanup across modules.

# 3. IPC Abstraction Layer

The system supports multiple IPC methods: FIFO, pipe, message queue, socket, and shared memory. All IPC implementations inherit from a common interface that defines basic methods: `send()`, `receive()`, `init()`, and `cleanup()`. This allows runtime switching of IPC type based on configuration, and enables easier testing and extension.

```cpp
class IPCInterface {
public:
    virtual bool send(const std::string& data) = 0;
    virtual bool receive(std::string& data_out) = 0;
    virtual void init() = 0;
    virtual void cleanup() = 0;
    virtual ~IPCInterface() {}
};
```

# 4. Shared Memory & Synchronization
Shared memory is used for high-speed data transfer between the **collector** and the **logger**. To avoid race conditions, a binary semaphore (mutex-style) is used to control access. Only one process/thread can write or read from the shared memory region at any given time.

`sem_wait()` before accessing

`sem_post()` after done

This guarantees consistency of data and prevents log corruption.

# 5. Configuration Handling
System parameters such as IPC type, number of sensors, and sensor interval are defined in an external configuration file (e.g., config.txt). This file is parsed during system startup. The system supports dynamic configuration reload via the SIGHUP signal. This enables tuning of runtime behavior without restarting the system.