<img src="../assets/shentu-step-04.svg" width="100%" alt="Shentu step 04 — Maintenance">

[← 03 Validator](./03-validator.md) · **04 Maintenance** · [05 Cheat sheet →](./05-cheatsheet.md)

---

```bash
source $HOME/.shentu-env
```

## ⬆️ Chain upgrades

Shentu ships **mandatory upgrades** through governance. Each [release](https://github.com/shentufoundation/shentu/releases) names the halt height. Recent ones:

| Version | Height | When |
|---|---|---|
| **v2.20.0** | 31 812 210 | 2026-10-29 |
| v2.19.0 | 31 150 500 | ~2026-09-15 |
| v2.18.0 | 29 367 500 | ~2026-05-19 |
| v2.17.0 | 28 124 200 | ~2026-02-25 |

Patch releases like `v2.16.1` and `v2.16.2` (CometBFT security fixes) are **not state-breaking**. Apply them with a rolling restart, without waiting for a height.

```bash
shentud q gov proposals --proposal-status voting-period
shentud q upgrade plan
```

**At the halt height** the node logs `UPGRADE "<name>" NEEDED` and stops. Then:

```bash
sudo systemctl stop shentud
sed -i "s|^export SHENTU_VERSION=.*|export SHENTU_VERSION=<new-version>|" $HOME/.shentu-env
source $HOME/.shentu-env
# re-run the install block from step 01 (binary or source)
shentud version
sudo systemctl start shentud
journalctl -u shentud -f -o cat
```

> [!WARNING]
> Never swap a state-breaking binary **before** the halt height, because your node will fork off. Use `--halt-height <height>` in `ExecStart` if you want a clean planned stop.

## 📈 Monitoring

| Alert | Check |
|---|---|
| Height stalled | `cometbft_consensus_height` flat for 2 min (`:26660`) |
| Missed blocks | `missed_blocks_counter` in `q slashing signing-info` |
| Peers | `cometbft_p2p_peers` < 10 |
| Jailed | `q staking validator $VALOPER` → `jailed: true` |
| Upgrade | `q upgrade plan` returns a plan |
| Disk | < 15% free |

## 🚚 Migration

1. Sync the new server **without** the key.
2. Stop and disable `shentud` on the **old** server.
3. Copy `config/priv_validator_key.json` and `data/priv_validator_state.json`.
4. Start the new node.

## ♻️ Reset

```bash
sudo systemctl stop shentud
cp $DAEMON_HOME/data/priv_validator_state.json $HOME/pvs.backup.json
shentud comet unsafe-reset-all --home $DAEMON_HOME --keep-addr-book
# state sync or snapshot (step 02), then:
cp $HOME/pvs.backup.json $DAEMON_HOME/data/priv_validator_state.json
sudo systemctl start shentud
```

## 🧹 Remove

```bash
sudo systemctl stop shentud && sudo systemctl disable shentud
sudo rm /etc/systemd/system/shentud.service && sudo systemctl daemon-reload
rm -rf $DAEMON_HOME $HOME/.shentu-env
sudo rm /usr/local/bin/shentud
```

---

[← 03 Validator](./03-validator.md) · **04 Maintenance** · [05 Cheat sheet →](./05-cheatsheet.md)
