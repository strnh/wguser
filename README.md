# wguser
For booking/manage wireguard road-warrior endpoint.

# Prerequisite
 - Python3
 - wireguard-tools (`wg`)
 - ip-pool https://pypi.org/project/ip-pool/
<pre>
$ pip install ip-pool
</pre>

# Usage:

  - wguser -a [username]  : add user (client config is written to `./[username].conf`)
  - wguser -r [username]  : remove user
  - wguser -l             : show assigned
  - wguser -s, --status   : show connected accounts and their remote IP on wg0 (needs root)
  - wguser -p, --peers    : all peers of wg0 as TSV (needs root)
  - wguser -m, --metrics  : statistics in Prometheus text format (needs root)

# Configuration

Settings are read from a config file in shell syntax (see `wguser.conf.example`),
searched in this order:

 1. `wguser -c FILE` / `--config FILE`
 2. `WGUSER_CONF` environment variable
 3. `wguser.conf` in the directory of the script

Without a config file the defaults are used: interface `wg0`, data files
(`vpn.json`, `servertmpl`, `wg0.conf`) in the directory of the script,
client configs written to the current directory.

| Setting | Default | |
|---|---|---|
| `INTERFACE` | `wg0` | WireGuard interface |
| `DATADIR` | script directory | base directory of the data files |
| `DBFILE` | `$DATADIR/vpn.json` | ip-pool database |
| `SERVERINTCONF` | `$DATADIR/$INTERFACE.conf` | server config |
| `SERVERTMPL` | `$DATADIR/servertmpl` | `[Peer]` section for client configs |
| `CLIENTDIR` | `.` | output directory of client configs |
| `HANDSHAKE_TIMEOUT` | `180` | seconds; newer handshake = connected |

The config file is sourced as a shell script, so keep it owned by root.

Changes to `wg0.conf` are not applied to the running interface; reload it yourself, e.g.
<pre>
# wg syncconf wg0 <(wg-quick strip wg0)
</pre>
Usernames may contain only `A-Z a-z 0-9 _ -`.

# Statistics

A peer is counted as connected if its latest handshake is within `HANDSHAKE_TIMEOUT` (180s).

## Prometheus (node_exporter textfile collector)
<pre>
# /etc/cron.d/wguser
* * * * * root /path/to/wguser -m > /var/lib/node_exporter/textfile/wguser.prom.tmp && mv /var/lib/node_exporter/textfile/wguser.prom.tmp /var/lib/node_exporter/textfile/wguser.prom
</pre>
Metrics (labels: `interface`, `user`, `public_key`):
`wguser_interface_up`, `wguser_connected_peers`,
`wguser_peer_receive_bytes_total`, `wguser_peer_transmit_bytes_total`,
`wguser_peer_latest_handshake_timestamp_seconds`, `wguser_peer_connected`.

## Munin
<pre>
# ln -s /path/to/munin/wguser /etc/munin/plugins/wguser
# cat /etc/munin/plugin-conf.d/wguser
[wguser]
user root
env.WGUSER /path/to/wguser
env.WGUSER_CONF /path/to/wguser.conf
</pre>
Graphs: traffic per user (`wguser_traffic`), connected users (`wguser_peers`).
