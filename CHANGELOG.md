# CHANGELOG

## [0.1.0] - 2026-05-22

### Role

- Rebuilt the role as a modern standalone Ansible role for installing and
  configuring Fail2Ban.
- Added support for `present` and `latest` package states.
- Added compatibility handling so string boolean values such as `"false"` do
  not accidentally select the `latest` package state.
- Added apt cache refresh support for apt-based targets.
- Added EPEL repository package installation for Enterprise Linux targets.
- Added managed `jail.local` rendering from namespaced `fail2ban_*`
  variables.
- Added optional custom filter and action template rendering.
- Added Fail2Ban service management controls.
- Added post-install verification for `fail2ban-client`.
- Added role input validation for package, configuration, service, and
  verification settings.

### Platform Support

- Added Galaxy metadata for current Debian, Ubuntu, Fedora, and Enterprise
  Linux releases.
- Added Docker validation for Debian 13, Debian 12, Rocky Linux 10, Rocky
  Linux 9, Ubuntu 26.04 LTS, and Ubuntu 24.04 LTS.

### Test Harness and CI

- Added a Workspace-managed Docker test harness using the Inviqa Ansible 2.20
  and Python 3.13 image track.
- Kept the Docker test harness limited to repository and Docker socket mounts,
  without SSH-agent or SSH config dependencies.
- Pinned the local Docker harness interpreter to avoid Ansible localhost
  interpreter-discovery warnings.
- Namespaced Docker harness image and platform variables as test-only inputs.
- Kept the Jenkinsfile lint image limited to plugins required by the local
  pipeline validator.
- Added reusable Workspace commands for Ansible lint, syntax checks, Docker
  tests, Jenkinsfile linting, GitHub release checks, and Ansible Galaxy import.
- Added Jenkins pipeline scaffolding that runs Workspace lint, syntax, Docker
  tests, release preflight, and optional main-branch publication stages.
- Added release-helper guards so GitHub release checks fail clearly when the
  GitHub token is missing.

### Documentation

- Added README documentation for installation, role variables, examples,
  supported platforms, testing, CI, publishing, support, inspiration, and
  licensing.
- Added test-harness, Jenkins CI, and Galaxy release documentation.
- Added source-backed Mermaid flowcharts for role execution, Docker testing,
  Jenkins CI, and release publication workflows.
- Added repository agent instructions and TODO tracking for release blockers.
- Added agent guidance for Workspace helper exit-status propagation and
  release-token guards.
