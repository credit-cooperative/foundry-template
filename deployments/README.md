# Deployments registry

This directory is the **canonical, committed record** of this project's deployed contract addresses —
one file per chain, named `<chainId>.json` (e.g. `8453.json` for Base, `11155111.json` for Sepolia).

Raw Foundry `broadcast/` artifacts stay gitignored; these curated JSON files are the source of truth that
the [address-book](https://github.com/credit-cooperative/address-book) aggregator consumes.

## How it's maintained

Do not hand-edit these files. They are generated from broadcast artifacts by
[`@credit-cooperative/devkit`](https://github.com/credit-cooperative/devkit):

```sh
# Deploy + broadcast + update the registry in one step (add --verify, --private-key, etc. as args)
just deploy scripts/solidity/Deploy.s.sol <chain>

# Or, update the registry from the latest broadcast of a script you ran manually
just deployments-extract scripts/solidity/Deploy.s.sol

# Verify every registry address has code on-chain
just deployments-check --rpc-url <chain>
```

After deploying: **review the JSON diff and commit it.** Merging the change to `main` triggers the
`Deployments` workflow, which notifies the address-book aggregator.

## Schema

```jsonc
{
  "chain": "base", // RPC alias from foundry.toml
  "chainId": 8453,
  "contracts": {
    "PaymentRails": {
      "address": "0x…", // current deployed address
      "deployer": "0x…",
      "txHash": "0x…",
      "blockNumber": 123,
      "commit": "abc1234", // git commit the deploy was built from
      "version": "1.2.0", // package.json version at deploy time
      "constructorArgs": ["0x…"], // present only when the deploy had args
      "history": [
        // present only after a re-deploy to a new address
        {
          "address": "0x…",
          "version": "1.1.0",
          "commit": "def5678",
          "deprecated": "2026-06-12",
          "...": "…",
        },
      ],
    },
  },
}
```

Re-deploying a contract to a **new** address moves the previous entry into its `history` array (tagged with a
`deprecated` date); re-running the same deployment just refreshes metadata in place.

To read an address back inside a deploy script, use `readDeployment("<Name>")` from `BaseScript`.
