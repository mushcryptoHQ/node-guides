<img src="../assets/akash-step-04.svg" width="100%" alt="Akash step 04 — Maintenance">

[← 03 Validator](./03-validator.md) · **04 Maintenance** · [05 Cheat sheet →](./05-cheatsheet.md)

---

```bash
source $HOME/.akash-env
```

## ⬆️ Chain upgrades

Akash upgrades through **governance software-upgrade proposals**. At the approved height, the node halts with `UPGRADE "<name>" NEEDED`, you swap the binary, and the node continues.

```bash
# watch for proposals and the planned height
akash q gov proposals --proposal-status voting-period
akash q upgrade plan
```

**Swap at the halt height.** Once `meta.json` points to the new version, the "Install `akash`" block from [step 01](./01-requirements-install.md) fetches it:

```bash
sudo systemctl stop akash
# re-run the "Install akash" block from step 01, then:
akash version
sudo systemctl start akash
journalctl -u akash -f -o cat
```

> [!TIP]
> To stop at an exact height for a planned swap, add `--halt-height <height>` to `ExecStart`. Remove it after the upgrade.

<details>
<summary><b>Cosmovisor</b> (automatic swaps)</summary>
<br>

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest   # needs Go
export DAEMON_NAME=akash
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
cp $(which akash) $DAEMON_HOME/cosmovisor/genesis/bin/
# for each upgrade: put the new binary in $DAEMON_HOME/cosmovisor/upgrades/<plan-name>/bin/akash
```

Then run `cosmovisor run start` in systemd, with `DAEMON_NAME`, `DAEMON_HOME` and `DAEMON_RESTART_AFTER_UPGRADE=true`.
</details>

## 📈 Monitoring

Prometheus is on `:26660` (enabled in step 02).

| Alert | Check |
|---|---|
| Height stalled | `cometbft_consensus_height` flat for 2 min |
| Missed blocks | `akash q slashing signing-info …` → `missed_blocks_counter` rising |
| Peers | `cometbft_p2p_peers` < 10 |
| Jailed | `akash q staking validator $VALOPER` → `jailed: true` |
| Upgrade | `akash q upgrade plan` returns a plan |
| Disk | < 15% free |

## 🚚 Migration

1. Sync the new server **without** the key. Wait for `catching_up: false`.
2. `sudo systemctl stop akash && sudo systemctl disable akash` on the **old** server.
3. Copy `config/priv_validator_key.json` and `data/priv_validator_state.json` to the new server.
4. Start the new node.

## ♻️ Reset

```bash
sudo systemctl stop akash
cp $DAEMON_HOME/data/priv_validator_state.json $HOME/pvs.backup.json
akash comet unsafe-reset-all --home $DAEMON_HOME --keep-addr-book
rm -rf $DAEMON_HOME/wasm
# restore a snapshot (step 02), then:
cp $HOME/pvs.backup.json $DAEMON_HOME/data/priv_validator_state.json
sudo systemctl start akash
```

## 🧹 Remove

```bash
sudo systemctl stop akash && sudo systemctl disable akash
sudo rm /etc/systemd/system/akash.service && sudo systemctl daemon-reload
rm -rf $DAEMON_HOME $HOME/.akash-env
sudo rm /usr/local/bin/akash
```

> [!WARNING]
> If this is a validator, unbond or move the key **before** you delete anything.

---

[← 03 Validator](./03-validator.md) · **04 Maintenance** · [05 Cheat sheet →](./05-cheatsheet.md)
