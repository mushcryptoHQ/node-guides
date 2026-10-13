<a href="https://mushcrypto.world/">
  <img src="./assets/hero.svg" width="100%" alt="MushCrypto Node Guides">
</a>

<p align="center">
  <a href="./celestia/"><img src="https://img.shields.io/badge/Celestia-guides-7b2bf9?style=for-the-badge&labelColor=060a14" alt="Celestia guides"></a>
  <a href="https://mushcrypto.world/"><img src="https://img.shields.io/badge/mushcrypto.world-060a14?style=for-the-badge&logo=googlechrome&logoColor=00b6fd" alt="Website"></a>
  <a href="https://mushcrypto.world/scorecard/"><img src="https://img.shields.io/badge/Governance_Scorecard-060a14?style=for-the-badge&logo=googlesheets&logoColor=e2a63c" alt="Scorecard"></a>
  <a href="https://mushcrypto.world/docs/"><img src="https://img.shields.io/badge/DA_Node_Docs-060a14?style=for-the-badge&logo=readthedocs&logoColor=00b6fd" alt="Docs"></a>
</p>

<p align="center"><i>The runbooks we actually use to run our validators, written down so you can use them too.</i></p>

<br>

## 🍄 Networks

<p align="center">
  <a href="./celestia/"><img src="./assets/net-mainnet.svg" width="49%" alt="Celestia — mainnet & Mocha"></a>
  <a href="./axelar/"><img src="./assets/net-axelar.svg" width="49%" alt="Axelar — axelar-dojo-1"></a>
  <a href="./akash/"><img src="./assets/net-akash.svg" width="49%" alt="Akash — akashnet-2"></a>
  <a href="./shentu/"><img src="./assets/net-shentu.svg" width="49%" alt="Shentu — shentu-2.2"></a>
</p>

| Network | Chains | What's covered |
|---|---|---|
| 🟣 [**Celestia**](./celestia/) | `celestia` · `mocha-5` | Consensus node · Validator · **Fibre server** · Upgrades & signalling · Cheat sheet |
| ⚪ [**Axelar**](./axelar/) | `axelar-dojo-1` | Full node · Validator · **tofnd + vald** · Broadcaster · **EVM chain maintainers** · Cheat sheet |
| 🔴 [**Akash**](./akash/) | `akashnet-2` | Full node · Validator (`validator.json`) · Upgrades · Cheat sheet |
| 🟠 [**Shentu**](./shentu/) | `shentu-2.2` | Full node · Validator (`validator.json`) · Upgrades · Cheat sheet |

<sub>Axelar, Akash and Shentu guides cover <b>mainnet only</b>. Celestia covers both mainnet and the Mocha testnet.</sub>

<br>

## 🧭 How these guides are written

| | Principle |
|---|---|
| 🔀 | **One runbook, two networks.** You set `CHAIN_ID` once and every command works for both mainnet and testnet. |
| 📌 | **Pinned versions.** Each guide states which release it was checked against, and the table in each network hub stays current. |
| 🔐 | **Safety first.** Double-signing and key-handling warnings come *before* the command that could hurt you. |
| 📎 | **Sourced.** Every network-specific value links back to the official docs or release notes. When they disagree, the official source wins. |

> [!IMPORTANT]
> Chains move quickly. Before running anything on mainnet, compare versions, hashes and endpoints with the official documentation linked in each guide.

<br>

## 🤝 Contributing

Did you find a stale version, a broken command or a better way to do something? Open an issue or a PR, or write to **hello@mushcrypto.xyz**.

<img src="./assets/footer.svg" width="100%" alt="Infrastructure, unseen.">
