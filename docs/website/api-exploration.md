---
sidebar_position: 6
title: API Exploration
description: Authenticate to CDA, inspect the simulated ECU, and explore diagnostic sessions and variant detection.
---

# API Exploration

Use these examples after [starting the stack](getting-started.md). They run from your host in Bash and require `curl` and `jq`. They use the default host ports; adjust the base URLs if you changed port mappings.

These are demo APIs and credentials, not production security guidance. Do not reuse the demo secrets in a real deployment.

## Authenticate to CDA

Request an access token and store it in the current shell:

```bash
export SOVD_BASE_URL=http://localhost:20002/vehicle/v15
export SIM_BASE_URL=http://localhost:8181

ACCESS_TOKEN=$(curl --fail-with-body -sS -X POST \
  -H 'Content-Type: application/json' \
  --data '{"client_id":"test", "client_secret":"secret"}' \
  "$SOVD_BASE_URL/authorize" | jq -er '.access_token')
```

If the request fails or `jq` reports a missing token, resolve that error before continuing. Repeat this step in a new terminal or when the token expires.

## Inspect the ECU Component

Retrieve the standardized resource collection for the ECU and its detected variant:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  "$SOVD_BASE_URL/components/blueprint-ecu" | jq .
```

To force variant detection again:

```bash
curl --fail-with-body -sS -X PUT \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  "$SOVD_BASE_URL/components/blueprint-ecu"
```

## Read the Current Powertrain Mode

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  "$SOVD_BASE_URL/components/blueprint-ecu/data/powertrain_mode" | jq .
```

While FMS is running, repeated reads may show different modes. Pause it with `docker compose stop fms` for a stable observation, and resume with `docker compose start fms` when finished.

## Explore Diagnostic State

The following requests change diagnostic state. Pause FMS first to avoid interference with the automatic walkthrough:

```bash
docker compose stop fms
```

### Acquire a Component Lock

```bash
curl --fail-with-body -sS -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H 'Content-Type: application/json' \
  --data '{"lock_expiration": 100000}' \
  "$SOVD_BASE_URL/components/blueprint-ecu/locks"
```

Inspect the response for the lock details. An active lock may affect access by other clients; do not assume it is shared with the Powertrain Mode Controller.

### Switch to the Extended Diagnostic Session

```bash
curl --fail-with-body -sS -X PUT \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H 'Content-Type: application/json' \
  --data '{"value": "extended"}' \
  "$SOVD_BASE_URL/components/blueprint-ecu/modes/session"
```

### Switch the Simulator to the Boot Variant

This uses the ECU Simulator control API directly, not SOVD:

```bash
curl --fail-with-body -sS -X PUT \
  -H 'Content-Type: application/json' \
  --data '{"variant": "BOOT"}' \
  "$SIM_BASE_URL/blueprint-ecu/state"
```

The boot variant may expose different diagnostic resources. Force variant detection again before inspecting its component collection. Do not expect the normal powertrain walkthrough to work unchanged while experimenting with another variant or diagnostic session.

### Return to the Baseline Demo

To discard the experimental diagnostic state, recreate the simulator and CDA, then resume FMS. This interrupts active diagnostic requests and connections:

```bash
docker compose up -d --force-recreate ecu-sim sovd-cda
docker compose ps --all
```

Wait for the simulator to be healthy and CDA to be ready. Then resume the automatic writer:

```bash
docker compose start fms
```

Confirm recovery in the [ECU configuration logs](ecu-configuration.md). For authentication errors, locks, or missing resources, see [Troubleshooting](troubleshooting.md#diagnostic-api-requests-fail).