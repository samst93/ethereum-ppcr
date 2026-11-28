# PPCR research checkpoint 3 — V24.0B3F8

Scope: research evidence only. No activation or recovery authorization.

Reference block 4501968; discovery interval 4500969–4501968. The discovery report covers 1,000 blocks and the exclusion report covers 5,976 distinct discovered addresses: 5,973 runtime nonmatches, 3 absent accounts, no new exact matches. The original 585-wallet population remains unchanged (535 structurally consistent, 50 quarantined). The positive control is not counted twice.

Commitment: `f175f915f02c733eebdc74795ed1cf95c7fb029c6e984090bef12ed00669aa35`

Merkle root: `bb2de28c6f946f8210673ef6769db13a456f591ee7dded0b7814f7f8527a7849`

Python and Go reports record 6,561 membership proofs, 585 prior membership proofs and 32 negative tests each. The uploaded Go report is byte-identical to the packaged report. These are decoded-report reconciliation and Merkle membership verifications, not independent raw Ethereum trie replay. The F4A input and corrected F4B evidence archives preserve the exclusion proofs; F4C contains the offline replay implementation. Follow the READMEs inside the packages. Earlier original wallet raw proofs and raw historical trace archives are not bundled here and must be retained separately.

Canonicality not attested; completeness not established; authorization and eligibility not determined. Cross-provider extended validation deferred. Alternative proxy dependencies, full authorization and consensus equivalence remain unresolved. No claim of exhaustive affected-wallet discovery or recoverability. Keep activation disabled.

SHA256SUMS.txt pins all other files in this checkpoint. The manifest is not a signature. Proposed publication tag: v0.2-research-checkpoint.3. Publication is not performed by creating this archive.
