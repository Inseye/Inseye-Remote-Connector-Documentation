# System state diagram

*created by [Mateusz Chojnowski](mailto:mateusz.chojnowski@inseye.com)*

This document describes user and system sequence diagram.
Diagrams include behavior of the user and both presentation layer and simplified business layer of all the systems taking part in communication.
Solid arrows represent communication between user and applications parts.
Dotted arrow signify communication between and within systems.

# Big picture

```mermaid
stateDiagram-v2
direction LR
EstablishingConnection: <a href="#establishing-connection">Establishing Connection</a>
state fork <<fork>>
state if_calibration <<choice>>
SystemInformation: <a href="#system-information">System Information</a>
Calibration: <a href="#calibration-procedure">Calibration Procedure</a>
ClientManagement: <a href="#client-management">Client Management</a>
RespondToClient: <a href="#handle-client-applications">Handle client applications</a>
[*] --> fork
fork --> ClientManagement
fork --> RespondToClient
fork --> EstablishingConnection
EstablishingConnection --> EstablishingConnection: user declines connection
EstablishingConnection --> EstablishingConnection: operation timeout
EstablishingConnection --> EstablishingConnection: failed to establish connection
EstablishingConnection --> if_calibration: user accepts
if_calibration --> Calibration: if calibration is ongoing
if_calibration --> SystemInformation: default
SystemInformation --> Calibration: starts calibration
SystemInformation --> EstablishingConnection: disconnects from service
Calibration --> EstablishingConnection: disconnect from service
Calibration --> SystemInformation: user decides to abort calibration
Calibration --> SystemInformation: default
```

## Establishing connection

**Goal: connect eye tracker with desktop service and make it manageable through desktop service**

Parties:
  + User: VR headset/desktop service end user
  + Android Service: application on android device that manages eye tracker and makes data accessible remotely
    + Initial: view after loading the application
    + Accept Incoming Connection: popup/view that will allow user accept incoming service connection
  + Desktop Service: Windows application that communicates with Android Service and provides gaze data access to desktop applications
    + Looking For Remote Service: view that allow user connect to remote service

```mermaid
sequenceDiagram
autonumber
actor User
participant Android Service
box Green Android Views
participant Initial
participant Accept Incoming Connection
end
participant Local Area Network
participant Desktop Service
box Gray Desktop Views
participant Looking For Remote Service
end

par Initialize desktop service
    activate Looking For Remote Service
    Looking For Remote Service ->> User: Informs that desktop service is waiting for remote eye tracker connection
    activate Looking For Remote Service
    loop Until user decides to connect
        Desktop Service --) Local Area Network: Queries for publishing<br/> android services
        Looking For Remote Service ->> User: Shows android services<br/>ready to serve eye tracker
    end
and Initialize android service 
    User ->> Android Service: starts application
    activate Initial
    User ->> Initial: Wants to connect with desktop service
    Android Service --) Local Area Network: Publishes information about<br/>service offering eye tracker
end

User ->> Looking For Remote Service: Decides which remote<br/>service to connect to
Desktop Service --) Android Service: Sends credentials for connection
deactivate Initial
activate Accept Incoming Connection
Accept Incoming Connection ->> User: Show information about desktop service attempting to connect
alt user accepts
    User ->> Accept Incoming Connection: Accepts desktop service offer
else 
    break user declines connection
        User ->> Accept Incoming Connection: Declines desktop service offer
        Looking For Remote Service ->> User: Inform that connection was refused from Android device
    end 
else 
    break operation timeout 
        Looking For Remote Service ->> User: Inform that connection failed due to timeout
    end
end

deactivate Accept Incoming Connection
activate Initial
Android Service --) Desktop Service: Sends authorization token
Desktop Service --) Android Service: Establishes secure<br/>connection with credentials
break failed to establish connection
    Looking For Remote Service ->> User: Inform user that services<br/>failed to connect
end
Looking For Remote Service ->> User: Informs user that<br/>connection was established
deactivate Initial
deactivate Looking For Remote Service
```

## System information


**Goal: provide user with information about eye tracker state, allow initiating calibration, allow user to disconnect from service, show what client applications are using eye tracking data currently**

Parties:
  + User: VR headset/desktop service end user
  + Android Service: application on android device that manages eye tracker and makes data accessible remotely
  + Desktop Service: Windows application that communicates with Android Service and provides gaze data access to desktop applications
    + Status view: view that allows user to get information about current system state 


