# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2026-01-11

### Added

- Initial release of RTLAMR WebSocket Add-on for Home Assistant
- RTL-SDR integration using rtl_tcp and rtlamr
- WebSocket server for real-time meter reading broadcasts
- Support for multiple meter protocols:
  - IDM (Interval Data Message)
  - NetIDM (Network IDM)
  - R900 (Badger Orion)
  - R900BCD (R900 with BCD encoding)
  - SCM (Standard Consumption Message)
  - SCM+ (Enhanced SCM)
- Multi-meter configuration support
- Configurable reading format with decimal point placement
- Health check endpoint at `/health`
- WebSocket endpoint at `/ws` with automatic client management
- Cached readings for new WebSocket clients
- S6-overlay service management
- Multi-architecture support (amd64, aarch64)
- GitHub Actions CI/CD workflows
- Comprehensive configuration schema validation

### Configuration Options

- `general.sleep_for` - Interval between reading cycles
- `general.verbosity` - Log level (debug, info, warning, critical, none)
- `general.device_id` - RTL-SDR device index
- `general.rtltcp_host` - Optional external rtl_tcp server
- `custom_parameters.rtltcp` - Additional rtl_tcp arguments
- `custom_parameters.rtlamr` - Additional rtlamr arguments
- `meters` - List of meter configurations with protocol, format, and Home Assistant attributes

[Unreleased]: https://github.com/alex-savin/hassio-addon-rtlamr-ws/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/alex-savin/hassio-addon-rtlamr-ws/releases/tag/v1.0.0
