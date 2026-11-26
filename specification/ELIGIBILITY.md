# PPCR Eligibility Specification

Status: Experimental research draft
Stage: 2D5
Date: 2026-11-26

## Mandatory requirements

- Eligibility must derive from authenticated chain history.
- Historical authorization must not be weakened.
- No new ownership rights may be introduced.
- Unresolved ownership evidence must remain quarantined.
- Recovery must not create additional ETH.
- Eligibility must be independently reproducible.
- Missing or invalid evidence must fail closed.

## Current research inventory

Candidate wallets: 284
Provisionally consistent: 260
Quarantined: 24
Formally declared eligible: 0

## Mandatory verification gates

- Full historical account and storage proof verification.
- Historical authorization equivalence.
- Independent implementation validation.
- Consensus state-transition invariant testing.
- Independent security review.

## Deferred verification

Stage 2D4.3 remains incomplete and is mandatory
before any final eligibility commitment.

## Deployment restrictions

This document does not authorize recovery, modify
Ethereum state, or establish production eligibility.

The proposal remains experimental.
