# `@bitflick/inscriptions`

[![CI](https://github.com/flick-ing/inscriptions/actions/workflows/ci.yml/badge.svg)](https://github.com/flick-ing/inscriptions/actions/workflows/ci.yml)
[![Changesets](https://img.shields.io/badge/changesets-enabled-blue.svg)](https://github.com/changesets)
[![npm version](https://img.shields.io/npm/v/@bitflick/inscriptions.svg)](https://www.npmjs.com/package/@bitflick/inscriptions)
[![npm downloads](https://img.shields.io/npm/dm/@bitflick/inscriptions.svg)](https://www.npmjs.com/package/@bitflick/inscriptions)
[![license](https://img.shields.io/npm/l/@bitflick/inscriptions.svg)](LICENSE)

TypeScript library for Bitcoin inscriptions and Ordinals/BRC-20 tooling, published as [`@bitflick/inscriptions`](https://www.npmjs.com/package/@bitflick/inscriptions) on npm.

The library builds Taproot inscription funding, reveal, and refund transactions; supports parent inscription flows and batching helpers; includes network helpers for mainnet, testnet, testnet4, and regtest; and provides mempool.space broadcast/funding utilities.

npm download counts are useful package-usage evidence, not a unique-user count.

## Installation

```bash
npm install @bitflick/inscriptions
```

## Concise Usage

```ts
import {
  broadcastTx,
  bitcoinToSats,
  generateFundableGenesisTransaction,
  generatePrivKey,
  generateRevealTransaction,
  waitForInscriptionFunding,
} from "@bitflick/inscriptions";

const funding = await generateFundableGenesisTransaction({
  address: "bc1p...",
  inscriptions: [
    { content: Buffer.from("Hello, Ordinals"), mimeType: "text/plain" },
  ],
  network: "mainnet",
  privKey: generatePrivKey(),
  feeRate: 10,
  tip: 10_000,
  padding: 546,
});

const [txid, vout] = await waitForInscriptionFunding(
  funding.inscriptionUtxo,
  funding.network
);

const revealTx = await generateRevealTransaction({
  inputs: [
    {
      amount: Number(bitcoinToSats(funding.fundingAmountBtc)),
      cblock: funding.initCBlock,
      leaf: funding.initLeaf,
      script: funding.initScript,
      padding: funding.padding,
      secKey: funding.secKey,
      tapkey: funding.initTapKey,
      txid,
      vout,
      inscriptions: funding.writableInscriptions,
      rootTapKey: funding.initTapKey,
    },
  ],
  feeRateRange: [10, 25],
  feeTarget: 10_000,
});

await broadcastTx(revealTx, funding.network);
```

See the package README for a longer walkthrough: [`packages/inscriptions/README.md`](packages/inscriptions/README.md).

## Supported Workflows

- Create fundable genesis transactions for one or more inscriptions.
- Generate reveal transactions from funded inscription UTXOs.
- Generate refund transactions for unspent inscription outputs.
- Work with parent inscription metadata and grouped funding inputs.
- Broadcast raw transactions and poll for inscription funding through mempool.space.
- Test flows against regtest using the checked-in integration test package.

## Public API Overview

The package exports transaction builders, network helpers, mempool helpers, grouping utilities, refund support, and shared TypeScript types from [`packages/inscriptions/src/index.ts`](packages/inscriptions/src/index.ts).

Key entry points include:

- `generateFundableGenesisTransaction`
- `generateRevealTransaction`
- `generateRefundTransaction`
- `broadcastTx`
- `waitForInscriptionFunding`
- `generatePrivKey`
- `networkNamesToTapScriptName`

## Release and Compatibility

- Package: [`@bitflick/inscriptions` on npm](https://www.npmjs.com/package/@bitflick/inscriptions)
- Source: [github.com/flick-ing/inscriptions](https://github.com/flick-ing/inscriptions)
- Changelog: [`packages/inscriptions/CHANGELOG.md`](packages/inscriptions/CHANGELOG.md)
- Releases are managed with Changesets.
- The repository includes unit tests plus a regtest-oriented integration package under `packages/test`.

## Packages

| Package                                           | Description                                                                                           | NPM                                                                                                                     |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| [`@bitflick/inscriptions`](packages/inscriptions) | A Node.js TypeScript library for creating Bitcoin inscriptions (BRC20) and interacting with ordinals. | [![npm](https://img.shields.io/npm/v/@bitflick/inscriptions.svg)](https://www.npmjs.com/package/@bitflick/inscriptions) |
| [`@bitflick/tsconfig`](packages/tsconfig)         | Shared TypeScript base configuration for all bitflick packages.                                       | [![npm](https://img.shields.io/npm/v/@bitflick/tsconfig.svg)](https://www.npmjs.com/package/@bitflick/tsconfig)         |

## Monorepo Development

Extend the shared TypeScript configuration in your `tsconfig.json`:

```json
{
  "extends": "@bitflick/tsconfig/tsconfig.base.json",
  "compilerOptions": {
    /* your overrides */
  }
}
```

## Contributing

Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on contributing, reporting issues, and release process.

## Code of Conduct

This project and its contributors abide by the [Code of Conduct](CODE_OF_CONDUCT.md).

## Security

See [SECURITY.md](SECURITY.md) for reporting security vulnerabilities.

## License

This project is licensed under the terms of the [MIT License](LICENSE).
