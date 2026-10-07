# AI-native Git: initial source analysis

Status: research proposal; no Git behavior or object format changed.
Date: 2026-10-07.
Fork: https://github.com/fusionrabbit/git (personal account).
Upstream: https://github.com/git/git.
Inspected baseline: `6de20f6092dcf9bdb1c8efe03db4b70c82b423dd`.
Research branch: `codex/ai-native-analysis`.

## Objective and boundaries

Investigate two connected directions: an AI-oriented storage/indexing system
and a native product model linking intent, actors, actions, use cases,
contracts, code, tests, documentation, planning and approval evidence.
All authoritative product/work records must be versioned in Git. The system
is standalone; CodingBud is only a possible future consumer.

This analysis is based on source inspection, not performance measurements.
No build, dependency installation, embedding request or application execution
has been performed. Repository compatibility and exact revision identity are
initial experimental constraints, not a decision to exclude deeper changes.

## Source map and initial findings

Paths below refer to the inspected upstream baseline.

| Concern | Source | Finding and implication |
| --- | --- | --- |
| Object database | `odb.h`, `odb.c` | `object_database` owns sources, caches and a transaction. `odb_read_object()` reads exact objects; `odb_write_object_ext()` hashes content before dispatching a write. Preserve this identity contract in any backend experiment. |
| Storage abstraction | `odb/source.h`, `odb/source.c` | Sources expose callbacks for reading, streaming, iterating, writing, transactions, maintenance and integrity checks. This is a concrete seam for backend research, not a public dynamic plugin API. |
| Current backend selection | `odb/source.c:27` | `odb_source_new()` directly constructs the files source. Callback abstraction does not mean an arbitrary database is selectable by configuration today. |
| Existing source implementations | `odb/source-files.c`, `odb/source-loose.c`, `odb/source-packed.c`, `odb/source-inmemory.c` | Loose/packed/in-memory implementations provide examples. An alternative must address the full lifecycle, not just key-value reads. |
| Object transactions | `odb/transaction.h` | Buffered object writes have a distinct commit/finalize lifecycle. This is separate from ref transactions and does not by itself establish an atomic product/workflow transaction. |
| Object identity and format | `object-file.c`, `object.h`, `Documentation/gitformat-pack.adoc` | Logical types are commit/tree/blob/tag; pack encodings also include deltas. New graph/vector object types affect readers, packing, validation and transfer. |
| Existing acceleration pattern | `commit-graph.c`, `chunk-format.c`, `Documentation/technical/commit-graph.adoc` | Commit-graph is a supplemental index that can be disabled while objects remain sufficient. Reuse the architectural pattern, not the history graph as a product graph. |
| Object annotations | `notes.c`, `notes-cache.c` | Notes associate blobs with object IDs without modifying source objects. `notes_cache` uses a validity marker; its write builds a parentless commit and updates without an expected old ID. It is not a ready-made durable concurrent approval/event journal. |
| Reference transitions | `refs.c`, `refs.h` | Ref transactions accept expected old object IDs. Use one conditional canonical ref advance for a validated state transition; Git does not provide domain permission or budget rules. |
| Batched reading | `builtin/cat-file.c` | Existing batch operations offer a baseline for reading repository content without launching one Git process per object. Compare against this before attributing latency to storage. |
| Command integration | `git.c` | Builtin dispatch and external command handling offer separate paths for an experimental command. Selecting a builtin does not require changing object formats. |
| Transfer | `upload-pack.c`, `fetch-pack.c`, `send-pack.c`, `builtin/receive-pack.c` | Durable graph/embedding data must be included through intentionally transferred reachable objects/refs. Local files in `.git` do not automatically travel with clone/push. |
| Retention | `reachable.c`, `builtin/prune.c` | Git retention follows recognized object reachability. An object ID written inside JSON is not a traversal edge that keeps its target alive. Preserve required source revisions through explicit reachable commits/refs. |
| Profiling/tests | `Documentation/technical/api-trace2.adoc`, `t/perf/`, `t/unit-tests/u-odb-inmemory.c` | Trace2 and existing test infrastructure provide starting points. Neither has been run in this analysis. |

## Storage alternatives to compare

1. Standard Git objects plus a supplemental semantic index. Product records
   and retained derived bytes are ordinary reachable blobs/trees/commits.
   Local text/vector/adjacency indexes are disposable query structures.
2. An alternative object source behind the existing ODB interface, preserving
   exact object identities and export to ordinary Git objects/packs. Prototype
   selection, transaction behavior, migration and compatibility explicitly.
3. New graph/vector object types or a new repository format. This is the most
   invasive alternative. List every affected reader, fsck, pack, GC, protocol
   and host component before implementation; demonstrate an export path.

