# About

This document describes how client applications communicate with Windows service. Communication protocol specification is named as **Service-Client communication protocol** or **SCCP**.

## Versions
Latest component version:
+ Windows service: 0.0.1
+ Native library: 0.1.0
+ SCCP: 0.1.0


# Introduction 
The SCCP protocol is intended for use where Inseye Desktop Service wants to share Inseye eye-tracker gaze data with client application on the same machine.

SCCP describes behavior of all parties participating in communication, medium used to exchange information and structure of shared information.

# Communication mediums

Windows service and client application communicate with each another through **named pipe** and **shared memory**.


### Named-pipes
Windows service creates named pipe server under path known by all parties - `\\.\pipe\inseye.desktop-service`. Desktop service is supposed to start serving as soon as service application is stared and close the pipe server when the service is shutting down.

Communication through named pipe is bidirectional based on request and response communication model. 
Only client is able to initialize request.
Client cannot start new request until request was not finished.
Server has unspecified time to response to client request.

### Shared memory
Windows service creates single shared memory file with a valid name on demand when required. 
Shared memory is shared between all clients.

Communication through shared memory is unidirectional - only service can send data through shared memory to clients. 
Service doesn't wait for client reads instead each client must implement algorithm that tracks shared memory content and adjusts its behavior to current state of shared memory. 

## Behavioral specification 

### Establishing connection with desktop service by client

Connection between client and service is established when client manages to connect to named pipe server.
Client can immediately start new request.

### Closing connection with desktop service by client

To break connection with server client must only close named pipe handle. 
No other action must be performed by the client. 

### Closing connection with client by desktop service

Current SCCP standard doesn't specify how client should handle case where service pipe server closes connection. 

## Structural specification

### Named pipe messages

Each named pipe message must be **at most 1024 bytes long**.
Messages are not fixed length, both communication parties are expected to accept messages shorted than 1024 bytes. 
Each message starts with 4 byte message type serialized as little endian 32 byte int.
Message type numbers remain reserved when message type is removed from specification and must not be reused when new message is added to specification.
All numerical values are serialized in little-endian format.

Known message types are:

| Message Type        | Numerical value `uint32` |
| ------------------- | ------------------------ |
| ServiceInfoRequest  | 0                        |
| ServiceInfoResponse | 1                        |

#### Service Info Request

Sent by the client to obtain information about communication protocol and shared memory path.

Structure:

| Name         | Offset | Type     | Value     |
| ------------ | ------ | -------- | --------- |
| Message Type | 0      | `uint32` | 0 (const) |

#### Service Info Response

Sent by the service in response to [Service Info Request](#service-info-request). Total message length depends on how long is shared memory path string in serialized message.

Structure:

| Name               | Offset | Type         | Value                                         |
| ------------------ | ------ | ------------ | --------------------------------------------- |
| Message Type       | 0      | `uint32`     | 1 (const)                                     |
| Version Major      | 4      | `uint32`     | communication protocol major version          |
| Version Minor      | 8      | `uint32`     | communication protocol minor version          |
| Version Patch      | 12     | `uint32`     | communication protocol patch version          |
| Shared memory path | 16     | `byte[1008]` | path to shared memory file managed by service, null terminated string. |



### Structure of shared memory

Shared memory is made of a single memory-mapped file that is split into two parts.
The first part (starting at first byte of the mapped file) of the shared memory is the header containing communication metadata and the second part (offset depends on header size) is a buffer where gaze data is written.

All numerical data written to shared memory is in little endian format.

### Shared memory header

Shared memory header is at least 12 bytes long and SCCP version is serialized as three unsigned 32-bit integers, where first one is major, seconds is minor and third is patch number. Communication protocol in header is serialized in format compatible with [SemVer](https://semver.org/). First 12 bytes are packed with alignment of 1 byte and are explicitly written in little endian format. Changes to the structure of first 12 bytes of memory are considered breaking and must **always** keep information about incompatibility that can be properly parsed/interpreted by any already released version of client library.

Published by service in version [Unreleased]
Supported by shared library 0.0.1.
Can be used to communicate with service in version from 0.0.1 to 1.0.0.
The structure is packed with 1 byte alignment.
`const` - data is written when service creates shared memory buffer and then it's not modified until service and all clients free shared memory.

Structure:

| Type           | Name            | Description                                                                                |
| -------------- | --------------- | ------------------------------------------------------------------------------------------ |
| `const uint32` | version_major   |                                                                                            |
| `const uint32` | version_minor   |                                                                                            |
| `const uint32` | version_patch   |                                                                                            |
| `const uint32` | header_size     | Length of the header in bytes.                                                             |
| `const uint32` | buffer_size     | Length of the whole mapped file in bytes.                                                  |
| `const uint32` | sample_size     | Length of single sample stored in buffer in bytes.                                         |
| `uint32`       | samples_written | Counter incremented by 1 each time new data is written to shared<br>buffer by the service. |

### Data buffer layout
Data buffer contains n-last samples of recorded gaze data in the form of circular buffer. Each sample is packed with 1 byte alignment all numerical values are serialized in little endian format.

Last (oldest) sample in the buffer is always considered 'hot' (in the process of being edited by service) and should not be read by the clients - clients could perform read while service is editing the same memory area.

Data samples may not be placed in the buffer one after another, client should be able to calculate each data samples beginning and end position in the mapped memory using `sample_size`, `buffer_size` `header_size` variables.

Published by service in version [Unreleased]
Introduced in shared library 0.0.1.
Can be used to communicate with service in version from 0.0.1 to 1.0.0.
The structure is packed with 1 byte alignment.

Structure:

| Type     | Name        | Description                                                                 |
| -------- | ----------- | --------------------------------------------------------------------------- |
| `uint64` | time        | Milliseconds since UTC Epoch. Timestamp of when data was recorder on device. |
| `float`  | left_eye_x  | Horizontal left eye position, radians.                                      |
| `float`  | left_eye_y  | Vertical eye position, radians.                                             |
| `float`  | right_eye_x | Horizontal right eye position, radians.                                     |
| `float`  | right_eye_y | Vertical right eye position, radians.                                       |
| `uint32` | gaze_event  | Gaze event id.                                                              |

