Masternode Config
=================

Megacoin Core allows controlling multiple remote masternodes from a single wallet. The wallet needs to have a valid collateral output of 4200 MEC for each masternode and uses a configuration file named `masternode.conf` which can be found in the following data directory (depending on your operating system):
 * Windows: `%APPDATA%\Roaming\Megacoin\`
 * Mac OS: `~/Library/Application Support/Megacoin/`
 * Unix/Linux: `~/.megacoin/`

`masternode.conf` is a space separated text file. Each line consists of an alias, IP address followed by port (default Mainnet: 7951, Testnet: 19444), masternode private key, collateral output transaction id and collateral output index.

Example:
```
mn1 192.0.2.1:7951 93HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg 7603c20a05258c208b58b0a0d77603b9fc93d47cfa403035f87f3ce0af814566 0
mn2 192.0.2.2:7951 92Da1aYg6sbenP6uwskJgEY2XWB5LwJ7bXRqc3UPeShtHWJDjDv 5d898e78244f3206e0105f421cdb071d95d111a51cd88eb5511fc0dbf4bfd95f 1
```

In the example above:
* the collateral output of 4200 Megacoin for `mn1` is output `0` of transaction `7603c20a05258c208b58b0a0d77603b9fc93d47cfa403035f87f3ce0af814566`
* the collateral output of 4200 Megacoin for `mn2` is output `1` of transaction `5d898e78244f3206e0105f421cdb071d95d111a51cd88eb5511fc0dbf4bfd95f`

_Note: IPs in `127.0.0.*` or private ranges are for local testing only. Make sure you have real reachable public IP addresses in your production `masternode.conf`._

The following RPC commands are available in Megacoin Core (type `help masternode` in Console or via `megacoin-cli` for more info):
* `masternode list-conf`
* `masternode start-alias <alias>`
* `masternode start-all`
* `masternode start-missing`
* `masternode start-disabled`
* `masternode outputs`
* `masternode status`
