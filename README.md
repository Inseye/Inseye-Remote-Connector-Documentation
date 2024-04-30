# About

Remote connector is a project that aims delivering gaze data from Inseye eye tracker to desktop application wireless in a manner that is usable by non-technically educated end user.

# Project structure

Project is made of multiple parts:
- [Remote-Connector-Documentation](https://github.com/Inseye/Inseye-Remote-Connector-Documentation) this repository, 
- [Remote-Connector-Android (private)](https://github.com/Inseye/Remote-Connector-Android) an Android library intended to be part of [Inseye-Android-Service (private)](https://github.com/Inseye/Inseye-Android-Service) that exposes API ways to discover the API for clients in local network
- [Remote-Connector-API](https://github.com/Inseye/Inseye-Remote-Connector-API) a repository with network public API definitions
- [Remote-Connector-Desktop](https://github.com/Inseye/Inseye-Remote-Connector-Desktop) an desktop (Windows for now) application that connects to [Inseye-Android-Service (private)](https://github.com/Inseye/Inseye-Android-Service) using [Remote-Connector-API](https://github.com/Inseye/Inseye-Remote-Connector-API) and exposes gaze data to other desktop application through IPC communication
- [Remote-Connector-Lib](https://github.com/Inseye/Inseye-Remote-Connector-Lib) a native library with C and C++ headers used to communicate with [Remote-Connector-Desktop](https://github.com/Inseye/Inseye-Remote-Connector-Desktop) within the same system

# Architecture Overview

::: mermaid 
flowchart TD
subgraph android_device ["Android Device"]
subgraph "Android Service"
android_plugin["Android Plugin<br>(Remote-Connector-Android)"]
end
end

subgraph "Network API<br>(Remote-Connector-API)"
grpc(["gRPC"])
sd(["DNS-SD<br>(service discoverability,<br>protocol version)"]) 
end

subgraph desktop_device ["Desktop"]
desktop_service["Desktop Service<br>(Remote-Connector-Service)"]
subgraph "SCCP"
shared_memory(["Shared memory"])
named_pipes(["Named pipes"])
end
subgraph "Client Application"
client_application["Library<br>(Remote-Connector-Lib)"]
end
end


sd --> desktop_service
grpc --> android_plugin
android_plugin --- sd
grpc --> desktop_service
desktop_service --- shared_memory --> client_application
named_pipes --> desktop_service
named_pipes --> client_application


classDef red fill:#fc2c03
class android_device red
class desktop_device red
:::

## Articles
- [SCCP](./SCCP.md) describes communication protocol between desktop service and desktop client application
- [System State Diagram](./SystemStateDiagram.md) provides board overview of states and flow of application and user