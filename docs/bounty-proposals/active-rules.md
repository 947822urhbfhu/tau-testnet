## Bounty 02 · Which rules protect this account?

A private prototype and its tests predate maintainer approval. This proposal targets `tau-testnet` at `c2b8c47` and its pinned Tau engine at `c3ea334`.

### What a wallet can verify

A wallet can install a rule in the first transaction of a block and replace it in the last. Both transactions can succeed. The proposed query shows the policy at a canonical chain tip. A receipt shows what each transaction changed.

```text
getactiverules <sender> [recipient] [--payment <id>] [--tx <hash>]
getrulesnapshot <block_hash> status
getrulesnapshot <block_hash> <content_id> [byte_offset]
```

The first query returns the genesis hash, block hash, height and active consensus ID. For each account, it reads the outgoing clause and rule-change guard from the same registry used by execution, keyed by normalized public key and target stream. Each installed clause includes its stored UTF-8 body and SHA-256 digest. An empty enforced slot returns `absent`. On this protocol version the incoming role returns `unsupported`. On a node also running the Bounty 03 recipient policy, the query checks its committed activation and reads the recipient’s incoming clause, maximum hold duration in seconds or blocks, and policy digest. A recipient’s outgoing spending limit applies when that recipient spends. The payment lookup checks the parties and adds the amount and release point of an open retractable payment.

For example, Alice can request `getactiverules <Alice> <Bob> --tx <install-hash>`. If the receipt records an installation with digest X and the sender’s current clause has digest Y, the wallet can show “Installed in block 81. Replaced by block 82.” A pending offer or unexecuted guarded change supplies no installation receipt. The same canonical receipt is available from `gettxstatus`.

If programmable grants also land, Alice funding Bob and Bob paying a shop are two different transactions. For a draw, the wallet needs the grant’s remaining principal, time conditions and caps, any shared allowance, Bob’s sending policy and the shop’s receiving policy. The proposed `--payment` option reads existing retractable transfers. The separate programmable-payments lookup reads grants. A wallet combining those responses must match their genesis and block hashes, retry on head movement, and preflight the actual draw. A historical grant lookup would require a separate design.

### Shared Tau rules after a partial replacement

Suppose the shared program receives `o26=7 && o27=9` followed by `o26=3`. Submitted text includes the superseded value 7. An effective view must expose the surviving 9 and replacement 3. There is also a case where one specification part retains several alternatives. The pinned C++ interpreter can report the last selected alternative or a disjunction before selection. Its existing Python `current_spec()` binding goes through `api::current_spec()`, which prints `current_spec_fm()` and can therefore return a disjunction even after a step. Different inputs can select a different alternative at the same revision. [Interpreter contract](https://github.com/IDNI/tau-lang/blob/c3ea334cd1e6612930ea9f404933211858f0009f/src/interpreter.h#L312) · [Native example with conditional alternatives](https://github.com/IDNI/tau-lang/blob/c3ea334cd1e6612930ea9f404933211858f0009f/tests/unit/test_interpreter_api_extensions.cpp#L427)

The proposed native accessor, `current_spec_parts()`, is read-only. It returns each conjunctive specification part with its formulas in the priority order used by the interpreter. A separate `current_selected_spec()` binding exposes the C++ step observation with its time point. When the node uses an older binding, it reports `selected_step_only`. The wallet can verify its registry slot and must label the shared view as partial. Full shared coverage requires building the accessor into the native dependency. Shared formulas include their representation plan and mapping for interned addresses. They describe shared constraints. A verdict for a concrete transfer still requires its inputs and execution history.

### Publication and consistency

- Capture registry replacements, neutral revocations, offers, guarded requests and releases, and native revision outcomes during execution. Discard effects from rejected candidates. Persist accepted effects with the block hash, transaction position, actual charged fee and bounded diagnostics in the same SQLite transaction as the canonical state. Join later releases by request ID and canonical ancestry. Return `not_recorded` for a confirmed historical transaction whose execution evidence is missing.
- Read both accounts, payment, receipt and shared-snapshot reference from one committed database view. Exclude intermediate fork replay. A separate reader reconstructs the journal using its committed representation and allocator mapping, verifies journal links, recorded step results, revision and time point, and renders the formulas. The read request never steps the live evaluator.
- Pin pages by block hash and SHA-256 of the full serialized document. Allow a wallet to finish a retained snapshot when the head extends. Reject cursors on an orphaned branch. A lagging reader reports its availability. A failed reconstruction leaves the registry and confirmed receipts readable.

These receipts are full-node observations. The existing block hash and state commitment continue to anchor consensus. Deployment requires no new block fields or genesis reset. [Atomic canonical commit](https://github.com/IDNI/tau-testnet/blob/c2b8c47e07342dee01ac27f1380872374e3bf6ff/db.py#L913) · [Authority reconstruction](https://github.com/IDNI/tau-testnet/blob/c2b8c47e07342dee01ac27f1380872374e3bf6ff/tau_authority.py#L236)

The proposed acceptance tests cover partial replacement, alternative priority, multiple installations in one block, repeat installations, revocations, refusals, guarded release and proposer/importer agreement. They also cover restart, head advancement, reorg and missing historical evidence. Please review the wallet contract and the native API changes, and confirm whether the native accessor should land before the node RPCs.

[Private testnet observations and remaining limits](https://github.com/947822urhbfhu/tau-testnet/blob/bounty/design-proposals/docs/bounty-proposals/design-evidence.md) · [Wallet and operator guide for an installation](https://github.com/947822urhbfhu/tau-testnet/blob/bounty/design-proposals/docs/bounty-proposals/checking-a-rule-installation.md)
