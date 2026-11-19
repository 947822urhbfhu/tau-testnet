## Bounty 02 · Which rules protect this account?

I began a private implementation before receiving maintainer approval. I understand that this can affect eligibility, and I’m asking whether you will consider the work and the design below. The proposal targets `tau-testnet` at `c2b8c47` and its pinned Tau engine at `c3ea334`.

### What a wallet can verify

A wallet can install a rule in the first transaction of a block and replace it in the last. Both transactions can succeed. I propose one query for the policy at a canonical chain tip and a receipt for what each transaction actually changed.

```text
getactiverules <sender> [recipient] [--payment <id>] [--tx <hash>]
getrulesnapshot <block_hash> status
getrulesnapshot <block_hash> <content_id> [byte_offset]
```

The first query returns the genesis hash, block hash, height and active consensus ID. For each account, it reads the outgoing clause and rule-change guard from the same registry used by execution, keyed by normalized public key and target stream. Each installed clause includes its stored UTF-8 body and SHA-256 digest. An empty enforced slot returns `absent`. On this protocol version the incoming role returns `unsupported`. On a node also running the Bounty 03 recipient policy, the query checks its committed activation and reads the recipient’s incoming clause, maximum hold duration in seconds or blocks, and policy digest. A recipient’s outgoing spending limit applies when that recipient spends. The payment lookup checks the parties and adds the amount and release point of an open retractable payment.

For example, Alice can request `getactiverules <Alice> <Bob> --tx <install-hash>`. If the receipt records an installation with digest X and the sender’s current clause has digest Y, the wallet can show “Installed in block 81. Replaced by block 82.” A pending offer or unexecuted guarded change supplies no installation receipt. The same canonical receipt is available from `gettxstatus`.

### Shared Tau rules after a partial replacement

Suppose the shared program receives `o26=7 && o27=9` followed by `o26=3`. Submitted text includes the superseded value 7. An effective view must expose the surviving 9 and replacement 3. There is also a case where one specification part retains several alternatives. The pinned C++ interpreter can report the last selected alternative or a disjunction before selection. Its existing Python `current_spec()` binding goes through `api::current_spec()`, which prints `current_spec_fm()` and can therefore return a disjunction even after a step. Different inputs can select a different alternative at the same revision. [Interpreter contract](https://github.com/IDNI/tau-lang/blob/c3ea334cd1e6612930ea9f404933211858f0009f/src/interpreter.h#L312) · [Native example with conditional alternatives](https://github.com/IDNI/tau-lang/blob/c3ea334cd1e6612930ea9f404933211858f0009f/tests/unit/test_interpreter_api_extensions.cpp#L427)

I request review of a read-only native accessor, `current_spec_parts()`. It returns each conjunctive specification part with its formulas in the priority order used by the interpreter. A separate `current_selected_spec()` binding exposes the C++ step observation with its time point. An older binding advertises `selected_step_only`, so a wallet can verify its registry slot but must treat the shared policy as partial. Full shared coverage requires building the accessor into the native dependency. Shared formulas include their representation plan and mapping for interned addresses. They describe shared constraints. A verdict for a concrete transfer still requires its inputs and execution history.

### Publication and consistency

- Capture registry replacements, neutral revocations, offers, guarded requests and releases, and native revision outcomes during execution. Discard effects from rejected candidates. Persist accepted effects with the block hash, transaction position, actual charged fee and bounded diagnostics in the same SQLite transaction as the canonical state. Join later releases by request ID and canonical ancestry. Return `not_recorded` for a confirmed historical transaction whose execution evidence is missing.
- Read both accounts, payment, receipt and shared-snapshot reference from one committed database view. Exclude intermediate fork replay. A separate reader reconstructs the journal using its committed representation and allocator mapping, verifies journal links, recorded step results, revision and time point, and renders the formulas. The read request never steps the live evaluator.
- Pin pages by block hash and SHA-256 of the full serialized document. Allow a wallet to finish a retained snapshot when the head extends. Reject cursors on an orphaned branch. A lagging reader reports its availability. A failed reconstruction leaves the registry and confirmed receipts readable.

These receipts are full-node observations. The existing block hash and state commitment continue to anchor consensus. Deployment requires no new block fields or genesis reset. [Atomic canonical commit](https://github.com/IDNI/tau-testnet/blob/c2b8c47e07342dee01ac27f1380872374e3bf6ff/db.py#L913) · [Authority reconstruction](https://github.com/IDNI/tau-testnet/blob/c2b8c47e07342dee01ac27f1380872374e3bf6ff/tau_authority.py#L236)

I propose acceptance tests for the partial replacement above, alternative priority, multiple same-block installations, repeat/revoke/refusal, guarded release, proposer/importer agreement, and queries across restart, head advancement, reorg and missing historical evidence. I’m requesting a decision on the design, the native dependency change and the eligibility of the private work.
