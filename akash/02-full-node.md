<img src="../assets/akash-step-02.svg" width="100%" alt="Akash step 02 — Full node">

[← 01 Requirements](./01-requirements-install.md) · **02 Full node** · [03 Validator →](./03-validator.md)

---

```bash
source $HOME/.akash-env
```

## 1. Initialise & genesis

```bash
akash genesis init "$MONIKER" --chain-id $CHAIN_ID --home $DAEMON_HOME

curl -sSf "$(curl -sSf $AKASH_META_URL | jq -r '.codebase.genesis.genesis_url')" \
  > $DAEMON_HOME/config/genesis.json
jq -r .chain_id $DAEMON_HOME/config/genesis.json     # → akashnet-2
```

## 2. Seeds, peers, gas

```bash
CFG=$DAEMON_HOME/config/config.toml
APP=$DAEMON_HOME/config/app.toml

SEEDS=$(curl -sSf $AKASH_META_URL | jq -r '(.peers.seeds // []) | map("\(.id)@\(.address)") | join(",")')
PEERS=$(curl -sSf $AKASH_META_URL | jq -r '(.peers.persistent_peers // []) | map("\(.id)@\(.address)") | join(",")')
sed -i -e "s|^seeds *=.*|seeds = \"$SEEDS\"|" \
       -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $CFG

sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0025uakt"|' $APP

EXT_IP=$(curl -s https://ifconfig.me)
sed -i -e "s|^external_address *=.*|external_address = \"$EXT_IP:26656\"|" $CFG
sed -i -e 's|^prometheus *=.*|prometheus = true|' $CFG
```

<details>
<summary><b>Validator-friendly pruning & indexer</b> (optional, saves disk)</summary>

```bash
sed -i -e 's|^pruning *=.*|pruning = "custom"|' \
       -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
       -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP
sed -i -e 's|^indexer *=.*|indexer = "null"|' $CFG
```
</details>

## 3. Sync

<details open>
<summary><b>📦 Official snapshot</b> (hourly, ~15 GB)</summary>
<br>

```bash
cd $DAEMON_HOME
rm -rf data wasm && mkdir -p data
wget -O akashnet-2_latest.tar.lz4 https://snapshots.akash.network/akashnet-2/latest --inet4-only
lz4 -d akashnet-2_latest.tar.lz4 - | tar -xf - -C $DAEMON_HOME
rm akashnet-2_latest.tar.lz4
ls $DAEMON_HOME/data | head
```

Check that the archive unpacked `data/` (and `wasm/`, if present) **directly under `$DAEMON_HOME`**. Adjust the `-C` path if the archive layout differs.
</details>

<details>
<summary><b>⚡ State sync</b></summary>
<br>

```bash
RPC="https://akash-rpc.polkachu.com:443"
LATEST=$(curl -s $RPC/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((LATEST - 2000))
TRUST_HASH=$(curl -s "$RPC/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)
sed -i -E "/^\[statesync\]/,/^\[/ {
  s|^enable *=.*|enable = true|
  s|^rpc_servers *=.*|rpc_servers = \"$RPC,$RPC\"|
  s|^trust_height *=.*|trust_height = $TRUST_HEIGHT|
  s|^trust_hash *=.*|trust_hash = \"$TRUST_HASH\"|
}" $CFG
```
</details>

> [!WARNING]
> Restoring a snapshot onto an **existing validator**? Back up `data/priv_validator_state.json` first and put it back before you start the node. `unsafe-reset-all` does **not** clear `~/.akash/wasm`, so delete that folder by hand.

## 4. systemd

```bash
sudo tee /etc/systemd/system/akash.service > /dev/null <<EOF
[Unit]
Description=Akash node ($CHAIN_ID)
After=network-online.target
Wants=network-online.target

[Service]
User=$USER
ExecStart=$(which akash) start --home $DAEMON_HOME
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now akash
journalctl -u akash -f -o cat
```

## 5. Synced?

```bash
curl -s localhost:26657/status | jq '.result.sync_info | {latest_block_height, catching_up}'
curl -s localhost:26657/net_info | jq '.result.n_peers'
```

When you see `catching_up: false`, continue to the validator step.

---

[← 01 Requirements](./01-requirements-install.md) · **02 Full node** · [03 Validator →](./03-validator.md)
