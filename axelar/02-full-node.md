<img src="../assets/axelar-step-02.svg" width="100%" alt="Axelar step 02 — Full node">

[← 01 Requirements](./01-requirements-install.md) · **02 Full node** · [03 Validator →](./03-validator.md)

---

```bash
source $HOME/.axelar-env
```

## 1. Initialise with Axelar's configs

Axelar maintains reference `config.toml` and `app.toml` files, and `config.toml` already contains the **vald** sections (`[broadcast]` and `[[axelar_bridge_evm]]`):

```bash
axelard init "$MONIKER" --chain-id $CHAIN_ID --home $AXELAR_HOME

REPO=https://raw.githubusercontent.com/axelarnetwork/axelarate-community/main
curl -fL $REPO/configuration/config.toml   -o $AXELAR_HOME/config/config.toml
curl -fL $REPO/configuration/app.toml      -o $AXELAR_HOME/config/app.toml
curl -fL $REPO/resources/mainnet/seeds.toml -o $AXELAR_HOME/config/seeds.toml
curl -fL https://axelar-mainnet.s3.us-east-2.amazonaws.com/genesis.json -o $AXELAR_HOME/config/genesis.json
jq -r .chain_id $AXELAR_HOME/config/genesis.json     # → axelar-dojo-1
```

> [!IMPORTANT]
> **Remove the Moonbeam block.** The v1.5.3 upgrade notes require deleting the whole `[[axelar_bridge_evm]]` entry where `name = "Moonbeam"`, and the reference template still contains it:
> ```bash
> python3 - "$AXELAR_HOME/config/config.toml" <<'PY'
> import re, sys
> p = sys.argv[1]; s = open(p).read()
> s = re.sub(r'\[\[axelar_bridge_evm\]\]\s*\nname = "Moonbeam".*?(?=\n\[\[|\n#####|\n\[)', '', s, flags=re.S)
> open(p, 'w').write(s)
> PY
> grep -n Moonbeam $AXELAR_HOME/config/config.toml || echo "Moonbeam removed"
> ```

```bash
CFG=$AXELAR_HOME/config/config.toml
EXT_IP=$(curl -s https://ifconfig.me)
sed -i -e "s|^external_address *=.*|external_address = \"$EXT_IP:26656\"|" $CFG
sed -i -e 's|^prometheus *=.*|prometheus = true|' $CFG
```

Check that the values from the v1.5.3 notes are in place: `iavl-cache-size = 0` and `iavl-disable-fastnode = true` in `app.toml`, and `timeout_commit = "1s"` in `config.toml`.

```bash
grep -E "^iavl-(cache-size|disable-fastnode)" $AXELAR_HOME/config/app.toml
grep -E "^timeout_commit" $CFG
```

## 2. Sync

Mainnet has been through many upgrades since genesis, so **don't sync from block 1**.

<details open>
<summary><b>⚡ State sync</b> (Polkachu)</summary>
<br>

```bash
RPC="https://axelar-rpc.polkachu.com:443"
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

Always state-sync with the **current** binary, or you'll hit AppHash errors.
</details>

<details>
<summary><b>📦 Snapshot</b></summary>
<br>

Pick a current snapshot from [Polkachu](https://polkachu.com/tendermint_snapshots/axelar) or one of the providers listed in the [Axelar docs](https://docs.axelar.dev/validator/setup/manual/):

```bash
SNAP_URL="<paste the current snapshot link>"
axelard comet unsafe-reset-all --home $AXELAR_HOME --keep-addr-book
curl -o - -L "$SNAP_URL" | lz4 -c -d - | tar -x -C $AXELAR_HOME
```
</details>

## 3. systemd: axelard

```bash
sudo tee /etc/systemd/system/axelard.service > /dev/null <<EOF
[Unit]
Description=axelard ($CHAIN_ID)
After=network-online.target
Wants=network-online.target

[Service]
User=$USER
ExecStart=/usr/local/bin/axelard start --home $AXELAR_HOME
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now axelard
journalctl -u axelard -f -o cat
```

## 4. Synced?

```bash
curl -s localhost:26657/status | jq '.result.sync_info | {latest_block_height, catching_up}'
curl -s localhost:26657/abci_info | jq -r '.result.response.version'     # → 1.5.3
```

Wait for `catching_up: false` before starting tofnd and vald.

---

[← 01 Requirements](./01-requirements-install.md) · **02 Full node** · [03 Validator →](./03-validator.md)
