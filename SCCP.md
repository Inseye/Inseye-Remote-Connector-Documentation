# About

This document describes how client applications communicate with Windows service later mentioned as **Service-Client communication protocol** or **SCCP**.

## Versions
Latest component version:
+ Windows service: Unreleased
+ Native library: 0.0.1
+ SCCP: 0.0.1

# SCCP description

SCCP is made of structure of data shared and how the data is shared between Windows service and client application.
SCCP version numbering is compliant with [SemVer](https://semver.org/) numbering schema.

Windows application communicates with the client applications via file mapped as shared memory buffer in Windows service and its clients processes.

Name of the mapped file is well-known (`Local\\Inseye-Remote-Connector-Shared-Memory`) by both parties. Shared memory is created by the service, clients are only allowed open created file.

The communication is single-sided - only the service is writing to the shared memory, clients have no means of communicating with the service. 

Through the shared memory service shares metadata about itself and the communication channel such as communication protocol version, size of the buffer; and the eye tracking data. Service doesn't wait for the clients reads.

## Structure of shared memory

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
|----------------|-----------------|--------------------------------------------------------------------------------------------|
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
|----------|-------------|-----------------------------------------------------------------------------|
| `uint64` | time        | Millisconds since UTC Epoch. Timestamp of when data was recorder on device. |
| `float`  | left_eye_x  | Horizontal left eye position, radians.                                      |
| `float`  | left_eye_y  | Vertical eye position, radians.                                             |
| `float`  | right_eye_x | Horizontal right eye position, radians.                                     |
| `float`  | right_eye_y | Vertical right eye position, radians.                                       |
| `uint32` | gaze_event  | Gaze event id.                                                              |

