![GITHUB-BADGE](https://github.com/cmangla/AirSane-openwrt/actions/workflows/build.yml/badge.svg)
# AirSane for OpenWRT
The OpenWRT package for [AirSane](https://github.com/SimulPiscator/AirSane)

## Packages
Some OpenWRT packages are attached to [releases](https://github.com/cmangla/AirSane-openwrt/releases).
If you'd like more architectures or OpenWRT versions included in those, please raise a PR that adds them
to `.github/workflows/build.yml`.

## Usage

### Pre-built Packages
Pre-built `.apk` packages are available in [GitHub Releases](https://github.com/cmangla/AirSane-openwrt/releases). Currently built for:
- **OpenWrt 25.12.2** with mediatek-filogic target (uses `.apk` package format)

To request packages for additional OpenWRT versions or architectures, please open an issue or submit a PR to add them to `.github/workflows/build.yml`.

### Build from Source
Build the package for yourself using the OpenWRT SDK Docker image from https://hub.docker.com/r/openwrtorg/sdk

```bash
docker run --rm -v "$(pwd)"/bin/:/home/build/openwrt/bin -it openwrtorg/sdk:mediatek-filogic-25.12.2
```

Inside the container, follow the **Prepare**, **Compile**, and **Install** steps below.

The continuous integration in this repository also builds packages as artifacts for every commit. See the "Actions" tab for recent builds.
### Prepare
Inside the SDK container, add the AirSane repository to your feeds:
```bash
echo "src-git airsaned https://github.com/cmangla/AirSane-openwrt.git" >> feeds.conf.default
```

Update and install the feed:
```bash
./scripts/feeds update base packages airsaned && make defconfig && ./scripts/feeds install airsaned
```

### Compile
Build the package (adjust `CORES_NUM` based on your system):
```bash
make package/airsaned/compile V=s -j $(nproc)
```

The compiled package will be at `bin/packages/<ARCH>/airsaned/airsaned-<VERSION>.apk`

### Install
1. Copy the package to your router:
```bash
scp bin/packages/<ARCH>/airsaned/airsaned-<VERSION>.apk root@<YOUR_ROUTER_IP>:/tmp
```

2. SSH into your router:
```bash
ssh root@<YOUR_ROUTER_IP>
```

3. Install the package:
```bash
apk add /tmp/airsaned-<VERSION>.apk
```

### Configure
Edit the configuration file on your router:
```bash
vi /etc/config/airsaned
```

Available options include:
- `interface` - Network interface to bind to (default: `*` for all)
- `port` - Listen port (default: `8090`)
- `access_log` - Log destination (default: `-` for stderr)
- `hotplug` - Enable scanner hotplug (default: `true`)
- `mdns_announce` - Advertise via mDNS (default: `true`)
- `announce_secure` - Advertise secure mode (default: `false`)
- `web_interface` - Enable web UI (default: `true`)
- `local_scanners_only` - Block remote scanners (default: `true`)
- `debug` - Enable debug output (default: `false`)

See the [Copilot Instructions](/.github/copilot-instructions.md) for the full list of configuration options.

### Start the Service
Start the service once:
```bash
/etc/init.d/airsaned start
```

Enable the service to start on boot:
```bash
/etc/init.d/airsaned enable
```

Access the web interface at `http://<router_ip>:8090`

# Acknowledgements
This is a fork of the original implementation by
[polikasov](https://github.com/polikasov/AirSane-openwrt),
with changes by
[nevian427](https://github.com/nevian427/AirSane-openwrt),
[ypopovych](https://github.com/ypopovych/AirSane-openwrt),
[shawnking07](https://github.com/shawnking07/AirSane-openwrt)
and
[alryaz](https://github.com/alryaz)
merged in.
