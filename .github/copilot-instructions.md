# Copilot Instructions for AirSane-openwrt

## Overview

This repository is an **OpenWRT package** for [AirSane](https://github.com/SimulPiscator/AirSane), a SANE WebScan frontend with Apple AirScan protocol support. The package is maintained as an OpenWRT feed that integrates AirSane into custom router builds.

## Repository Structure

- **`airsaned/`** - OpenWRT package definition
  - `Makefile` - Package metadata, version info, build configuration, and dependencies (libsane, libjpeg, libpng, libavahi-client, libusb-1.0, libstdcpp, libatomic)
  - `files/` - Configuration and init files
    - `airsaned.init` - OpenWRT init script (uses procd for service management)
    - `airsaned.default` - Default configuration with 11 runtime options (port 8090, interface binding, logging, etc.)
- **`.github/workflows/`** - CI/CD pipelines
  - `build.yml` - Builds packages for specified OpenWRT releases and SDK targets
  - `release.yml` - Release automation
  - `update_release.yml` - Release update workflows
- **`.github/work.sh`** - Workflow helper script for extracting package version info from Makefile

## Build & Package Management

### Package Version Scheme

Package versions are generated dynamically from the Makefile:
- Format: `{PKG_SOURCE_DATE}-SV-{PKG_SOURCE_VERSION_SHORT}-{PKG_RELEASE}`
- Example: `2026-02-18-SV-291513f-1`
- Extract version info: `bash .github/work.sh pkg-ver airsaned/Makefile`

### Updating AirSane Source

To update to a new upstream AirSane release in `airsaned/Makefile`:
1. Update `PKG_SOURCE_VERSION` (commit hash from https://github.com/SimulPiscator/AirSane)
2. Update `PKG_SOURCE_DATE` (date of the commit)
3. Update `PKG_VERSION` (semantic version of AirSane itself)
4. Set `PKG_MIRROR_HASH=skip` for first build, then run `make` and replace with actual hash
5. Run CI to generate packages for configured targets

### Build Configuration

The build matrix in `.github/workflows/build.yml` defines which OpenWRT releases and SDK targets are built:
- Current matrix: OpenWRT 25.12.2 for mediatek-filogic target
- To add more architectures/releases: Edit the `matrix` section in `build.yml` and add them to the list

### Building Locally

Within an OpenWRT SDK environment:
```bash
# Single package
make package/airsaned/compile V=s -j$(nproc)

# Full rebuild with clean
make package/airsaned/clean package/airsaned/compile V=s -j$(nproc)
```

Output: `bin/packages/<ARCH>/airsaned/airsaned-<VERSION>.ipk`

## Configuration & Runtime

### Init Script (`airsaned.init`)

- Uses **procd** for service management (OpenWRT standard)
- Loads config from `/etc/config/airsaned` on startup
- Respawn policy: 3600s cooldown, max 15 retries, 5s between attempts
- Service start priority: 95, stop priority: 10

### Configuration File (`airsaned.default`)

Global section options (all boolean or string values):
- `interface` - Network binding (default: '*' = all)
- `port` - Listen port (default: 8090)
- `access_log` - Log destination (default: '-' = stderr)
- `hotplug` - Scanner hotplug support (default: true)
- `mdns_announce` - mDNS advertisement (default: true)
- `announce_secure` - Secure mode announcement (default: false)
- `unix_socket` - Unix socket path (default: empty)
- `web_interface` - Enable web UI (default: true)
- `reset_option` - Allow device reset (default: true)
- `disclose_version` - Advertise version (default: true)
- `local_scanners_only` - Block remote scanners (default: true)
- `compatible_path` - Use compatible URL paths (default: true)
- `debug` - Debug output (default: false)
- `random_paths` - Randomized URL paths (default: false)

## Key Conventions

### Makefile Metadata

- **PKG_NAME** - Exactly matches directory name (`airsaned`)
- **PKG_INSTALL=1** - Enables the `define Package/airsaned/install` block
- **PKG_FIXUP=autoreconf** - Runs autoconf before build (AirSane uses autoconf)
- **PKG_BUILD_PARALLEL=1** - Enables parallel make jobs
- **PKG_MIRROR_HASH** - Security hash; use `skip` during development

### File Placement in OpenWRT Package

Files are installed with specific paths in the init script and Makefile:
- Binaries: `/usr/bin/airsaned` (executable)
- Init script: `/etc/init.d/airsaned` (executable)
- Config: `/etc/config/airsaned` (config file, preserved on upgrades)

Configuration files are declared with `Package/airsaned/conffiles` to prevent overwrites during updates.

### CI/CD Matrix Expansion

When adding new OpenWRT versions or architectures:
1. Add to `matrix.wrtrel` list (e.g., `25.03.0-rc2`)
2. Add to `matrix.sdk` list (e.g., `ramips-mt7621`, `bcm27xx-bcm2708`)
3. The GitHub action automatically generates build names combining both dimensions
4. Artifacts uploaded with naming: `airsaned_<VERSION>_openwrt-<WRT_REL>_<SDK>.apk.d`

## Testing & Validation

- No unit tests in this repository (package-only)
- Test packages: Deploy `.ipk` to OpenWRT device and verify with:
  ```bash
  opkg install /tmp/airsaned-<VERSION>.ipk
  /etc/init.d/airsaned start
  # Verify: web interface at http://<router>:8090, scanner discovery via AirScan
  ```
- Validate against target architecture using the SDK container for that architecture

## Important Notes

- Package is GPL-3.0 licensed (inherited from AirSane upstream)
- All upstream AirSane changes are in the external git repository (not in this repository)
- This repository only maintains the OpenWRT packaging wrapper and CI/CD
- Releases are published to GitHub Releases with pre-built `.ipk` files
