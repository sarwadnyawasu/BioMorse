# BioMorse: Low-Bandwidth Embedded Communication System

> **ESP8266-based bidirectional wireless communication system using ESP-NOW and Morse/DNA encoding for infrastructure-independent transmission of text and genomic data.**

## Overview

**BioMorse** is a low-bandwidth, infrastructure-independent embedded communication system designed around two **ESP8266 NodeMCU** devices.

The system combines:

* ESP8266 microcontrollers
* ESP-NOW peer-to-peer wireless communication
* Morse-code-based encoding
* DNA sequence encoding
* Bidirectional communication
* RGB LED and buzzer feedback
* MAC-address-based node identification
* Transmission-latency measurement
* Local/offline operation

Unlike conventional Wi-Fi communication, BioMorse does not require a router, access point, Internet connection, or cloud broker.

The system was developed around the idea of creating a **low-cost communication link for constrained environments**, with genomic DNA sequences used as one of the primary demonstration data types.

---

# Problem Statement

Modern communication systems often depend on centralized infrastructure such as:

* Wi-Fi routers
* Cellular networks
* Internet connectivity
* Cloud services
* Network servers

This dependency becomes a limitation in environments where infrastructure is unavailable or unreliable.

BioMorse explores a different architecture:

```text
Node A  ←──── ESP-NOW ────→  Node B
  │                           │
  └──── No Router / Cloud ────┘
```

Each node can both transmit and receive, creating a **bidirectional embedded communication channel**.

The project additionally investigates whether DNA/genomic data can be represented through a compact symbolic encoding layer before wireless transmission.

---

# Objectives

The project was developed with the following objectives:

1. Build a direct ESP8266-to-ESP8266 communication link.
2. Eliminate dependence on Wi-Fi routers and Internet connectivity.
3. Implement Morse-based message encoding and decoding.
4. Support genomic DNA sequence transmission.
5. Implement bidirectional communication.
6. Provide local visual and auditory communication feedback.
7. Measure end-to-end transmission latency.
8. Evaluate packet delivery over different distances.
9. Measure decoding accuracy for different message types.
10. Validate system behaviour through functional test cases.
11. Explore an inexpensive architecture suitable for constrained communication environments.

---

# System Architecture

```text
                    BIO-MORSE NODE A
                 ┌────────────────────┐
                 │     ESP8266        │
                 │                    │
User Input ─────→│ Encoding           │
                 │                    │
                 │ ESP-NOW TX/RX      │
                 │                    │
                 │ Decoding           │
                 └─────────┬──────────┘
                           │
                           │ ESP-NOW
                           │
                           ▼
                 ┌────────────────────┐
                 │     ESP8266        │
                 │                    │
                 │ ESP-NOW RX/TX      │
                 │                    │
                 │ Decoding           │
                 │                    │
                 │ RGB LED + Buzzer   │
                 └─────────┬──────────┘
                           │
                           ▼
                    Received Message
```

Both ESP8266 nodes are configured to operate as transmitters and receivers, allowing communication in either direction.

---

# Hardware Architecture

Each BioMorse node consists of:

| Component               | Function                                    |
| ----------------------- | ------------------------------------------- |
| ESP8266 NodeMCU v3      | Processing + wireless communication         |
| Tactile push button     | Manual transmission trigger                 |
| RGB LED                 | Visual system-state indication              |
| Active 5 V buzzer       | Audio feedback                              |
| USB-to-Serial interface | Programming, serial communication and power |
| Breadboard              | Hardware prototyping                        |
| Jumper wires            | Interconnections                            |

The ESP8266 provides the processing capability as well as the integrated Wi-Fi radio used by ESP-NOW. The documented node uses an **80 MHz Tensilica L106 CPU** with integrated 802.11 b/g/n wireless hardware.

---

# Communication Architecture

The system uses **ESP-NOW** for direct wireless communication.

ESP-NOW operates at the MAC layer and allows peer-to-peer communication without the TCP/IP stack and without requiring a conventional Wi-Fi access point.

The communication architecture is therefore:

```text
ESP8266 A
   │
   │ MAC-addressed ESP-NOW packet
   │
   ▼
ESP8266 B
```

Each node stores the MAC address of its communication peer.

The system therefore avoids:

```text
Router
Cloud Server
MQTT Broker
Internet
```

for the core communication path.

---

# DNA-to-Morse Encoding

One of the distinctive features of BioMorse is its support for **genomic DNA sequences**.

DNA consists of four nucleotide bases:

```text
A → Adenine
T → Thymine
C → Cytosine
G → Guanine
```

The system first represents the four bases using a 2-bit representation:

```text
A → 00
T → 01
C → 10
G → 11
```

