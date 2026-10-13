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

<table>
  <tr>
    <td width="50%" valign="top">
      <h3>🟣 <a href="./celestia/">Celestia</a></h3>
      <b>Mainnet Beta</b> <code>celestia</code> · <b>Mocha</b> <code>mocha-5</code><br><br>
      Consensus node · Validator · <b>Fibre server</b> · Upgrades · Monitoring · Cheat sheet<br><br>
      <a href="./celestia/"><b>Open the Celestia runbook →</b></a>
    </td>
    <td width="50%" valign="top">
      <h3>🌱 Coming next</h3>
      <b>Axelar</b> <code>axelar-dojo-1</code><br>
      <b>Akash</b> <code>akashnet-2</code><br>
      <b>Shentu</b> <code>shentu-2.2</code><br><br>
      <sub>These guides grow from the same template as the networks we validate.</sub>
    </td>
  </tr>
</table>

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
