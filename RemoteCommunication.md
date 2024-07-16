# Remote Communication

*created by [Mateusz Chojnowski](mailto:mateusz.chojnowski@inseye.com)*

## Version: 0.1.0

This document specifies how Android Service communicates with Windows Service.


## Scope of communication

Windows Service and Android Service are meant to communicate within local area wireless network.

## Communication protocol

Windows Service communicates android service through gRPC overt HTTPS. 
Android Service serves as a server where Windows Service is a client that connects to it. 

## Service discoverable

Windows Service connects to Android Service by knowing address and port of gRPC server. 
Android Service advertises server endpoint in local are network through DDNS-SD protocol.

The service is advertised under arbitrary name (default is `inseye-remote-service`), with service type of `_inseye-et._tcp`, socked address, port number and additional attribute `version` which contains remote communication protocol version as sem-ver string.


### Encryption

Communication is encrypted with TLS with self signed certificates generated with Android Service.

The certificates are not validated by the client thus client is vulnerable server authentication forging. 