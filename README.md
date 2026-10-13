# Quevra Envio indexer

Indexes Quevra's Monad Testnet veMON lock lifecycle, validator identities and backing, staking intent and settlement, validator vote weights, reward funding and claims, and vault reward accounting. Contract addresses and start blocks are configured in `config.yaml`; vault, gauge, and bribe instances are registered from their factory events.

## Local development

Requirements: Node.js 22+, pnpm, and Docker. An Envio HyperSync API token is needed when indexing Monad Testnet.

1. Install dependencies with `pnpm install`.
2. Set `ENVIO_API_TOKEN` in a local `.env` file (do not commit it) to index Monad Testnet.
3. Run `pnpm codegen` after changing `config.yaml` or `schema.graphql`.
4. Run `pnpm indexer:dev` for Monad Testnet, or `pnpm indexer:solonet:dev` for local Solonet. Both expose GraphQL at `http://127.0.0.1:8080/v1/graphql`.

The Solonet config in `config.solonet.yaml` reads from `http://127.0.0.1:48080` and uses the local deployment addresses recorded in `packages/contracts/deployments/20143.json`. It is isolated from the Monad Testnet config so local development does not require an Envio API token. It starts from the current head because Solonet RPC only serves a recent block window; earlier local activity cannot be backfilled.

The hosted Monad testnet dapp is at [https://quevra-pearl.vercel.app](https://quevra-pearl.vercel.app). To run the dapp against the local Solonet indexer, set `NEXT_PUBLIC_CHAIN_ID=20143` in the dapp environment and start the dapp separately. Its `/api/indexer` route defaults to the local GraphQL URL; override it with `ENVIO_SOLONET_GRAPHQL_ENDPOINT` if needed. For Envio Cloud on Monad Testnet, set `ENVIO_GRAPHQL_ENDPOINT` to the deployed GraphQL URL and configure endpoint authentication with `ENVIO_GRAPHQL_AUTH_TOKEN` or `ENVIO_GRAPHQL_ADMIN_SECRET` in the dapp server environment.

## Indexed data

- `Position` tracks veMON ownership, locked MON, expiry/permanent-lock changes, and validator identity NFTs.
- `Validator` tracks veValidator ownership and its linked operator, vault, gauge, bribe, reward token, and Monad validator ID.
- `Activity` records lock and approval actions, MON deposits, stake intent and settlement, backing changes, vote-weight changes, reward funding/claims, validator configuration, and vault reward sync/payment events.
- `DynamicRewardContract` associates each created gauge or bribe instance with its validator identity so dynamic contract events can be joined to Quevra positions.

The indexer provides historical activity and identity discovery. Use direct contract reads for current balances and claimable amounts.

`NonStakingGauge` is present in the Solidity source but has no deployed instance or factory event in the current Monad Testnet deployments, so Envio has no address from which to discover and index `RewardsBeneficiarySwitched` yet. Add it when that contract receives an on-chain registration path or deployment.
