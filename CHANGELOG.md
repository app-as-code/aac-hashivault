# Changelog

All notable changes to this deployment definition are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

## [0.0.1]

### Added

- App-as-Code service definition for HashiCorp Vault `2.2.1` with single-node integrated (Raft) storage.
- Web UI and API on port 8200, routed through Traefik with TLS and automatic DNS.
- File audit device target and rendered Vault configuration as read-only volumes.
- README with runbook, configuration reference and project icon.
