PPCR — Parity Proxy Contract Recovery Protocol
Research Working Draft v0.1
Date: November 21, 2026
Proposed EIP Track: Standards Track — Core
Status: Research / Pre-Draft (not an assigned EIP)
Activation: To be determined
Author: To be specified before submission
Research proposal, not an accepted EIP. The mechanism, eligibility rule and owner-signature verification are not yet specified at consensus implementation precision. The historical asset figures below are observations, not claims that the assets are recoverable.

Abstract
Parity Proxy Contract Recovery (PPCR) investigates a narrowly scoped Ethereum consensus mechanism for restoring owner-authorized control of assets held by Parity multisignature proxy contracts disabled by destruction of their shared library in November 2017.

The intended design would distinguish verifiable loss of wallet execution capability from loss of private keys, contract exploits and unrelated ownership disputes. Eligibility would be determined using a closed historical evidence set and deterministic verification rather than discretionary case adjudication. Recovery must preserve historically valid multisignature authorization and must not permit protocol administrators to designate beneficiaries.

Historical research identified 284 distinct contracts matching a targeted Parity proxy runtime fingerprint. At the end of block 4,501,968, 93 of those contracts had nonzero ETH balances totaling 443,163.039443434519138007 ETH. Controlled local-fork experiments on the two largest wallets demonstrated threshold-controlled 1-wei transfers in intact-library and locally restored-library conditions, but not in the destroyed-library condition. Those experiments used impersonated historical owner addresses, not real owner signatures.

This is an initial technical research draft, not a consensus-ready specification or a recovery-eligibility determination.
1. Motivation
In November 2017, destruction of the Parity multisignature shared WalletLibrary removed code relied upon by dependent proxy wallets. The proxies retained state, including ETH balances and owner-configuration data, but lost execution paths that had previously enabled wallet operations.

The question posed by PPCR is whether Ethereum could support an exceptionally narrow, deterministic and owner-authorized recovery mechanism without making generalized contract losses reversible or introducing discretionary control over other people's assets.

This draft does not argue that financial loss alone warrants a state intervention.
2. Historical context
The destroyed library's historical address is:

0x863df6bfa4469f3ead0be8f9f2aae51c91a907b4

PPCR's archival observations show 5,903 bytes of library code at the end of block 4,501,968 and zero bytes at the end of block 4,501,969. The incident was previously addressed by EIP-999, which proposed restoring library code and was subsequently withdrawn. Any PPCR proposal must address the technical and social objections to that earlier approach.
3. Scope
PPCR is limited to the specific 2017 Parity library-destruction event and to contracts meeting a future fully specified eligibility predicate. Initial contract identification uses runtime-code Keccak-256:

0x5d85058ca470c3561426ba7b1675197b2aa848173843dc79d36e3ec14ce91eeb

A bytecode match does not itself establish frozen assets, library-dependency causality, genuine authorization or eligibility.

PPCR does not propose a general solution for lost private keys, compromised accounts, unrelated shared libraries, bridges, governance attacks, proxy-admin errors or arbitrary smart-contract failures. ETH and token assets require distinct accounting and safety analysis.
4. Terminology
- Affected proxy: A historical Parity wallet proxy whose relevant execution depends on the specified destroyed library.
- Reference state: Authenticated Ethereum state at a precisely defined block or transaction boundary.
- Historical owner set / threshold: The wallet's valid owner addresses and required number of distinct approvals at the selected historical reference state, accounting for any preceding owner changes.
- Loss causality: Evidence that destruction of the shared library disabled a previously executable wallet operation.
- Eligibility proof: A consensus-verifiable demonstration that all specified recovery conditions are met.
- Recovery transition: A proposed owner-gated operation with deterministic state changes and replay protection.
5. Proposed eligibility principles
An eventual Core EIP must define an exact predicate, not rely on the following prose alone:
1. Identity: The contract existed before the incident; creation provenance and execution code can be independently reproduced from the chain.
2. Dependency: The relevant wallet operation actually reaches the specified shared library, not merely a matching static PUSH20 and DELEGATECALL pattern.
3. Historical authorization: Storage layout, owner set and signature threshold are reconstructed from verifiable historical evidence, including legitimate changes up to the selected boundary.
4. Causality: The intended operation executes with intact historical library code but fails to have that effect after code destruction, under controlled equivalent conditions.
5. Genuine approval: A recovery request is authorized by the applicable threshold of distinct valid signers, cryptographically verified for the exact operation, beneficiary, amount, chain, domain and nonce.
6. Accounting: No ETH is minted; source debit and recipient credit balance exactly, and no value can be recovered twice.
7. Closed scope: Eligibility cannot be expanded by later discretion or by superficial similarity to unrelated historical incidents.
Owner-address impersonation on a local fork is a testing technique, not acceptable evidence of real signer consent.
6. Candidate consensus architectures
Option A: Library-code restoration
A scheduled protocol transition would restore a reviewed implementation at the destroyed library address. This closely models successful counterfactual tests, but may reactivate capabilities in every dependent contract, potentially including historically vulnerable behavior, and substantially overlaps EIP-999. It is not approved by the present research.
Option B: Narrow owner-gated recovery transition
A future consensus transition would permit a precisely defined recovery operation only when deterministic historical eligibility and current cryptographic owner-threshold authorization are both proven. A client could validate the predicate and the authorized transfer without granting general-purpose restoration powers.

