<img src="../assets/akash-step-01.svg" width="100%" alt="Akash step 01 — Requirements & install">

[← Akash hub](./README.md) · **01 Requirements & install** · [02 Full node →](./02-full-node.md)

---

## 🖥️ Hardware

| | Minimum | Recommended |
|---|---|---|
| CPU | 8 cores | 16 cores |
| RAM | 128 GB *(the validator figure from the Akash docs)* | 128 GB |
| Disk | 512 GB NVMe | 1 TB+ NVMe |
| Network | 100 Mbps | 1 Gbps |
| OS | Ubuntu 24.04 LTS | |

<sub>Source: <a href="https://akash.network/docs/node-operators/validators/running-a-validator/">Running a Validator</a>. A plain full node or RPC node can run on much less (the CLI build guide lists 4 cores / 8 GB / 100 GB as its minimum).</sub>

## 📦 Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget jq unzip lz4 aria2 git make build-essential ufw
```

## 🔀 Network variables

```bash
cat > $HOME/.akash-env <<'EOF'
export AKASH_META_URL=https://raw.githubusercontent.com/akash-network/net/main/mainnet/meta.json
export CHAIN_ID=akashnet-2
export MONIKER="your-moniker"
export WALLET=validator
export KEYRING_BACKEND=file
export DAEMON_HOME=$HOME/.akash
export GAS="--gas auto --gas-adjustment 1.5 --gas-prices 0.025uakt"
EOF
echo 'source $HOME/.akash-env' >> $HOME/.bashrc
source $HOME/.akash-env
```

## ⬇️ Install `akash`

The binary URL and its SHA-256 come straight from the official `meta.json`, so this block always installs the **recommended** version:

```bash
cd $HOME
BIN=$(curl -sSf $AKASH_META_URL | jq -r '.codebase.binaries["linux/amd64"]')
URL=${BIN%%\?*}; SUM=${BIN##*sha256:}
echo "$URL"

curl -fLo akash.zip "$URL"
echo "$SUM  akash.zip" | sha256sum -c -        # → akash.zip: OK
unzip -o akash.zip akash
sudo install -m 0755 akash /usr/local/bin/akash
rm -f akash.zip akash

akash version                                 # → v2.1.0
```

<details>
<summary>arm64 hosts</summary>

Use `.codebase.binaries["linux/arm64"]` instead of `linux/amd64`.
</details>

## 🔌 Ports

| Port | Service | Exposure |
|---|---|---|
| `26656` | P2P | 🌍 public |
| `26657` | RPC | 🔒 localhost |
| `9090` | gRPC | 🔒 localhost |
| `1317` | REST API | 🔒 localhost |
| `26660` | Prometheus | 🔒 monitoring |

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
sudo ufw allow ssh && sudo ufw allow 26656/tcp
sudo ufw enable
```

---

[← Akash hub](./README.md) · **01 Requirements & install** · [02 Full node →](./02-full-node.md)
