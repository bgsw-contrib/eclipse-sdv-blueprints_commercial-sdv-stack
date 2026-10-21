---
sidebar_position: 1
title: Introduction
description: Run a container-based commercial vehicle blueprint for firmware deployment orchestration and ECU configuration with Eclipse SDV components.
---

# Commercial SDV Stack

Explore how a fleet backend can orchestrate firmware deployments and change an ECU's configuration using Eclipse SDV software. This blueprint runs the backend and a simulated vehicle together with Docker Compose; no physical ECU or vehicle is required.

| Topic | Details |
| --- | --- |
| Use cases | OTA firmware deployment orchestration and OTA ECU configuration changes |
| Eclipse projects | Eclipse Symphony, Eclipse uProtocol, Eclipse OpenSOVD, Eclipse Kuksa, and Eclipse Mosquitto |
| Diagnostic protocols | SOVD over HTTP, UDS over DoIP |
| Runtime | Docker Engine or Docker Desktop with Docker Compose v2 |
| Source | [eclipse-sdv-blueprints/commercial-sdv-stack](https://github.com/eclipse-sdv-blueprints/commercial-sdv-stack) |

## What You Will Learn

- Create a Symphony deployment target and observe the in-vehicle ECU Updater respond.
- Follow a fleet management command over uProtocol to an SOVD diagnostic operation.
- Observe the ECU Simulator process a UDS powertrain mode write.
- Inspect services through container logs, the Symphony Portal, and diagnostic APIs.

## Choose a Walkthrough

Begin with [Getting Started](getting-started.md) to launch and verify the stack, then try either use case:

| Walkthrough | You do | You observe |
| --- | --- | --- |
| [Firmware Update](firmware-update.md) | Run a script to create and later delete a deployment target | Symphony forwards the desired deployment state to the ECU Updater |
| [ECU Configuration](ecu-configuration.md) | Watch the automatically started Fleet Management System | Powertrain mode changes travel through uProtocol, CDA, and UDS to the simulator |

Read the [Architecture](architecture.md) for context, use [API Exploration](api-exploration.md) for manual diagnostic requests, and consult [Troubleshooting](troubleshooting.md) when a step fails.

## Scope and Limitations

The firmware use case demonstrates **deployment orchestration**, not ECU flashing. The ECU Updater only maintains deployment state in memory; it does not download or install the example firmware images. Connecting this path to the OpenSOVD Classic Diagnostic Adapter for an actual UDS firmware update is a future extension.

The configuration use case does reach the **simulated ECU**: the Classic Diagnostic Adapter translates an SOVD data write into a UDS operation over DoIP. It does not modify a physical vehicle.

This is a demonstration environment, not a production deployment. It uses demo credentials and tokens, unsecured internal communication, and a log viewer with Docker socket access. Run it only on a trusted development machine. The diagnostic ports are not bound exclusively to loopback by default; see the [setup safety notes](getting-started.md#before-you-start).