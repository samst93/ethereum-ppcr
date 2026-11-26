# PPCR v0.2 — Parity Proxy Contract Recovery

**Consensus Design Research Draft — for technical criticism, not deployment**  
**Date:** 2026-11-24  
**Proposed EIP type/category:** Standards Track / Core  
**Formal EIP status:** Pre-Draft; no assigned EIP number  
**GitHub project:** https://github.com/samst93/ethereum-ppcr  
**Discussion:** https://ethereum-magicians.org/t/research-draft-ppcr-parity-proxy-contract-recovery-protocol/30469

> **Important limitation.** This document defines one concrete *candidate* state-transition architecture, including a checkable eligibility algorithm and owner-signature format. It is **not a complete, implementable network-upgrade specification**. Historical state roots, an EIP-2718 transaction-type identifier, fee constants, proof bounds, source-code/storage-layout audit, and conformance vectors remain to be finalized. None of the proposed recovery transactions exist on Ethereum mainnet.

## Abstract

This proposal investigates a **one-time, ETH-only, owner-threshold-authorized sweep** for an objectively defined subset of Parity multisignature proxy wallets disabled by the destruction of the shared `WalletLibrary` in November 2017. Rather than restoring library code, a new Ethereum typed transaction would prove a wallet's 2017 state against fixed historical state roots, verify an EIP-712 authorization signed by the historical wallet owners, and make a narrow consensus-level balance and replay-marker change. It would not call the wallet, execute recipient code, restore library functionality, or give an administrator control over beneficiaries.

The principal unresolved question is whether such an exceptional consensus mechanism is justified at all. The design deliberately fails closed when historical identity, ownership, provenance, or authorization is ambiguous.

## 1. Motivation and exclusions

The original Parity wallet proxies delegated functionality to a shared library. Archive-state analysis for this project reports 5,903 code bytes at the library address after block 4,501,968 and zero code bytes after block 4,501,969. A direct transfer from an affected wallet is not available merely because historical owners still hold their signing keys.

**Included in v0.2:** a single, explicit transfer of **the entire ETH balance currently held** by an eligible proxy, to one recipient explicitly approved by the applicable historical multisignature threshold.

**Excluded:** ERC-20/ERC-721/ERC-1155 token recovery; arbitrary contract calls; restoration of wallet or library code; owner replacement; lost-key recovery; dispute adjudication; wallets using incompatible proxy code or unverifiable owner state; recovery of future ETH deposited after the single sweep. A one-time sweep intentionally leaves later deposits unaddressed, which is a substantial limitation for review.

## 2. Historical constants and commitment anchors

The following values are fixed for **Ethereum mainnet only** (`chainId = 1`):

| Identifier | Value |
|---|---|
| `PRE_BLOCK` | 4,501,968, end-of-block state |
| `POST_BLOCK` | 4,501,969, end-of-block state |
| `LIBRARY` | `0x863df6bfa4469f3ead0be8f9f2aae51c91a907b4` |
| `PROXY_CODE_HASH` | `0x5d85058ca470c3561426ba7b1675197b2aa848173843dc79d36e3ec14ce91eeb` |
| `PRE_LIBRARY_CODE_HASH` | `0x15e25447f2cb376d8b0f5c72a2f8053c293dbe32dd1c3a354ce1fdb6dc76dfb4` |
| `EMPTY_CODE_HASH` | `0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470` |
| Historical wallet storage | `m_required` at slot `0`; `m_numOwners` at `1`; `m_owners[i]` at `5 + i`, with indices `1..N`; `m_ownerIndex[uint(owner)]` at mapping slot `261` — **provisional, requires bytecode/storage audit** |
| `MAX_OWNERS_V02` | 16 — deliberately restrictive candidate bound, not a fact about the Parity ecosystem |
| `PRE_STATE_ROOT` | **UNFILLED:** obtain and independently attest canonical block-header `stateRoot` at `PRE_BLOCK` |
| `POST_STATE_ROOT` | **UNFILLED:** obtain and independently attest canonical block-header `stateRoot` at `POST_BLOCK` |
| `PPCR_TX_TYPE` | **UNASSIGNED:** a fresh, non-conflicting EIP-2718 transaction type in `[0x00, 0x7f]` |
| Activation fork | **UNSCHEDULED** |

