# Which rules protect this payment?

**Status.** This is reviewer documentation for Bounty 02. Codex developed and tested a private prototype on the contributor’s behalf before maintainer approval. The proposal asks Tau to decide eligibility and review the node and native API changes. The proposed commands below require that prototype. The upstream examples use public revision `c2b8c47`. [Bounty 02 and review discussion](https://github.com/IDNI/tau-testnet/issues/42)

## What an upstream wallet can check today

Set `SENDER` to the account’s 96-hex public key and `INSTALL_TX` to the 64-hex transaction hash. These lookups require no signing key.

```sh
tau-testnet rpc "getruleoffers $SENDER"
tau-testnet rpc "gettxstatus $INSTALL_TX"
```

Read `accepted_clauses` for the registered outgoing clause and `rule_change_guard` for the guard on future edits. Pending offers and mempool entries appear separately. A confirmed transaction status identifies its block and confirmation count. These commands are separate observations, so their answers can span different heads. The submitted application-rule text can also contain conditions superseded by a later revision. [Account registry RPC](https://github.com/IDNI/tau-testnet/blob/c2b8c47e07342dee01ac27f1380872374e3bf6ff/commands/getruleoffers.py) · [Transaction status RPC](https://github.com/IDNI/tau-testnet/blob/c2b8c47e07342dee01ac27f1380872374e3bf6ff/commands/gettxstatus.py)

## Verify an installation in the proposed interface

```sh
tau-testnet rpc "getactiverules $SENDER $RECIPIENT --tx $INSTALL_TX"
```

The response names the genesis, block hash and height used for both accounts and the receipt. For an outgoing spending limit, find the receipt effect for the sender’s account and target stream 5. Compare its `new_body_sha256` with `sender.outbound.body_sha256`. A match verifies that the recorded installation produced the clause currently registered at that head. Different digests tell the wallet the installed clause has since been replaced. A null new digest records removal. A confirmed older transaction can return `not_recorded` when execution evidence is missing.

An account’s outgoing limit applies when that account spends. The incoming role reports `unsupported` on the baseline protocol. With the separate Bounty 03 enforcer and committed activation, the recipient’s incoming role also exposes its clause and maximum hold duration. For an open retractable transfer, use `--payment $HOLD_ID` to check the parties, amount and release height or time together. Use `checktx` on a concrete transaction to include its actual inputs in a preflight. [Upstream preflight command](https://github.com/IDNI/tau-testnet/blob/c2b8c47e07342dee01ac27f1380872374e3bf6ff/commands/checktx.py)

## Read the shared rules

Take `BLOCK_HASH` from the account lookup. If the shared view is building, poll for that block. When it is ready, use its `CONTENT_ID` to fetch the first page.

```sh
tau-testnet rpc "getrulesnapshot $BLOCK_HASH status"
tau-testnet rpc "getrulesnapshot $BLOCK_HASH $CONTENT_ID 0"
```

Follow `next_offset` until it is null, concatenate the decoded `data_base64` bytes in order, then verify their SHA-256 against the content ID. On `STALE_VIEW`, start a fresh account lookup. A retained snapshot can be read after its chain extends. A branch switch can invalidate it.

With the proposed native binding, `ordered_alternatives` means every specification part is included. Within each part, Tau tries the formulas in order and executes the first solvable alternative. An older binding yields `selected_step_only` and `parts = null`. Its printed formula can include a disjunction even after a step. Show the shared view as partial in this case. [Pinned native interpreter](https://github.com/IDNI/tau-lang/blob/c3ea334cd1e6612930ea9f404933211858f0009f/src/interpreter.h#L312) · [Existing Python API implementation](https://github.com/IDNI/tau-lang/blob/c3ea334cd1e6612930ea9f404933211858f0009f/src/api.tmpl.string.h#L524)

## Operator checks from the private test

Two disposable local nodes used the patched native binding. A proposer mined a signed outgoing rule and charged a fee of 10 test units. A follower imported the block through its normal block-ingestion code. Both returned the same clause digest, confirmed effect and shared rules. After restart, the follower reproduced the same receipt and shared document. A second run with recipient controls active also verified both registry effects and a 17-block hold cap. These tests exercised wallet TCP and block ingestion. The harness delivered the block locally.

For a proposed deployment, record the genesis and block hashes before restarting, then repeat the lookup and compare the receipt and snapshot content ID at that same block. If shared reconstruction reports `unavailable`, inspect the node log for the block hash and check the installed native binding, program baseline and committed journal. The account registry and confirmed receipt remain independently readable.