The fork enables experiments in all three. Initial investigation should use
the same workloads and baseline for meaningful comparison. Merely storing a
vector beside content does not provide nearest-neighbor search.

## Product graph and authority

Keep authored intent, extracted implementation facts and AI-proposed links
distinct even when exposed in one graph view. Actors and actions describe
behavior; use cases are named flows referencing action occurrences. Shared
contracts, schemas, events and other independently referenced artifacts may
be supporting records. Private implementation details need not be authored
as individual product nodes.

Stable product IDs survive renames. Revision identities bind definitions,
code and verification to exact snapshots. An action can span several files
and a file can serve several actions. Planning/task dependencies, behavioral
flows and runtime evidence are related models with different semantics.

Each relationship needs origin, source revision and applicability. An AI
suggestion cannot silently become approved intent or an authorization rule.
Parser observations do not resolve all dynamic calls or prove runtime behavior.

## Derived data and retrieval

An embedding record should identify source content, chunk boundaries,
embedding model/version, dimensions, preprocessing and any context included.
An isolated-content embedding can be reused across branches; code analysis
depending on imports/build configuration requires broader dependency identity.
Model migrations must not change the identity of unchanged code.

Query against a requested repository revision, with a separately identified
workspace overlay for uncommitted edits. Exact identifiers and text search,
symbol navigation, semantic candidates and bounded graph expansion should
cooperate. Validate candidate membership in the requested snapshot before
returning it. Return exact source references and coverage gaps.

Persist derived bytes in Git when portability or historical evidence requires
them. Rebuilding from an external model is not a guarantee of identical bytes.
Supplemental `.git` index files need an explicit build/recovery path; a clone
must preserve authoritative data even when those indexes are absent. Define
fetch/push/backup behavior for every non-default ref.

## Synchronization experiment

Use a generic order fixture with actors, browse/create/cancel actions,
branching use cases, a shared repository contract, code, tests and docs.
Include objectives, tasks and revision-bound review records.

Change cancellation from "before shipment" to "before payment capture".
Detect impact, propose contract/code/test/doc changes, execute verification
and record independent review and human approval. Direct code edits enter
the same change-impact process. Never rewrite approved intent or weaken a
test to hide an implementation regression.

Include an action rename with stable identity; a missing relationship; a
misleading similar function; two branches that merge cleanly as text but
contradict each other; and evidence becoming stale after a relevant change.
Mechanical checks and semantic findings must report their coverage and
uncertainty rather than claim universal equivalence of prose and code.

## Bounded research tasks

| Task | Question | Deliverable / acceptance |
| --- | --- | --- |
| A1: ODB lifecycle audit | Can an alternate backend satisfy the complete source interface? | Callback inventory covering streaming, transaction/failure semantics, fsck, maintenance, alternates, pack generation and migration. No implementation commitment yet. |
| A2: Versioned graph fixture | Can product identity and links be represented portably? | Small schema/fixture, parser validation rules and rename/branch scenarios; all authored facts preserved by normal Git. |
| A3: Incremental index design | What invalidates each derived record? | Manifest schema and source/dependency/model keys; stale-revision rejection; rebuild and workspace-overlay design. |
| A4: Storage benchmark | Is storage a material bottleneck? | Compare repeated subprocess reads, persistent cat-file batch, warm/cold reads, index construction, disk use and clone/transfer overhead. Report hardware, tool versions and dataset. |
| A5: Retrieval evaluation | Do vectors/graphs improve complete tasks? | Matched text/symbol, plus vectors, plus graph, combined modes; same tasks/models/budgets; repeated runs and held-out cases. Measure completion, regression, context volume, time, cost and missed relevant context. |
| A6: Synchronization prototype | Can relevant drift be detected without corrupting intent? | Requirement-change/direct-edit/concurrent-change demonstrations, explicit unresolved findings and approval applicability. |
| A7: Backend experiment decision | Which measured limitation requires core changes? | Written comparison; select standard index, compatible backend prototype or new format with stated evidence and compatibility costs. |

No speedup, accuracy improvement, universal call graph or automatic semantic
merge is established yet. A successful initial result must survive an index
loss/rebuild and a normal clone of all required refs without losing product
definitions, decisions or preserved evidence.

## Next source investigation

Start A1 with `odb/source-files.c` and its transaction/pack paths. Identify
dependencies on filesystem assumptions outside the callback interface.
Then define A2/A3 data contracts before selecting a vector engine or backend.
Build/test requirements and dependency availability must be assessed before
any performance claim. Keep each experiment on a scoped research branch and
preserve the upstream baseline for comparison.
