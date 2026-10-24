# cyphras-vault-archive

The public event history of the Cyphras private-payments vaults on Stellar, kept here so a wallet
can rebuild its notes from the seed without any Cyphras server and after the RPC has dropped old
ledgers. It is chain data only. Everything in this repository is released under CC0 1.0 (see
LICENSE).

```
mainnet/xlm/manifest.json
mainnet/xlm/windows/<first ledger, 10 digits>.jsonl.gz
testnet/xlm/...
```

Each window is one day of ledgers, gzip of one JSON event per line. The manifest lists the vault,
the network, every window and the SHA-256 of its content. The format is `cyphras-vault-archive/1`,
defined by `services/internal/vaultarchive` in
[cyphras/cyphras-contracts](https://github.com/cyphras/cyphras-contracts).

## Trust nothing here

A reader does not have to trust this repository, its history or Cyphras. Every window is checked
against the manifest, and the whole archive is checked against the vault on chain: the commitments
must rebuild a tree root the vault has held, and every nullifier must exist in the vault. A copy that
fails a check is wrong, wherever it came from; any clone or mirror is as good a source as this one.

## Verify it

Get the `archive` tool from a release of cyphras/cyphras-contracts (`services-v*`, image
`ghcr.io/cyphras/cyphras-contracts/archive`, built and attested by that repository's release
workflow) or build it from the release tag with `go build ./cmd/archive` in `services/`. Take the
deployment file for the network from `deployments/` in the same repository.

```
archive verify --dir mainnet/xlm --deployment mainnet.json --rpc <rpc 1> --rpc <rpc 2>
```

This checks the format and then each RPC against the archive; give RPCs run by different
providers. `--format-only` checks the files alone.

To reproduce the mainnet windows without trusting the published files, rebuild them from the public
Stellar ledger metadata archive and compare the manifests' `windows`:

```
archive rebuild --ledger-meta https://aws-public-blockchain.s3.us-east-2.amazonaws.com/v1.1/stellar/ledgers/pubnet \
  --deployment mainnet.json --out rebuilt --to <coveredTo of the manifest>
```

## How it is kept

A daily workflow copies the published archive from
`https://private.cyphras.com/<network>/xlm/archive` with `archive fetch --rpc`, verifies it
against two independent RPCs and commits it with the workflow's own token. A window already
committed is never replaced: a published archive that is behind this one or changes an old window
fails the run. A weekly workflow rebuilds the mainnet windows added since its last pass from the
ledger metadata archive, records how far it got in `mainnet/crosschecked`, and fails if any
differs from what is committed.
