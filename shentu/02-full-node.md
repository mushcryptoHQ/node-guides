<img src="../assets/shentu-step-02.svg" width="100%" alt="Shentu step 02 — Full node">

[← 01 Requirements](./01-requirements-install.md) · **02 Full node** · [03 Validator →](./03-validator.md)

---

```bash
source $HOME/.shentu-env
```

## 1. Initialise & genesis

```bash
shentud init "$MONIKER" --chain-id $CHAIN_ID --home $DAEMON_HOME

curl -fL https://raw.githubusercontent.com/shentufoundation/mainnet/main/shentu-2.2/genesis.json \
  -o $DAEMON_HOME/config/genesis.json
jq -r .chain_id $DAEMON_HOME/config/genesis.json      # → shentu-2.2
```

## 2. Seeds, peers, gas

Seeds and peers come from the [cosmos chain-registry](https://github.com/cosmos/chain-registry/blob/master/shentu/chain.json):

```bash
CFG=$DAEMON_HOME/config/config.toml
APP=$DAEMON_HOME/config/app.toml
REG=https://raw.githubusercontent.com/cosmos/chain-registry/master/shentu/chain.json

SEEDS=$(curl -sSf $REG | jq -r '[.peers.seeds[] | "\(.id)@\(.address)"] | join(",")')
PEERS=$(curl -sSf $REG | jq -r '[.peers.persistent_peers[]? | "\(.id)@\(.address)"] | join(",")')
sed -i -e "s|^seeds *=.*|seeds = \"$SEEDS\"|" \
       -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $CFG

sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01uctk"|' $APP

EXT_IP=$(curl -s https://ifconfig.me)
sed -i -e "s|^external_address *=.*|external_address = \"$EXT_IP:26656\"|" $CFG
sed -i -e 's|^prometheus *=.*|prometheus = true|' $CFG
```

<details>
<summary><b>Validator-friendly pruning & indexer</b> (optional)</summary>

```bash
sed -i -e 's|^pruning *=.*|pruning = "custom"|' \
       -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
       -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP
sed -i -e 's|^indexer *=.*|indexer = "null"|' $CFG
```
</details>

## 3. Sync

The chain has been through many upgrades since genesis, so syncing from block 1 would mean replaying every historical binary. **Use state sync or a snapshot.**

<details open>
<summary><b>⚡ State sync</b></summary>
<br>

```bash
RPC="https://shentu-rpc.polkachu.com:443"
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

<details>
<summary><b>📦 Snapshot</b></summary>
<br>

Polkachu publishes a daily snapshot. The file name contains the block height, so copy the current link from [polkachu.com/tendermint_snapshots/shentu](https://polkachu.com/tendermint_snapshots/shentu):

```bash
SNAP_URL="<paste the current .tar.lz4 link>"
cd $HOME
curl -o - -L "$SNAP_URL" | lz4 -c -d - | tar -x -C $DAEMON_HOME
```
</details>

## 4. systemd

```bash
sudo tee /etc/systemd/system/shentud.service > /dev/null <<EOF
[Unit]
Description=Shentu node ($CHAIN_ID)
After=network-online.target
Wants=network-online.target

[Service]
User=$USER
ExecStart=$(which shentud) start --home $DAEMON_HOME
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now shentud
journalctl -u shentud -f -o cat
```

## 5. Synced?

```bash
curl -s localhost:26657/status | jq '.result.sync_info | {latest_block_height, catching_up}'
curl -s localhost:26657/abci_info | jq -r '.result.response.version'     # → v2.19.0
```

---

[← 01 Requirements](./01-requirements-install.md) · **02 Full node** · [03 Validator →](./03-validator.md)
