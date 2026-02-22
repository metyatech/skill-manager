# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-02-22

### Changed

- Updated monitoring section: prescribe non-blocking behavior (never use `Status(wait=true)`, call `Status(wait=false)` on every response, use background `agents-mcp wait` on Claude Code).
- Updated dispatch workflow: return control to user immediately after spawning, check status on every subsequent message.

## [0.1.0] - Initial release
