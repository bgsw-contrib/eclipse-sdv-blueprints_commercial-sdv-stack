---
sidebar_position: 7
title: Troubleshooting
description: Diagnose container startup, Symphony deployment, powertrain configuration, and diagnostic API problems.
---

# Troubleshooting

Run commands from the repository root, with the same `CDA_PORT` and `SIM_CONTROL_PORT` environment values used at startup, if any.

## Start with Container Status and Logs

```bash
docker compose ps --all
docker compose logs --tail=100
```

Look for exited containers, an unhealthy ECU Simulator or MQTT broker, and repeated connection or authentication errors. Dozzle at http://localhost:8080 provides a browser view of the same container logs.

## Docker Is Not Reachable

If `docker version` cannot contact the server, start Docker Engine or Docker Desktop and confirm your user can access its socket. Do not change the socket to world-writable permissions to bypass an access error.

## Container Images Cannot Be Pulled

Check registry access, your network connection, the image name in the Compose file, and whether the published image supports your host architecture:

```bash
docker compose pull
```

If you need to build an image locally, the Compose file includes build definitions. For example, to build the local Rust services:

```bash
docker compose build fms powertrain-mode-controller vehicle-properties
docker compose up -d
```

Building requires access to dependency registries and more time and disk space than pulling images. Some other build contexts are remote Git repositories, so not every build is self-contained in this checkout. Consult the [uServices](https://github.com/eclipse-sdv-blueprints/commercial-sdv-stack/blob/main/uservices/README.md), [ECU Simulator](https://github.com/eclipse-sdv-blueprints/commercial-sdv-stack/blob/main/ecu-sim/README.md), and [ODX](https://github.com/eclipse-sdv-blueprints/commercial-sdv-stack/blob/main/odx/README.md) documentation for development details.

## A Port Is Already in Use

Stop the conflicting local service or change the host mapping. CDA and the simulator have configurable host ports:

```bash
export CDA_PORT=127.0.0.1:20003
export SIM_CONTROL_PORT=127.0.0.1:8182
docker compose up -d
```

In this example, use `http://localhost:20003/vehicle/v15` and `http://localhost:8182` from the host. Internal Compose service addresses remain unchanged. Keep the same variables set for subsequent Compose commands. Other published ports require a Compose configuration change.

## Browser Services Are Unreachable

Confirm the corresponding container is running and the published port is correct. For a remote Docker host, configure private port forwarding to the machine running your browser. Host-side URLs use `localhost`; names such as `sovd-cda` resolve only inside the relevant Compose network.

## Firmware Target Does Not Reach the ECU Updater

Confirm the Symphony greeting works and the target appears in the portal, then inspect the orchestration chain:

```bash
docker compose logs --tail=150 symphony-api symphony-agent ecu-updater mosquitto
```

Check that the agent and updater connect to MQTT. The trigger script's waiting prompt does not validate the HTTP response; an API error can still be followed by the prompt. Review its response output and check that `jq` is installed and authentication returned a token.

Only one walkthrough should manage `ecu-updater-target` at a time. Clean up a stale target before retrying.

## Firmware Script Was Interrupted

If the script was stopped before its deletion step, authenticate to Symphony and remove its target manually:

```bash
SYMPHONY_BASE_URL=http://localhost:8082/v1alpha2
SYMPHONY_TOKEN=$(curl --fail-with-body -sS -X POST \
  -H 'Content-Type: application/json' \
  --data '{"username":"admin","password":""}' \
  "$SYMPHONY_BASE_URL/users/auth" | jq -er '.accessToken')
```

After confirming authentication succeeded:

```bash
curl --fail-with-body -sS -X DELETE \
  -H "Authorization: Bearer $SYMPHONY_TOKEN" \
  "$SYMPHONY_BASE_URL/targets/registry/ecu-updater-target"
```

If the target is already absent, no further deletion is needed. Otherwise, wait for reconciliation and verify its removal in the portal before running the script again.

## Powertrain Modes Do Not Change

```bash
docker compose logs --tail=150 fms powertrain-mode-controller sovd-cda ecu-sim mosquitto
```

Check the chain in order: FMS requests, uProtocol/MQTT connectivity, the controller's SOVD destination, CDA responses, then simulator UDS requests. The controller's success message alone does not prove CDA accepted the write; verify the simulator log or [read the current mode](api-exploration.md#read-the-current-powertrain-mode).

If you previously paused FMS, resume it:

```bash
docker compose start fms
```

If you experimented with locks, variants, or sessions, follow [Return to the Baseline Demo](api-exploration.md#return-to-the-baseline-demo). Startup readiness can also cause transient RPC failures; persistent failures after initialization need investigation.

## Diagnostic API Requests Fail

- **401 or 403:** obtain a new demo access token and ensure the `Authorization: Bearer` header is present. The controller has its own configured token, separate from the shell token used in API examples.
- **404 or missing resources:** verify the base URL includes `/vehicle/v15`, the component is `blueprint-ecu`, and the expected variant is detected.
- **Lock or session errors:** check whether an earlier API experiment left an active lock or changed the diagnostic session.
- **Connection refused:** confirm CDA and the simulator are running and use the host ports selected at startup.

Use `curl --fail-with-body -sS` as shown in the [API guide](api-exploration.md) to retain useful HTTP error responses.

## Start Fresh

First finish or clean up any firmware walkthrough. To recreate the stack while preserving Mosquitto volumes:

```bash
docker compose down
docker compose up -d
```

For a complete reset of the named Mosquitto data and log volumes, use `docker compose down --volumes` instead. This permanently deletes those volumes' contents.

## Report an Issue

Open an issue in the [source repository](https://github.com/eclipse-sdv-blueprints/commercial-sdv-stack/issues) with your host OS and architecture, Docker and Compose versions, the failing step, container status, and relevant log excerpts. Redact tokens, credentials, and sensitive vehicle data before sharing output.