`RECOVERY_SLOT := keccak256(UTF8("ethereum.ppcr.parity2017.v0.2.recovery-marker"))` is a dedicated wallet-storage slot for the one-time marker. Proving this slot zero in the historical and current wallet state is mandatory. Clients would set it to the integer `1` on successful recovery. **This is a proposed, privileged storage update**, not behavior currently supported by EVM calls to the proxy. A storage-conflict and client-compatibility review is essential.

The complete **925-byte runtime** corresponding to `PROXY_CODE_HASH` must be published and audited byte-for-byte to establish that its execution paths actually depend on `LIBRARY`. A hash match alone is not a causal proof unless the matched code has been analyzed. The historical library's pre/post `codeHash` must likewise be independently verified against the two canonical roots, either as protocol constants justified by reproducible proofs or via a specified validation routine. Do not assume a researcher's CSV manifest is authoritative consensus input.

## 3. Historical state proof format and verification

Eligibility is proved against historical Ethereum Merkle-Patricia trie roots; it does **not** require a client to maintain an archive node or trust an off-chain scanner. A proposer supplies authenticated witness nodes alongside the recovery transaction.

For each historical account at block `B`, the verifier:

1. Computes the secure state-trie key `keccak256(address20)`.
2. Validates a canonical Merkle-Patricia account proof against the fixed block `stateRoot`.
3. Decodes the canonical Ethereum account RLP `[nonce, balance, storageRoot, codeHash]`.
4. For each required slot `s`, verifies a storage-trie proof with key `keccak256(uint256_to_32_bytes_be(s))`, decoding the RLP-encoded storage integer (absence means zero).
5. Rejects malformed RLP, invalid trie branches, unused or inconsistent proof nodes, missing required values, integer overflow, and ambiguous duplicate witness entries.

The owner-index mapping slot is computed using Solidity's fixed-key mapping storage convention:

`ownerIndexSlot(a) = uint256(keccak256(uint256_to_32_bytes_be(uint160(a)) || uint256_to_32_bytes_be(261)))`.

The draft's canonical witness grouping is:

```text
HistoricalWitness {
    pre_wallet_account_proof:  [RLP trie nodes],
    post_wallet_account_proof: [RLP trie nodes],
    pre_slot_proofs:          [(uint256 slot, [RLP trie nodes])],
}
```

`pre_slot_proofs` must cover slots `0`, `1`, `6..5+N`, `RECOVERY_SLOT`, and the derived mapping slots for each of the `N` listed historical owners. Slot entries must appear in **strictly ascending numerical order** without duplicates. All trie nodes are listed in root-to-leaf order. Node-size/aggregate-proof-byte bounds and whether shared proof-node deduplication is permitted are **unfinalized**; no claim is made that the format is production-feasible yet.

**Historical library commitment check:** independently establish from witnesses anchored to `PRE_STATE_ROOT` and `POST_STATE_ROOT` that `codeHash(LIBRARY, PRE_BLOCK) = PRE_LIBRARY_CODE_HASH` and `codeHash(LIBRARY, POST_BLOCK) = EMPTY_CODE_HASH`. This may be a separately published conformance fixture rather than repeated in each recovery transaction, but the normative anchor values and validation procedure must be finalized before EIP submission.

## 4. Exact candidate eligibility predicate (v0.2)

Given a proposed wallet address `W`, historical witness `H`, and the state `S` *immediately before* this transaction, `EligibleV02(W,H,S)` is **true if and only if** each test below passes:

1. `chainId == 1`; the PPCR activation fork is active; `W` is a 20-byte address, not zero and not `LIBRARY`.
2. The `PRE_BLOCK` account proof for `W` is valid against `PRE_STATE_ROOT` and has `codeHash == PROXY_CODE_HASH`.
3. The `POST_BLOCK` account proof for `W` is valid against `POST_STATE_ROOT` and also has `codeHash == PROXY_CODE_HASH`.
4. The *current* account at `W` exists and has `codeHash == PROXY_CODE_HASH`; the wallet is not recreated with different code.
5. The wallet's pre-incident storage reads as `T = slot[0]` and `N = slot[1]`, satisfying `1 <= T <= N <= MAX_OWNERS_V02`.
6. For **every** `i = 1..N`, `A_i = address(uint160(slot[5+i]))` is nonzero; the original 256-bit word has no bits set above the low 160 bits; the addresses `A_i` are pairwise distinct; and `slot[ownerIndexSlot(A_i)] == i`.
7. Current wallet storage slots `0`, `1`, `6..5+N`, and `ownerIndexSlot(A_i)` all equal their authenticated `PRE_BLOCK` values. Any observed change **fails closed**, even if legitimate; this restriction may exclude wallets that changed ownership during block 4,501,969.
8. The historic and current storage word at `RECOVERY_SLOT` is zero. This slot must be shown not to conflict with the audited wallet's expected storage layout.
9. The wallet has a **positive current ETH balance**; `transaction.amount == balance_S(W)`; the recipient `R` is nonzero and `R != W`.
10. The recipient is explicitly authorized by at least `T` unique EOA owner signatures as specified in Section 5. A verified signature is counted only if its recovered address is one of `A_1..A_N` and its historical *and current* `m_ownerIndex` value is exactly the array index of that owner.

Any failure returns **ineligible**. The historical ETH balance is not an eligibility condition: a wallet with zero ETH in 2017 may have current ETH, but the 2017 proxy/ownership proofs and all other restrictions still apply. **Token-only wallets cannot be swept by this ETH-only transition.**

This is a conservative *subset rule*, not a claim to enumerate all affected wallets. It imposes additional restrictions beyond historical Parity behavior and may exclude genuine owners, especially where the historic owner array had gaps, owner slots were reorganized, contract-based owners were used, or ownership changed after the reference snapshot.

## 5. Cryptographic owner authorization

The transaction carries EIP-712 signed structured data. To prevent confusion with generic account recovery, the domain is fixed as follows:

```text
EIP712Domain(
    string name,       // "PPCR-Parity2017"
    string version,    // "0.2"
    uint256 chainId,   // 1
    bytes32 salt       // keccak256(UTF8("PPCR/ETH-mainnet/4501968/4501969/v0.2"))
)

PPCRWithdrawal(
    address wallet,
    address recipient,
    uint256 amount,
    uint256 walletNonce,
    uint256 expiryBlock,
    address sponsor
)
```

The signing digest is **exactly** `keccak256(0x19 || 0x01 || domainSeparator || hashStruct(PPCRWithdrawal))` using EIP-712 encoding and type hashes for the declarations above. No `personal_sign` prefix is used. `walletNonce` must equal the wallet account's current consensus nonce, preventing authorization intended for a different account state. `expiryBlock` must satisfy `currentBlock <= expiryBlock`; it must not be zero. `sponsor` must equal the recovered transaction fee payer. `recipient`, `amount`, `wallet`, `walletNonce`, `expiryBlock`, and `sponsor` are all bound by *every* owner's signature.

The ordered signature list contains tuples `(yParity, r, s)` where `yParity` is `0` or `1`; `1 <= r < secp256k1_n`; `1 <= s <= secp256k1_n / 2` (integer floor); recovered addresses are nonzero and **strictly increasing numerically**; a duplicate or malformed signature invalidates the transaction. Accept at most `N` signatures and require `count >= T`.

A counted signer must be an actual owner under the historical and current storage proofs, and must be an **EOA** (the signer has empty code at the reference state and at transaction execution). **EIP-1271 contract signatures, EIP-7702 delegated-code signers, BLS signatures, and threshold/contract signer adapters are not supported in v0.2.** They require separate security and compatibility analysis. A historical contract owner cannot be silently converted into an EOA signature.

Signers authorize the **specific recipient and full present wallet balance**, not a change in owner configuration, and not a reusable permission. The sender signing the transaction is the independent gas sponsor; it does not inherit any wallet-owner authority.

**Important:** The exact design is proposed cryptography. D3I.15's address-impersonation tests are not ECDSA signature-verification tests.

## 6. Candidate EIP-2718 typed transaction envelope

