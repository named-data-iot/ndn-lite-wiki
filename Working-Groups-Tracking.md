# NDN IOT Working Groups Tracking

#### Current Status
##### Zhiyi, Tianyuan, Xinyu (Stack and App Support Development Team)
- Finished: 
  * NDN Stack
  * Forwarder and Face (need some update)

- In progress
  * access control

- TODOs: 
  * Move fragmentation support into base face "class"
  * Add face create functions in base face "class" and achieve polymorphism by void* parameter list and function pointers
  * We haven't started working on Application Support Layer

##### Yanbiao, Edward (BLE Development Team)
- In progress:
  * Bootstrapping over BLE

- TODOs:
  * Some functionality of BLE is still missing because of the hardware and software issues

##### Bo (802155 over Nordic Development Team)
- Finished:
  * Integrating ndn-lite into Noridc SDK

- TODOs:
  * Figure out 802154 using Noridc SDK APIs (Aborted)
  * Application development using ndn-lite

## 2018 Dec 3 Meeting at 2 PM

#### Issues
- To all: decided not to use SDK 802154.
- To all: we need another meeting on Dec 4 to discuss application implementation details

- To Zhiyi: Add base face: create face API, Add ability to parse service entry and create face, distinguish multicast face and unicast face
- To Bo: start working on application

## 2018 Dec 1 Meeting at 10 PM

#### Issues
- To Zhiyi and Tianyuan: stop working on service discovery, work on access control
- To Yanbiao and Edward: see if BLE in SDK can work
- To Bo: see if 802154 in SDK can work

#### Plan before Dec 3
- Zhiyi, Tianyuan: Access Control
  * Quick thoughts are: handshakes between producer and consumer
- Bo: Nordic SDK 802154 Experiments
- Yanbiao, Edward: Nordic SDK BLE Experiments
- Access Control Issues
  * Should we still need having a **dedicated access controller**, like what Wentao did before with [NDN-ACE](https://named-data.net/publications/techreports/ndn-0036-1-ndn-ace/)? Current boards are powerful enough to derive ECDSA keys. If not, we must re-design the access control part from the bottom. 

#### TODOs for Next Meeting
* Decide which to use over Nordic SDK: BLE or 802154

