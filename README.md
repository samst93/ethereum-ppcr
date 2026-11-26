
# ethereum-ppcr
Parity Proxy Contract Recovery — Ethereum Core EIP research, historical evidence, and consensus validation
# Ethereum PPCR — Parity Proxy Contract Recovery

**Research status:** Pre-EIP / Standards Track — Core candidate

PPCR explores a deterministic, historically bounded Ethereum consensus mechanism for restoring owner-authorized control of assets affected by the November 2017 Parity multisignature library destruction.

Etereum-Magicians https://ethereum-magicians.org/t/research-draft-ppcr-parity-proxy-contract-recovery-protocol/30469

PPCR_Protocol_Working_Draft_v0.1 [PPCR_Protocol_Working_Draft_v0.1.pdf](https://github.com/user-attachments/files/35221397/PPCR_Protocol_Working_Draft_v0.1.pdf)

Visual Explanation of PPCR_Protocol [PPCR_Explained_Research_Overview_v0.1.pdf](https://github.com/user-attachments/files/35221576/PPCR_Explained_Research_Overview_v0.1.pdf)

Overview

Parity Proxy Contract Recovery (PPCR) explores a narrowly scoped Ethereum consensus mechanism for restoring owner-authorized control of assets held in multisignature proxy contracts affected by the November 2017 Parity WalletLibrary destruction.

The goal is to investigate whether a historically verifiable and non-discretionary recovery mechanism can be implemented without introducing a generalized ability to reverse transactions, change contract ownership, or redistribute assets.

PPCR is currently an independent research project, not an accepted Ethereum Improvement Proposal.

The Problem

In November 2017, the shared library used by a class of Parity multisignature wallets was destroyed.

Affected wallets depended on this library for execution through DELEGATECALL. Although their historical owner configurations and assets could remain in contract state, the disappearance of the library code disabled essential wallet operations.

The central issue is that valid multisignature authorization alone could no longer execute the original wallet functionality.

The Proposed Solution

PPCR investigates a deterministic consensus-level recovery mechanism restricted to contracts satisfying a precisely defined historical eligibility predicate.

The proposed approach would:

Verify historical identity: Establish that a contract belonged to the relevant Parity proxy wallet class.

Verify historical dependency: Confirm that the wallet relied on the destroyed shared library.

Verify eligibility: Apply objective, consensus-verifiable criteria to a historically bounded set of contracts.

Verify owner authorization: Require cryptographically valid authorization consistent with the applicable historical multisignature rules.

Execute a restricted recovery transition: Permit only the specifically authorized recovery operation while preserving asset conservation and preventing replay.

These are design objectives. The complete consensus rules, signature format, and transaction execution mechanism remain under development.

PPCR does not propose recovery based solely on a claimant's assertion of ownership.

Historical Research

Historical Ethereum execution-state research has produced the following preliminary results:

Research Area

Finding

Historical scan

227,740 unique blocks examined

Scan ending block

4,501,969

Matching proxy candidates

284

Candidates holding ETH at the historical reference snapshot

93

Historical ETH balance total across those 93 contracts

443,163.039443434519138007 ETH

Detailed pilot wallets

2

Counterfactual local-fork transfer experiments

2/2 successful

Unresolved runtime-creation events

945

Important: The historical ETH balance total is not a claim that this amount remains locked today, is eligible for PPCR, or can ultimately be recovered.

The identified proxy candidates must undergo further eligibility and authorization analysis.

Experimental Results

Research stages D3I.10J through D3I.15 have examined historical contract creation, runtime code fingerprints, ETH balances, proxy-library dependencies, owner configurations, execution traces, and local state transitions.

Two significant historical wallets were used for detailed pilot experiments:

0x3bfc20f0b9afcace800d73d2191166ff16540258 — historical 4-of-7 multisignature configuration.

0x376c3e5547c68bc26240d8dcc6729fff665a4448 — historical 2-of-3 multisignature configuration.

Controlled local-fork experiments demonstrated that threshold-governed ETH transfers were possible when the historical library code was present or counterfactually restored, but were not completed when the library code was absent.

The successful experiments used local impersonation of historical owner addresses, not actual cryptographic signatures from those owners.

Consequently, these tests demonstrate a limited form of execution causality and multisignature threshold behavior. They do not prove real-world owner consent or consensus-level recovery readiness.

Proposed Security Principles

PPCR is being developed around the following principles:

Deterministic eligibility: All validating clients must reach the same conclusion about whether a wallet qualifies.

Historically closed scope: The recovery mechanism must not become a general-purpose contract rescue facility.

Owner authorization: Recovery must not permit arbitrary parties to select beneficiaries.

Value conservation: The mechanism must not create ETH or permit unintended asset transfers.

Replay protection: A valid authorization must not be reusable outside its permitted scope.

Independent verification: Historical evidence and state-transition results should be reproducible.

No discretionary selection: Protocol rules, rather than case-by-case governance decisions, must determine eligibility.

Relationship to EIP-999

EIP-999 previously proposed restoring the historical Parity WalletLibrary code through a consensus state change.

PPCR investigates whether a more restricted, owner-authorized recovery transition could address the same underlying loss of functionality while avoiding global restoration of the historical library.

Whether this alternative sufficiently resolves the technical and governance objections raised against earlier recovery proposals remains an open research question.

Research Documents

Ethereum Magicians discussion

PPCR — Parity Proxy Contract Recovery Protocol

Protocol working draft

PPCR Protocol Working Draft v0.1 (PDF)

Explanatory overview

PPCR Explained — Research Overview v0.1 (PDF)

Development Roadmap

Stage

Objective

Status

D3I.10J

Historical candidate enumeration

Completed

D3I.11

Historical balance and code validation

Completed

D3I.12

Library dependency and ownership reconstruction

Pilot completed

D3I.13–14

Execution causality and restoration simulations

Pilot completed

D3I.15

Threshold-governed state-transition experiments

Pilot completed

D3I.16

Expanded historical eligibility validation

Planned

D3I.17

Cryptographic authorization design and validation

Planned

D3I.18

Deterministic consensus transition specification

Planned

D3I.19

Reference implementation and test vectors

Planned

Formal EIP

Submit complete Core EIP specification for review

Not submitted

Open Research Questions

Can the eligible contract set be defined through an objective, historically bounded predicate?

How should the protocol validate authentic owner authorization without weakening the original multisignature security model?

How should unresolved creation events and non-ETH assets be handled?

What is the safest consensus mechanism for recovery without broadly restoring vulnerable historical behavior?

Can the proposed recovery mechanism withstand independent security and consensus review?

Would Ethereum's core developers and broader community support such an exceptional protocol change?

Contributing and Discussion

Technical criticism, independent reproduction, security analysis, and alternative designs are welcome.

The immediate focus is formalizing eligibility and authorization rules before advancing toward a complete Core EIP specification.

Please use the linked Ethereum Magicians thread for protocol-level discussion.

Disclaimer: PPCR is experimental research. No recovery capability is currently available on Ethereum mainnet through this proposal. The existence of historical assets or a matching proxy bytecode fingerprint does not establish present eligibility, ownership, consent, or recoverability.