State:
- eye tracker state
- remote service name and address
- list of desktop client applications waiting for eye tracker data

```mermaid
sequenceDiagram
autonumber
actor User
box Gray Desktop Views
participant Status View
end
participant Desktop Service
participant Android Service
activate Status View
par show eye tracking state
    loop Status Loop
        Status View ->> User: Show current state
        Android Service -->> Android Service: Wait for State change
        Android Service -->> Desktop Service: Send updated state
    end
and client applications state
    loop Status loop
        Status View ->> User: Show desktop applications<br/>that are accessing eye tracker
        Desktop Service -->> Desktop Service: Wait for accessing<br/>applications change
    end
and disconnect from service
    break disconnect from service
        alt User choice
            User ->> Status View: Decides to disconnect
            Desktop Service -->> Android Service: Disconnects from
        else Connection error
            Status View ->> User: Inform about <br/>connection errors
        end
    end
and starts calibration
    break
        User ->> Status View: Initiates calibration
    end
end
deactivate Status View
```

## Calibration procedure

**Goal: Inform user about current calibration state, allow user cancelling ongoing calibration**

Parties:
  + User: VR headset/desktop service end user
  + Android Service: application on android device that manages eye tracker and makes data accessible remotely
  + Desktop Service: Windows application that communicates with Android Service and provides gaze data access to desktop applications
    + Calibration view: shows current state of calibration (in progress, finished successfully, finished failed), allows user to abort ongoing calibration

```mermaid
sequenceDiagram
autonumber
actor User
box Gray Desktop Views
participant Calibration View
end
participant Desktop Service
participant Android Service
activate Calibration View
par show calibration state
    loop until calibration finishes
        Calibration View ->> User: Show current state
        Android Service -->> Android Service: Wait for State change
        Android Service -->> Desktop Service: Send updated state
    end
    Calibration View ->> User: Inform user about<br/>calibration result
and
    break disconnect from service
        Calibration View ->> User: Inform that calibration<br/>failed due to connection<br/>issues
    end
and 
    break  user decides to abort calibration
        User ->> Calibration View: Aborts calibration
    end
end
deactivate Calibration View
```

## Handle client applications

**Goal: allow user respond to desktop applications that attempts to access gaze data**

Parties:
  + User: VR headset/desktop service end user
  + Desktop application: application on desktop that connects to desktop service to gain access to eye tracking data
  + Desktop Service: Windows application that communicates with Android Service and provides gaze data access to desktop applications
    + Acceptance View: view/popup that shows in response to client application that neither have already been granted or revoked permissions to access gaze data

```mermaid
sequenceDiagram
autonumber
actor User
box Gray Desktop Views
participant Acceptance View
end
participant Desktop Service
participant Desktop Application
loop
    Desktop Service --) Desktop Service: Wait for client application
    Desktop Application --) Desktop Service: Attempt to connect
    alt unknown application
        activate Acceptance View
        Acceptance View ->> User: Inform about new application<br/>trying to access gaze data
        alt granted
            User ->> Acceptance View: Grant permission
            Desktop Service --) Desktop Application: Send connection details
        else dismissed
            User ->> Acceptance View: Don't grant permission
            Desktop Service --) Desktop Application: Inform that permission<br/>was not granted
        end 
        deactivate Acceptance View
    else known application
        Desktop Service --) Desktop Application: Send connection details
    end
end
```

## Client management
**Goal: allow user to access information on which desktop applications have granted access to gaze data, revoke existing permissions**

Parties:
  + User: VR headset/desktop service end user
  + Desktop Service: Windows application that communicates with Android Service and provides gaze data access to desktop applications
    + Granted Accesses: list of applications that had been granted or revoked permissions to access gaze data 

```mermaid
sequenceDiagram
autonumber
actor User
box Gray Desktop Views
participant Granted Accesses 
end
participant Desktop Service
activate Granted Accesses
Granted Accesses ->> User: Show list of applications<br/>with and without permissions
loop until user decides to exit
    User ->> Granted Accesses: Selects application<br/>to change permissions
    Granted Accesses -->> Desktop Service: Changes permissions
end
User ->> Granted Accesses: Close
deactivate Granted Accesses
```