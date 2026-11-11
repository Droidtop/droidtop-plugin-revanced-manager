# Changelog

All notable changes this fork makes are documented in this file. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Upstream's own
changes arrive through the upstream sync and are recorded in ReVanced/revanced-manager's history; the
fork does not cut versioned releases of its own, so its changes are recorded
under Unreleased.

## [Unreleased]

### Added

- The plugin plan, rewritten so plugins run in droidtop's own context (2026-09-25), the licence note and the daily upstream sync (2026-09-25).
- A commit-hygiene check and the upstream sync, both called from the shared workflows in `Droidtop/droidtop-platforms`.

### Changed

- The private-fork framing is dropped now that the plugin is public (2026-09-28).
- The upstream merge keeps this fork's README (2026-10-09): the daily sync had stopped on that one conflict since 2026-09-30.

### Removed

- Upstream's "Pull strings" workflow is disabled in the repository settings (2026-10-09): it pulls translations from a Crowdin project this fork has no access to and checks out a branch the fork does not have.
