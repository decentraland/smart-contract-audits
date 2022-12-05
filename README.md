# Decentraland Smart Contracts Audits

### Table of Contents

- [Smart Contracts Audits](#smart-contract-audits)
  - [LAND](#land-registry)
  - [Estate](#estate-registry)
  - [Marketplace](#marketplace)
  - [Bids](#bids)
  - [Names](#dcl-names)
  - [Collections](#collections)
  - [LAND Auction](#land-second-auction)
  - [PassThrough](#passthrough)
  - [Burners](#burners)
  - [Rentals](#rentals)
  - [Common Contracts](#common-contracts)
  - [Periodic Token Vesting](#periodic-token-vesting)
- [Well Known Issues](#well-known-issues)

## Smart Contract Audits

### LAND Registry

- [Deploy 2022/03/12](https://github.com/decentraland/land/releases/tag/Deploy%2F2022-03-12)
  - 2022/04/01 - [LANDRegistry By Facundo Spagnuolo](./reports/LANDRegistry_04-01-2022_Spagnuolo.md)
  - 2022/04/01 - [LANDRegistry By Certik](./reports/LANDRegistry_04-01-2022_Certik.pdf)
- [Deploy 2020/07/17](https://github.com/decentraland/land/releases/tag/Deploy%2F2020-07-17)
  - 2020/06/19 - [Add Minime token for on-chain voting](./reports/Decentraland_LAND_Estate_MINIME_06_19_2020.pdf)
- [Deploy 2018/08/31](https://github.com/decentraland/land/releases/tag/deploy%2F2018-08-31)
  - 2018/07/06 - [LANDRegistry By LevelK](./reports/Decentraland_Land_Registry_Audit_07-06-2018_LevelK.pdf)

### Estate Registry

- [Deploy 2020/07/17](https://github.com/decentraland/land/releases/tag/Deploy%2F2020-07-17)
  - 2022/05/28 - [Estate Registry re-audit by Facundo Spagnuolo](./reports/Decentraland_Estates_05_28_2022.md)
  - 2020/06/19 - [Add Minime token for on-chain voting](./reports/Decentraland_LAND_Estate_MINIME_06_19_2020.pdf)
- [Deploy 2018/08/31](https://github.com/decentraland/land/releases/tag/deploy%2F2018-08-31)
  - 2018/08/22 - [Estate Registry by Levelk](./reports/Decentraland_Estates_Audit_08-22-2018)

### Marketplace

- [v3.0.0](https://github.com/decentraland/marketplace-contracts/releases/tag/v3.0.0)
  - 2021/12/01 - [Marketplace V2 With Meta Tx and Royalties by Perssimistic](./reports/Marketplace_v2_01_12_2021_by_pessimistic.md)
  - 2021/12/03 - [Marketplace V2 With Meta Tx and Royalties by Certik](./reports/Decentraland_Marketplace_and_Bids_V2_With_Royalties_Certik_12_03_2021.pdf)
- [v1.0.0](https://github.com/decentraland/marketplace-contracts/releases/tag/1.0.0)
  - 2018/09/28 - [Marketplace By LevelK](./reports/Decentraland_Marketplace_V2_Audit_09-28-2018.pdf)

### Bids

- [v2.0.0](https://github.com/decentraland/bid-contract/releases/tag/v2.0.0)
  - 2021/09/20 - [Bids V2 By Quantstamp](./reports/Decentraland_Bids_in_Polygon_and_Collection_Bridges_09_20_2021.pdf)
  - 2021/12/01 - [Bids V2 With Meta Tx and Royalties by Perssimistic](./reports/Bid_v2_01_12_2021_by_pessimistic.md)
  - 2021/12/03 - [Bids V2 With Meta Tx and Royalties by Certik](./reports/Decentraland_Marketplace_and_Bids_V2_With_Royalties_Certik_12_03_2021.pdf)
- [v1.0.0](https://github.com/decentraland/bid-contract/releases/tag/v1.0.0)
  - 2019/02/18 - [Bids by Nomic Labs](./reports/Bid_Contract_Audit_Report_02_19_2019.pdf)

### DCL Names

- [v2.0.0](https://github.com/decentraland/avatars-contract/releases/tag/v2.0.0)
  - 2020/01/20 - [DCL Names with ENS](./reports/Decentraland_ENS_Avatars_audit_01_20_2020.pdf)
- [v1.0.0](https://github.com/decentraland/avatars-contract/releases/tag/v1.0.0)
  - 2019/06/03 - [DCL Names](./reports/Decentraland_Names_Audit_Agusin_Aguilar_06_03_2019.pdf)

### Collections

- [v1.32.0](https://github.com/decentraland/wearables-contracts/releases/tag/v1.32.0)
  - 2022/05/01 - [Upgradable Colllections](./reports/collections_05_01_2022.md)
- [v1.31.0](https://github.com/decentraland/wearables-contracts/releases/tag/v1.31.0)
  - 2021/11/10 - [Third Party Registry By Certik](./reports/Decentraland_TPR_Tiers_Certik_10_28_2021.pdf)
  - 2021/11/10 - [Third Party Registry By Pessimistic](./reports/Decentraland_TPR_Pessimistic_11_10_2021.pdf)
  - 2022/04/11 - [Third Party Registry With Merkle Root By Certik](./reports/Decentraland_TPR_with_Merklee_root_Certik_04_11_2022.pdf)
  - 2022/04/11 - [Rarities with oracles By Certik](./reports/TPRegistry_and_Rarities-with_Oracle_2022_04_11.pdf)
- [v1.29.9](https://github.com/decentraland/wearables-contracts/releases/tag/v1.29.0)
  - 2021/09/10 - [Collection Bridges By Pessimistic](./reports/Decentraland_Collections_Bridge_Security_Analysis_09_10_2021_Pessimistic.pdf)
  - 2021/09/20 - [Collection Bridges By Quantstamp](./reports/Decentraland_Bids_in_Polygon_and_Collection_Bridges_09_20_2021.pdf)
- [v1.23.0](https://github.com/decentraland/wearables-contracts/releases/tag/v1.23.0)
  - 2021/04/26 - [Collection V2 With By Agustin Aguillar](./reports/Decentraland_collections_v2_audit_04_26_2021.pdf)
- [v1.16.0](https://github.com/decentraland/wearables-contracts/releases/tag/v1.16.0)
  - 2020/10/13 - [Collection V2 Secon Iteration By Agustin Aguillar](./reports/Decentraland_Collections_v2_10_13_2020.pdf)
- [v1.3.0](https://github.com/decentraland/wearables-contracts/releases/tag/v1.3.0)
  - 2020/05/04 - [Collections V2 First Iteration By Agustin Aguillar](./reports/Decentraland_Collections_contract_05_04_2020.pdf)
- [v1.3.0](https://github.com/decentraland/wearables-contracts/releases/tag/v1.3.0)
  - 2020/05/04 - [Collections V2 First Iteration By Agustin Aguillar](./reports/Decentraland_Collections_contract_05_04_2020.pdf)

### LAND Second Auction

- [v1.0.0](https://github.com/decentraland/land-auction/releases/tag/v1.0.0)
  - 2018/10/12 - [LAND Auction by Agustin Aguilar](./reports/Decentraland_Land_Auction_10-12-2018.pdf)

### PassThrough

- [v1.0.0](https://github.com/decentraland/pass-through/releases/tag/v1.0.0)
  - 2019/01/07 - [PassThrough By Agustin Aguilar](./reports/Decentraland_PassThrough_contracts_Agustin_Aguilar_01_07_2019.pdf)
  - 2019/01/15 - [PassThrough By Nomic Labs](./reports/PassThrough_audit_report_nomic_01_15_2019.pdf)

### Burners

- [v1.0.0](https://github.com/decentraland/aux-contracts/releases/tag/v1.0.0)
  - 2019/01/10 - [Marketplace Burner by Nomic Labs](./reports/MarketplaceBurner_audit_report_01_10_2019.pdf)
  - 2019/06/03 - [Marketplace Burner by Agustin Aguilar](./reports/Decentraland_Burner_Contract_audit_06_03_2019.pdf)

### Rentals
  - 2022/10/27 - [Rentals by Xinghui Chen](./reports/Rentals_and_Common_Contracts_10_27_2022_Xinghui_Chen.pdf)
  - 2022/10/05 - [Rentals by Facundo Spagnuolo](./reports/Rentals_10_05_2022_Spagnuolo.md)
  - 2022/09/30 - [Rentals (Preliminary Report) by Certik](./reports/Rentals_and_Common_Contracts_preliminary_08_25_2022_Certik.pdf)

### Common Contracts
  - 2022/10/27 - [Common Contracts by Xinghui Chen](./reports/Rentals_and_Common_Contracts_10_27_2022_Xinghui_Chen.pdf)
  - 2022/10/05 - [Common Contracts by Facundo Spagnuolo](./reports/Common_Contracts_10_05_2022_Spagnuolo.md)
  - 2022/09/30 - [Common Contracts (Preliminary Report) by Certik](./reports/Rentals_and_Common_Contracts_preliminary_08_25_2022_Certik.pdf)

### Periodic Token Vesting

  - 2022/12/05 - [Periodic Token Vesting by Nethermind](./reports/PeriodicTokenVesting_Nethermind_12_05_2022.pdf)
  - 2022/12/02 - [Periodic Token Vesting by Certik](./reports/PeriodicTokenVesting_Certik_12_02_2022.pdf)
  - 2022/12/02 - [Periodic Token Vesting by Xinghui Chen](./reports/PeriodicTokenVesting_XinghuiChen_12_02_2022.pdf)
  - 2022/11/10 - [Periodic Token Vesting by Facundo Spagnuolo](./reports/PeriodicTokenVesting_FacundoSpagnuolo_11_10_2022.pdf)

## Well Known Issues

- [Smart Contracts](./smart_contracts_well_known_issues.md)
- [Apps/Websites](./app_well_known_issues.md)
