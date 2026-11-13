# wifi-surveyor

OpenWrt package that provides:

- AP-side metrics agent: `/usr/sbin/wifi-surveyor-agent --json`
- AP-side cache refresher: `/usr/sbin/wifi-surveyor-cache`
- AP key helper: `/usr/sbin/wifi-surveyor-authorize-key`
- Main-router API: `/www/cgi-bin/wifi-surveyor-api`

## Security model

- Main router calls APs over SSH using a dedicated key.
- APs should install the public key with a forced command:
  - `command="/usr/sbin/wifi-surveyor-agent --json"`
  - no PTY/forwarding permissions.
- Main API requires `X-Auth-Token` (or `?token=`).

## Configure

Edit `/etc/config/wifi-surveyor`:

- `auth_token` for API access
- `ssh_user`, `ssh_key`, `ssh_connect_timeout`
- `sample_interval` (seconds; default `5`) for background cache refresh
- `cache_max_age` (seconds; default `20`) accepted staleness for cached stats
- `push_enabled` (set `1` on APs)
- `push_target` (default `http://10.0.0.1/cgi-bin/wifi-surveyor-api?action=push`)
- `push_token` (must match main router API token)
- `self_ip` (optional AP LAN IP; auto-detected if empty)
- AP list in `config ap` sections (`ip`, `enabled`)

The init service runs a lightweight loop that refreshes `/tmp/wifi-surveyor/latest.json`
every `sample_interval` seconds. The API serves cached data first (local and remote),
with on-demand fallback to a live collection if cache is missing.

With `push_enabled=1` on APs, each AP also POSTs the latest JSON to the main router.
The aggregator then serves AP data from push-cache immediately (no per-request SSH fanout).

Agent snapshots include:
- per-interface station stats (`mac`, `signal_dbm`, `inactive_ms`, `connected_s`, `rx_mbps`, `tx_mbps`)
- recent roam/auth events from `hostapd` logs (for post-analysis and txpower/channel tuning)

## Install SSH key on APs

On main router:

```sh
mkdir -p /etc/wifi-surveyor
ssh-keygen -t ed25519 -N '' -f /etc/wifi-surveyor/id_ed25519
pub="$(cat /etc/wifi-surveyor/id_ed25519.pub)"
for ip in 10.0.0.11 10.0.0.14 10.0.0.18 10.0.0.20 10.0.0.21; do
  ssh root@$ip "/usr/sbin/wifi-surveyor-authorize-key '$pub'"
done
```

## API usage

Local only:

```sh
curl -s -H 'X-Auth-Token: change-me-please' 'http://10.0.0.1/cgi-bin/wifi-surveyor-api?action=local'
```

Single AP:

```sh
curl -s -H 'X-Auth-Token: change-me-please' 'http://10.0.0.1/cgi-bin/wifi-surveyor-api?action=ap&ip=10.0.0.18'
```

All APs:

```sh
curl -s -H 'X-Auth-Token: change-me-please' 'http://10.0.0.1/cgi-bin/wifi-surveyor-api?action=all'
```

Roam lookup by connected BSSID:

```sh
curl -s -H 'X-Auth-Token: change-me-please' 'http://10.0.0.1/cgi-bin/wifi-surveyor-api?action=roam&bssid=aa:bb:cc:dd:ee:ff'
```

Local LAN throughput endpoint (token protected):

```sh
curl -s -H 'X-Auth-Token: change-me-please' 'http://10.0.0.1/cgi-bin/wifi-surveyor-throughput?bytes=3000000' > /dev/null
```

For higher-speed LAN testing, a static blob is also maintained at:

```sh
http://10.0.0.1/wifi-surveyor-blob.bin
```

Blob size is controlled by `throughput_blob_bytes` in `/etc/config/wifi-surveyor`.
