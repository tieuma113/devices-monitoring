# 1. Overview

This document describes the software module decomposition of the Embedded Sensor Collector System. Each module is designed to encapsulate a specific responsibility and interact through well-defined interfaces. The design follows the Single Responsibility Principle to ensure maintainability and testability.

# 2. Module List

|Module Name                 |Description|
|----------------------------|-----------------|
|sensor.cpp/h                |Simulates a sensor process that generates and sends data|
|collector.cpp/h             |Receives data from sensors via IPC and writes to shared memory|
|logger.cpp/h                |Reads from shared memory and logs to a file|
|ipc_xxx.cpp/h               |IPC abstraction layer (FIFO, MQ, socket...)|
|signal_handler.cpp/h        |Handles SIGINT and SIGHUP signals|
|config.cpp/h                |Parses configuration file|
|utils.cpp/h                 |Utility functions: timestamp, random, etc.|

# 3. Module Interfaces

3.1 sensor.h

void sensor_process(int id);
std::string generate_sensor_data(int id);

3.2 collector.h

void collector_main();
bool receive_sensor_data(std::string& out_data);

3.3 logger.h

void logger_run();
void write_log(const std::string& message);

3.4 ipc_interface.h

```c++ class IPCInterface {
public:
    virtual bool send(const std::string& data) = 0;
    virtual bool receive(std::string& out) = 0;
    virtual void init() = 0;
    virtual void cleanup() = 0;
    virtual ~IPCInterface() {}
};
```

3.5 signal_handler.h

void setup_signal_handlers();
void handle_sigint(int sig);
void handle_sighup(int sig);

3.6 config.h

void load_config(const std::string& path);
std::string get_ipc_type();
int get_sensor_interval();

3.7 utils.h

std::string current_timestamp();
int random_between(int min, int max);

# 4. Data Structures

```c++ // Common data packet structure
struct SensorData {
    int sensor_id;
    std::string timestamp;
    std::string value;
};
```

Can be update in the future

