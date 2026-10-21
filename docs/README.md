# Commercial SDV Stack Documentation

This documentation expands the repository README into a hands-on guide. The English-language pages in `website/` are designed for both GitHub and the Eclipse SDV Blueprints Docusaurus website.

## Reader Guide

1. [Introduction](website/introduction.md): what the blueprint demonstrates and where to start.
2. [Architecture](website/architecture.md): components, networks, and communication protocols.
3. [Getting Started](website/getting-started.md): prerequisites, startup, verification, and shutdown.
4. [Firmware Update Walkthrough](website/firmware-update.md): create, observe, and remove a Symphony deployment target.
5. [ECU Configuration Walkthrough](website/ecu-configuration.md): trace a powertrain mode change through the stack.
6. [API Exploration](website/api-exploration.md): authenticate and explore the diagnostic interfaces.
7. [Troubleshooting](website/troubleshooting.md): diagnose startup and use-case failures.

## Publishing

Follow [Integrating with the Eclipse SDV Blueprints Website](WEBSITE_INTEGRATION.md). It contains the remote-content configuration, sidebar entries, local preview instructions, and publication checklist.

Keep website pages self-contained: use relative `.md` links between imported pages, absolute GitHub links for source files, and embedded Mermaid diagrams instead of repository-local image dependencies. No separate documentation site or package installation is needed in this repository.