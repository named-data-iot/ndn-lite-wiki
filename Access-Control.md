# NDN-Lite 

## An Overview

NDN-Lite provides application support including security bootstrapping, service discovery, access control, and etc.
In this document, we describe the spec of the access control protocol implemented in NDN-Lite.

The access control module aims to

  * enable an application to fetch encrypted data, along with encryption keys
  * lightweight keys distribution and management

As shown in the figure, the NDN IoT Access Control is as follow.

* **Access Control Request**
  IoT devices fetch anchor issued certificate and communication keys from bootstrapping. Devices using communication key signed interest to request a encryption key for certain prefixes. The access key is symmetric and distributed with validity time. Timeout access keys must be updated through a new Access Control Request.

* **Key Encryption Key (KEK)**
  Key Encryption Key should be negotiated by ECDH. Diffie-Hellman bits are included in Request Interest. Upon receiving the Request, producer verifies the signed interest with anchor and negotiate a symmetric key, used for encrypt content encryption key (CEK)


* **Content Encryption Key (CEK)**
  Content Encryption Key should be negotiated by information provided from Access Requester and Access Provider (**Zhiyi, what about a string CEK `/A/ENCRYPTED-BY/B/VALID/5MIN` ?**). A valid CEK is a symmetric key, used for AES-128 content encryption and decryption.

```
consumer device                                      producer device
      |                                                     |
      |          Access Control Request Interest            |
      | --------------------------------------------------> |
      |                                                     |
      |       Key Encryption Key, Content Encryption Key    |
      | <-------------------------------------------------- |

consumer device                                      producer device
      |                                                     |
      |                     Interest                        |
      | --------------------------------------------------> |
      |               Content Encrypted Data                |
      | <-------------------------------------------------- |
      |                                                     |
 Decryption                                                 |
      |                        ...                          |

consumer device                                      producer device
      |                                                     |
      |                     Interest                        |
      | --------------------------------------------------> |
      |               Content Encrypted Data                |
      | <-------------------------------------------------- |
      |                                                     |
Decryption Failed                                           |
      |          Access Control Request Interest            |
      | --------------------------------------------------> |
      |                                                     |
      |                        ...                          |
```

## Packet Format

### Access Request Interest Packet Format

#### Interest Packet Format

  ```
  Interest Name: /{home_prefix}/{Identity}/AC_KEK
  Parameters: Diffie-Hellman bits
  Signature: Required
  ```

#### Data Packet Format

  Data Content TLV format

  ```
  Content = TLV_CONTENT_TYPE Length
            Diffie-Hellman Bits
            KEK encrypted CEK
  ```