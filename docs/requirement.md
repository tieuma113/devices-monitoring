# 1. Project Title:

Embedded Sensor Collector System

# 2. Purpose:

This project is designed to practice key concepts in Linux system programming, including inter-process communication (IPC), multithreading, and multiprocess architecture. It simulates a sensor data collection system, where multiple sensor processes generate data and send it to a central collector for processing and logging. The goal is to build hands-on experience with real-world system components commonly found in embedded and automotive applications.

# 3. Scope:

The system consists of the following main components:

- **Sensor Simulator**: simulates multiple sensors generating random data at fixed intervals.
- **Collector**: receives data from all sensors using Linux IPC mechanisms and stores it temporarily in shared memory.
- **Logger**: reads data from shared memory and writes it to a log file, along with system notifications and error messages.
- **Signal Handler**: handles termination and configuration reload requests (e.g., via SIGINT, SIGHUP).
- **IPC Layer**: abstracts the communication mechanism, supporting FIFO, pipe, message queue, socket, or shared memory.

# 4. Functional Requirements

| ID   | Requirement                                | Description |
|------|--------------------------------------------|-------------|
| FR01 | Sensor data generation                     | Each sensor must generate data periodically and send it to the collector process. |
| FR02 | IPC communication                          | The system must use Linux IPC mechanisms to transfer data between processes. |
| FR03 | Data collection                            | The collector must receive and store sensor data into shared memory. |
| FR04 | Logging                                    | The logger must write all received data, system events, and errors into a log file. |
| FR05 | Signal handling                            | The system must handle SIGINT and SIGHUP signals to gracefully terminate or reload configuration. |
| FR06 | Configurable IPC type                      | The IPC type used must be selectable via a configuration file. |
| FR07 | Multiple sensor support                    | The system must support multiple sensor processes running concurrently. |
| FR08 | Timestamp in log entries                   | Each log entry must include a timestamp indicating when the event occurred. |
| FR09 | Fault recovery for unresponsive modules    | The system must detect unresponsive modules and automatically restart them if they fail to respond within a predefined timeout. |