A new transaction type is proposed so an ordinary EOA **sponsor** pays gas while the locked proxy is the **balance source**. Standard legacy/EIP-1559 transaction signing from the proxy itself is not possible because it is a contract account; EIP-3607 normally rejects a transaction sender with deployed code. The sponsor is therefore recovered from a distinct outer transaction signature.

After assignment of `PPCR_TX_TYPE`, use the envelope:

```text
PPCR_TX_TYPE || rlp([
    chainId,
    sponsorNonce,
    maxPriorityFeePerGas,
    maxFeePerGas,
    gasLimit,
    wallet,
    recipient,
    amount,
    walletNonce,
    expiryBlock,
    historicalWitness,
    ownerSignatures,
    sponsorYParity,
    sponsorR,
    sponsorS
])
```

The sponsor signs `keccak256(PPCR_TX_TYPE || rlp(fields preceding sponsorYParity/sponsorR/sponsorS))`. The recovered sponsor must have empty code, the correct `sponsorNonce`, sufficient ETH for the maximum gas liability, and valid low-`s` secp256k1 signature. The transaction fee follows EIP-1559 base-fee/priority-fee semantics and the ordinary sender-nonce increment; the **wallet ETH must never pay gas**. No `to`, EVM calldata, contract creation, access list, value transfer by the sponsor, or blob fields are defined for this transaction type.

This is an **interface proposal**, not an assigned transaction type. Final RLP nesting and canonical historic-witness encoding, max witness size, intrinsic gas, proof-validation metering, sponsor fee settlement, typed receipt encoding, mempool rules, and interaction with later transaction standards remain **hard blockers** for a complete EIP.

## 7. Consensus transition: successful PPCR withdrawal

All account reads and proofs are evaluated against state `S` **immediately before execution** in block order. If any eligibility, signature, nonce, proof, fee, expiry, or balance check fails, the transaction is *invalid for block inclusion*, not a partially successful EVM call.

For a valid recovery transaction:

1. Validate `EligibleV02(wallet, witness, S)` and Section 5 signatures.
2. Charge the sponsor's transaction fee, burn the base-fee share, credit the block fee recipient with the priority-fee share, and increment the sponsor's account nonce, according to the finalized transaction fee rules.
3. Debit exactly `amount` wei from the wallet account.
4. Credit exactly `amount` wei to `recipient`, **without executing recipient code**. Treat this as a protocol balance update, not an EVM `CALL`, `DELEGATECALL`, or `SELFDESTRUCT`.
5. Write the storage integer `1` at `RECOVERY_SLOT` of the wallet. Do **not** change the wallet's code, other storage slots, or wallet account nonce.
6. Emit a successful typed receipt with **no EVM logs**. The transaction hash and normal receipt are the evidence of the state change; a separate event contract is not invented.

The transition must be **atomic**. The wallet debit equals the recipient credit. The sponsor's ordinary gas fee is accounted for separately; no ETH is minted by PPCR. A second attempt necessarily fails because the recovery marker is no longer zero. Any ETH sent to the abandoned wallet **after** its single sweep remains outside this v0.2 mechanism.

The one-time storage marker deliberately limits the exception. However, modifying a legacy contract's storage without executing its code is itself a notable protocol precedent and must be explicitly debated.

## 8. Verification pseudocode