Working research direction: Explore Option B for a distinctive, bounded PPCR design; use Option A as a historical and experimental comparison. The exact recovery transaction format, state-transition function, gas rules, client validation algorithm and signature model are not yet specified.

A manifest root could commit to candidate evidence but would not independently establish a complete or correct eligible set. An independent derivation or other sound inclusion-and-completeness argument is required.
7. Required invariants
- Authorization: An operation cannot succeed without the required distinct, valid, historically authorized approvals.
- Conservation: ETH credited to a beneficiary is debited from the source, without unauthorized supply creation.
- Isolation: Ineligible accounts cannot be altered by the PPCR rule.
- Replay safety: A completed authorization cannot transfer the same value twice.
- Determinism: All conforming clients produce the same state root for the same inputs.
- Historical integrity: Past blocks and receipts remain unchanged.
- Non-discretion: No administrator selects wallets or beneficiaries through a subjective process.
- Atomicity: Failure cannot cause a partial recovery-induced transfer.
These are proposed properties, not all formally proven.
8. Research evidence (D3I.10J–D3I.15)
D3I.10J — Historical enumeration
The targeted scan through block 4,501,969 recorded 227,740 unique historical blocks, 782,601 deduplicated creation-attempt records and 284 unique matching proxy candidates. The exported manifest passed its internal source audit. 945 creation events had unresolved runtime classification and therefore limit any completeness claim.

Candidate manifest CSV SHA-256:

3F9A3468DA8014A442BED14A93A0FA7A9FA954107EFDD7881BD0270EB9061A44

Final batch SHA-256:

16718A03C88379EFDCE7D623FCF98093B3EA1DF9C1384F7D45E0D8DA5512B2E4
D3I.11 — Historical balance and code
All 284 candidates reportedly matched the expected 925-byte proxy code at both historical snapshots. 93 held nonzero ETH at block 4,501,968, totaling 443,163.039443434519138007 ETH. ETH balance observations do not prove frozen assets or current entitlement; zero-ETH candidates may also hold tokens.

D3I.11 JSON SHA-256:

B9458B4BD6A67A24D14480FF1743724EC7F04F8829C7BC4248E91F26FED510FB
D3I.12 — Library dependency and owner configuration
For two high-balance pilot wallets, historical storage and getter responses agreed on the following configurations:
Proxy	Historical ETH	Owner configuration
0x3bfc20f0b9afcace800d73d2191166ff16540258	306,276.272251399926202	4-of-7
0x376c3e5547c68bc26240d8dcc6729fff665a4448	114,939	2-of-3


Both proxies included the library address and DELEGATECALL instructions. The library had code at block 4,501,968 and none at block 4,501,969. Static references alone are not a causality proof.

D3I.12 JSON SHA-256:

98E63E6C8905B6BB652E5A9CB95427B559430B9DF1BC27B079577E1E2E278834
D3I.13–D3I.14 — Execution-path comparison
In historical eth_call/trace simulations, test calls to each wallet reached the library. With library code present, traces showed an internal wallet call and one event log; after destruction these effects disappeared. An ephemeral counterfactual restoration of historical library code in simulated post-incident state restored the observed execution path for both wallets.

D3I.13 SHA-256:

5F030EEC5E240F63830F0F2D0455B329E239C30E4A3153C160BBC8FF0F6119B9

