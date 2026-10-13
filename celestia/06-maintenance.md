<img src="../assets/step-06.svg" width="100%" alt="Step 06 — Maintenance">

[← 05 Fibre](./05-fibre.md) · **06 Maintenance** · [07 Cheat sheet →](./07-cheatsheet.md)

---

```bash
source $HOME/.celestia-env
```

## ⬆️ Upgrading the binary

Celestia ships **multiplexer** builds, so the same binary carries old and new app versions and switches at activation. A routine upgrade is a binary swap:

```bash
NEW=v10.4.0          # or v10.4.0-mocha, etc. Always read the release notes first
ARCH=$(uname -m); [ "$ARCH" = "aarch64" ] && ARCH=arm64
cd $HOME
curl -fLO "https://github.com/celestiaorg/celestia-app/releases/download/$NEW/celestia-app_Linux_${ARCH}.tar.gz"
curl -fLO "https://github.com/celestiaorg/celestia-app/releases/download/$NEW/checksums.txt"
sha256sum --ignore-missing --check checksums.txt && tar -xzf celestia-app_Linux_${ARCH}.tar.gz

sudo systemctl stop celestia-appd
sudo install -m 0755 celestia-appd /usr/local/bin/celestia-appd
sudo systemctl start celestia-appd
celestia-appd version

sed -i "s|^export APP_VERSION=.*|export APP_VERSION=$NEW|" $HOME/.celestia-env
```

Validators also replace `fibre` from the **same release** and restart `fibre.service`.

After a major release, review new config fields:

```bash
celestia-appd config sync --home $CELESTIA_HOME --fibre-home $FIBRE_HOME --dry-run
celestia-appd config sync --home $CELESTIA_HOME --fibre-home $FIBRE_HOME
```

`config sync` backs up the files first, keeps your existing values and **does not touch `app.toml`**.

## 🗳️ Signalling an app version upgrade

Since v3, app versions activate through **in-protocol signalling** (`x/signal`), not governance proposals. Once validators holding **5/6 of voting power** have signalled the same version and a `try-upgrade` succeeds, the upgrade is scheduled about **7 days later** (232,616 blocks on mainnet).

> [!IMPORTANT]
> **Upgrade your binary first, then signal.** Signalling says "my node is ready", so only send it when that's true.

```bash
# 1. signal readiness (example: v10)
celestia-appd tx signal signal 10 --from $WALLET --chain-id $CHAIN_ID \
  --keyring-backend $KEYRING_BACKEND $FEES -y

# 2. watch the tally and who is still missing
celestia-appd query signal tally 10
celestia-appd query signal missing-validators 10

# 3. once ≥ 5/6 have signalled, anyone can trigger scheduling
celestia-appd tx signal try-upgrade --from $WALLET --chain-id $CHAIN_ID \
  --keyring-backend $KEYRING_BACKEND $FEES -y

# 4. pending upgrade height
celestia-appd query signal upgrade

# 5. after activation
curl -s localhost:26657/abci_info | jq -r '.result.response.app_version'
```

| Network | App v10 |
|---|---|
| `mocha-5` | ✅ Activated at block **1082619** (2026-09-24 20:20 UTC) |
| `celestia` | ⏳ `v10.4.0` released 2026-10-07. Activation is pending, so watch `query signal upgrade` and the status page |

Follow [status.celestia.org](https://status.celestia.org) for announcements. Celestia also maintains an [upgrade monitor](https://docs.celestia.org/operate/consensus-validators/upgrade-monitor/).

## 📈 Monitoring

You enabled Prometheus in [step 03](./03-consensus-node.md#4-configuration), so metrics are served at `:26660/metrics`.

```bash
curl -s localhost:26660/metrics | grep -E "^cometbft_consensus_height |^cometbft_p2p_peers "
```

| Alert | Metric / check | Suggested trigger |
|---|---|---|
| Node stalled | `cometbft_consensus_height` not increasing | 2 min |
| Missing blocks | `cometbft_consensus_validator_missed_blocks` | any increase |
| Low peers | `cometbft_p2p_peers` | < 10 |
| Slow signing (Fibre) | `cometbft_privval_signing_latency_*` | median > 10 ms |
| Jailed | `q staking validator $VALOPER` → `jailed: true` | immediately |
| Fibre down | `systemctl is-active fibre`, `:7980` reachability | 1 min |
| Disk | free space on both disks | < 15% |
| Upgrade pending | `query signal upgrade` | new height appears |

> [!TIP]
> Don't want to babysit dashboards? That's literally what [MushCrypto](https://mushcrypto.world/) does: alerts in under 120 ms, and zero missed so far.

## 🚚 Migrating to a new server

> [!CAUTION]
> The safe order is **sync new → stop old → move key → start new**. Running both at once means double signing and tombstoning.

1. Set up and fully sync the new node *without* the validator key. Check `catching_up: false`.
2. Stop and disable the old one:
   ```bash
   sudo systemctl stop celestia-appd fibre
   sudo systemctl disable celestia-appd fibre
   ```
3. Copy `config/priv_validator_key.json` to the same path on the new server, plus `data/priv_validator_state.json`. Compare the two files with `sha256sum`.
4. Start the new validator, then Fibre. Run `valaddr set-host` again if the public address changed.

## ♻️ Resetting

| Command | What it does | Validator-safe? |
|---|---|---|
| `celestia-appd tendermint reset-state` | Clears chain data and keeps validator state and the address book | ✅ |
| Move `data/` aside, keep `priv_validator_state.json` | Manual reset (recommended for validators) | ✅ |
| `celestia-appd tendermint unsafe-reset-all --home $CELESTIA_HOME` | Wipes everything **including** validator state | ⚠️ Only if not signing, and back up keys first |

## 🔑 Backups

Keep these off the server, encrypted:

- The wallet mnemonic
- `$CELESTIA_HOME/config/priv_validator_key.json`
- `$CELESTIA_HOME/config/node_key.json` *(optional, keeps your peer ID)*
- Fibre TLS material, if you use a remote signer

## 🧹 Removing a node

```bash
sudo systemctl stop fibre celestia-appd
sudo systemctl disable fibre celestia-appd
sudo rm /etc/systemd/system/fibre.service /etc/systemd/system/celestia-appd.service
sudo rm -rf /etc/systemd/system/fibre.service.d
sudo systemctl daemon-reload
rm -rf $CELESTIA_HOME $FIBRE_HOME $HOME/.celestia-env
sudo rm /usr/local/bin/celestia-appd /usr/local/bin/fibre
```

> [!WARNING]
> If this is a validator, **unbond or move the key first**. Back up the key before running `rm -rf`.

---

[← 05 Fibre](./05-fibre.md) · **06 Maintenance** · [07 Cheat sheet →](./07-cheatsheet.md)
