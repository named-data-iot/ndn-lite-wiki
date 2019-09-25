Overview
============

The NDN-Lite library implements the Named Data Networking Stack with the high-level application support functionalities and low-level OS/hardware adaptations for Internet of Things (IoT) scenarios.


### NDN-LITE is 
* Designed to run on **resource constrained devices**
* A framework for network systems with built-in highly level system modules for security bootstrapping, service discovery, access control, etc.
* A development platform for IoT application developers


### Design Principles
* A **System framework**
* Universality for different platforms (that supports **raw C**)
* With minimal memory overhead and zero dynamic memory allocation
* Minimal memory and better efficiency
* Runs application and NDN forwarder within the same process
* Fit into IoT platforms that only support single thread/process

Relations to Existing NDN Packages
-----------------------------------
NDN-LITE is designed to be an **IoT system framework** that can work with all IoT platforms in **raw C**.

With system framework principle in mind, NDN-LITE has ** application support** to take care of bootstrapping, resource discovery, and access control. 

### Differences from NFD + NDN-CXX
* Designed to be an IoT system framework: bootstrapping, access control, etc.
* Lightweight forwarder with reduced functionalities: no support of forwarding hint, RIB management, etc.
* Lightweight encoding/decoding: reduced memory overhead
    
#### Differences from NDN-RIOT
* The original goal to renew/improve NDN-RIOT codebase.
* Designed not only for RIOT but as a framework that can work with all IoT platforms **(raw C)**
* Doesn't support multi-process design of forwarder and applications

#### Differences from CCN-LITE
 * Designed to be an IoT system framework

Where we are now
-------------------

* High-level support has not been fully realized. Now developers still need to manually configure face and work with Interest/Data
* Can be adapt to **any platform that supports C**. So far the supported platforms are:
    * nRF52 boards
    * Rasberry PIs

* Some system modules are missing now:
    * Access Control (design outdated), Pub Sub
 
Our team is actively maintaining the codebase and exploring the design and implementation for the remaining issues.

Architecture
------------

The architecture of NDN-Lite library is independent of the OS and the development kit.
The exposed API makes the library easily pluggable to IoT Software Development Kit (SDK) and IoT Operating Systems, by creating a thin adaptation layer between the platform and the NDN-Lite.

The library was designed to provide more than core NDN network stack.
The library allows applications to directly integrate supporting functionalities including Access Control, Service Discovery, Schematized Trust and so on.

The following block diagram presents the architecture of the project.