D3I.14 SHA-256:

78BB630EA33B9F5F2664765190BDEEF6D139956392362204BDA5D566F8C76E4B
D3I.15 v1.2 — Local threshold-controlled transfers
Disposable Anvil forks tested 1-wei transfers from both pilot wallets under three conditions:
Condition	4-of-7 proxy	2-of-3 proxy
Historical library intact	+1 wei to recipient	+1 wei to recipient
Historical library destroyed	0 wei	0 wei
Original library locally restored	+1 wei to recipient	+1 wei to recipient


The local-fork evidence recorded corresponding source-wallet debits, threshold execution events, and negative tests for non-owner and duplicate approvals. A previous v1.1 run failed because of an incorrect function selector; v1.2 corrected the selector and passed.

Limitations: This experiment impersonated historical owner addresses on a local fork. It did not demonstrate real cryptographic signatures, contemporaneous consent, current control, or permission to recover an entire wallet balance. No mainnet transactions were broadcast.

Successful D3I.15 v1.2 JSON SHA-256:

E911D46D670CDE6A00CCE0ADD9D127E7B4EADEA43CB16D4D3B7150721CABF35A
9. Rationale and governance considerations
Historical determinism is useful for limiting discretionary intervention, but determinism alone is not legitimacy. A Core EIP would need to explain why this historical exception should be adopted and how it differs from past proposals that failed to obtain consensus.

A genuine owner-gated recovery approach may differ from a blanket restoration of code, but that difference only becomes material when its exact state-transition semantics and attack surface are specified.
10. Backwards compatibility
Any selected PPCR mechanism would require coordinated Ethereum execution-layer changes and likely a scheduled network upgrade. An exact activation block, recovery transaction encoding, fee schedule and client-validation procedure have not been established. Interaction with existing wallets, historical approvals, ERC-20 tokens, contract-based owner addresses and current account-abstraction patterns requires analysis.
11. Security considerations
Principal risks include unauthorized beneficiary selection, compromised historical keys, unauthorized replay, inconsistent historical storage interpretation, incomplete candidate enumeration, token/accounting edge cases, unanticipated global library reactivation, client implementation divergence, and normalizing discretionary intervention in otherwise immutable contract state.

Especially important is the gap between a stored historical owner address and valid present-day consent. That gap must be resolved explicitly rather than presumed away.
12. Relationship to EIP-999
EIP-999 proposed restoring code at the destroyed library address and was withdrawn. PPCR explores a more constrained owner-gated eligibility model, but it does not yet demonstrate a complete consensus design distinct from EIP-999. Prior objections, including precedent and state surgery, must be confronted directly.
13. Required test vectors
Before formal submission, publish independent vectors covering affected and unaffected proxies; historical ownership/threshold changes; signed, unsigned and malformed authorizations; duplicates; replays; ETH conservation; token edge cases; unresolved creation events; account and library state changes; and multi-client identical state roots.
14. Outstanding work
1. Specify a complete deterministic, independently verifiable eligibility predicate.
2. Resolve or safely bound 945 unresolved runtime-creation events.
3. Expand causality and ownership validation beyond the two pilot wallets.
4. Specify signature verification and revocation/compromise treatment.
5. Decide the asset scope (ETH only versus tokens) and the recovery transition architecture.
6. Formalize state isolation, replay safety, conservation and failure atomicity.
7. Build independent multi-client prototypes and publish reproducible conformance tests.
8. Seek candid review from execution-client developers and the Ethereum community.
15. Conclusion
The D3I.10J–D3I.15 program provides an evidence-backed starting point: a reproducible historical candidate set, observed balances, reconstructed multisignature configurations for two large wallets, and local-fork demonstrations linking library destruction to loss of threshold-controlled transfer functionality.

It does not yet prove broad eligibility or justify a mainnet change. PPCR v0.1 is intended to invite independent technical criticism and identify precisely which claims remain to be proven.
References
1. EIP-1 — EIP Purpose and Guidelines
2. Parity Technologies — A Postmortem on the Parity Multi-Sig Library Self-Destruct
3. OpenZeppelin — The Parity Wallet Hack Reloaded
4. EIP-999 — Restore Contract Code at the Parity WalletLibrary Address
5. PPCR D3I.10J–D3I.15 historical and local-fork research records (to be added to the repository after review and secret sanitization).
