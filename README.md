# PeakMC

> High-Performance Server Networking, Deep Packet Inspection & Core Extension Framework for Minecraft: Bedrock Dedicated Server (LeviLamina)

[![Platform](https://img.shields.io/badge/platform-BDS%20%7C%20Windows%20x64-blue.svg)](https://github.com/LiteLDev/LeviLamina)
[![Framework](https://img.shields.io/badge/framework-LeviLamina-green.svg)](https://github.com/LiteLDev/LeviLamina)
[![C++ Standard](https://img.shields.io/badge/C%2B%2B-20%2F23-orange.svg)](https://en.wikipedia.org/wiki/C%2B%2B23)
[![License](https://img.shields.io/badge/license-GPLv3-lightgrey.svg)](LICENSE)

---

## 📖 Overview

**PeakMC** is an advanced native C++ server mod and core extension framework built on top of [LeviLamina](https://github.com/LiteLDev/LeviLamina) for Minecraft: Bedrock Dedicated Server (BDS). 

While conventional server plugins rely on high-level gameplay event abstractions, PeakMC operates at the lower levels of the BDS engine. Its goal is to provide enterprise-grade server infrastructure: low-latency packet routing, custom packet rate-limiting, deep packet inspection (DPI) to mitigate network-level exploits, sub-client multi-device session management, and granular protocol-level lifecycle hooks.

---

## 🎯 Engine Integration & Internal Symbols

In the Bedrock Dedicated Server architecture, inbound packet handling is strictly routed through `ServerNetworkHandler`. Unlike engines that use a generic dispatch queue, BDS declares explicit overload methods for every individual packet structure:

```cpp
virtual void handle(NetworkIdentifier const& source, PacketType const& packet);
void $handle(NetworkIdentifier const& source, PacketType const& packet);
```

To provide zero-overhead packet filtering, pre-dispatch sanitization, and state validation before internal engine mutations occur, PeakMC directly interfaces with:

1. **Inbound Packet Dispatchers (`ServerNetworkHandler::handle`)**:
   - Authentication and Handshake (`ClientToServerHandshakePacket`, `SubClientLoginPacket`).
   - Gameplay Actions & Inventory (`PlayerHotbarPacket`, `ActorPickRequestPacket`, `BookEditPacket`, `CompletedUsingItemPacket`).
   - World & Structural Operations (`StructureBlockUpdatePacket`, `StructureTemplateDataRequestPacket`, `LecternUpdatePacket`).
   - Session & Sync (`ClientMovementPredictionSyncPacket`, `PlayerToggleCrafterSlotRequestPacket`, `ServerboundLoadingScreenPacket`).

2. **Connection Lifecycle & Security Callbacks**:
   - Connection establishment and negotiation (`onConnect`, `completeHandshake`, `onPlayerReady`).
   - Malformed packet containment and policy enforcement (`handlePacketViolation`, `sendPacketViolationWarningPacket`).
   - Multi-session sub-client synchronization (`sendSubClientLoginMessageLocal`, `onTransferRequest`).
   - Disconnect handling and state cleanup (`onTick`, `removeFromDenyList`).

PeakMC hooks and overrides these handlers to implement strict protocol analysis, custom rate limiting, and exploit mitigations. Interacting with these packet types and lifecycle handlers at native speed requires linking against these underlying BDS internal routines.

---

## 🏗️ Architecture & Subsystems

```
                     +---------------------------------------+
                     |         Minecraft Client(s)           |
                     +---------------------------------------+
                                         │
                                [Network Packets]
                                         ▼
                     +---------------------------------------+
                     |     ServerNetworkSystem (RakNet)      |
                     +---------------------------------------+
                                         │
                     ┌───────────────────▼───────────────────┐
                     │           PeakMC Network Hook         │
                     │  - Deep Packet Inspection (DPI)       │
                     │  - Protocol Violation Containment    │
                     │  - Rate Limiting & Filter Matrix      │
                     └───────────────────┬───────────────────┘
                                         │  (validated & sanitized)
                     ┌───────────────────▼───────────────────┐
                     │         ServerNetworkHandler          │
                     │  - handle(Source, Packet) Overloads   │
                     │  - Connection Lifecycle & Auth        │
                     └───────────────────┬───────────────────┘
                                         │
                                         ▼
                     +---------------------------------------+
                     |            BDS Engine Core            |
                     +---------------------------------------+
```

### Core Subsystems:
* **`PeakMC::Network`**: Network packet routing engine, packet interception matrix, and direct `ServerNetworkHandler` hook points.
* **`PeakMC::Security`**: Inbound payload validation, packet flood dampening, malformed packet violation handling.
* **`PeakMC::Session`**: Handshake verification, connection negotiation, sub-client multi-seat routing.
* **`PeakMC::Sync`**: Low-latency player movement prediction verification and server-side state confirmation.

---

## 🔧 Building & Requirements

* **Operating System**: Windows Server 2022+ / Windows 10/11 x64
* **Compiler**: Visual Studio 2022 (MSVC v143 toolset) with C++20/C++23 support
* **Framework**: [LeviLamina](https://github.com/LiteLDev/LeviLamina) SDK (v0.14+)
* **Build System**: [XMake](https://xmake.io/) or CMake 3.25+

```powershell
# Clone the repository
git clone https://github.com/LordBombardir/PeakMC.git
cd PeakMC

# Configure and build via XMake
xmake f -m release -a x64
xmake
```

---

## 📜 License

PeakMC is distributed under the GNU General Public License v3.0 (GPLv3).
Minecraft is a registered trademark of Mojang Synergies AB.
