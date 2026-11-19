# Reading a conditional grant and the rules for its next payment

Alice funded a 5,000-unit grant for Bob. Bob wants to pay a merchant that accepts incoming payments only from him. On a disposable private node, Codex combined the programmable-payments and active-rules prototypes to check which restrictions govern the draw. The CLI used the node’s TCP interface. A validator mined signed transactions with the native Tau engine. Both critical designs still require Tau’s approval. This describes the private integration and gives reviewers a reproducible reading procedure. [Programmable-payments design](programmable-payments.md) · [Active-rules design](active-rules.md) · [Signed demonstration evidence](design-evidence.md)

## Read one canonical head

Set `ALICE`, `BOB` and `MERCHANT` to their full public keys and `GRANT` to the funding instruction ID. Point all commands at the same node. These commands print server JSON.

```sh
tau-testnet rpc "getactiverules $ALICE $BOB" > funding-rules.json
tau-testnet --json payments show "$GRANT" > grant.json
ALLOWANCE=$(jq -r '.data.instruction.terms.allowance_id // empty' grant.json)
if [ -n "$ALLOWANCE" ]; then
  tau-testnet --json payments show "$ALLOWANCE" > allowance.json
fi
tau-testnet rpc "getactiverules $BOB $MERCHANT" > draw-rules.json
```

Check each response’s `status`. In both rules responses, the chain is `data.head.genesis_hash` and the block is `data.head.block_hash`. In each payment response, those fields are `data.genesis_hash` and `data.head_hash`. Require one matching genesis and block hash across every result. An empty block mined between two reads produced different hashes during the test. Discard the partial set and repeat all reads on a mismatch or stale view. These account and grant queries read the current tip.

In `grant.json`, check that `data.instruction.kind` is `grant`, `id` is the requested ID, `owner` is Alice and `terms.recipient` is Bob. A referenced allowance must have kind `allowance` and the same parties. Read the grant’s terms, `remaining`, `period_id` and `spent` alongside the allowance’s own period, cap and meter. Interpret each meter in its recorded period. The node resets it on the first successful draw in a new period.

At height six the grant held 4,600. Its weekly cap was 900 and Bob had drawn 400. The separate allowance had a weekly cap of 500 with 400 spent, leaving **100** available under those two caps. Bob’s outgoing rule and the merchant’s incoming rule govern this proposed draw. The merchant’s registered clause appeared under `data.recipient.inbound.clause`. If Bob has an isolated account spend policy, also read `payments list "$BOB"` at the matching head for that policy and `data.spend_totals`; follow `next_after_id` at the same head if needed. Shared Tau rules have their own coverage status. For a building view, poll `getrulesnapshot BLOCK_HASH status` and check the reported coverage. This run reached `ready` with `selected_step_only`, which covers the interpreter’s observed selection at that step. The active-rules `--payment` option currently looks up retractable transfers. The tested grant returned `not_open` there while `payments show` returned its live restrictions.

## Preflight the actual draw

These commands sign a draft locally, then ask the node to evaluate its policies and fee. The signing step uses Bob’s wallet key.

```sh
jq -n --arg id "$GRANT" --arg to "$MERCHANT" --slurpfile g grant.json \
  '{action:"grant_spend",id:$id,version:$g[0].data.instruction.version,
    destination:$to,amount:100}' > draw.json
GENESIS=$(jq -r .data.genesis_hash grant.json)
tau-testnet --json payments submit --key bob --genesis "$GENESIS" \
  --file draw.json --fee 100 --dry-run > signed.json
tau-testnet rpc "checktx estimate $(jq -c . signed.json)"
```

In the test, a 300-unit draft initially returned `admissible: true`, `tau_evaluated: true`, `fee_checked: true` and `estimated_fee: "10"`. Escrow remained 5,000. A competing 400-unit draw then confirmed, consuming Bob’s nonce. Retrying the old signature returned `INVALID_SEQUENCE`. A freshly signed 101-unit draw returned `PAYMENT_PERIOD_CAP`. A fresh draft for 100 passed. The estimate identifies the candidate height and timestamp under `data.payment`. Its response omits the base block hash. Read the head again after estimating and repeat if it advanced. Escrow and Bob’s sequence remain available to other transactions until inclusion. Refresh the head, terms and sequence before signing a retry.

Operators can inspect activation with `payments status`. Refunds settle at the first eligible block. After a reorg, reacquire the canonical head and check `payments history "$GRANT"` for the refund on that branch. The separate nine-block demonstration records the refund disappearing on a sibling branch and returning when the original branch was restored. [Demonstration ledger](design-evidence.md)
