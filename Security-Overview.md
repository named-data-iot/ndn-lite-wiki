# NDN-Lite Security Overview

## NDN-Lite Security Frontend

The frontend of the NDN-Lite security library is divided into multiple components, and thus the library supports more flexible implementation. Frontend components of the NDN-Lite library will use different backend implementations depending on the compilation flags used.

Description of the compilation flags that can be used to utilize different security backend implementations within NDN-Lite can be found in the security/ndn-lite-sec-config.h file.

Description of how frontends are mapped to different backends based on the compilation flags referenced above can be found in the "NDN-Lite Security Backend" section.

The NDN-Lite security frontend contains the following components:

#### AES Component

The header file describing AES related functionality can be found at `./security/ndn-lite-aes-.h`.

In brief, NDN-Lite provides an AES key structure, as well as AES encryption and decryption.

#### SHA component

The header file describing SHA hashing related functionality can be found at `./security/ndn-lite-sha.h`.

In brief, NDN-Lite provides hash generation and verification using SHA algorithms (e.g., SHA256).

#### HMAC Component 

The header file describing HMAC related functionality can be found at `./security/ndn-lite-hmac.h`.

In brief, NDN-Lite provides an HMAC key structure, as well as HMAC signing and verification.

#### ECC Component

The header file describing ECC related functionality can be found at `./security/ndn-lite-ecc.h`.

In brief, NDN-Lite provides ECC public and private key structures, as well as ECC signing and verification.

#### RNG Component

The header file describing RNG related functionality can be ofund at `./security/ndn-lite-rng.h`.

In brief, NDN-Lite provides random number generation functionality IF a security backend other than the default
security backend is selected.

The reason that the default security backend does not provide RNG functionality is that RNG number generation is often
platform specific; therefore, NDN-Lite RNG functionality is only provided in the case that a security backend for
a specific platform that has RNG functionality is selected (e.g., the NRF CRYPTO backend).

## NDN-Lite Security Backend

#### Default Backend

A backend that implements all the exposed APIs in the frontend.
This backend is a pure software implementation.

#### Nordic SDK NRF Backend