The resulting representation is then mapped to the corresponding Morse symbols used for transmission.

For example:

```text
DNA sequence
     ↓
   ATCG
     ↓
00 01 10 11
     ↓
Morse representation
     ↓
ESP-NOW packet
```

This layered encoding provides a structured representation of genomic sequences before wireless transmission.

---

# Text Message Encoding

BioMorse also supports ordinary alphanumeric messages.

The ESP8266 uses a **36-entry lookup table covering A–Z and 0–9**.

The process is:

```text
User Text
   ↓
Uppercase Conversion
   ↓
Character Lookup
   ↓
Morse Symbol
   ↓
Encoded Message
```

Word boundaries are represented using slash separators.

The encoded Morse string is additionally prefixed with a millisecond timestamp so that transmission latency can be calculated at the receiver.

---

# Six-Stage Communication Pipeline

The complete BioMorse operation can be divided into six major stages.

```text
1. Input
      ↓
2. Encoding
      ↓
3. ESP-NOW Transmission
      ↓
4. Reception
      ↓
5. Decoding
      ↓
6. Feedback / Output
```

Each stage performs a specific task in the end-to-end communication process.

---

# 1. Input Stage

The user enters either:

* Normal text
* DNA sequence

through the connected laptop's serial monitor.

The serial interface operates at **115200 baud**.

The input is:

1. Read until newline
2. Trimmed
3. Converted to uppercase
4. Passed to the encoding stage

This approach avoids requiring a dedicated keypad or display input interface.

---

# 2. Encoding Stage

For ordinary text, each character is converted using the Morse lookup table.

For DNA data, the sequence passes through the DNA representation layer before being mapped into the transmission format.

The encoded message is then combined with a timestamp.

Conceptually:

```text
Input
  ↓
Data Type Detection
  ↓
Encoding
  ↓
Timestamp
  ↓
Packet Construction
```

---

# 3. ESP-NOW Packet Transmission

The encoded data is packaged into a **250-byte structure**.

The packet is transmitted through the ESP-NOW API using the registered peer MAC address.

The wireless process is:

```text
Encoded Data
     ↓
Timestamp + Payload
     ↓
Packet Structure
     ↓
ESP-NOW API
     ↓
Peer MAC Address
     ↓
Wireless Transmission
```

ESP-NOW provides the direct device-to-device communication mechanism.

---

# 4. Reception Stage

The receiving ESP8266 listens for incoming ESP-NOW packets using a registered receive callback.

Once a packet arrives, the receiver extracts:

* Timestamp
* Encoded Morse payload

The timestamp is then compared with the receiver's local time to estimate end-to-end transmission latency.

---

# 5. Morse Decoding

The receiver reconstructs the original message by reversing the encoding process.

```text
ESP-NOW Packet
      ↓
Extract Payload
      ↓
Separate Morse Symbols
      ↓
Lookup Morse Pattern
      ↓
Recover Characters
      ↓
Reconstruct Message
```

The receiver uses the same lookup table employed during transmission.

This symmetric encoding/decoding approach allows the transmitted message to be reconstructed at the destination node.

---

# 6. Visual & Audio Feedback

BioMorse provides hardware-level feedback through an RGB LED and buzzer.

### Idle

```text
Blue LED
```

indicates the node is idle.

### Successful Reception

```text
Green LED
+
Two short buzzer beeps
```

indicates successful reception and decoding.

### Transmission Failure

```text
Red LED
+
Four buzzer beeps
```

indicates communication failure.

This allows the system to communicate its state even without continuously monitoring the serial terminal.

---

# Bidirectional Communication

A major feature of BioMorse is that both ESP8266 nodes can function as transmitters and receivers.

```text
        ESP-NOW
A  ←────────────────→  B
TX                     TX
RX                     RX
```

Both nodes use a combined ESP-NOW operating role and maintain the peer MAC address.

This eliminates the need to permanently designate one device as the transmitter and the other as the receiver.

---

# Node Identification

Each communication node is identified using its **MAC address**.

This provides a transparent mechanism for:

* Peer identification
* Debugging
* Packet routing
* Communication diagnostics

The MAC address therefore becomes part of the practical embedded communication workflow rather than relying on an external network identifier.

---

# Latency Measurement

Transmission latency is measured using timestamps.

The sender attaches a millisecond timestamp to the encoded payload.

At the receiver:

```text
Current Receiver Time
        -
Sender Timestamp
        =
Estimated End-to-End Latency
```

This allows the communication system to be quantitatively evaluated rather than simply checking whether a message arrived.

---

# Experimental Validation

The system was evaluated under controlled indoor laboratory conditions across distances of approximately **5–50 m**.

