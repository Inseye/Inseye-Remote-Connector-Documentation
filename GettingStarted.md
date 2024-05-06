# Getting started 

*created by [Mateusz Chojnowski](mailto:mateusz.chojnowski@inseye.com)*

1. Connect headset to local wireless network. Make sure that VR headset is in the same local network as PC.

2. Enable remote streaming in android service.
- Open the Android service.
![](./assets/getting-started/pic1.jpeg)
- Go to settings.
![](./assets/getting-started/pic2.jpeg)
- Enable `OpenXR PCVR remote server`.
![](./assets/getting-started/pic3.jpeg)

3. Prepare desktop service.
- Download and run [Inseye-Remote-Connecto-Desktop](https://github.com/Inseye/Inseye-Remote-Connector-Desktop/releases).
- Connect to remote android service by double clicking on remote service offer.
![](./assets/getting-started/image001.png)
- Check eye tracker state and optionally calibrate eye tracker before use.
![](./assets/getting-started/image002.png)

4. Run client application that connects to Windows service through [the library](https://github.com/Inseye/Inseye-Remote-Connector-Lib/releases).

