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
    <artifactId>hj1239-sdk-java8</artifactId>
    <version>1.1.0</version>
</dependency>
```

> Published to **GitHub Packages**. Configure `~/.m2/settings.xml`:
>
> ```xml
> <server>
>     <id>github</id>
>     <username>YOUR_USERNAME</username>
>     <password>YOUR_GITHUB_TOKEN</password>
> </server>
> ```
>
> And add the repository:
>
> ```xml
> <repository>
>     <id>github</id>
>     <url>https://maven.pkg.github.com/DarkInno/hj1239-java8-sdk</url>
> </repository>
> ```

### Gradle

```groovy
implementation 'io.darkinno:hj1239-sdk:1.1.0'
```

## Integration Guide

### 1. SDK Initialization

```java
// Default configuration (strict mode + validation enabled)
Gb1239Sdk sdk = new Gb1239Sdk();

// Custom configuration
Gb1239Config config = new Gb1239Config();
config.setStrictMode(false);        // Relax strict byte-boundary checks
config.setEnableValidation(false);  // Skip automatic validation during decode
Gb1239Sdk sdk = new Gb1239Sdk(config);
```

### 2. Vehicle Login

```java
VehicleInfo vi = VehicleInfo.builder()
    .vin("LSVAM41Z6F2000001")
    .plateNumber("京A12345")
    .plateColor("1")                        // 1=Blue, 2=Yellow, 3=Black, 4=White, 5=Green
    .fuelType(FuelType.DIESEL)
    .emissionStandard(EmissionStandard.CHINA_VI)
    .manufacturer("Example Motors")
    .model("Truck-500")
    .modelYear(2023)
    .build();

byte[] loginPacket = sdk.encodeVehicleLogin(vi, 1);
```

### 3. Realtime Emission Data (Table 5 — DPF+SCR)

```java
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
    .positionStatus(0x01)   // bit0=GPS valid, bit1=N/S, bit2=E/W
    .longitude(116.397128)
    .latitude(39.916527)
    .odometer(12345.6)
    .build();

byte[] realtimePacket = sdk.encodeRealtimeData(em, "LSVAM41Z6F2000001", 1);
```

### 4. OBD Engine Data (Table 6)

```java
ObdEngineData obd = ObdEngineData.builder()
    .timestamp(LocalDateTime.now())
    .milOn(false)
    .dtcCount(0)
    .egrErrorRate(2.5)
    .scrSystemStatus(1)
    .dpfSystemStatus(1)
    .dpfSootLoad(45.0)
    .dpfAshLoad(5.0)
    .engineRuntime(3600)
    .positionStatus(0x01)
    .longitude(116.397128)
    .latitude(39.916527)
    .odometer(12345.6)
    .build();

byte[] obdPacket = sdk.encodeObdEngineData(obd, "LSVAM41Z6F2000001", 1);
```

### 5. Hybrid Vehicle Data (Table 7)

```java
HybridData hd = HybridData.builder()
    .timestamp(LocalDateTime.now())
    .vehicleSpeed(55.0)
    .engineSpeed(1800.0)
    .motorSpeed(3200.0)
    .motorTorque(150.0)
    .batterySoc(78.5)
    .batteryVoltage(355.0)
    .batteryCurrent(-15.0)
    .fuelConsumptionRate(6.8)
    .engineCoolantTemp(82.0)
    .hybridMode(2)            // 0=EV, 1=Engine, 2=Hybrid, 3=Charge
    .positionStatus(0x01)
    .longitude(116.397128)
    .latitude(39.916527)
    .odometer(23456.7)
    .build();

byte[] hybridPacket = sdk.encodeHybridData(hd, "LSVAM41Z6F2000001", 1);
```

### 6. Emission Check Data (Table 8 — PEMS)

```java
EmissionCheckData ec = EmissionCheckData.builder()
    .timestamp(LocalDateTime.now())
    .checkType(1)               // 1=In-use compliance, 2=Roadside pull
    .noxPemsValue(0.45)
    .coPemsValue(1.2)
    .pmPemsValue(0.02)
    .exhaustFlow(185.0)
    .engineSpeed(1600.0)
    .engineTorquePercent(65.0)
    .engineCoolantTemp(88.0)
    .positionStatus(0x01)
    .longitude(116.397128)
    .latitude(39.916527)
    .odometer(34567.8)
    .build();

byte[] checkPacket = sdk.encodeEmissionCheck(ec, "LSVAM41Z6F2000001", 1);
```

### 7. Heartbeat, Logout, Time Sync

```java
// Heartbeat — sent periodically to maintain connection
byte[] heartbeat = sdk.encodeHeartbeat("LSVAM41Z6F2000001", 1);

// Vehicle logout
byte[] logout = sdk.encodeVehicleLogout("LSVAM41Z6F2000001", 1);

// Time synchronization
byte[] timeSync = sdk.encodeTimeSync("LSVAM41Z6F2000001", 1);
```

### 8. Retransmit (Complement) Data

```java
// Re-upload historical data that failed to send
byte[] complement = sdk.encodeComplementData(em, "LSVAM41Z6F2000001", 1);
```

### 9. Platform Login / Logout

```java
// Platform authentication (for multi-platform deployments)
byte[] platformLogin = sdk.encodePlatformLogin("platform-001", "user", "pass", 1);
byte[] platformLogout = sdk.encodePlatformLogout("platform-001", 1);
```

### 10. Key Exchange (Encryption)

```java
// Request key exchange with plain algorithm
byte[] keyReq = sdk.encodeKeyExchangeRequest("LSVAM41Z6F2000001", 1, 0x01);

// Request with built-in RSA-2048 key pair generation
byte[] keyReqRsa = sdk.encodeKeyExchangeRequestWithRsaKey("LSVAM41Z6F2000001", 1);