```python
# SPECIFICATION SKETCH ONLY; NOT EXECUTABLE CONSENSUS CODE
# verify_mpt_account / verify_mpt_storage use canonical RLP and Ethereum
# secure Merkle-Patricia trie rules. Unknown constants must be finalized.

def validate_ppcr_v02(tx, state, block_number):
    assert CHAIN_ID == 1 and fork_active(block_number)
    assert tx.type == PPCR_TX_TYPE  # unassigned
    sponsor = recover_outer_sponsor(tx)
    assert valid_eoa_sponsor(sponsor, state, tx)
    assert 0 < tx.expiryBlock and block_number <= tx.expiryBlock

    pre = verify_mpt_account(PRE_STATE_ROOT, tx.wallet, tx.witness.pre)
    post = verify_mpt_account(POST_STATE_ROOT, tx.wallet, tx.witness.post)
    assert pre.codeHash == post.codeHash == PROXY_CODE_HASH
    assert state.code_hash(tx.wallet) == PROXY_CODE_HASH

    T = historical_slot(pre.storageRoot, 0, tx.witness)
    N = historical_slot(pre.storageRoot, 1, tx.witness)
    assert 1 <= T <= N <= 16
    owners = []
    for i in range(1, N + 1):
        word = historical_slot(pre.storageRoot, 5 + i, tx.witness)
        assert word != 0 and word < (1 << 160)
        owner = address(word)
        assert owner not in owners
        assert historical_slot(pre.storageRoot, owner_index_slot(owner), tx.witness) == i
        assert state.storage(tx.wallet, owner_index_slot(owner)) == i
        assert state.storage(tx.wallet, 5 + i) == word
        owners.append(owner)
    assert state.storage(tx.wallet, 0) == T
    assert state.storage(tx.wallet, 1) == N
    assert historical_slot(pre.storageRoot, RECOVERY_SLOT, tx.witness) == 0
    assert state.storage(tx.wallet, RECOVERY_SLOT) == 0

    assert tx.walletNonce == state.account_nonce(tx.wallet)
    assert tx.amount > 0 and tx.amount == state.balance(tx.wallet)
    assert tx.recipient != ZERO_ADDRESS and tx.recipient != tx.wallet
    digest = eip712_ppcr_digest(tx, sponsor)
    signers = recover_strictly_sorted_low_s_signers(digest, tx.ownerSignatures)
    assert T <= len(signers) <= N
    for signer in signers:
        assert signer in owners
        assert is_eoa_at_pre_and_current(signer, tx.witness, state)
        assert state.storage(tx.wallet, owner_index_slot(signer)) == owners.index(signer) + 1

    # Additional canonical proof encoding, fee, and block-gas rules: TBD.
    return (sponsor, tx.wallet, tx.recipient, tx.amount)


def apply_ppcr_v02(validated, state):
    sponsor, wallet, recipient, value = validated
    # Fee and sponsor nonce handled by finalized typed-transaction rules.
    state.balance[wallet] -= value
    state.balance[recipient] += value  # no EVM call
    state.storage[wallet, RECOVERY_SLOT] = 1
    return SUCCESS_RECEIPT_WITH_EMPTY_LOGS
```

**Implementation note:** Owner-account code status at the historical snapshot also requires authenticated account proofs for counted signers. These additional witnesses are not yet represented in the prototype `HistoricalWitness` field list above; the schema and proof bounds **must be expanded** before implementation. This deliberate outstanding issue prevents readers from mistaking the pseudocode for executable consensus code.

## 9. Invariants and security model

- **Closed historical identity:** No alternate proxy hash, library address, reference block, or externally curated list can qualify a wallet.
- **Conservative authorizer rule:** Every counted signer is present in the array *and* active in the original owner-index mapping in both historical and current state; all signers are distinct.
- **Exact approval:** An approval binds source, recipient, amount, wallet nonce, expiry, chain, domain, and sponsor.
- **ETH conservation:** Wallet debit = recipient credit; normal sponsor gas fees remain separate.
- **No general capability:** No contract call, code restore, arbitrary wallet slot write, token transfer, or administrator-selected recipient.
- **Replay prevention:** `RECOVERY_SLOT == 0` is mandatory before execution and permanently becomes `1`.
- **Atomicity:** Invalid proof/signature/balance never produces a partial wallet transfer.
- **Chain isolation:** Historical roots, transaction type, EIP-712 domain, and network are mainnet-specific.
- **History preservation:** Historical blocks, transactions, receipts, code, and ownership are not rewritten.

**Adversarial considerations:** stolen historical keys; obsolete ownership policies; compromised EOAs; contract owners whose authorization cannot be expressed as ECDSA; unexpected storage layouts; incomplete enumeration; gas-denial by oversized proof witnesses; signature malleability; transaction or signature replay; mismatched client trie/RLP validation; beneficiary accounts with code; future deposited ETH; global precedent for bypassing EVM contract restrictions. A historically valid ECDSA key does not by itself establish the present legal entitlement of the signer.

## 10. Research evidence supporting (and not supporting) the design

