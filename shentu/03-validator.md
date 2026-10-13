<img src="../assets/shentu-step-03.svg" width="100%" alt="Shentu step 03 — Validator">

[← 02 Full node](./02-full-node.md) · **03 Validator** · [04 Maintenance →](./04-maintenance.md)

---

> [!CAUTION]
> **Double signing on Shentu = 5% slash + permanent tombstone.** Never run the same `priv_validator_key.json` on two machines.

```bash
source $HOME/.shentu-env
```

## 1. Wallet

```bash
shentud keys add $WALLET --keyring-backend $KEYRING_BACKEND            # save the mnemonic offline
# shentud keys add $WALLET --recover --keyring-backend $KEYRING_BACKEND

WALLET_ADDR=$(shentud keys show $WALLET -a --keyring-backend $KEYRING_BACKEND)
VALOPER=$(shentud keys show $WALLET --bech val -a --keyring-backend $KEYRING_BACKEND)
echo "export WALLET_ADDR=$WALLET_ADDR" >> $HOME/.shentu-env
echo "export VALOPER=$VALOPER"         >> $HOME/.shentu-env

shentud q bank balances $WALLET_ADDR
```

## 2. Back up the consensus key

```bash
cp $DAEMON_HOME/config/priv_validator_key.json $HOME/priv_validator_key.shentu.backup.json
chmod 600 $HOME/priv_validator_key.shentu.backup.json
```

## 3. `validator.json`

Shentu runs Cosmos SDK 0.53, so `create-validator` takes a **JSON file**.

```bash
cat > $HOME/validator.json <<EOF
{
  "pubkey": $(shentud comet show-validator --home $DAEMON_HOME),
  "amount": "1000000uctk",
  "moniker": "$MONIKER",
  "identity": "<keybase-id>",
  "website": "<https://your-website>",
  "security": "<security-contact-email>",
  "details": "<short description>",
  "commission-rate": "0.05",
  "commission-max-rate": "0.20",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
EOF
jq . $HOME/validator.json
```

## 4. Create

```bash
shentud tx staking create-validator $HOME/validator.json \
  --from $WALLET --chain-id $CHAIN_ID --keyring-backend $KEYRING_BACKEND $GAS -y
```

## 5. Verify

```bash
shentud q staking validator $VALOPER | grep -E "moniker|status|jailed|tokens"
curl -s localhost:26657/status | jq '.result.validator_info.voting_power'
```

The active set holds **125** validators. `BOND_STATUS_BONDED` with a non-zero `voting_power` means you're in.

## 6. Stay out of jail

| Rule | Value |
|---|---|
| Window | 10 000 blocks |
| Must sign | **≥ 10%** |
| Downtime jail | 10 min + 0.01% slash |
| Double sign | 5% + tombstone |

```bash
shentud q slashing params
shentud q slashing signing-info $(shentud comet show-validator --home $DAEMON_HOME)

shentud tx slashing unjail --from $WALLET --chain-id $CHAIN_ID \
  --keyring-backend $KEYRING_BACKEND $GAS -y
```

## 7. Edit, delegate, withdraw

```bash
shentud tx staking edit-validator --new-moniker="<new-moniker>" --website="<https://your-website>" \
  --from $WALLET --chain-id $CHAIN_ID --keyring-backend $KEYRING_BACKEND $GAS -y

shentud tx staking delegate $VALOPER 1000000uctk \
  --from $WALLET --chain-id $CHAIN_ID --keyring-backend $KEYRING_BACKEND $GAS -y

shentud tx distribution withdraw-rewards $VALOPER --commission \
  --from $WALLET --chain-id $CHAIN_ID --keyring-backend $KEYRING_BACKEND $GAS -y
```

---

[← 02 Full node](./02-full-node.md) · **03 Validator** · [04 Maintenance →](./04-maintenance.md)
