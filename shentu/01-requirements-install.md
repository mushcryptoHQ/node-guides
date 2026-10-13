<img src="../assets/shentu-step-01.svg" width="100%" alt="Shentu step 01 — Requirements & install">

[← Shentu hub](./README.md) · **01 Requirements & install** · [02 Full node →](./02-full-node.md)

---

## 🖥️ Hardware

Shentu doesn't publish official specs. This is the baseline we run, which is typical for a Cosmos SDK 0.53 chain of this size:

| | Recommended |
|---|---|
| CPU | 8 cores |
| RAM | 32 GB |
| Disk | 1 TB NVMe |
| Network | 1 Gbps |
| OS | Ubuntu 22.04 / 24.04 LTS (x86_64) |

> [!NOTE]
> Releases ship a **linux/amd64** binary only. On arm64, build from source.

## 📦 Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget jq lz4 git make build-essential ufw
```

## 🔀 Network variables

```bash
cat > $HOME/.shentu-env <<'EOF'
export CHAIN_ID=shentu-2.2
export SHENTU_VERSION=v2.19.0
export MONIKER="your-moniker"
export WALLET=validator
export KEYRING_BACKEND=file
export DAEMON_HOME=$HOME/.shentud
export GAS="--gas auto --gas-adjustment 1.5 --gas-prices 0.025uctk"
EOF
echo 'source $HOME/.shentu-env' >> $HOME/.bashrc
source $HOME/.shentu-env
```

## ⬇️ Install `shentud`

<details open>
<summary><b>A. Release binary</b> (linux/amd64)</summary>
<br>

```bash
cd $HOME
V=${SHENTU_VERSION#v}
curl -fLo shentud "https://github.com/shentufoundation/shentu/releases/download/$SHENTU_VERSION/shentud_${V}_linux_amd64"
sudo install -m 0755 shentud /usr/local/bin/shentud && rm shentud
shentud version          # → 2.19.0 (or v2.19.0)
```

The releases page publishes no checksum file. If you need a verifiable build, use option B.
</details>

<details>
<summary><b>B. Build from source</b> (Go 1.23)</summary>
<br>

```bash
GO_VERSION=1.23.12
wget "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go${GO_VERSION}.linux-amd64.tar.gz
rm go${GO_VERSION}.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && source $HOME/.bashrc

cd $HOME && rm -rf shentu
git clone https://github.com/shentufoundation/shentu.git && cd shentu
git checkout $SHENTU_VERSION
make install
sudo install -m 0755 $HOME/go/bin/shentud /usr/local/bin/shentud
shentud version --long | grep -E "version|commit|go"
```
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

[← Shentu hub](./README.md) · **01 Requirements & install** · [02 Full node →](./02-full-node.md)
