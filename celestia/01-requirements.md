<img src="../assets/step-01.svg" width="100%" alt="Step 01 — Requirements">

[← Celestia hub](./README.md) · **01 Requirements** · [02 Install →](./02-install-celestia-app.md)

---

## 🖥️ Hardware

| Role | CPU | RAM | Disk | Network |
|---|---|---|---|---|
| **Validator** | 32 cores | 32 GB | 2 TB NVMe | 1 Gbps |
| Consensus node, pruned | 32 cores | 32 GB | 2 TB NVMe | 1 Gbps |
| Consensus node, archival | 32 cores | 64 GB | 16 TB NVMe | 1 Gbps |
| **Fibre server** *(validators, app v10+)* | shares the validator host | **+ ~27 GiB** for receive buffers at default limits | **a separate disk** from consensus data | public port `7980` |

<sub>Source: <a href="https://docs.celestia.org/operate/getting-started/hardware-requirements/">Hardware requirements</a> and the <a href="https://docs.celestia.org/operate/consensus-validators/fibre/">Fibre guide</a>. Disk figures are usable capacity after RAID and filesystem overhead.</sub>

> [!IMPORTANT]
> **Validators must pass the CPU benchmark.** The recommended CPU features are **GFNI** and **SHA-NI**. Check them before you pay for a server:
> ```bash
> grep -o -w -E 'gfni|sha_ni' /proc/cpuinfo | sort -u
> ```
> You want both `gfni` and `sha_ni` in the output.

> [!NOTE]
> **RAM for Fibre.** The worst-case receive buffers are `max_connections × max_concurrent_streams × 132 MiB`. With the defaults (16 × 13) that comes to about **27 GiB**, *on top of* what celestia-app and the OS use. A validator host running Fibre therefore needs well over the 32 GB baseline. Plan for 64 GB or more.

## 🐧 Operating system

- **Ubuntu 24.04 LTS** (x86_64) is the OS tested in CI.
- celestia-app multiplexer builds (v9+) require **glibc ≥ 2.38**, so Ubuntu 22.04 and older will **not** work. The Fibre binary needs glibc ≥ 2.34.

```bash
ldd --version | head -1
```

## 📦 Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget tar aria2 lz4 jq git make clang pkg-config \
  libssl-dev build-essential ufw ncdu
```

<details>
<summary><b>Go toolchain</b> (only needed to build from source)</summary>
<br>

celestia-app `v10.4.0` builds with **Go 1.26.6**. Check the release notes when you upgrade.

```bash
GO_VERSION=1.26.6
cd $HOME
wget "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz"
rm "go${GO_VERSION}.linux-amd64.tar.gz"
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc
source $HOME/.bashrc
go version
```
</details>

## 🚀 BBR congestion control

Celestia recommends BBR on Linux hosts. If you build from source, `make enable-bbr && make check-bbr` does this for you (see [step 02](./02-install-celestia-app.md)). If you install the prebuilt binary, set it manually:

```bash
sudo modprobe tcp_bbr
echo tcp_bbr | sudo tee -a /etc/modules
printf "net.core.default_qdisc=fq\nnet.ipv4.tcp_congestion_control=bbr\n" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
sysctl net.ipv4.tcp_congestion_control   # → bbr
```

> [!NOTE]
> Run this on the **host**, not inside a container. It changes network settings for the whole host.

## 💽 Disks

Put Fibre on its own disk, separate from the consensus data. For example:

```
/home/<user>/.celestia-app     → NVMe #1  (consensus data)
/mnt/fibre/.celestia-fibre     → NVMe #2  (Fibre shards)
```

## 🔌 Ports

| Port | Service | Exposure |
|---|---|---|
| `26656` | CometBFT P2P | 🌍 **public** |
| `7980` | Fibre client port *(app v10+, validators)* | 🌍 **public** |
| `26657` | CometBFT RPC | 🔒 localhost (or a trusted network) |
| `9090` | Application gRPC, used by Fibre and bridge nodes | 🔒 localhost / private, **plaintext** |
| `26669` | Priv-validator gRPC, used by Fibre for signing | 🔒 **localhost only**, unless you set up mutual TLS |
| `9098` | CometBFT gRPC (`[rpc] grpc_laddr`), used by bridge nodes | 🔒 private |
| `26660` | Prometheus metrics | 🔒 monitoring network |

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 26656/tcp
sudo ufw allow 7980/tcp        # validators running Fibre
sudo ufw enable
```

> [!CAUTION]
> **Never expose `26669`** (the signer) to the internet without mutual TLS. Anyone who can reach it can request signatures from your validator key.

## 🔀 Pick your network

Every later step reads these variables. Create the file **once**, choosing **one** of the two blocks.

<table>
<tr><th>Mainnet Beta</th><th>Mocha testnet</th></tr>
<tr valign="top"><td>

```bash
cat > $HOME/.celestia-env <<'EOF'
export CHAIN_ID=celestia
export APP_VERSION=v10.4.0
export KEYRING_BACKEND=file
export MONIKER="your-moniker"
export WALLET=validator
export CELESTIA_HOME=$HOME/.celestia-app
export FIBRE_HOME=/mnt/fibre/.celestia-fibre
export FEES="--fees 21000utia --gas 220000"
EOF
```

</td><td>

```bash
cat > $HOME/.celestia-env <<'EOF'
export CHAIN_ID=mocha-5
export APP_VERSION=v10.4.0-mocha
export KEYRING_BACKEND=test
export MONIKER="your-moniker"
export WALLET=validator
export CELESTIA_HOME=$HOME/.celestia-app
export FIBRE_HOME=/mnt/fibre/.celestia-fibre
export FEES="--fees 21000utia --gas 220000"
EOF
```

</td></tr>
</table>

```bash
echo 'source $HOME/.celestia-env' >> $HOME/.bashrc
source $HOME/.celestia-env
echo "$CHAIN_ID / $APP_VERSION"
```

> [!WARNING]
> Do **not** use `KEYRING_BACKEND=test` on mainnet. It stores keys unencrypted on disk.

---

[← Celestia hub](./README.md) · **01 Requirements** · [02 Install →](./02-install-celestia-app.md)