The published D3I.10J scan examined 227,740 unique blocks through 4,501,969, recorded 782,601 deduplicated creation-attempt records, identified **284** unique targeted proxy-bytecode matches, and flagged **945** unresolved creation events. D3I.11 reported **93** candidates with nonzero ETH at the reference snapshot, totaling **443,163.039443434519138007 ETH**. Those are *historical observations*, **not** a claim that this amount remains locked, is eligible, or can be recovered.

D3I.12 reconstructed a historical **4-of-7** configuration for `0x3bfc20f0b9afcace800d73d2191166ff16540258` and **2-of-3** for `0x376c3e5547c68bc26240d8dcc6729fff665a4448`. D3I.13–14 traced library-dependent execution and local counterfactual restoration. D3I.15 v1.2 then demonstrated 1-wei threshold-controlled transfers on two Anvil forks when historical library code was available, but not in the post-destruction condition. The simulations used address impersonation, **not real private-key signatures**, and did **not** test the novel typed transaction, the historical MPT witnesses, or the privileged balance/storage transition described here.

Source SHA-256 audit references:

- Candidate manifest CSV: `3F9A3468DA8014A442BED14A93A0FA7A9FA954107EFDD7881BD0270EB9061A44`
- D3I.11 JSON: `B9458B4BD6A67A24D14480FF1743724EC7F04F8829C7BC4248E91F26FED510FB`
- D3I.12 JSON: `98E63E6C8905B6BB652E5A9CB95427B559430B9DF1BC27B079577E1E2E278834`
- D3I.15 v1.2 JSON: `E911D46D670CDE6A00CCE0ADD9D127E7B4EADEA43CB16D4D3B7150721CABF35A`

**Completeness warning:** This design is predicate-based, not manifest-based; it may prove some wallets eligible without resolving every scanner event. That does **not** remove the need to assess historical coverage, false negatives, contract variants, and the broader affected population before presenting a credible impact estimate.

## 11. Test matrix required before claiming implementation readiness

| ID | Test | Expected result |
|---|---|---|
| P01 | Historical proxy with verified 4-of-7, 4 genuine EOA signatures, exact full balance | Accept (if all other conditions pass) |
| P02 | Verified 2-of-3, 2 genuine EOA signatures | Accept (if all other conditions pass) |
| N01 | Correct proxy code hash but missing historical owner-index proof | Reject |
| N02 | Invalid `PRE_STATE_ROOT` or `POST_STATE_ROOT` Merkle proof | Reject |
| N03 | Wrong 2017 proxy code hash, or replaced current runtime | Reject |
| N04 | Threshold minus one signatures | Reject |
| N05 | Duplicate, unsorted, high-`s`, or malformed signatures | Reject |
| N06 | Historical owner present in array but not active in `m_ownerIndex` | Reject |
| N07 | Historical owner was replaced or threshold changed since reference | Reject |
| N08 | Mismatched recipient, amount, sponsor, wallet, chain, or wallet nonce | Reject |
| N09 | Expired signature | Reject |
| N10 | Second recovery after marker becomes `1` | Reject |
| N11 | Amount smaller or larger than current full wallet balance | Reject |
| N12 | Recipient is wallet or zero address | Reject |
| N13 | Owner signs using an EIP-1271 contract or delegated code | Reject in v0.2 |
| N14 | Non-owner EOA signs | Reject |
| N15 | Sponsor cannot afford gas or has wrong account nonce | Reject |
| N16 | Invalid proof exceeds bounded transaction limits | Reject |
| C01 | ETH total across wallet and recipient | Unchanged by PPCR balance move |
| C02 | Recipient is a contract | Credit balance but do **not** execute code |
| C03 | Two valid recoveries for same wallet in one block | First succeeds, second invalid |
| C04 | Signatures and historical witnesses across at least two independently developed clients | Identical state roots and receipts |

These are **required future vectors**, not presently passing tests.

## 12. Mandatory open decisions before a formal EIP PR

