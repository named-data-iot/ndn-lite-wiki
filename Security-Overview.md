NDN-Lite Security Overview
=================

The Frontend
------------

The frontend of the NDN-Lite security library is divided into multiple components, and thus the library supports more flexible implementation, e.g., some components use the NDN-Lite default software backend while some other components use the platform-specific backend.

The frontend contains the following components:
* Hash component
* Signing and Verification component (`./security/sign-verify.h`)
* AES Encryption and Decryption component (`./security/aes.h`)
* Randomness component, including randomness generation and diffle-hellman key neigotiation (`./security/crypto-key.h`, `./security/randomness.h`)


The Backend
-----------

#### Default Backend

A backend that implements all the exposed APIs in the frontend.
This backend is a pure software implementation.

#### Nordic SDK NRF Backend