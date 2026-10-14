<!-- solve-problem:10106821-6bf2-4c1d-afd1-8736819e2cf9 -->
# Weighted decryption committees under churn and bribery

A [Tangle Solve problem](https://solve.tangle.tools/problems/10106821-6bf2-4c1d-afd1-8736819e2cf9).

## What is the problem?

In encrypted-mempool protocols (e.g., threshold-encrypted order-flow auctions, fair-ordering systems like Shutter-style decrypting committees), a committee of n nodes holds shares of an epoch key; any threshold t of shares decrypts the batch of encrypted transactions. The current frontier is **weighted** threshold encryption: each committee member i has stake weight w_i, and the decryption threshold is expressed in total weight (e.g., any subset with weight ≥ W decrypts, no subset with weight < W can). The open problem asks to build a weighted batched threshold decryption scheme that simultaneously achieves:

1. **Churn / dynamism:** committee membership can change across epochs, with proactive key refresh (new shares, same public key or a cleanly rotated one), so that an adversary corrupting a weight-threshold of nodes *cumulatively over time* — never a weight-threshold within a single epoch — still cannot decrypt.
2. **Bribery resistance (early reveal / withholding):** a coalition holding weight just below the decryption threshold should not profitably be able to (a) decrypt early (before the commit point) and sell ordering information, or (b) withhold shares to force decryption failure, even when bribed by a party wanting to front-run or censor.
3. **Efficiency preserved:** retain the 2025 state of the art — quasilinear computation and communication complexity independent of the number of distinct weights.

The subtlety motivating the problem: **weights are meant to model trust/stake, but stake can be rehypothecated or split across pseudonyms, so "uncorrelated weights" is a fiction.** Security must be stated against adversaries with *correlated* control of weight, not just independent per-node corruption.

Formalization sketch: define an epoch game where the adversary adaptively corrupts nodes (possibly via bribery at per-unit-weight price), weights may be correlated via a hidden ownership graph, and security notions are (i) confidentiality of plaintexts until the reveal round, (ii) robustness/liveness of decryption, (iii) proactive security across epochs. Deliverable: a scheme with proofs under stated corruption and bribery models, plus matching or explicit communication/computation bounds.

## Why does it matter?

Encrypted mempools are the leading deployed defense against MEV (front-running, sandwiching), and threshold-decryption committees are their lynchpin; if a bribed near-threshold coalition can early-decrypt or withhold, the entire MEV protection collapses exactly when it's most profitable to break it. Weighted committees let real protocols use stake instead of one-person-one-node, but correlated ownership means naive weighting understates adversarial power. A solution unlocks safely rotating validator-set decryption committees for proof-of-stake chains and hybrid order-flow-auction designs; leaving it open means either unweighted committees (poor decentralization economics) or unproven weighted ones.

## What's already known?

- **Agarwal et al. (2025)** achieve weighted batched threshold encryption with quasilinear computation and weight-independent communication — this is the baseline the census record calls "beyond." It addresses the *static* weighted setting; treat it as the starting schema whose security game must be extended. (I have not verified the paper's exact theorems; read it first and update this brief's assumptions.)
- **Classical dynamic/proactive secret sharing** (Herzberg et al., Ostrovsky–Yung): share refresh defeats mobile adversaries, but in unweighted, small-committee settings with independent corruptions.
- **Bribery-related work:** threshold-signature/decryption bribery ("verifiable bribery," time-lock puzzles as anti-early-reveal, e.g., in Shutter-like designs and commit-reveal MEV literature) shows two standard partial mitigations: economic penalties via slashable collateral, and forcing decryption to occur at a fixed round so early reveal has no market. Neither is proven against *weighted coalitions with correlated ownership*.
- Known gap: no published scheme composes all three (weights, churn/refresh, bribery) with the 2025 efficiency guarantees, nor is there a lower bound showing the combination is hard.

## What would count as solving it?

1. **Construction:** a weighted batched threshold decryption protocol supporting epoch-based membership churn and share refresh, with (a) IND-CPA-style confidentiality until reveal under an adversary controlling any coalition of weight < W, where coalitions may be correlated across epochs up to a stated cumulative budget; (b) bribery-resistance formalized as: no bribing strategy with cost bounded by B yields expected advantage ε in early decryption or liveness failure — or an explicit impossibility/lower bound showing confidentiality and bribery-resistance trade off.
2. **Proofs:** reductions to standard assumptions in both an honest-majority-of-weight and a rational-adversary (game-theoretic equilibrium) model.
3. **Efficiency:** computation quasi-linear in total weight or committee size; communication independent of the weight distribution — matching or beating the 2025 baseline.
4. Acceptable weaker outputs: impossibility results separating dynamic from static weighted security; or a scheme secure only against independent corruptions plus a formal counterexample under correlated ownership.

## How to start

1. Read Agarwal et al. (2025) ePrint 2025/2115 carefully — extract the exact corruption model, weight semantics, and where quasilinearity comes from; identify which lemma breaks when corruptions are adaptive across epochs.
2. Write down the adversary model precisely: cumulative-weight corruption budget across epochs with refresh, and a bribery price function per unit weight. This formalization is half the problem and is genuinely missing.
3. Try the natural composition: 2025 weighted scheme + Herzberg-style share refresh. Expect the obstacle that refresh with weights requires re-randomizing weighted shares without a trusted dealer, and that correlated ownership makes "weight < W" an overestimate of the adversary's cost.
4. For bribery, consider combining a fixed decrypt-round (delayed decryption so early reveal is worthless) with collateral slashing; formalize as a mechanism-design layer atop the cryptographic layer.

## Key references

- Agarwal et al., "Weighted Batched Threshold Encryption with Applications to Mempool Privacy," IACR ePrint 2025/2115 — the baseline scheme achieving quasilinear computation and weight-independent communication for static weighted committees; your starting point. https://eprint.iacr.org/2025/2115 (verify)
- Herzberg, Jakobsson, Jawurek, Krawczyk, Rabin, "Proactive Secret Sharing or: How to Cope with Perpetual Leakage" (CRYPTO'95 / journal version 2015) — the foundational share-refresh technique for mobile adversaries you must adapt to weights. (verify)
- Ostrovsky & Yung, "How to Withstand Mobile Virus Attacks" (PODC 1991) — origin of proactive security against mobile adversaries.
- Shutter Network documentation / related threshold-encryption mempool papers — deployed context for fixed-round decryption as an anti-early-reveal heuristic. (verify exact citation)

## Warnings

- The census record is a one-liner; the "problem" is under-specified, so any solution must first pin down the adversary model (corruption budget, bribery pricing, correlation structure of weights) — referees will attack exactly this.
- "Weight models trust" is the trap: explicitly model correlated/split stake (Sybil/rehypothecation), or your threshold is vacuous.
- Early-reveal bribery can be *mitigated* by making decryption deterministic-round (nothing to sell early) — don't claim novelty for that alone; the withholding side is the harder, unsolved half.
- Do not assume proactive refresh composes with weighted thresholds for free; weighted share redistribution likely needs a weighted dealer/DKG redesign.
- I could not verify the contents of ePrint 2025/2115 or whether the Agarwal et al. results are peer-reviewed; confirm before building on stated bounds.

---
*`BCGT-000319` · flagged for review*

## Acceptance guidance

1. **Construction:** a weighted batched threshold decryption protocol supporting epoch-based membership churn and share refresh, with (a) IND-CPA-style confidentiality until reveal under an adversary controlling any coalition of weight < W, where coalitions may be correlated across epochs up to a stated cumulative budget; (b) bribery-resistance formalized as: no bribing strategy with cost bounded by B yields expected advantage ε in early decryption or liveness failure — or an explicit impossibility/lower bound showing confidentiality and bribery-resistance trade off.
2. **Proofs:** reductions to standard assumptions in both an honest-majority-of-weight and a rational-adversary (game-theoretic equilibrium) model.
3. **Efficiency:** computation quasi-linear in total weight or committee size; communication independent of the weight distribution — matching or beating the 2025 baseline.
4. Acceptable weaker outputs: impossibility results separating dynamic from static weighted security; or a scheme secure only against independent corruptions plus a formal counterexample under correlated ownership.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/10106821-6bf2-4c1d-afd1-8736819e2cf9?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000319.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000319)
