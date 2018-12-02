# NDN-Lite Access Control

## An Overview

NDN-Lite provides application support including security bootstrapping, service discovery, access control, and etc.
In this document, we describe the spec of the access control protocol implemented in NDN-Lite.

The access control module aims to

  * provide data confidentiality and access control and thus improve the security and privacy of the system.
  * provide an automatic and lightweight keys distribution and management implementation

As shown in the figure, the NDN IoT Access Control is as follow.

* **Encryption Key (EK) Negotiation Between Encryptor and Controller**

  To obtain the key for the encryptor to encrypt content, a data producer (called encryptor in NDN-Lite Access Control) will negotiate the `encryption key (EK)` with the system controller.
  The key will be negotiated via Elliptic Curve Diffie-Hellman (ECDH), that is, the encryptor first sends a EK Request Interest containing encryptor's ECDH pub key bits and the controller will check the access control policy and replies the Data packet containing controller's ECDH pub key bits.
  The EK will be derived through a key derivation function (KDF).

* **Decryption Key (DK) Distribution Between Decryptor and Controller**

  Notice that NDN-Lite Access Control uses symmetric key for content encryption and decryption.
  Therefore, we have `EK == DK`.

  To obtain the key for the decryptor to decrypt content, a data consumer (called decryptor in NDN-Lite Access Control) will negotiate an `ephemeral key` with the system controller and system controller will encrypt the EK with the ephemeral key and deliver it to the decryptor.
  The ephemeral key will be negotiated via Elliptic Curve Diffie-Hellman (ECDH), that is, the encryptor first sends a Decryption Key Request Interest containing decryptor's ECDH pub key bits and the controller will check the access control policy and replies the Data packet containing controller's ECDH pub key bits and also encrypted EK.
  The ephemeral key will be derived through a key derivation function (KDF) and be used to decrypt the EK.

```
EK Negotiation:

  Encryptor                                Controller
      |                                         |
      |          EK Request Interest            |
      | --------------------------------------> |
      |                                         |
      |         EK Request Response Data        |
      | <-------------------------------------- |

DK Negotiation:

  Decryptor                                Controller
      |                                         |
      |          DK Request Interest            |
      | --------------------------------------> |
      |                                         |
      |         DK Request Response Data        |
      | <-------------------------------------- |

Data Consumption:

  Decryptor                                Encryptor
      |                                         |
      |            Content Interest             |
      | --------------------------------------> |
      |                                         |
      |          Encrypted Content Data         |
      | <-------------------------------------- |
```

## Packet Format

### Encryption Key (EK) Negotiation Between Encryptor and Controller

#### Interest Packet Format

  ```
  Interest Name: /[home_prefix]/AC/<encryptor_identity>/<parameters_digest>
  Parameters: {Key_Type}; {Key_ID}; {EC_Pub_Key};
  Signature: Signed by producer's identity key
  ```

  The Parameters TLV will be:
  ```
  AC_Key_Type = TLV_AC_KEY_TYPE_TYPE Length
                  nonNegativeInteger
  AC_Key_ID = TLV_NAME_TYPE Length
                NameValue
  ECDH_Public_Key = TLV_ECDH_Pub_TYPE Length
                      Bytes
  ```

  AC Key Type:

    * 0: Encryption Key (EK) (This should be selected for EK request)
    * 1: Decryption Key (DK)

  AC Key ID is the Name of the Key following the naming convention:

  ```
  AC Key ID = /[encryptor_prefix]/[data_prefix]/KEY/<random_number>
  ```

#### Data Packet Format

  ```
  Name: /[home_prefix]/AC/<encryptor_identity>/<parameters_digest>
  Content: EK Response TLV
  Signature: Signed by controller's identity key
  ```

  EK Response TLV

  ```
  ECDH_Public_Key = TLV_ECDH_Pub_TYPE Length
                      Bytes
  Salt = TLV_SALT_TYPE Length
           Bytes
  LiftTime = TLV_AC_KEY_LIFETIME_TYPE Length
               NonNegativeInteger
  ```

#### Instructions

After receiving the EK request Interest, the controller should

  * Check the access control policy to see whether the encryptor is authorized to encrypt the content and how long the key lifetime should be
  * If yes, the controller should first generate a ECDH public key and get the shared secret.
  * The controller derives the AES KEY using KDF with an optional salt value from the shared secret.
  * The controller records the KEY ID associated with the AES KEY bits and the generate time.
  * The controller replies the ECDH public key, key lifetime and the optional salt value back.

### Decryption Key (DK) Negotiation Between Decryptor and Controller

#### Interest Packet Format

  Same as EK request but there should be `AC Key Type = 1`, which means to get the Decryption Key.

#### Data Packet Format

  ```
  Name: /[home_prefix]/AC/<decryptor_identity>/<parameters_digest>
  Content: DK Response TLV
  Signature: Signed by controller's identity key
  ```

  EK Response TLV

  ```
  ECDH_Public_Key = TLV_ECDH_PUB_TYPE Length
                      Bytes
  Salt = TLV_SALT_TYPE Length
           Bytes
  LiftTime = TLV_AC_KEY_LIFETIME_TYPE Length
               NonNegativeInteger
  Encrypted_DK = TLV_CIPHER_DK_TYPE Length
                   Bytes
  ```

#### Instructions

After receiving the DK request Interest, the controller should

  * Check the access control policy to see whether the decryptor is authorized to decrypt the content
  * If yes, the controller should first generate a ECDH public key and get the shared secret.
  * The controller derives the AES KEY using KDF with an optional salt value from the shared secret.
  * The controller uses the AES KEY to encrypt the required EK
  * The controller gets the lifetime of the EK and calculated the lifetime of DK `DK_lifetime = EK_generated_time + EK_liftime - current_time`
  * The controller replies the ECDH public key, key lifetime, the optional salt value, and the encrypted EK back.

## Default Settings of NDN-Lite Access Control

* ECDH Protocol: NIST p-256 Curve
* KDF: Hmac-based Key Derivation Function (HKDF)
* AES Key: 128 bits from HKDF
* AES IV: derived from HKDF
* AES Encryption: CBC mode

## Encrypted Content Data TLV Format

When Data content is supposed to be encrypted, NDN-Lite defines the encrypted content TLV.

```
Content = ENCRYPTED_CONTENT_TLV Length
            AC_Key_ID
            Encrypted_Payload

AC_Key_ID = TLV_NAME_TYPE Length
                NameValue

AES_IV = TLV_AES_IV_TYPE Length
           Bytes

Encrypted_Payload = TLV_ENCRYPTED_PAYLOAD_TYPE Length
                Bytes
```

When the consumer receives the Data packet, the consumer should

  * check whether it has the corresponding DK or not
  * if yes, the controller can directly decrypt the content
  * if no, the controller should try to apply for the key from the controller using NDN-Lite Access Control protocol.
  * after obtaining the DK, the consumer should cache the decryption until the key expires.