![](https://github.com/Zhiyi-Zhang/ndn_standalone/wiki/iot-framework.jpg)


### Application Support Component
* **Boostrapping**: To obtain identity certificate and trust anchor.

* **Service Discovery**: To discover available services around and advertise one's own services.

* **Pub Sub**(in progress): To subscribe to services and publish new content to one's own services.


* **Access Control**:To protect data confidentiality and grant access rights to authorized identities. Old version is now removed. New version work in progress.

* **Schematized Trust**: To only execute commands from authorized identities.



### Network Component
* **Forwarder**: In the same process as the application. **No memory copy** in packet processing. Content Store is currently not supported given the limited RAM of most IoT deviceS.

* **NDN Packet Encoding**: Interfaces for Interest Data packet encoding/decoding.

* **Face**: Abstract face for network interface. Will be implemented by platform-dependent adaptations

### Utilities Component
* **Key Storage**: In-memory storage to keep device's identity certificate, trust anchor.

* **Clock**: To provide time support, especially for network timeout. It has two unified interfaces that need to be implemented
 * ``time()`` to get the time of steady clock
 * ``delay()`` to sleep for a period of time

* **Message Queue**: An event queue which keeps event from both NDN forwarder and an application. This event queue will be run by a main loop of the program.

* ** Fragmentation**: To fragment/assemble NDN's wire format packets into/from byte chunks to fit link layer MTUs.

* **Crypto Support**: To support crypto, including SHA2 module, AES module, ECC module, HMAC module, and RNG module with default backend.




### Adaptation Component (Platform Dependent)
* **Face adaptation**: To extend the abstract face to real link layer interfaces supported by different platforms: IEEE 802.15.4, BLE, LoRa.

* **Clock adaptation**: Provide the realization of two unified interfaces

* **Crypto adaptation**: The adaptation is to provide a different realization of crypto modules to achieve stronger security (hardware TPM, hardware RNG) and higher efficiency (hardware ECC signing/verification). This is **optional**. If no crypto adaptation, NDN-LITR's default realization will be used.

Features
--------

In the current version, this library implements the following features:

NDN Layer:
* NDN Encoding Decoding, which is compatible with NDN TLV format 0.3.
* NDN IoT Forwarder, a lightweight forwarding module implementation for IoT. Will support Content Store in the near future.
* Abstract Face that can be inherited by OS/SDK specific adaptations.
* DirectFace to support app-forwarder communication for single-thread applications.
* DummyFace for test only.
* Fragmentation: reuses the [ndn-riot](https://github.com/named-data-iot/ndn-riot) fragmentation header (3 bytes header)
```
    0           1           2           3
    0 1 2  3    8         15           23
    +-+-+--+----+----------------------+
    |1|X|MF|Seq#|    Identification    |
    +-+-+--+----+----------------------+

    First bit: header bit, always 1 (indicating the fragmentation header)
    Second bit: reserved, always 0
    Third bit: MF bit, 1 indicating the last frame
    4th to 8th bit: sequence number (5 bits, encoding up to 31)
    9th to 24th bit: identification (2-byte random number)
```

Security:
* Crypto front end which supports OS/SDK specific crypto back-end implementation.
* A default pure-software crypto backend using tinycrypto and micro-ecc.
* Interest and Data signing and verification.
* AES Encrypted Content TLV for Data packet.

Application Support Layer:
* Ease-of-use Security Bootstrapping Module to achieve efficient and secured trust anchor installation and identity certificate issuance. Check the protocol details at [here](https://github.com/named-data-iot/ndn-lite/wiki/Security-Bootstrapping).
* Lightweight Name-based Access Control to provide data confidentiality and control of access to data. Check the protocol details at [here](https://github.com/named-data-iot/ndn-lite/wiki/Access-Control).
* Lightweight Service Discovery Protocol Module to enable an application provide services to the network or utilize existing services in the network system. Check the protocol details at [here](https://github.com/named-data-iot/ndn-lite/wiki/Service-Discovery).

Platform Adaptation:
* Nordic NRF 802154 Raw Driver Adaptation, including an adaptation layer and a face implementation called `ndn-nrf-802154-face`.
* Nordic SDK adaptation, including an adaptation layer and a face implementation called `ndn-nrf-ble-face`.

Code Base Structure
-----------------

* `./encode` directory: NDN packet encoding and decoding.
* `./forwarder` directory: NDN lightweight forwarder implementation and Network Face abstraction.
* `./face` directory: Dummy face
* `./security` directory: Security support.
* `./app-support` directory: Access Control, Service Discovery, and other high-level modules that can facilitate application development.
* `./util` directory: Tools used in forwarder, including time and message queue.

Instructions
------------
#### Before Starting
Please check if your platform are already supported by existing packages. If so, those would be helpful to start with and you can jump to Step4

* [nRF52 package](https://github.com/named-data-iot/ndn-iot-package-over-nordic-sdk)
* [POSIX package](https://github.com/named-data-iot/ndn-iot-package-over-posix)
* [esp8266 package (not official)](https://github.com/yoursunny/esp8266ndn)

**Warning**: You are only supposed to use the adaptation layer that is designed for your platform and the face implementation(s) that can work with your platform. Using incompatible adaptation and faces will lead to compilation failures.

#### 1. Download the NDN-Lite into your project
```
git clone https://github.com/named-data-iot/ndn-lite
```

#### 2. Preparation for platform adaptation
Check what communication technologies (e.g., Unix sockets, 802.15.4, BLE) would be used in your project. They will be adpated to the NDN-Lite face-forwarder system.

Check if you're going to other security-backend to replace default security-backend. Usually platform-specific backend are preferred, to achieve the best performance.

**Warning**: NDN-Lite does have default RNG, but it's fake a RNG which always fails. It is highly recommended to use platform-specific RNG replace it.

#### 3. Create platform adaptation
Create `./adaptation` folder, in the same directory of `./ndn-lite`, and put face implementation there. You need to implement a new Face structure in the header and source files. Developers need to make sure that the first structure element is an `ndn_face_intf` instance. In the source file, implement the functions defined in the `ndn_face_intf` structure. In the header file, create a construction function to create a face instance or get the face instance if the face is a singleton. The function pointers binding should also be done in this function. APIs are defined in `./ndn-lite/forwarder/face.h`

Inside `./adaptation` folder, create `./security` folder, and put implementation you want to replace corresponding default backend with there. Backend APIs are defined header files in `./ndn-lite/security` (HMAC, ECC, RNG, AES, SHA256). Don't forget to use `register_platform_security_init` to register your backend before security module initialization. Make sure `ndn_security_init` be after `register_platform_security_init`, otherwise your platform backend won't be loaded.

Existing examples:
* (Recommended) [POSIX Adaptation](https://github.com/named-data-iot/ndn-iot-package-over-posix): Unix Sockets, UDP, and POSIX-RNG backend.
* Nordic SDK Adaptation with `ndn-nrf-ble-face`: The files under `./adaptation/ndn-nrf-ble-adaptation/` is the adaptation layer and the files `./face/ndn-nrf-ble-face.c`, `./face/ndn-nrf-ble-face.h` are the face implementation.
* NRF 802154 driver Adaptation with `ndn-nrf-802154-face`: The files under `./adaptation/ndn-nrf-802154-driver/` is the adaptation layer and the files `./face/ndn-nrf-802154-face.c`, `./face/ndn-nrf-802154-face.h` are the face implementation.

#### 4. Add source files and headers into your IoT project's Makefile or equivalent.
An example would be:
```
# In the MakeFile
SRC_FILES := \
ndn-lite/encode/data.c \
ndn-lite/encode/decoder.c \
ndn-lite/encode/encoder.c \
ndn-lite/encode/interest.c \
... # and all the needed files under ./encode, ./security, ./forwarder, ./face, ./app-support, and ./adaptation

CFLAGS += -I/path/to/ndn-lite
```

#### 5. Compilation
If you're going to use RNG (from your platform backend) in your project, please `-DFEATURE_PERIPH_HWRNG` or do equivalent thing. Otherwise default ECC backend will still do deterministic signing, which is less strong than normal signing.
