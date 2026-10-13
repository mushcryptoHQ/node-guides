<img src="../assets/axelar-step-01.svg" width="100%" alt="Axelar step 01 — Requirements & install">

[← Axelar hub](./README.md) · **01 Requirements & install** · [02 Full node →](./02-full-node.md)

---

## 🖥️ Hardware

| | Minimum | Recommended |
|---|---|---|
| CPU | 16 cores | 32 cores |
| RAM | 16 GB | 32 GB |
| Disk | 1.5 TB NVMe | 2 TB+ NVMe |
| Network | 1 Gbps | 1 Gbps |
| OS | Ubuntu 22.04 / 24.04 LTS (x86_64) | |

<sub>Source: <a href="https://docs.axelar.dev/validator/setup/overview/">Validator setup overview</a>. This covers <b>axelard + vald + tofnd</b> only. The EVM nodes you need for <a href="./04-external-chains.md">step 04</a> run on separate hardware.</sub>

## 📦 Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget jq unzip lz4 zstd aria2 git ufw
```

## 🔀 Network variables

```bash
cat > $HOME/.axelar-env <<'EOF'
export CHAIN_ID=axelar-dojo-1
export AXELARD_VERSION=v1.5.3
export TOFND_VERSION=v1.0.1
export MONIKER="your-moniker"
export AXELAR_HOME=$HOME/.axelar
export TOFND_HOME=$HOME/.tofnd
export GAS="--gas auto --gas-adjustment 1.4 --gas-prices 0.007uaxl"
EOF
echo 'source $HOME/.axelar-env' >> $HOME/.bashrc
source $HOME/.axelar-env
```

The keyring password goes into a **separate root-only file**, never into `.bashrc`:

```bash
sudo install -d -m 0700 /etc/axelar
sudo install -m 0600 /dev/null /etc/axelar/keyring.env
echo 'KEYRING_PASSWORD=<choose-a-strong-password>' | sudo tee /etc/axelar/keyring.env > /dev/null
```

## ⬇️ Install `axelard` (static) and `tofnd`

The **`-static`** build bundles libwasmvm v2.3.4. With the dynamic build you would have to install and pin the library yourself, and a stale libwasmvm is the classic cause of **app hash mismatches**.

```bash
cd $HOME
BASE=https://github.com/axelarnetwork/axelar-core/releases/download/$AXELARD_VERSION
F=axelard-linux-amd64-$AXELARD_VERSION-static
curl -fLO $BASE/$F && curl -fLO $BASE/$F.sha256
echo "$(cut -d' ' -f1 $F.sha256)  $F" | sha256sum -c -        # → OK
sudo install -m 0755 $F /usr/local/bin/axelard && rm -f $F $F.sha256

BASE=https://github.com/axelarnetwork/tofnd/releases/download/$TOFND_VERSION
F=tofnd-linux-amd64-$TOFND_VERSION
curl -fLO $BASE/$F && curl -fLO $BASE/$F.sha256
echo "$(cut -d' ' -f1 $F.sha256)  $F" | sha256sum -c -
sudo install -m 0755 $F /usr/local/bin/tofnd && rm -f $F $F.sha256

axelard version            # → 1.5.3
tofnd --help | head -3
```

## 🔌 Ports

| Port | Service | Exposure |
|---|---|---|
| `26656` | P2P | 🌍 public |
| `26657` | RPC, used by vald | 🔒 localhost |
| `9090` | gRPC | 🔒 localhost |
| `1317` | REST | 🔒 localhost |
| `50051` | tofnd gRPC | 🔒 **localhost only** |
| `26660` | Prometheus | 🔒 monitoring |

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
sudo ufw allow ssh && sudo ufw allow 26656/tcp
sudo ufw enable
```

---

[← Axelar hub](./README.md) · **01 Requirements & install** · [02 Full node →](./02-full-node.md)
