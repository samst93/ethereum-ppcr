# PPCR research checkpoint 2 — V23.7

Date: 2026-11-27. Proposed tag: `v0.2-research-checkpoint.2`.
Status: research evidence only; activation must remain disabled.

## Reproduction
Python V23.5 and Go V23.6 were rerun from their packaged sources on Linux; all package SHA-256 manifest checks passed. Python: 585 proofs, 21 tests; Go 1.23.2: 585 proofs, 28 tests. The uploaded Windows V23.5 report equals the reproduced Python report. Windows V23.6 reproduction has not been supplied.

Commitment: `e6840ccdc8bbd61222c3f074065196e3c97000d757c4452aa1c24e105cc97e5c`

Merkle root: `f0560b92eb9de5fd3cbcebc4479804c8bf974060ad52059a3afe7e829cbc9022`

## Evidence scope
585 indexed exact-runtime matches, not an exhaustive candidate set. Prior reports record 1,755 account snapshots and 15,354 selected storage proofs independently replayed against pinned roots at blocks 4501968, 4501969 and 26494170. 535 structurally consistent; 50 quarantined. 228 bounded native EVM observations are bound into the evidence. Four anomaly flag refinements preserved. No final eligibility determination.

Exact snapshot totals (wei): 513537869434745980492533, 513537869434745980492533, 515035136676569840784788.
Quarantine totals (wei): 92113565929498391369, 92113565929498391369, 128108411940159014798.

## Reproduce offline
Verify SHA256SUMS.txt. Extract PPCR_V23_5.zip into a separate directory and run `python ppcr_v23_5.py`. Extract PPCR_V23_6.zip into another directory and run `go run main.go`. Compare generated reports with the included reports. The Go verifier uses the standard library. Package manifests include their own inputs; verify before executing. No RPC needed.

## Open safety gates
- Chain canonicality and exhaustive candidate-set completeness are not established.
- Proxy dependency semantics and full historical authorization equivalence are not established.
- Native tests used synthetic surrounding state, selected storage, direct library calls, no signed transactions, and unverified Geth provenance.
- Global library restoration does not enforce individual wallet quarantine.
- Activation-state and sequential/parallel consensus equivalence, independent client implementation, and external security/EIP review remain pending.
- Commitment verification does not rerun raw trie proofs or native EVM observations. Raw evidence archives are referenced by hashes, not shipped here. Go verifies assembled bundle, not independent reconstruction from all eleven inputs.

No approval, deployment, or recovery entitlement is implied. Do not activate Geth.
