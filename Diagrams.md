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
participant Android Service Initial View
participant Looking For Desktop View
end
participant Local Area Network
participant Desktop Service
box Gray Desktop Views
participant Looking For Remote View
end

par Initialize desktop service
    User ->> Desktop Service: Turns on
    Looking For Remote View ->> User: Informs that desktop service is waiting for remote eye tracker connection
    activate Looking For Remote View
    Desktop Service ->> Local Area Network: Publishes information about<br/>service waiting for eye tracker
and Initialize android service 
    User ->> Android Service: Turns on
    activate Android Service Initial View
    User ->> Android Service Initial View: Wants to connect with desktop service
    deactivate Android Service Initial View
    activate Looking For Desktop View
    loop Until user decides to connect
        Android Service --) Local Area Network: Queries for listening<br/> desktop services
        Looking For Desktop View ->> User: Shows desktop services<br/>waiting for eye tracker
    end
end

User ->> Looking For Desktop View: Decides which desktop<br/>service to connect to

Android Service --) Desktop Service: Sends credentials for connection
Desktop Service --) Android Service: Establishes secure<br/>connection with credentials
break When failed to establish connection
Looking For Desktop View ->> User: Inform user that services<br/>failed to connect [restart sequence from step 6]
end
Looking For Desktop View ->> User: Informs user that<br/>connection was established

deactivate Looking For Desktop View
activate Android Service Initial View
Looking For Remote View ->> User: Informs user that<br/>connection was established
deactivate Android Service Initial View
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