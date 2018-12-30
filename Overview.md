Overview
============

The NDN-Lite library implements the Named Data Networking Stack with the high-level application support functionalities and low-level OS/hardware adaptations for Internet of Things (IoT) scenarios.


Architecture
------------

The architecture of NDN-Lite library is independent of the OS and the development kit.
The exposed API makes the library easily pluggable to IoT Software Development Kit (SDK) and IoT Operating Systems, by creating a thin adaptation layer between the platform and the NDN-Lite.

The library was designed to provide more than core NDN network stack.
The library allows applications to directly integrate supporting functionalities including Access Control, Service Discovery, Schematized Trust and so on.

The following block diagram presents the architecture of the project.

![](https://github.com/Zhiyi-Zhang/ndn_standalone/wiki/ndn-iot-framework.pdf)


Features
--------

In the current version, this library implements the following features:

NDN Layer:
* NDN Encoding Decoding, which is compatible with NDN TLV format 0.3.
* NDN IoT Forwarder, a lightweight forwarding module implementation for IoT. Will support Content Store in the near future.
* Abstract Face that can be inherited by OS/SDK specific adaptations.
* DirectFace to support app-forwarder communication for single-thread applications.
* DummyFace for test only.

Security:
* Crypto front end which supports OS/SDK specific crypto back-end implementation.
* A default pure-software crypto backend using tinycrypto and micro-ecc.
* Interest and Data signing and verification.
* AES Encrypted Content TLV for Data packet.

Application Layer:
* Access Control
* Service Discovery

Platform Adaptation:
* Nordic NRF 802154 Raw Driver Adaptation, including an adaptation layer and a face implementation called `ndn-nrf-802154-face`.
* Nordic SDK adaptation, including an adaptation layer and a face implementation called `ndn-nrf-ble-face`.


Instructions
------------


#### 1. Download the NDN-Lite into your IoT project
```
git clone https://github.com/Zhiyi-Zhang/ndn_standalone.git
```

#### 2. Select the adaptation layer and network face implementation that can work with your project's platform.

**Warning**: You are only supposed to use the adaptation layer that is designed for your platform and the face implementation(s) that can work with your platform. Using incompatible adaptation and faces will lead to compilation failures.

If there is no existing adaptation layer and faces for your current development platform, you can easily create a new adaptation layer with your customized face implementation following the instructions described in later sections.

#### 3. (Optional) Select the security backend that can work with your project's platform.

You can config which security/crypto backend to use by defining the macro value to be 1 with compiler's c flags.
You can check `./security/config.h` for details.
By default, you can use the software backend provided by NDN-Lite.

However, to achieve the best performance, it is recommended to use the platform-specific back end.

If there is no existing backend for your current development platform, you can easily create a new backend following the instructions described in later sections.

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



Creating an Adaptation Layer with a Face for a New Platform
---------------------------------------------

To create a Face implementation based on platform-specific network API, one needs to create an adaptation layer.

#### Step One
* Under the `./adaptation` directory, create a new folder with the name that can reflect the platform.
* In the folder, create new header and/or source files to include platform-specific header files and implement your helper functions.

#### Step Two
* Under the `./face` directory, create a new header file and a new source file for the face (use the face name as the file name, e.g., ndn-nrf-802154-face).
* Implement a new Face structure in the header and source files. Developers need to make sure that the first structure element is an `ndn_face_intf` instance.
* In the source file, implement the functions defined in the `ndn_face_intf` structure.
* In the header file, create a construction function to create a face instance or get the face instance if the face is a singleton. The function pointers binding should also be done in this function.

Existing examples:
* Nordic SDK Adaptation with `ndn-nrf-ble-face`: The files under `./adaptation/ndn-nrf-ble-adaptation/` is the adaptation layer and the files `./face/ndn-nrf-ble-face.c`, `./face/ndn-nrf-ble-face.h` are the face implementation.
* NRF 802154 driver Adaptation with `ndn-nrf-802154-face`: The files under `./adaptation/ndn-nrf-802154-driver/` is the adaptation layer and the files `./face/ndn-nrf-802154-face.c`, `./face/ndn-nrf-802154-face.h` are the face implementation.


Creating an NDN Security Back End for a New Platform
---------------------------------------------

To create a new security/crypto backend for an OS/SDK platform, please follow the steps.

#### Step One
* Under the `./adaptation` directory, create a new folder with the name that can reflect the platform.
Of course, you don't need to create a new directory if the adaptation for the platform already exists.
* In the folder, create new header and/or source files to include platform-specific header files and implement your helper functions.

#### Step Two
* Under the `./security` directory, create a new folder with the name can reflect the platform (e.g., `./security/nordic-sdk-crypto-back/`).
* If you want to use platform-specific APIs for digital signature generation and verification, implement the APIs defined in `./security/sign-verify.h`.
* If you want to use platform-specific APIs for AES encryption and decryption, implement the APIs defined in `./security/aes.h`.
* If you want to use platform-specific APIs for randomness, implement the APIs defined in `./security/random.h`.


#### Step Three

In the `./security/config.h` file, create a new macro for your backend and config the included files.