1. **Historical root attestation:** Obtain canonical headers/roots for 4,501,968 and 4,501,969, validate the library proofs, publish full proof fixtures, and freeze normative constants.
2. **Code and storage audit:** Prove the full 925-byte proxy's dependency and storage layout, including `m_ownerIndex` slot `261`, mapping keys, owner reorganization, and possible conflicts with `RECOVERY_SLOT`.
3. **Witness completeness:** Add historical owner account/code proofs, canonical proof-byte encoding, proof-length and owner-count limits; measure worst-case witness size.
4. **Typed-transaction specification:** Assign a collision-free EIP-2718 type and finalize exact RLP, intrinsic gas, proof costs, maximum gas usage, fee settlement, receipts, and peer-to-peer handling.
5. **Actual signature trials:** Obtain only authorized, voluntary test signatures from controlled keys; demonstrate EIP-712 verification, threshold negatives, and replay rejection. Never request owners' private keys.
6. **Eligibility expansion:** Run D3I.16 on the remaining ETH-funded candidates and investigate 945 unresolved creation events; characterize intentional false negatives of the conservative predicate.
7. **Execution-spec tests:** Implement this *new* state transition in a reference execution specification and then in at least two independent execution clients for cross-client comparison.
8. **Economic and governance legitimacy:** Compare against EIP-999, explain the exceptional precedent, examine alternatives without protocol state surgery, and seek candid Ethereum Magicians and execution-client feedback.

## 13. Relationship to EIP-999 and other recovery work

[EIP-999](https://eips.ethereum.org/EIPS/eip-999) proposed restoring code at the historical Parity library address and was withdrawn. PPCR v0.2 instead proposes a one-time, opt-in, threshold-approved ETH balance move. It is narrower in capability, but arguably introduces an even more explicit precedent: **consensus rules overriding ordinary contract-enforced access to ETH**. It is not automatically preferable to code restoration and must stand on its own security and governance argument.

Account-level timelock recovery mechanisms for modern smart accounts, such as [KoinLabs Timelock Account Recovery](https://github.com/koinlabs/Timelock-account-recovery), solve a different class of problem and do not supply a general justification for changing Ethereum's consensus state transition for 2017 legacy contracts.

## 14. Backwards compatibility

This would require a hard fork and coordinated execution-client adoption. All non-PPCR transactions should retain existing semantics. The targeted wallets would change balances and one reserved storage key **only when owners and a sponsor submit a valid new transaction**. Historical blocks remain untouched. Future ETH deposits after a one-time sweep cannot be recovered using v0.2; contract-based owner signatures and tokens remain unsupported. Exact handling of account-cleanup, witness metering, future EIP interactions, and system-state accounting must be verified across clients.

## 15. Conclusion

PPCR v0.2 is a **specific research candidate**: a deterministic historical proof, genuinely signed owner-threshold approval, and a tightly constrained one-time ETH sweep without restoring the original Parity library. This makes the debate more technically precise but does not establish network feasibility or social acceptance. The first claim requiring validation is **not** “we can recover 443,163 ETH”; it is “independent clients can recognize the same limited set of wallets and safely apply exactly the same authorized state transition.”

## References

- [EIP-1 — EIP Purpose and Guidelines](https://eips.ethereum.org/EIPS/eip-1)
- [EIP-2718 — Typed Transaction Envelope](https://eips.ethereum.org/EIPS/eip-2718)
- [EIP-712 — Typed Structured Data Hashing and Signing](https://eips.ethereum.org/EIPS/eip-712)
- [EIP-3607 — Reject Transactions from Senders with Deployed Code](https://eips.ethereum.org/EIPS/eip-3607)
- [EIP-999 — Restore the Parity WalletLibrary](https://eips.ethereum.org/EIPS/eip-999)
- [Parity Technologies — A Postmortem on the Parity Multi-Sig Library Self-Destruct](https://medium.com/paritytech/a-postmortem-on-the-parity-multi-sig-library-self-destruct-63daca3a4cf7)
- [Historical WalletLibrary source illustration](https://gist.github.com/muellerberndt/812987f947e2e56da622ba8599dbd9cb) — mapping and layout still require source-to-bytecode attestation
- [PPCR research repository](https://github.com/samst93/ethereum-ppcr)
- [Ethereum Magicians PPCR discussion](https://ethereum-magicians.org/t/research-draft-ppcr-parity-proxy-contract-recovery-protocol/30469)

*Document license for eventual EIP submission: CC0-1.0; confirm repository license and copyright compliance before copying third-party material.*
