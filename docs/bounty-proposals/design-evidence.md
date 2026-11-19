# Evidence for the critical bounty designs

Both private prototypes and their tests predate maintainer approval. The payment prototype covered later stages before their prerequisites merged, and work on auto-split overlapped with recipient controls. The results below come from private test networks using disposable keys. Tau’s review is requested on the designs, merge order and public testnet activation. [Bounty program](https://tau.net/bounty-program)

## Programmable payments

Alice starts with 10,000 units and funds a 5,000-unit grant. Bob can draw up to 500 per Monday-based UTC week from December 1 through December 13, 2026. The shop routes 20% of incoming payments to savings. Separately, Alice authorizes monthly rent of 101. Bob acts as the keeper and pays its execution fee. The node charges 10 units for each accepted fee-bearing transaction in this example.

| UTC date | What the two ledgers must show |
| --- | --- |
| November 30 | Grant funding reduces Alice’s spendable balance. An early draw is refused. All 5,000 remain in grant escrow. |
| December 1 | The rent credits 81 to the shop and 20 to savings. A second execution for December fails. Draws of 300 and 200 succeed. A further unit is refused. The grant holds 4,500. |
| December 7 | A draw of 500 succeeds in the new week. The grant holds 4,000. |
| December 14 | The expiry block returns 4,000 to Alice without another fee. The escrow is empty. A signed sibling block one second before expiry leaves the 4,000 escrowed. Selecting either sibling changes both the balance and the refund history. |

The validator and an independent importer agreed on the nine-block chain, including the state hash, payment-book root, nonces and receipts after restart. Their final balances were Alice 8,879, Bob 9,960, shop 10,871, savings 10,220 and validator 70. A separate loopback test ran two node servers with separate databases. The follower disconnected before expiry, reconnected over libp2p to receive the refund block, and recovered the same state from disk while the validator was offline.

A native royalty rule paid 50 on a confirmed gross sale of 1,000. A funded escrow required an attestor. Signed-block tests check a refused final leg, an insufficient keeper fee and two competing claims. Failed settlement leaves the event available for a valid retry. A bounded in-memory sweep returned 172,032 units from 8,192 simultaneous expiries. Its nine timings ranged from 79.6 to 202.5 ms. End-to-end block cost at that population remains unmeasured.

The private wallet and operator guide covers fee estimation, standing orders, offline endorsements, grantee consent, feature activation, receipt lookup, expiry and restart. The condition stages depend on agreement about protected policy administration in #33. The five-stage prototype supports owner-plus-cosigner changes. In a separate private experiment, an owner preauthorized the exact text of a spend rule. A named delegate installed it, used its own sequence and paid its own fee while the owner’s key was offline. Revocation prevented reuse. That bounded permission covers one use case in #33. General delegation and recovery, and whether the condition stages meet their prerequisite, remain for review. [Five-stage design](programmable-payments.md) · [Protected wallet-policy issue](https://github.com/IDNI/tau-testnet/issues/33)

## Active rules

Consider a wallet that pays to install a rule and replaces it in the same block. The final account slot establishes the current protection. Two separate receipts establish what happened to each signed change, including the prior and resulting rule digests and the charged fee. A repeat or a refusal can be distinguished from a replacement.

For shared rules, the native test installs constraints corresponding to outputs 26 and 27 having values 7 and 9, then replaces only the constraint on output 26. The effective values are 3 and 9. A separate native interpreter regression checks a part with multiple alternatives whose priority matters. The older published binding returns a partial view that can still contain a disjunction after a step. A full answer requires the proposed ordered-alternatives accessor.

Two independent server processes observed the same accepted account-policy installation, 10-unit fee, clause digest and block. The follower reproduced the result after restart. A joint test with recipient controls also installed an outgoing clause and a 17-block incoming hold cap in one transaction. The proposer and importer agreed on both installed digests after restart. Additional tests cover a guarded change that never takes effect, reading during a concurrent canonical commit, orphaned pagination cursors and a historical transaction without recorded evidence. The wallet reports missing evidence explicitly. The operator guide describes the native build, retention, readiness and recovery. [Active-rule design](active-rules.md) · [Wallet capability requested in #42](https://github.com/IDNI/tau-testnet/issues/42)

[Read a grant and its shared allowance](reading-a-conditional-grant.md) · [Check a confirmed rule installation](checking-a-rule-installation.md)
