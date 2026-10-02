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

`vpn.json`, `servertmpl` and `wg0.conf` are read from the directory of the script
(override with `WGUSER_DIR=/path/to/dir`).
Changes to `wg0.conf` are not applied to the running interface; reload it yourself, e.g.
<pre>
# wg syncconf wg0 <(wg-quick strip wg0)
</pre>
Usernames may contain only `A-Z a-z 0-9 _ -`.
