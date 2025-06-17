# 1. Overview

This system simulates a multi-sensor environment where each sensor process generates data periodically. The data is collected by a central collector process, which stores it and forwards it to a logger for monitoring and storage. All communication between components is handled through Linux Inter-Process Communication (IPC) mechanisms. The system is designed to run entirely in user space on a Linux environment, with a focus on modularity and real-time data flow.

# 2. Components
- **Sensor Simulation**:  
  A sensor is implemented as a separate process that periodically generates simulated data (e.g., temperature, humidity). The data values are randomly generated and sent to the collector through a configurable IPC mechanism (such as FIFO, message queue, or socket).

- **Collector**:  
  The collector is a central process that receives data from multiple sensor processes via IPC. It may perform basic validation or monitoring logic, and then forwards the data to a shared memory region for the logger to access. It acts as the core of the data pipeline.

- **Logger**:  
  The logger is a process that reads sensor data from shared memory and writes it to log files. Each log entry includes a timestamp for traceability. The logger ensures that both data and system-level events (such as errors or configuration changes) are persistently recorded for later analysis.

- **Signal Handler**:  
  The signal handler is responsible for intercepting system signals such as `SIGINT` and `SIGHUP`. It ensures that the system can gracefully terminate or reload its configuration at runtime. On receiving a termination signal, it cleans up resources like IPC channels, shared memory, and child processes to prevent memory leaks or zombie processes.

# 3. Data Flow

1. Each **Sensor** process periodically generates random data.
2. The data is sent to the **Collector** through a configurable IPC mechanism (e.g., FIFO, message queue, or socket).
3. The **Collector** receives the data, performs optional validation, and writes it into a **shared memory** region.
4. The **Logger** reads the data from shared memory, appends a timestamp, and writes it to a **log file**.
5. A **Signal Handler** listens for system signals (e.g., SIGINT, SIGHUP) to gracefully terminate or reload configuration. It coordinates cleanup across all modules.

```
+-------------+         IPC          +------------+        Shared Memory        +----------+
|   Sensor 1  |  ------------------> |            |  -------------------------> |          |
|   Sensor 2  |  ------------------> |  Collector |                             |  Logger  |
|   Sensor N  |  ------------------> |            |  |          |
+-------------+                     +------------+                              +----------+
                                        ^   |
                                        |   | signal (reload)
                                        |   v
                                   +---------------+
                                   | Signal Handler|
                                   +---------------+
```

# 4. IPC Usage

The system supports multiple Linux IPC mechanisms to enable communication between sensor processes and the collector:

- **FIFO (Named Pipe)**: Simple and easy to use for one-way communication. Suitable for basic testing.
- **Pipe (Unnamed Pipe)**: Used when sensor processes are created using `fork()` and communication is required between parent and child.
- **Message Queue (SysV)**: Provides structured messages and supports message prioritization.
- **Shared Memory**: Used between the collector and logger for fast data access. Requires synchronization via semaphore.
- **Socket (UNIX domain)**: Suitable for flexible N-to-1 communication, especially when sensors are created as independent processes.
  
The IPC type between sensor and collector is configurable through a setting in the configuration file. All IPC implementations follow a common interface to allow easy switching and modular design.

# 5. Signal Behavior

The system responds to two primary POSIX signals:

- **SIGINT** (`Ctrl+C` or kill):  
  When this signal is received, the signal handler initiates a graceful shutdown process. It sends termination signals to all sensor and collector processes, releases IPC resources (e.g., message queues, shared memory), and ensures that the logger flushes and closes the log file.

- **SIGHUP** (`kill -HUP`):  
  This signal triggers a configuration reload. The system re-reads the configuration file to update parameters such as IPC type or sensor interval. Active modules may restart or update their behavior without requiring a full shutdown.

Additional signals (e.g., `SIGTERM`, `SIGUSR1`) can be implemented in future versions for finer control if needed.
