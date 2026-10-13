<img src="../assets/step-04.svg" width="100%" alt="Step 04 — Validator">

[← 03 Consensus node](./03-consensus-node.md) · **04 Validator** · [05 Fibre →](./05-fibre.md)

---

> [!CAUTION]
> **Double signing** gets you slashed (2% of stake) and **permanently tombstoned**. It almost always happens the same way: the same `priv_validator_key.json` runs on two machines at once. Never start a second copy "just to test".

```bash
source $HOME/.celestia-env
```

## 1. Wallet

```bash
# new wallet: write the mnemonic down offline
celestia-appd keys add $WALLET --keyring-backend $KEYRING_BACKEND

# or restore an existing one
celestia-appd keys add $WALLET --recover --keyring-backend $KEYRING_BACKEND
```

```bash
WALLET_ADDR=$(celestia-appd keys show $WALLET -a --keyring-backend $KEYRING_BACKEND)
VALOPER=$(celestia-appd keys show $WALLET --bech val -a --keyring-backend $KEYRING_BACKEND)
echo "export WALLET_ADDR=$WALLET_ADDR" >> $HOME/.celestia-env
echo "export VALOPER=$VALOPER"         >> $HOME/.celestia-env
```

## 2. Fund it

| Network | How |
|---|---|
| `mocha-5` | [mocha.celenium.io/faucet](https://mocha.celenium.io/faucet), or Discord `#mocha-faucet` with `$request <address>` (10 TIA per week) |
| `celestia` | Transfer real TIA. Keep a buffer for fees. |

```bash
celestia-appd q bank balances $WALLET_ADDR
```

## 3. Back up the consensus key

```bash
cp $CELESTIA_HOME/config/priv_validator_key.json $HOME/priv_validator_key.backup.json
chmod 600 $HOME/priv_validator_key.backup.json
```

Move that file **off the server** to an encrypted location. Together with the mnemonic, it *is* your validator.

## 4. Create the validator

Check `catching_up: false` first (see [step 03](./03-consensus-node.md#7-is-it-synced)).

Since celestia-app moved to the Cosmos SDK 0.50+ CLI, `create-validator` takes a **JSON file** instead of flags (`create-validator [path/to/validator.json]`).

**4.1 Write `validator.json`.** Replace the `<…>` placeholders with your own values, or delete the optional lines.

```bash
cat > $HOME/validator.json <<EOF
{
  "pubkey": $(celestia-appd comet show-validator --home $CELESTIA_HOME),
  "amount": "1000000utia",
  "moniker": "$MONIKER",
  "identity": "<keybase-id>",
  "website": "<https://your-website>",
  "security": "<security-contact-email>",
  "details": "<short description>",
  "commission-rate": "0.10",
  "commission-max-rate": "0.20",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
EOF
```

| Field | Meaning |
|---|---|
| `pubkey` | Consensus public key. It is filled in from `comet show-validator` |
| `amount` | Initial self-delegation, in `utia` (1 TIA = 1 000 000 utia) |
| `identity` / `website` / `security` / `details` | Optional. Leave them empty (`""`) if you don't need them |
| `commission-*` | Rates as decimals: `0.10` = 10% |
| `min-self-delegation` | In utia. If your self-delegation drops below it, the validator is jailed |

**4.2 Check the file.** It must be valid JSON, and `pubkey` must contain the `@type` and `key` fields:

```bash
jq . $HOME/validator.json
```

**4.3 Send the transaction:**

```bash
celestia-appd tx staking create-validator $HOME/validator.json \
  --from $WALLET --chain-id $CHAIN_ID --keyring-backend $KEYRING_BACKEND \
  $FEES -y
```

A successful response contains `code: 0` and a `txhash`.

> [!NOTE]
> `commission-max-rate` and `commission-max-change-rate` **can never be changed** after creation. Choose them carefully.

## 5. Verify

```bash
celestia-appd q staking validator $VALOPER | grep -E "moniker|status|tokens|jailed"
curl -s localhost:26657/status | jq '.result.validator_info'
```

`status: BOND_STATUS_BONDED` means you are in the active set (max **100** validators). Check your validator on an explorer too: [Celenium](https://celenium.io) or [Mintscan](https://mintscan.io/celestia), and their testnet equivalents.

## 6. Stay out of jail

| Rule | Value |
|---|---|
| Downtime | Jailed when you miss **more than 25%** of the signing window |
| Downtime jail | 1 minute, then `unjail` |
| Double sign | **2% slash + tombstone** (permanent) |

```bash
celestia-appd q slashing params
celestia-appd q slashing signing-info $(celestia-appd comet show-validator --home $CELESTIA_HOME)
```

```bash
# unjail after the jail period
celestia-appd tx slashing unjail --from $WALLET --chain-id $CHAIN_ID \
  --keyring-backend $KEYRING_BACKEND $FEES -y
```

## 7. Edit, delegate, withdraw

```bash
# edit (keep only the flags you are changing)
celestia-appd tx staking edit-validator --new-moniker="<new-moniker>" --website="<https://your-website>" \
  --from $WALLET --chain-id $CHAIN_ID --keyring-backend $KEYRING_BACKEND $FEES -y

# self-delegate more
celestia-appd tx staking delegate $VALOPER 1000000utia \
  --from $WALLET --chain-id $CHAIN_ID --keyring-backend $KEYRING_BACKEND $FEES -y

# withdraw rewards and commission
celestia-appd tx distribution withdraw-rewards $VALOPER --commission \
  --from $WALLET --chain-id $CHAIN_ID --keyring-backend $KEYRING_BACKEND $FEES -y
```

> [!IMPORTANT]
> From **app version 10**, every bonded validator is expected to run a **Fibre server**. On Mocha that is already active, so continue straight to [step 05](./05-fibre.md). On mainnet, prepare it now and start it after activation.

> [!TIP]
> Submit your node as a peer to the [`celestiaorg/networks`](https://github.com/celestiaorg/networks) repository so other operators can find you.

---

[← 03 Consensus node](./03-consensus-node.md) · **04 Validator** · [05 Fibre →](./05-fibre.md)
