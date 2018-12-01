# NDN-Lite Service Discovery

## An Overview

NDN-Lite provides application support including security bootstrapping, service discovery, access control, and etc.
In this document, we describe the spec of the service discovery protocol implemented in NDN-Lite.

The service discovery module aims to

  * enable an application to advertise the services provided by itself
  * help an application to automatically discover available services in the local network

A service in NDN-Lite refers to a service offered by a device in NDN IoT network.
Based on NDN's Interest and Data packet exchange, a service must satisfy the following requirements:

  * A service must have a prefix registered and available to other nodes in the local network.
  * The prefix follows the pre-defined naming convention so that other nodes/applications can direct invoke the service without manual configuration in advance.

As shown in the figure, the NDN IoT service discovery is as follow.

* **Service Discovery Advertisement**

  After the bootstrapping, the IoT device will start broadcasting its available services for a period of time.
  When the service changes or restarts, the IoT device will also broadcast the change.

* **Service Status Query(Optional)**

  When another IoT device wants the service, it will send a query for the service and get the meta data of the service.
  The replied Data packet will inform the requester whether the service is available or not and whether the requester has permission to use the service or not.
  This step is optional depending on the service.

* **Service Invocation**

  If the requester has permission and the service is available, the requester can then follow the pre-defined naming conventions to invoke the service.


```
bootstrapped device                                Other devices
   |                                                     |
   |   Periodically Broadcast Advertisement Interest     |
   | --------------------------------------------------> |
   |                                                     |

bootstrapped device                               Service Provider
   |                                                     |
   |           Service Status Query Interest             |
   | --------------------------------------------------> |
   |       The Meta Data Packets of the service          |
   | <-------------------------------------------------- |
   |            Service Invocation Interest              |
   | --------------------------------------------------> |
   |                        ...                          |
   |    More service invocation and optional replies     |
   |                        ...                          |

```

## Packet Format

### Service Advertisement Packet Format

#### Interest Packet Format

  ```
  Interest Name: /{home_prefix}/SD-ADV/{OwnIdentity}
  Parameters: A list of services provided by the device/application
  Signature: Not required
  ```

#### Data Packet Format

  No Data packet required.

#### Instructions

  Every node should register a prefix `/{home_prefix}/SD-ADV` after the device being bootstrapped.
  After receiving the advertisement Interest packet, the receiver will update its cached service table.

### Service Status Query Packet Format

#### Interest Packet Format

  ```
  Interest Name: /{home_prefix}/SD/<TargetIdentity>/QUERY/<Service_ID>
  Parameters: Depends on the service
  Signature: Signed by the requester
  ```

  When a Diffie Hellman process is needed, the Parameters TLV will be:
  ```
  (Optional)ECDH_Public_Key = TLV_ECDH_Pub_TYPE Length
                                  Bytes
  ```

#### Data Packet Format

  ```
  Name: /{home_prefix}/SD/<TargetIdentity>/QUERY/<Service_ID>
  Content: Service Meta Data TLV
  Signature: Signed by TargetIdentity's identity key
  ```

  Data Content TLV format

  ```
  Content = TLV_CONTENT_TYPE Length
              Service_Status = TLV_SD_STATUS_TYPE Length
                                 nonNegativeNumber
   (Optional) ECDH_Pub_Key = TLV_ECDH_Pub_TYPE Length
                               Bytes
  ```

  The status code:
    * 0: unavailable
    * 1: available
    * 2: busy
    * 3: permission denied

  Data Verification Process

    * The KeyLocator in Data Signature Info element must starts with the /<home_prefix>/<TargetIdentity>/KEY
    * The signature value must be verified using the public key in the TargetIdentity's certificate. TargetIdentity's certificate must be verified using the home's trust anchor key.

#### Instructions

  A prefix `/{home_prefix}/SD/<OwnIdentity>/QUERY` should be registered after the device being bootstrapped.

### Service Invocation Packet Format

#### Interest Packet Format

  ```
  Interest Name: /{home_prefix}/SD/<TargetIdentity>/<Service_ID>/<Command>/...
  Parameters: Depends on the service
  Signature: Signed by the requester (signature type depends on the service)
  ```

## NDN-lite Pre-defined Services

### **Service ID: SD_LED**
Description: Control the LED light on the other device.

| Requirements | Status |
|--------------|:------:|
| Need QUERY | No |
| Need Diffie-Hellman in QUERY | No |
| Need Signature in QUERY | No |
| Need Signature in invocation Interest | Yes |

Available commands:

| Command | Name Format | Parameters | Description |
|---------|:-----------:|:----------:|:-----------:|
| ON | SD_LED/ON/id | None | turn on LED with id |
| OFF | SD_LED/OFF/id | None | turn off LED with id |

Example:
```
Interest: /zhiyi-home/SD/board-0/SD_LED/ON/1
Interest Signature: Signed by the requester
```

### **Service ID: SD_TEMP**
Description: Read the temperature value from the target device. The content in replied Data will be **encrypted**.

| Requirements | Status |
|--------------|:------:|
| Need QUERY | No |
| Need Diffie-Hellman in QUERY | No |
| Need Signature in QUERY | No |
| Need Signature in invocation Interest | No |

Available commands:

| Command | Name Format | Parameters | Description |
|---------|:-----------:|:----------:|:-----------:|
| READ    | none | none | read the current temperature from the sensor |

Example:
```
Interest: /zhiyi-home/SD/board-0/SD_TEMP
```