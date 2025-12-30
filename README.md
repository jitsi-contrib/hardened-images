# Hardened Images

This repository provides **hardened Docker images** for Jitsi Meet.

The goal is to offer a more secure version of the Jitsi stack while staying as
close as possible to
[the official images](https://github.com/jitsi/docker-jitsi-meet). These images
are designed for compatibility; however, because of the hardening steps, **minor
configuration changes** are required compared to the standard setup.

#### Security Hardening

- **Non-Root Processes** \
  All components are configured to run with unprivileged user accounts to limit
  potential impact if a process is compromised.

- **Read-Only Filesystem** \
  Containers are built to support running with a read-only root filesystem
  (_with specific paths mounted as writable volumes_), preventing unauthorized
  persistent changes to the image at runtime.

#### Key Focus

- **Reduced Attack Surface** \
  Focused strictly on security-first builds.

- **Compatibility** \
  Designed to function with official Jitsi logic with minimal adjustments.

## Volumes

```bash
mkdir -p ~/.jitsi-meet-volumes/prosody/{config,prosody-plugins-custom}
mkdir -p ~/.jitsi-meet-volumes/{jibri,jicofo,jigasi,jvb,web}

mkdir -p ~/.jitsi-meet-volumes/storage/{jibri,prosody,transcripts}
chmod 777 ~/.jitsi-meet-volumes/storage/jibri
chmod 777 ~/.jitsi-meet-volumes/storage/prosody
chmod 777 ~/.jitsi-meet-volumes/storage/transcripts

mkdir -p ~/.jitsi-meet-volumes/tmp/{web-crontabs,web-load-test}
chmod 777 ~/.jitsi-meet-volumes/tmp/web-crontabs
chmod 777 ~/.jitsi-meet-volumes/tmp/web-load-test
```

## Environment file

Run the following commands for a quick test in a local environment. Set your own
local IP for `MY_IP`:

```bash
cp env.example .env
bash gen-passwords.sh

mv .env .env.myconfig

MY_IP=172.18.18.1

cat <<EOF >>.env.myconfig

# Customized values
PUBLIC_URL=https://${MY_IP}:8443
JVB_ADVERTISE_IPS=${MY_IP}

ENABLE_RECORDING=1
IGNORE_CERTIFICATE_ERRORS=true
EOF
```

## Running

```bash
# environment variables
export COMPOSE_PATH_SEPARATOR=","
export COMPOSE_FILE="docker-compose.yml,jibri.yml,jigasi.yml"
export COMPOSE_ENV_FILES=".env.myconfig"

# pull
docker compose pull

# run
docker compose up -d

# list running containers
docker compose ps

# stop
docker compose down --remove-orphans
```

## Implementation details

- [s6-overlay](https://github.com/just-containers/s6-overlay) is upgraded to the
  latest stable version (`v3.2.1.0`)
- All processes are executed by `s6` user without `root` privileges.
- Containers' filesystems are read-only.
- Mounted volumes:
  - `/config` \
    _read-only volume containing custom config files_
  - `/storage` \
    _writable volume containing files (recording, logs, etc.) created during the
    runtime_
  - `/tmp` \
    _writable volume containing temporary files created during the runtime_
- The following `tmpsfs` folders are also writable:
  - `/run`
  - `/tmp`
- Generated config files are created inside `/run` using templates.
