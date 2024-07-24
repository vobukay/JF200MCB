# Safety warning 
This board uses higt voltages and can cause death or serious injury.
Capacitors on this board might retain their charge with the power disconnected.
# Warning 
The author assumes no responsibility or liability for any errors or omissions in the content of this repository.
Use at your own risk

# Reverse engineering protocol of Treadmill Motor Controller JF200
#### Connection


JF200 ----------------Red   +8v--------------------------- Console

JF200 ----------------Black GND------------------------- Console

JF200 ----------------Yellow   Inverted RX TX  0- 8v ---------- Console


### Protocol
UART based, bits=8, baudrate =2400, parity=None, stop=1 
TX/RX  one wire, inverted, max +8V

#### UART Message

| Byte  |Value   |Description|
| ------------ | ------------ | ------------ |
| 0  | 0xF7  |Start message |
|  1 | 0xF8  |Start message |
|  2 |   | Command |
|  3 - (N-2) |  - | Data? |
|  N-1 |    | CheckSum8 Modulo 256  (Sum of Bytes 2 to N-2 % 256) |
|  N |  0xFD | End message |

##### Commands 

|  Command |  Description | Direction|
| ------------ | ------------ | ------ |
|   0x04   | Handshake |Console -----> JF200|
|  0x0C  |  Handshake  |JF200--------->Console|
|  0x12 |  Set params|Console -----> JF200|
|  0x1A| Set params reply |JF200--------->Console|

**Example**

Sequence  from handshake to start motor. Msg pairs 1-2,  5- 6 send one time, pairs  3-4, 7-8 each  ~320ms  to keep link between JF200 and console
 
| MSG Num  | Msg   | Description   |
| ------------ | ------------ | ------------ |
|  1 |   F7-F8-4-1-1-1-7-FD| Handshake
|  2|    F7-F8-C-1-2-1-0-8C-37-4-86-10-CC-46-7F-FD| Handshake reply
| 3  |  F7-F8-12-1-1-2-**0**-1-18-0-0C-86-0F-0-0C-B7-35-10-CC-46-EA-FD | Set params **Motor off** , speed 0.8, angle 0  | 
|4 | F7-F8-1A-1-2-2-0-0-0-0-0-0-86-0-0-0-3-CD-0-0-0-2-0-0C-B7-35-3-5F-D1-FD | Set param reply |
|5 | F7-F8-12-1-1-2-**0D**-1-18-0-0C-86-0F-0-0C-B7-35-10-CC-46-F7-FD| Set params Motor  **? prepare to start** , speed 0.8, angle 0 
|6| F7-F8-1A-1-2-2-0-0-0-0-0-0-86-0-0-0-3-CD-0-0-1-2-0-0C-B7-35-3-5F-D2-FD| Set params reply |
| 7|  F7-F8-12-1-1-2-**0C**-1-18-0-0C-86-0F-0-0C-B7-35-10-CC-46-F6-FD |Set params **Motor  on** , speed 0.8, angle 0 
|8 |F8-1A-1-2-2-0-1-18-0-52-0-86-10-10-9-3-CD-0-0-1-19-0-0C-B7-35-3-5F-7D-FD |Set params reply

Set params command 

|  Byte |  Value |  Description |
| ------------ | ------------ | ------------ |
|  0 |  F7 | Start message  |
|  1 |  F8 | Start message  |
|  2 | 12  |  Command Set Params |
|  3 |  1 |  ? Unknown param  |
|  4 |  1 |  ? Unknown param  |
|  5 | 2  |  ? Unknown param  |
|  6 | 0  |  On/Off/Preparing?  See example sequence above|
|  7 | 1  |  Speed HByte |
|  8 | 18  |  Speed  LByte |
|  9 |  0 |  Angle (0x0-0xF) |
|  10 |  C | ? Unknown param |
|  11 | 86  | ? Unknown param  |
|  12 | F  |  ? Unknown param |
|  13 |  0 |  ? Unknown param |
|  14 | C  |  ? Unknown param  |
|  15 |  B7 |  ? Unknown param |
| 16 |  35 |  ? Unknown param |
| 17 |  10 |  ? Unknown param |
|  18 |  CC |  ? Unknown param |
|  19 |  46 |  ? Unknown param |
|  20 |  EA |  CheckSum |
|  21 |  FD |   End message |

##### Speed  word
|  HByte |  LByte |  Speed | |
| ------------ | ------------ | ------------ | ------------ |
|  0x1 |  0x18 |  0.8 km/s ||
|  0x1 |  0x3B |  0.9 km/s |+0x23 step|
|  0x1 |  0x5E |  1 km/s |+0x23 step|
|  - |  - | - |
|  0x13 |  0x24 |   14 km/s  |






