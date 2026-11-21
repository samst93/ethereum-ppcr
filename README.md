# ethereum-ppcr
Parity Proxy Contract Recovery — Ethereum Core EIP research, historical evidence, and consensus validation
# Ethereum PPCR — Parity Proxy Contract Recovery

**Research status:** Pre-EIP / Standards Track — Core candidate

PPCR explores a deterministic, historically bounded Ethereum consensus mechanism for restoring owner-authorized control of assets affected by the November 2017 Parity multisignature library destruction.

Etereum-Magicians https://ethereum-magicians.org/t/research-draft-ppcr-parity-proxy-contract-recovery-protocol/30469

## Research completed

- Historical enumeration through Ethereum block 4,501,969
- 284 identified Parity proxy candidates
- 93 candidates with nonzero historical ETH balances
- 443,163.039443434519138007 ETH observed at the historical reference snapshot
- Historical library and owner-configuration validation for two major wallets
- Successful local-fork multisignature state-transition experiments for those two wallets

## Current limitations

These findings do not establish that all matching wallets are recoverable. Historical eligibility, genuine owner authorization, and consensus implementation remain under investigation. The 945 unresolved runtime-creation events require further analysis.

## Project goals

Develop reproducible historical evidence, a deterministic eligibility specification, and a security-reviewed Ethereum recovery mechanism that preserves multisignature authorization without discretionary beneficiary selection.

## Status

Research Working Draft v0.1. Not approved or scheduled for Ethereum mainnet activation.

## Discussion

Technical feedback and independent verification are welcome.
