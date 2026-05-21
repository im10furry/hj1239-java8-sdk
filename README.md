# HJ1239 Java SDK (Java 8 Compatible)

> *"We are DarkInno. Like a stout beer, our best ideas are brewed slowly in the dark, away from the hype."*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Java](https://img.shields.io/badge/Java-8%2B-orange.svg)](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html)
[![Build](https://img.shields.io/badge/Build-Passing-brightgreen.svg)]()

HJ 1239.3-2021 — *Technical Specification for Remote Emission Monitoring of Heavy-Duty Vehicles, Part 3: Communication Protocol and Data Format* — Java SDK implementation, strictly following Section 5 (Enterprise Platform Communication Protocol) and Section 4.5 (Data Unit Format) defined in the standard.

This branch is the **Java 8 compatible edition**, fully API-compatible with the main branch, suitable for enterprise environments still running on Java 8 runtime.

## Protocol Compliance

| Standard Section | Content | Implementation |
|---|---|---|
| 5.6.3 Table 16 | Packet structure (`~~` + cmd + resp + VIN + encrypt + len + data + BCC) | `DataPacket.java` |
| 5.6.4 Table 17 | Command unit identifiers (0x01–0x09) | `DataType.java` |
| 4.4.3 Table 1 | On-board terminal data units | `DataType.java` |
| 4.5.2 Table 2 | Real-time information data format | `PacketEncoder.java` |
| 4.5.2.3 Table 4 | Information type flags | `MessageType.java` |
| 4.5.2.4 Table 5 | DPF/SCR engine information (37 bytes, 19 fields) | `EmissionData.java` |
| 4.6 Table 14 | Time definition (BYTE[6]) | `TimeUtil.java` |
| 4.5.2.9 Table 10 | Position status bit definitions | `EmissionData.positionStatus` |

## Quick Start

### Maven

```xml
<dependency>
    <groupId>io.darkinno</groupId>
    <artifactId>hj1239-sdk</artifactId>
    <version>1.1.0</version>
</dependency>
```

### Usage

```java
Gb1239Sdk sdk = new Gb1239Sdk();

// Encode real-time emission data (Table 5 DPF+SCR)
EmissionData em = EmissionData.builder()
    .timestamp(LocalDateTime.now())
    .vehicleSpeed(60.0)
    .engineSpeed(1500.0)
    .fuelConsumptionRate(8.5)
    .engineCoolantTemp(85.0)
    .scrUpstreamNox(45.0)
    .scrDownstreamNox(5.0)
    .reagentRemaining(80.0)
    .intakePressure(100.0)
    .exhaustFlow(200.0)
    .dpfDifferentialPressure(1.5)
    .reagentLevel(75.0)
    .positionStatus(0x01)  // GPS valid
    .longitude(116.397128)
    .latitude(39.916527)
    .odometer(12345.6)
    .build();

byte[] packet = sdk.encodeRealtimeData(em, "LSVAM41Z6F2000001", 1);

// Decode
DataPacket decoded = sdk.decode(packet);
EmissionData result = sdk.decodeRealtimeEmission(decoded);

// Validate
ValidationResult vr = sdk.validateEmission(result);
```

## Packet Format (Table 16)

```
Offset | Size | Field             | Description
0      | 2    | Start marker      | 0x7E 0x7E
2      | 1    | Command ID        | 0x01=Login, 0x02=Realtime, 0x03=Retransmit, 0x04=Logout, 0x05=TimeSync
3      | 1    | Response flag     | 0xFE=Command, 0x01=Success, 0x02=Failure
4      | 17   | Vehicle VIN       | 17-character ASCII
21     | 1    | Encryption method | 0x01=None, 0x02=SM2, 0x03=SM4, 0x04=RSA, 0x05=AES128
22     | 2    | Data unit length  | 0–65531 (big-endian)
24     | N    | Data unit         | See Table 2 + Table 5
24+N   | 1    | BCC checksum      | XOR(command..last data byte)
```

## Build

```bash
mvn compile   # Compile (zero runtime dependencies)
mvn test      # Run tests
mvn package   # Package JAR
```

## Key Features

- **Java 8 Compatible** — Fully compatible with JDK 8+
- **Zero External Runtime Dependencies** — Uses Java standard library only
- **BCC (XOR) Checksum** — From command byte through last data unit byte
- **Invalid Value Handling** — 0xFF/0xFFFF/0xFFFFFFFF indicate unavailable sensor
- **Time Encoding** — BYTE[6] GMT+8 (YY,MM,DD,hh,mm,ss)
- **Position Status Bits** — bit0=Valid, bit1=North/South, bit2=East/West
- **Builder Pattern** — Fully immutable data models
- **Thread-Safe** — All encode/decode/validate operations are stateless

## License

MIT © [DarkInno](https://github.com/darkinno)
