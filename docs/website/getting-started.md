---
sidebar_position: 3
title: Getting Started
description: Prepare your development machine, start the Commercial SDV Stack, and verify the services before running the use cases.
---
<!--
 Copyright (c) 2026 Contributors to the Eclipse Foundation

 See the NOTICE file(s) distributed with this work for additional
 information regarding copyright ownership.

 This program and the accompanying materials are made available under the
 terms of the Apache License Version 2.0 which is available at
 https://www.apache.org/licenses/LICENSE-2.0

 Assisted-by: GPT-6
 SPDX-License-Identifier: Apache-2.0
-->

# Getting Started

## Prerequisites

You need:

- A development machine with Docker Engine or Docker Desktop running and Docker Compose v2 installed.
- Git to clone the repository.
- Bash, `curl`, and `jq` to run the firmware update script and API examples.
- A browser for Dozzle and the Symphony Portal.
- Internet access to GitHub Container Registry (`ghcr.io`) and Docker Hub to pull container images.

No physical ECU, Kubernetes cluster, local Rust toolchain, or local JVM is needed when using the published container images. If an image is unavailable for your host architecture, see [Troubleshooting](troubleshooting.md#container-images-cannot-be-pulled).

Verify the command-line tools:

```bash
docker version
docker compose version
git --version
bash --version
curl --version
jq --version
```

`docker version` must report a reachable server, not just a client version.

## Before You Start

Use a trusted development host. The stack includes demo credentials and tokens, a Kuksa connection without TLS, and Dozzle with a mounted Docker socket. Do not deploy the configuration unchanged on a public server or production vehicle.

Most published ports are bound to `127.0.0.1`, but the CDA and ECU Simulator control ports bind to all host interfaces by default. To keep those diagnostic APIs local, run the following in the same Bash session as the Compose commands:

```bash
export CDA_PORT=127.0.0.1:20002
export SIM_CONTROL_PORT=127.0.0.1:8181
```

The Compose port mappings append the container ports to these values. Keep the same variables set for later Compose commands. Review firewall rules and access controls before sharing any service.

## 1. Clone the Repository

```bash
git clone https://github.com/eclipse-sdv-blueprints/commercial-sdv-stack.git
cd commercial-sdv-stack
```

All subsequent shell commands assume this repository root as the working directory unless stated otherwise.

## 2. Start the Stack

```bash
docker compose up -d
docker compose ps --all
```

The first start pulls several images and may take longer than subsequent starts. Give the services time to initialize. The MQTT broker and ECU Simulator have health checks; a running container alone does not prove every downstream API is ready.

To inspect startup logs:

```bash
docker compose logs --tail=100 mosquitto ecu-sim sovd-cda
docker compose logs --tail=100 symphony-api symphony-agent
```

## 3. Verify Access

| Service | Default host address | Verification |
| --- | --- | --- |
| Dozzle | http://localhost:8080 | Container list and live logs are visible |
| Symphony API | http://localhost:8082/v1alpha2/greetings | Returns `Hello from Symphony K8s control plane (S8C)` |
| Symphony Portal | http://localhost:3000 | Log in as `admin` with an empty password; inspect **Targets** |
| SOVD API | http://localhost:20002/vehicle/v15 | Use the authenticated requests in [API Exploration](api-exploration.md) |
| ECU Simulator control API | http://localhost:8181 | The control server responds |
| Kuksa Databroker | `localhost:55555` | gRPC endpoint, not a browser UI |
| MQTT broker | `localhost:1883` | MQTT endpoint, not a browser UI |

You can check the Symphony greeting directly:

```bash
curl --fail-with-body http://localhost:8082/v1alpha2/greetings
```

When using a remote development host or dev container, `localhost` refers to the machine running Docker. Forward the required ports to your browser's machine. Do not make demo endpoints publicly accessible to work around a forwarding issue.

## 4. Run the Use Cases

- [Firmware Update](firmware-update.md): create a deployment target and watch the ECU Updater process it.
- [ECU Configuration](ecu-configuration.md): observe the Fleet Management System automatically cycle powertrain modes.

The configuration use case starts with the stack. The firmware use case starts only when you run its script.

## 5. Stop and Clean Up

Finish the firmware script by pressing Enter so it deletes its target, then stop the stack:

```bash
docker compose down
```

Compose preserves the named Mosquitto data and log volumes. For a fresh environment, the following command also deletes those volumes and their stored contents:

```bash
docker compose down --volumes
```

Neither command deletes your repository files. The ECU Updater's in-memory deployment state is lost when its container stops.

## Next Steps

Continue with the [Firmware Update Walkthrough](firmware-update.md), or read [Troubleshooting](troubleshooting.md) if a service did not start.