# NDN IOT Working Groups Tracking

## 2018 Dec 1 Meeting at 10 PM

#### Current Status
##### Zhiyi, Tianyuan, Xinyu (Stack and App Support Development Team)
- Finished: 
  * NDN Stack
  * Forwarder

- TODOs: 
  * Move fragmentation support into base face "class"
  * We haven't started working on Application Support Layer

##### Yanbiao, Edward (BLE Development Team)
- Finished:
  * Bootstrapping over BLE

- TODOs:
  * Some functionality of BLE is still missing because of the hardware and software issues

##### Bo (802155 over Nordic Development Team)
- Finished:
  * Integrating ndn-lite into Noridc SDK

- TODOs:
  * Figure out 802154 using Noridc SDK APIs

#### Plan before Dec 3
- Zhiyi, Tianyuan: Access Control
  * Quick thoughts are: handshakes between producer and consumer
- Bo: Nordic SDK 802154 Experiments
- Yanbiao, Edward: Nordic SDK BLE Experiments
- Access Control Issues
  * Should we still need having a **dedicated access controller**, like what Wentao did before with [NDN-ACE](https://named-data.net/publications/techreports/ndn-0036-1-ndn-ace/)? Current boards are powerful enough to derive ECDSA keys. If not, we must re-design the access control part from the bottom. 

#### TODOs for Next Meeting
* Decide which to use over Nordic SDK: BLE or 802154