The evaluation included:

* Multiple message types
* Different message lengths
* Different transmission distances
* DNA sequences
* Alphanumeric messages
* Special characters
* Long messages
* Bidirectional communication
* Peer-unavailability conditions

A total of **240 transmission events** were recorded across 12 test scenarios, with repeated trials used for validation.

---

# Communication Performance

The measured end-to-end latency ranged from approximately:

```text
3.2 ms @ 5 m
       ↓
22.1 ms @ 50 m
```

Packet loss remained below **1% up to 30 m**, while performance degraded beyond 40 m because of multipath effects in the indoor environment.

The documented performance table reports approximately **85% packet delivery at 45 m**.

---

# Message-Length Performance

Transmission time increased approximately linearly with message length.

Reported measurements include:

| Message Length | Approx. End-to-End Time |
| -------------: | ----------------------: |
|   5 characters |                  ~12 ms |
| 100 characters |                 ~204 ms |

The study reports an approximate **2 ms/character** relationship for the tested configuration at 10 m.

This provides a useful engineering relationship when deciding practical payload sizes.

---

# Morse Decoding Accuracy

Decoding performance was evaluated across several message categories.

The reported results include:

| Message Type               |        Result |
| -------------------------- | ------------: |
| Standard alphanumeric      |    Up to 100% |
| DNA sequences              |     **98.6%** |
| Special-character messages |        ~94.5% |
| Other tested categories    | High accuracy |

The DNA-sequence result demonstrates that the encoding/decoding pipeline can successfully handle genomic data in the tested configuration.

---

# Power Consumption

Power consumption was measured under different operating conditions.

Reported values include:

| Operating State                            |     Current |
| ------------------------------------------ | ----------: |
| Idle                                       |   ~80–90 mA |
| ESP-NOW transmission peak                  | ~170–210 mA |
| Approx. average in simulated usage profile |      ~88 mA |

The reported average power consumption was approximately **0.44 W from a 5 V USB supply** under the tested transmission profile.

---

# Functional Validation

The project included **12 functional test cases** covering communication correctness and boundary conditions.

Testing included scenarios such as:

* Normal message transmission
* DNA sequence transmission
* Empty messages
* Space handling
* Long messages
* Bidirectional communication
* Simultaneous transmit/receive operation
* Peer unavailability
* Communication failure handling

The paper reports that all 12 test cases passed under the documented validation procedure.

---

# Error Handling

The system was designed to handle communication failures without crashing or requiring a reboot.

When a peer becomes unavailable, the system activates the failure-state indication through:

```text
Red LED
+
Buzzer Pattern
```

This demonstrates that the communication callback layer includes explicit failure handling rather than assuming that every packet transmission will succeed.

---

# Encoding Overhead

Morse encoding introduces additional representation overhead compared with raw ASCII.

The study reports:

* Approximately **5.8× overhead** relative to unencoded ASCII
* Approximately **2.9× effective overhead** relative to the 2-bit DNA representation used for genomic sequences

A timestamp adds a small fixed packet overhead.

This creates an engineering trade-off:

```text
More Encoding
      ↓
Higher Data Representation Overhead
      ↓
But
      ↓
Structured / Symbolic Representation
+
Low-Infrastructure Communication
```

The project therefore focuses on constrained communication rather than maximizing raw throughput.

---

# Development Workflow

The project follows a complete embedded communication development cycle:

### Step 1 — Define Communication Architecture

Design two ESP8266 nodes capable of bidirectional communication.

### Step 2 — Establish ESP-NOW Link

Register peer MAC addresses and configure direct communication.

### Step 3 — Implement Encoding

Create the Morse lookup table and DNA representation mechanism.

### Step 4 — Build Packet Structure

Combine timestamp and encoded payload into the ESP-NOW packet.

### Step 5 — Implement Reception

Use ESP-NOW callbacks to receive incoming data.

### Step 6 — Implement Decoding

Reverse the Morse encoding process to reconstruct the original message.

### Step 7 — Add Hardware Feedback

Implement RGB LED and buzzer state indication.

### Step 8 — Add Latency Measurement

Calculate transmission delay using sender timestamps.

### Step 9 — Test Communication Range

Evaluate the system at multiple indoor distances.

### Step 10 — Validate Edge Cases

Test empty messages, long messages, special characters, simultaneous communication and peer failures.

---

# Key Technical Concepts

### Embedded Systems

* ESP8266
* GPIO
* Serial communication
* Callback-based communication
* Embedded state machines

### Wireless Communication

* ESP-NOW
* MAC-address-based peer communication
* Direct device-to-device communication
* Packet transmission
* Wireless latency measurement

### Data Encoding

