# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - Initial Public Release

### Added
- Core `cronprobe` bash wrapper script.
- Zero-dependency architecture relying only on standard utilities (`bash`, `curl`).
- Slack, Discord, Ntfy, and custom generic JSON webhook alert integrations.
- Capture and delivery of `stdout` and `stderr` on cron job failure (non-zero exit).
- Configurable truncation for large outputs to prevent payload limits.

### Limitations (Honest Scope)
- This is a local wrapper, not a SaaS monitor. It cannot detect "missed runs" (if a server dies or cron daemon stops).
- Alert delivery relies on the host network condition at the exact time of job failure.
- No historical dashboards, metrics, or retry logic.