// Decode key exchange response
KeyExchangeHandler.KeyExchangeData keyData = sdk.decodeKeyExchange(decodedPacket);
System.out.println(keyData.algorithm);   // encryption algorithm
System.out.println(keyData.publicKey);   // peer's public key bytes
```

### 11. Decoding Incoming Packets

```java
// Step 1: decode raw bytes into structured DataPacket
byte[] raw = ...; // received from TCP stream
DataPacket pkt = sdk.decode(raw);

// Step 2: decode DataPacket into typed data based on command ID
DataType cmd = Gb1239Sdk.getCommandType(pkt);
switch (cmd) {
    case REALTIME:
        EmissionData emission = sdk.decodeRealtimeEmission(pkt);
        break;
    case VEHICLE_LOGIN:
        VehicleInfo vehicle = sdk.decodeVehicleLogin(pkt);
        break;
    case OBD_ENGINE:
        ObdEngineData obd = sdk.decodeObdEngineData(pkt);
        break;
    case HYBRID:
        HybridData hybrid = sdk.decodeHybridData(pkt);
        break;
    case EMISSION_CHECK:
        EmissionCheckData check = sdk.decodeEmissionCheckData(pkt);
        break;
    case KEY_EXCHANGE:
        KeyExchangeHandler.KeyExchangeData keys = sdk.decodeKeyExchange(pkt);
        break;
}

// Utility checks
boolean isUpload = Gb1239Sdk.isUpload(pkt);  // true = data from terminal
boolean isResp   = Gb1239Sdk.isResponse(pkt); // true = response from platform
```

### 12. Validation

```java
// VIN validation
boolean validVin = sdk.validateVin("LSVAM41Z6F2000001");
boolean validVinDigit = sdk.validateVinWithCheckDigit("LSVAM41Z6F2000001");
boolean validPlate = sdk.validatePlateNumber("京A12345");

// Emission data validation (returns detailed result)
ValidationResult vr = sdk.validateEmission(emissionData);
if (!vr.isValid()) {
    for (String err : vr.getErrors()) {
        System.err.println("Validation error: " + err);
    }
}

// Also available for other data types
sdk.validateObdEngineData(obdData);
sdk.validateHybridData(hybridData);
sdk.validateEmissionCheckData(checkData);
```

### 13. TCP Client (Built-in Transport)

```java
Gb1239Sdk sdk = new Gb1239Sdk();

// Create client (host, port, heartbeatIntervalMs, reconnectDelayMs, maxReconnects)
TcpClient client = new TcpClient("platform.example.com", 8800, 30000, 5000, 10);

// Handle incoming packets
client.setPacketHandler(pkt -> {
    DataType cmd = Gb1239Sdk.getCommandType(pkt);
    if (cmd == DataType.REALTIME) {
        EmissionData em = sdk.decodeRealtimeEmission(pkt);
        // store / process emission data
    }
});

// Handle errors
client.setErrorHandler(err -> System.err.println("Error: " + err.getMessage()));

// Enable automatic heartbeat
client.setHeartbeatPacket(sdk.encodeHeartbeat("LSVAM41Z6F2000001", 1));

// Connect and start reading
client.connect();
client.start();

// Send data
byte[] login = sdk.encodeVehicleLogin(vehicleInfo, 1);
client.send(login);

// Send and wait for response
DataPacket resp = client.sendAndWait(
    sdk.encodeVehicleLogin(vi, 1),
    5000,                      // timeout 5s
    DataType.VEHICLE_LOGIN.getCode()  // expected response command
);

// Graceful shutdown
client.stop();
// or use try-with-resources:
try (TcpClient c = new TcpClient("host", 8800)) {
    c.connect();
    c.start();
    ...
}
```

### 14. Complete End-to-End Example

```java
Gb1239Sdk sdk = new Gb1239Sdk();
String vin = "LSVAM41Z6F2000001";
int seq = 0;

try (TcpClient client = new TcpClient("monitor-platform.com", 8800)) {

    client.setPacketHandler(pkt -> {
        DataType cmd = Gb1239Sdk.getCommandType(pkt);
        System.out.println("Received: " + cmd);
    });

    client.setHeartbeatPacket(sdk.encodeHeartbeat(vin, seq));

    client.connect();
    client.start();

    // Login
    VehicleInfo vi = VehicleInfo.builder()
        .vin(vin).plateNumber("京A12345").plateColor("1")
        .fuelType(FuelType.DIESEL)
        .emissionStandard(EmissionStandard.CHINA_VI)
        .build();
    DataPacket loginResp = client.sendAndWait(
        sdk.encodeVehicleLogin(vi, ++seq),
        5000, DataType.VEHICLE_LOGIN.getCode()
    );

    if (loginResp.getResponseFlag() != 0x01) {
        System.err.println("Login failed");
        return;
    }

    // Report realtime data periodically
    while (true) {
        EmissionData em = EmissionData.builder()
            .timestamp(LocalDateTime.now())
            .vehicleSpeed(60.0).engineSpeed(1500.0)
            .fuelConsumptionRate(8.5).engineCoolantTemp(85.0)
            .scrUpstreamNox(45.0).scrDownstreamNox(5.0)
            .reagentRemaining(80.0).intakePressure(100.0)
            .exhaustFlow(200.0).dpfDifferentialPressure(1.5)
            .reagentLevel(75.0).positionStatus(0x01)
            .longitude(116.397128).latitude(39.916527)
            .odometer(12345.6)
            .build();

        client.send(sdk.encodeRealtimeData(em, vin, ++seq));

        Thread.sleep(10000); // 10s interval per standard requirement
    }
}
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
