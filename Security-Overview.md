# NDN-Lite Security Overview
=================

## NDN-Lite Security Frontend
------------

The frontend of the NDN-Lite security library is divided into multiple components, and thus the library supports more flexible implementation. Frontend components of the NDN-Lite library will use different backend implementations depending on the compilation flags used.

Description of the compilation flags that can be used to utilize different security backend implementations within NDN-Lite can be found in the security/ndn-lite-sec-config.h file.

Description of how frontends are mapped to different backends based on the compilation flags referenced above can be found in the "NDN-Lite Security Backend" section.

The frontend contains the following components:
### Hash component
### Signing and Verification component (`./security/sign-verify.h`)
### AES Encryption and Decryption component (`./security/aes.h`)
### Randomness component, including randomness generation and diffle-hellman key negotiation (`./security/crypto-key.h`, `./security/randomness.h`)


## NDN-Lite Security Backend
-----------

#### Default Backend

A backend that implements all the exposed APIs in the frontend.
This backend is a pure software implementation.

#### Nordic SDK NRF Backend
