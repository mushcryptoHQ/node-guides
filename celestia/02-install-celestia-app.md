<img src="../assets/step-02.svg" width="100%" alt="Step 02 — Install celestia-app">

[← 01 Requirements](./01-requirements.md) · **02 Install** · [03 Consensus node →](./03-consensus-node.md)

---

`celestia-appd` ships as a **multiplexer**: one binary that embeds older app versions and switches to the new one automatically when an upgrade activates. The v10 multiplexer still embeds the older app versions (e.g. v9.0.8), so a node can replay history from before the v10 activation at height 15 116 658.

> [!WARNING]
> For the v9 → v10 transition, **do not** use the `celestia-app-standalone` build and **do not** use Cosmovisor. The multiplexer handles the switch itself.

```bash
source $HOME/.celestia-env
echo "Installing $APP_VERSION for $CHAIN_ID"
```

## Option A: Prebuilt binary (recommended)

<details open>
<summary><b>A1. Manual download with checksum verification</b></summary>
<br>

```bash
cd $HOME
ARCH=$(uname -m); [ "$ARCH" = "aarch64" ] && ARCH=arm64
PKG="celestia-app_Linux_${ARCH}.tar.gz"
BASE="https://github.com/celestiaorg/celestia-app/releases/download/$APP_VERSION"

curl -fLO "$BASE/$PKG"
curl -fLO "$BASE/checksums.txt"
sha256sum --ignore-missing --check checksums.txt     # → OK

tar -xzf "$PKG"
sudo install -m 0755 celestia-appd /usr/local/bin/celestia-appd
rm -f "$PKG" checksums.txt
```
</details>

<details>
<summary><b>A2. Official install script</b></summary>
<br>

The script downloads the release, verifies its checksum, and asks where to put the binary. Choose `/usr/local/bin`.

```bash
bash -c "$(curl -sL https://docs.celestia.org/celestia-app.sh)" -- -v $APP_VERSION
```
</details>

## Option B: Build from source

```bash
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app
git checkout tags/$APP_VERSION

make enable-bbr     # Linux hosts only; needs sudo
make check-bbr      # → BBR is enabled

make install        # builds the multiplexer into $HOME/go/bin
```

## ✅ Verify

```bash
which celestia-appd
celestia-appd version
```

The version should match `$APP_VERSION`. If `which` shows two different paths across reboots, delete the stale copy, because a leftover old binary is a classic cause of upgrade-day incidents.

> [!TIP]
> Validators also need the **Fibre** binary from the same release. You install it in [step 05](./05-fibre.md), but you can download it now so you're ready before activation.

---

[← 01 Requirements](./01-requirements.md) · **02 Install** · [03 Consensus node →](./03-consensus-node.md)
