# Changelog

All notable changes to this plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.2.0] — 2026-10-16

### Added

- A Claude Desktop extension, published as a `.mcpb` asset on this
  repository's releases. It installs in one click, asks for the API key in its
  own settings window instead of an environment variable, and keeps that key in
  the operating system's secure storage.
- README sections covering the Desktop install, which surfaces each install
  reaches, and the two authentication paths.

### Changed

- The environment-variable step is now marked as the Claude Code path; the
  extension does not need it.

## [1.1.0] — 2026-07-23

### Changed

- Replaced the broad `chemistry-guide` with the
  `chembricks-computational-chemistry` skill.
- Added explicit completion contracts for requested candidate counts,
  multi-strategy work, failed-candidate replacement, and long-running jobs.
- Updated scientific method-selection and provenance rules from the live MCP
  schemas, including energy-dataset compatibility and model-specific caveats.
- Kept arithmetic, unit conversion, planning, memory, code generation, and
  generic data science native to Claude while retaining chemistry-specific
  reaction validation and molecular optimization tools.
- Documented family-specific polling for async IP/EA, metal-docking, and
  generic queued jobs.

## [1.0.0] — 2026-07-22

### Added

- Initial release of the Chembricks AI Claude Code plugin.
- MCP server configuration connecting to `https://mcp.cmbx.ai/mcp/`.
- Chemistry guide skill with tool selection guidance and common workflows.
- Full tool catalog available live from the MCP server across 21 categories.
- Self-pointing marketplace configuration for direct installation.
