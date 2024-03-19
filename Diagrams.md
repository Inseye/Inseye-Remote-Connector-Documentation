# About

This document describes user and system sequence diagram.
Diagrams include behavior of the user and both presentation layer and business layer of all the systems taking part in communication.
Solid arrows represent communication between user and applications parts.
Dotted arrow signify communication between and within applications. 


## System initialization

**Goal: connect eye tracker with desktop service and make it manageable through desktop service**

```mermaid
sequenceDiagram
autonumber
actor User
participant Android Service
box Green Android Views
participant Initial View
participant Accept Incoming Connection View
end
participant Local Area Network
participant Desktop Service
box Gray Desktop Views
participant Looking For Remote View
end

par Initialize desktop service
    User ->> Desktop Service: Starts application
    activate Looking For Remote View
    Looking For Remote View ->> User: Informs that desktop service is waiting for remote eye tracker connection
    activate Looking For Remote View
    loop Until user decides to connect
        Desktop Service --) Local Area Network: Queries for publishing<br/> android services
        Looking For Remote View ->> User: Shows android services<br/>ready to serve eye tracker
    end
and Initialize android service 
    User ->> Android Service: starts application
    activate Initial View
    User ->> Initial View: Wants to connect with desktop service
    Android Service --) Local Area Network: Publishes information about<br/>service offering eye tracker
end

User ->> Looking For Remote View: Decides which remote<br/>service to connect to
Desktop Service --) Android Service: Sends credentials for connection
deactivate Initial View
activate Accept Incoming Connection View
Accept Incoming Connection View ->> User: Show information about desktop service attempting to connect
alt user accepts
    User ->> Accept Incoming Connection View: Accepts desktop service offer
else user declines
    break
        User ->> Accept Incoming Connection View: Declines desktop service offer
        Looking For Remote View ->> User: Inform that connection was refused from Android device
    end 
else operation timeout 
    break
        Looking For Remote View ->> User: Inform that connection failed due to timeout
    end
end

deactivate Accept Incoming Connection View
activate Initial View
Android Service --) Desktop Service: Sends authorization token
Desktop Service --) Android Service: Establishes secure<br/>connection with credentials
break Failed to establish connection
Looking For Remote View ->> User: Inform user that services<br/>failed to connect
end
Looking For Remote View ->> User: Informs user that<br/>connection was established
deactivate Initial View
deactivate Looking For Remote View
```

## System information


**Goal: provide user with information about eye tracker state, allow initiating calibration, allow user to disconnect from service**

```mermaid
sequenceDiagram
autonumber
actor User

participant Desktop Service
participant Android Service
par Show Eye Tracking State
    loop Status Loop
        Desktop Service ->> User: Show current state
        Android Service -->> Android Service: Wait for State change
        Android Service -->> Desktop Service: Send updated state
    end
and Disconnect From Service
User ->> Desktop Service: Decides to disconnect
Desktop Service -->> Android Service: Disconnects from

end
```