* Morse code
* DNA nucleotide representation
* 2-bit encoding
* Lookup-table-based encoding/decoding

### Hardware Interface

* RGB LED
* Active buzzer
* Push button
* USB-UART interface

### System Validation

* Latency measurement
* Packet delivery analysis
* Accuracy testing
* Range testing
* Fault handling
* Power measurement

---

# Technologies Used

| Category          | Technology                      |
| ----------------- | ------------------------------- |
| MCU               | ESP8266 NodeMCU v3              |
| Wireless Protocol | ESP-NOW                         |
| Encoding          | Morse Code + DNA representation |
| Programming       | Embedded C/C++                  |
| Input             | Serial Monitor + Push Button    |
| Visual Feedback   | RGB LED                         |
| Audio Feedback    | 5 V Active Buzzer               |
| Identification    | MAC Address                     |
| Communication     | Peer-to-Peer                    |
| Power             | 5 V USB                         |
| Development       | Arduino/ESP8266 environment     |

---

# Applications

The architecture described in the project can be adapted for:

* Remote bioinformatics data relay
* Emergency communication
* Disaster-response networks
* Rural IoT deployments
* Local embedded telemetry
* Infrastructure-independent communication
* Short-range field communication

The paper specifically discusses situations where conventional communication infrastructure may be unavailable or unreliable.

---

# Limitations

The current implementation has several documented limitations:

* Short-range communication compared with long-range LPWAN technologies
* Indoor RF performance affected by multipath
* Morse encoding introduces data overhead
* Special-character handling can reduce decoding accuracy
* The current implementation depends on a laptop serial interface for user input
* The system does not currently implement cryptographic encryption at the payload layer

These limitations define the areas for further development rather than being hidden from the system evaluation.

---

# Future Scope

The paper proposes several extensions.

### Long-Range Communication

Integration of **LoRa/SX1276** could extend the communication range from the current short-range implementation toward kilometre-scale open-terrain communication.

### Cryptographic Security

AES-128 payload encryption could be added to provide actual cryptographic confidentiality rather than relying only on the encoding mechanism.

### Mobile Interface

A mobile application communicating through BLE or USB-OTG could remove the dependency on a laptop serial monitor.

### ESP-NOW Mesh

Multiple ESP8266 nodes could be connected into a mesh architecture for multi-hop communication.

### Expanded Biological Encoding

The encoding architecture could be extended toward protein-sequence transmission using variable-length DNA codons.

### Gateway Connectivity

A designated gateway could bridge the infrastructure-independent local network to MQTT/cloud services when external connectivity becomes available.

---

# Repository Structure

```text id="wq9xv3"
BioMorse/
│
├── README.md
│
├── Firmware/
│   ├── node_a/
│   └── node_b/
│
├── Communication/
│   ├── espnow/
│   └── packet_structure/
│
├── Encoding/
│   ├── morse/
│   └── dna/
│
├── Hardware/
│   ├── circuit/
│   └── pinout/
│
├── Testing/
│   ├── latency/
│   ├── packet_delivery/
│   ├── decoding_accuracy/
│   └── functional_tests/
│
├── Results/
│   ├── graphs/
│   └── measurements/
│
└── Documentation/
    └── paper/
```

---

# Skills Demonstrated

This project demonstrates practical experience in:

* ESP8266 embedded development
* ESP-NOW wireless communication
* Peer-to-peer networking
* Embedded C/C++
* GPIO interfacing
* Wireless packet handling
* Callback-based firmware
* Data encoding and decoding
* Lookup-table implementation
* Serial communication
* RGB LED and buzzer interfacing
* Latency measurement
* Wireless range testing
* Embedded system validation
* Power measurement
* Fault handling
* Low-bandwidth communication architecture

---

# Project Takeaway

BioMorse demonstrates how a relatively simple embedded platform can combine **wireless communication, custom data encoding, hardware feedback, and system-level validation** into a complete communication system.

The complete signal path is:

```text
User Message / DNA Sequence
            ↓
       Data Encoding
            ↓
      Packet Formation
            ↓
        ESP-NOW TX
            ↓
      Wireless Channel
            ↓
        ESP-NOW RX
            ↓
      Payload Extraction
            ↓
       Morse Decoding
            ↓
      Original Message
            ↓
    LED / Buzzer Feedback
```

The project therefore goes beyond simply sending data between two ESP8266 boards. It demonstrates the design of an **end-to-end embedded communication architecture**, from data representation and wireless transport to hardware-level feedback and quantitative validation.

The documented prototype achieved **3.2–22.1 ms end-to-end latency over 5–50 m**, **98.6% DNA decoding accuracy**, and successful validation across the project's 12 functional test scenarios.
