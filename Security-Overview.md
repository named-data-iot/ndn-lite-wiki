# NDN-Lite Security Overview

## NDN-Lite Security Frontend

The frontend of the NDN-Lite security library is divided into multiple components, and thus the library supports flexible implementation. Frontend components of the NDN-Lite library will use different backend implementations depending on the compilation flags used.

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

The way that NDN-Lite maps security backend implementations to security frontend API's requires that the ***ndn_security_init*** function, declared in `./security/ndn-lite-sec-config.h`, be called.

In brief, NDN-Lite selects security backends for the frontend security API's through compilation flags; see `./security/ndn-lite-sec-config.h` for more details regarding what compilation flags should be used for different security backends. 

It should be noted that in many cases, multiple backends will be simultaneously used for different security frontends. For example, the NDN_LITE_SEC_BACKEND_NRF_CRYPTO compilation flag indicates that the RNG frontend will use the Nordic SDK NRF Backend, while the rest of the security frontend API's will use the Default Backend. On the other hand, the NDN_LITE_SEC_BACKEND_DEFAULT compilation flag indicates that all frontend API's besides the RNG frontend will use the Default Backend, while frontend RNG will have no backend and should thus not be used.

The different NDN-Lite security backends currently implemented are listed below.

#### Default Backend

The frontend components implemented by this backend are the AES component, 

This backend is a pure software implementation.

#### Nordic SDK NRF Backend

This frontend components implemented by this backend are the RNG component.

This backend is a pure software implementation.

