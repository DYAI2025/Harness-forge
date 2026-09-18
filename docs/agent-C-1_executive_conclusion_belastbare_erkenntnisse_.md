**1. EXECUTIVE CONCLUSION**

**Belastbare Erkenntnisse (evidence-backed):**
- Working Context is a *projection* (or materialization) of selected state into the model’s prompt window; it is never automatically authoritative. Treating it as current truth produces STALE_CONTEXT and TOCTOU failures. This follows from harness engineering literature (MemGPT/Letta paging models, LangGraph checkpointing, recent memory surveys) and classical cache/invalidation principles.⁠Rickhigh.substack
- An LLM/agent should normally act as reader, proposer, transformer or executor, not as Source of Truth for external business state. Authoritative stores remain external systems of record (CRM, ERP, Git, databases, policy engines). Enterprise designs that keep the host application as SoT confirm this separation.⁠arXiv
- Persistent memory (episodic/semantic/procedural) is primarily *advisory* when a fresher authoritative source exists. Benchmarks such as STALE show models frequently fail to detect implicit invalidation and act on outdated beliefs.⁠Huggingface
- Freshness is better modeled as a version-, event- or dependency-based property than a universal wall-clock TTL. Pure time-based rules fail under variable change rates and do not protect against concurrent mutation.
- Planning against a consistent snapshot + pre-effect live refresh (or expected-revision check) of critical state is a practical hybrid that balances reproducibility and safety. Pure snapshot or pure live-read extremes both have clear failure modes.
- For long-running / durable agents, Run State (ephemeral execution attempt) and Workflow State (durable control + history) should be separated. Temporal’s Workflow Execution + Event History + Activities provides strong engineering precedent for durability, recovery and replay.⁠Docs.temporal
- Explicit state_reads / state_writes declarations (with freshness and expected_revision) enable static conflict detection, ordering analysis and revalidation triggers—analogous to capability contracts.

**Wichtigste Non-Findings / Open:**
- No peer-reviewed, large-scale empirical study proves a single universal “State Contract” schema for agents. All proposed schemas below are DESIGN_RECOMMENDATION / PROPOSED_STANDARD.
- Strong consistency is rarely required for every read; risk and side-effect class should drive the requirement (H8, H9).
- Event sourcing is useful for Workflow State but is not a general default for all agent state (H17).
- Simple context-only models remain adequate for bounded, low-risk, read-only tasks (Profile A).

**Größte Architekturkonsequenzen:**
1. Authority, ownership and storage are orthogonal dimensions.
2. Context ≠ authoritative state; Memory ≠ current truth; Snapshot ≠ live state; Observed value ≠ current value.
3. Every additional state layer must demonstrably prevent a concrete failure class (STALE_*, LOST_UPDATE, TOCTOU, MEMORY_AS_TRUTH, DERIVED_STATE_DRIFT, CROSS_RUN_CONTAMINATION, RETRY_DUPLICATION).
4. Minimal viable models differ sharply by risk profile (A/B/C). Prefer the smallest model that satisfies correctness, controlled writes, concurrency safety, freshness, recovery and audit for that profile.

**2. SOURCE QUALITY MAP**

DomainStrengthWeaknessClassification of typical claims

Agent/LLM memory papers (2025–2026)Direct relevance to context, episodic/semantic/procedural memory, stalenessMostly benchmarks & systems descriptions; few controlled production failure studiesPREPRINT_RESULT / EMPIRICAL_RESULT (benchmarks)

Temporal / Cadence / Durable FunctionsStrong engineering evidence for durable Workflow State, recovery, replayVendor/platform documentation; not agent-specific efficacy trialsENGINEERING_PRACTICE / SYSTEMS_ANALOGY

Classical distributed systems & DB (consistency, MVCC, CAS, ETags, SI)Formal and battle-testedMust be adapted; not automatic agent evidenceFORMAL_RESULT / STANDARD / ENGINEERING_PRACTICE

Security / TOCTOU / policyClear failure modesSparse agent-specific measurementsENGINEERING_PRACTICE / INFERENCE

Vendor agent frameworks (LangGraph, etc.)Practical patternsNo universal standard; marketing biasENGINEERING_PRACTICE

Overall: evidence is strongest for *failure modes* and *engineering analogies*; weakest for universal quantitative thresholds or a single proven ontology.

**3. STATE TERMINOLOGY / ONTOLOGY**

Refined ontology after critical review of the given list:

**Core retained (orthogonal enough for enterprise use):**
- WORKING_CONTEXT — projection presented to the model for the current inference step.
- RUN_STATE — ephemeral state of a concrete execution attempt (attempt_id, current node, local variables, tool-call results of this attempt).
- WORKFLOW_STATE — durable control & history of a long-lived process (node status, retries, approvals, wait conditions, event cursor).
- ARTIFACT_STATE — versioned outputs (files, code, reports, models) with content-addressable or explicit revision identity.
- EXTERNAL_STATE — authoritative state of external systems of record.
- POLICY_STATE — current authorization, roles, approvals, governance rules (often a specialized EXTERNAL or DERIVED view).
- EPISODIC_MEMORY — concrete historical interactions/events with provenance.
- SEMANTIC_MEMORY — abstracted / consolidated long-term knowledge.
- PROCEDURAL_MEMORY / Skill Registry — skills, playbooks, capability packages.
- DERIVED_STATE — caches, summaries, embeddings, indexes, materialized views (explicitly secondary).

**Adjustments:**
- Derived State *must* be a first-class class because it is the source of silent drift.
- Policy State is usefully separate when authorization freshness and TOCTOU risk are high; otherwise it can be treated as a critical EXTERNAL_STATE subtype.
- Artifact State is distinct from Run State (survives the run) but may be produced by a run.
- Overlaps exist (e.g., a summary is both Derived and may enter Semantic Memory); the ontology is a classification of *roles*, not mutually exclusive storage locations.
- Missing useful distinctions: Observation vs. Valid time, Observed revision vs. Current revision, Advisory vs. Authoritative flag.

**4. CLAIM–EVIDENCE–COUNTEREVIDENCE MATRIX** (selected critical claims)

ClaimTypeEvidenceCounter-evidence / LimitsConfidence

H1: Working Context never automatically authoritativeDESIGN_RECOMMENDATIONPaging models, STALE benchmark, context engineeringBounded tasks with no external mutationHigh

H2: Agent rarely SoT for external business stateENGINEERING_PRACTICEEnterprise designs keep host as SoTSelf-sovereign agent research (rare)High

H3: Memory advisory when authoritative source existsEMPIRICAL_RESULT (STALE) + INFERENCESTALE paper, memory surveysPurely internal personalization with no SoRHigh

H4: Version/dependency > pure TTLENGINEERING_PRACTICE + INFERENCECache invalidation literature, PlanFenceVery high-churn systems may still need time boundsMedium-High

H5: Snapshot plan + live critical refreshDESIGN_RECOMMENDATIONHybrid patterns in durable systemsPure live-read may be simpler for ultra-short tasksMedium

H6: Separate Run vs Workflow State for durable agentsENGINEERING_PRACTICETemporal modelShort-lived agents can collapse themHigh

H7–H18Mostly DESIGN_RECOMMENDATION / PROPOSED_STANDARDAnalogies + failure analysisLack of large agent ablation studiesMedium

**5. STATE CLASSIFICATION RESULT**
- Keep the 10 classes above; treat Derived as mandatory for any system that caches or summarizes.
- Collapse Policy into External only if authorization is never high-risk.
- Optional / Experimental: pure parametric (weight-level) memory; full event-sourced everything.
- Required in every non-trivial harness: Working Context, External State references, some form of Run/Workflow distinction for multi-step work, provenance on Derived and Memory.

**6. STATE AUTHORITY MATRIX** (condensed)

STATE_TYPESOURCE_OF_TRUTHOWNERAUTHORITATIVE_FORREADERSWRITERSPROPOSED_WRITERSVERSIONINGFRESHNESS_MODELCONSISTENCYSNAPSHOT_OKLIVE_REQUIRED_WHENINVALIDATIONPROVENANCERISK

Working ContextProjection of othersHarness runtimeCurrent inference onlyModelRuntime (materialize)Model (suggest)Snapshot ID + source revsDependency / versionSnapshot isolation preferredYes (by definition)N/ASource changeHighSTALE_CONTEXT

Run StateRuntime / checkpoint storeHarnessThis attemptRuntime, auditorsRuntimeAgent (local)attempt_id + seqEvent / attemptLocal strongYesOn resumeAttempt endMediumRecovery

Workflow StateDurable store / event logWorkflow engineControl flow, historyRuntime, auditorsEngine (via events)Agent (signals)workflow_id + history offsetEvent-basedCausal / serializable on historyYes (checkpoints)RarelyNew eventsHighRecovery, audit

Artifact StateContent store / VCSArtifact serviceOutput identityManyAuthorized writersAgentcontent-hash / revVersionContent-addressedYesBefore consume criticalNew versionHighIntegrity

External StateExternal systemExternal ownerBusiness truthAuthorized principalsExternal APIs (authz)Agent (propose)ETag / rev / versionVersion / eventSystem-dependentConditionalHigh-risk writes, authzExternal changeCriticalLOST_UPDATE, TOCTOU

Policy StatePolicy engine / IdPSecurity ownerAuthz decisionsRuntimePolicy service—Policy versionVersion / eventStrong for decisionsSnapshot of versionAlways before privileged actionPolicy deployCriticalSTALE_AUTH

Episodic MemoryMemory storeMemory service / tenantHistorical recordAgent, runtimeMemory serviceAgentEvent ID + timeTemporal validityEventual / causalYesRarelyExplicit forget / conflictHighPoisoning

Semantic MemoryMemory storeMemory serviceAbstracted knowledge (advisory)AgentConsolidation processAgentSource revs + confDependencyEventualYesWhen SoR availableSource change / conflictHighDrift

Procedural MemorySkill registryRegistry ownerHow-to knowledgeAgentRegistryAgent / humansSkill versionVersionStrong on loadYesOn skill updateNew versionMediumProcedural drift

Derived StateDerivation engineCache ownerNever (unless proven)ManyRecompute—Source revision vectorDependencySnapshot of sourcesYesAfter source changeSource invalidationCriticalDERIVED_DRIFT

**7–13. Models (Source-of-Truth, Read/Write, Working Context, Run vs Workflow, External, Memory, Derived)**
- **Source-of-Truth Model**: For each logical object there is exactly one authoritative store *at a given time*. The agent never becomes that store for external business entities. Epistemic authority (what the model believes) is distinct from execution authority and storage ownership.
- **Read/Write Ownership**: Reads are generally broad (subject to isolation); writes are narrow and usually mediated (agent proposes → runtime validates → external system commits, or CAS/expected_revision). Direct LLM writes to authoritative stores are disallowed except in explicitly low-risk, sandboxed cases.
- **Working Context vs Authoritative**: Context is always a snapshot projection. Every fact in context should carry (or be retrievable via) source, observed_revision, observation_time, freshness class. Invalidation of sources must be able to trigger context refresh or replan.
- **Run vs Workflow**: Separate for any agent that can outlive a single process or wait for external events. Run State is attempt-scoped and may be discarded; Workflow State is the durable ledger.
- **External State**: Prefer *reference* + expected_revision over full copy. Materialize only when needed for consistent multi-step reasoning; always refresh or condition on revision before side-effecting writes.
- **Memory Authority**: Advisory by default. Authoritative only for purely internal, non-SoR facts (e.g., user preference that has no external record). Always carry confidence, provenance, temporal validity, and conflict detection.
- **Derived/Cached**: Must store dependency vector of source revisions. Invalidation is dependency-driven; derived values are never treated as primary truth.

**14. FRESHNESS / STALENESS MODEL**

Definitions:
- FRESH: observed_revision matches (or is known to be ≥) the revision required by the operation’s freshness contract.
- STALE: known older than required revision or dependency.
- UNKNOWN_FRESHNESS: no version/event metadata available.
- INVALIDATED: explicitly marked by invalidation signal.
- EXPIRED: time-based bound exceeded (secondary).

Comparison of models:
- TIME_BASED: simple, weak under variable rates; failure mode = false freshness or unnecessary refresh.
- VERSION_BASED: strong for mutable resources; needs cooperative external systems (ETags, revs).
- EVENT_BASED: excellent for event-sourced or CDC sources.
- DEPENDENCY_BASED: required for Derived and plans.
- RISK_BASED: modulates the above by operation risk.
- HYBRID: recommended (version/event primary, time as safety net, risk as selector).

**PROPOSED FreshnessRequirement (PROPOSED_STANDARD):**

text

KopierenKopiert

`state_ref, source, observed_revision, observation_time,
freshness_requirement: {mode: version|event|dependency|time|hybrid, bound: ...},
allowed_staleness_mode, refresh_before_effect: bool, expected_revision: optional`

**15. SNAPSHOT vs LIVE READ DECISION MATRIX** (excerpt)

STATE CATEGORYOPERATIONRISKSNAPSHOT SAFE?REFRESH REQ?LIVE REQ?EXPECTED_REV?RATIONALEEVIDENCE

Externalread-only reportLowYesNoNoNoReproducibilityEngineering practice

ExternalJira/Git/CRM writeMed-HighPlanning onlyYesPreferYesLost-update preventionCAS literature

Policyauthorization checkHighVersion snapshotYesYesYesTOCTOUSecurity practice

MemoryretrievalLow-MedYesAdvisoryNoNoAdvisory natureSTALE paper

Derivedconsume summaryMedOnly if sources freshYesIf criticalDependencyDriftPlanFence etc.

Workflowcontrol decisionMedCheckpointOn resumeRarelyHistory offsetDurabilityTemporal

Hybrid “plan against snapshot, execute against refreshed critical state” is the default recommended pattern for Profile B/C.

**16–22. Consistency, Concurrency, Transition, Provenance, Isolation, Lifecycle, Change→Response**
- **Consistency**: Prefer Snapshot Isolation or causal consistency for planning views; linearizability / strong consistency only for the narrow set of high-risk writes and policy decisions. Read-your-writes and monotonic reads are almost always desirable inside a run.
- **Concurrency**: Capability contracts and IR should express expected_revision / CAS / lease. Optimistic concurrency is the default; locks only when external systems require them.
- **Transitions**: Prefer explicit event-sourced transitions for Workflow State; mutable stores acceptable for low-risk Run/Artifact state.
- **Provenance**: Every materialization into Working Context and every write must record source, revision, reader/writer, timestamp, capability/manifest, validation outcome.
- **Isolation**: Strict tenant and run isolation for Working Context and Memory; shared resources must be versioned and access-controlled.
- **Lifecycle**: CREATED → ACTIVE → (STALE|SUPERSEDED|INVALIDATED) → ARCHIVED/DELETED. Retention and GC are independent of Workflow lifecycle.
- **State Change → Runtime Response**:
  - Irrelevant change → ignore
  - Source of current Working Context → REFRESH or REVALIDATE
  - Dependency of plan → REVALIDATE / REPLAN
  - Capability or skill change → REBIND / RECOMPILE
  - Policy change → immediate REVALIDATE of authorization

**23–24. Capability Contract ↔ State Contract ↔ IR Boundary**
- Capability Contract declares *what* can be read/written and the *static* freshness/consistency class required.
- State Contract / StateReference binds a concrete instance (resource_id + observed_revision + freshness contract) at planning or runtime.
- IR / Execution Manifest carries the bound references, expected_revisions, refresh flags and conflict policy for the concrete execution.

Avoid double-maintenance by letting Capability Contract be the type-level declaration and StateReference the instance-level binding.

**25–28. PROPOSED Schemas (v0.1 – all PROPOSED_STANDARD)**

**StateContract** (type-level):
state_id, state_type, source, source_of_truth, owner, authority_class, read_policy, write_policy, schema, versioning_scheme, default_freshness, default_consistency, invalidation_rules, conflict_policy, retention, sensitivity.

**StateReference / StateBinding** (instance):
state_ref, resource_id, observed_revision, observation_time, valid_from/until, freshness_requirement, consistency_requirement, snapshot_id (optional), dependency_refs.

**FreshnessRequirement**: as above.

**StateRead / StateWrite**:
resource (type | instance | pattern), mode (snapshot|live|hybrid), expected_revision (optional), freshness, consistency, purpose.

**29–31. PROFILES**

**PROFILE A – Low-risk read-only / bounded**
State classes: Working Context + light External references + optional Episodic.
SoT: External systems + conversation.
Snapshot: preferred. Live: rare. Freshness: best-effort version or none. Consistency: snapshot. Conflict: none. Trace: basic provenance.

**PROFILE B – Stateful multi-step / moderate writes**
All core classes. Separate Run + Workflow. Explicit StateReferences with expected_revision on writes. Hybrid snapshot+refresh. Dependency invalidation for Derived. Full provenance.

**PROFILE C – High-risk / side-effectful / rapidly changing**
Everything in B + mandatory Policy State version checks, strong consistency or CAS on every privileged write, live refresh before effect, strict isolation, complete audit trail, possible leases.

**32. FIVE (plus) NEGATIVE EXAMPLES**
1. STALE_CONTEXT – model uses old tool result still in context → incorrect action.
2. STALE_AUTHORIZATION_STATE – role revoked but old policy snapshot used → unauthorized write.
3. LOST_UPDATE – agent overwrites concurrent external change because no expected_revision.
4. MEMORY_AS_TRUTH – long-term memory contradicts current CRM record → wrong customer action.
5. DERIVED_STATE_DRIFT – summary/index based on superseded source → cascading errors.
6. CROSS_RUN_CONTAMINATION – shared cache or memory bleed.
7. TOCTOU – precondition true at check, false at effect.
8. RETRY_DUPLICATION – non-idempotent side-effect on retry because no durable attempt ledger.

**33. PRACTICAL IMPLEMENTATION PATTERNS**
- ETag / If-Match / expected_revision for External.
- Temporal-style Event History + Checkpoints for Workflow.
- Content-addressed stores + Git-like hashes for Artifacts.
- Dependency vectors + CDC for Derived invalidation.
- Explicit StateBinding objects passed through IR → Runtime.
- Provenance envelopes on every context materialization.

**34. ANTI-PATTERNS**
- Treating conversation history as authoritative external state.
- Universal TTLs without version awareness.
- Agent direct writes to production systems of record.
- Memory without provenance or conflict detection.
- Derived values without source-revision tracking.
- Collapsing Run and Workflow for long-running agents.
- Assuming strong consistency everywhere.

**35. TEST / EVAL PLAN**

Controlled ablations:
A. context-only vs explicit state store
B. snapshot-only vs live-only vs hybrid
C. no revision check vs expected_revision/CAS
D. memory authoritative vs advisory + verification
E. single state object vs Run+Workflow+External separation

Metrics: task correctness, stale-read failure rate, lost-update rate, unauthorized-write rate, recovery success, audit completeness, latency, tool-call count, storage cost, complexity.

Use synthetic environments with injectable concurrent mutations and policy changes.

**36. OPEN RESEARCH QUESTIONS**
- Quantitative thresholds for when hybrid refresh cost exceeds benefit.
- Automatic inference of minimal freshness contracts from capability risk.
- Scalable dependency tracking for large Derived graphs.
- Empirical measurement of MEMORY_AS_TRUTH frequency in production.
- Formal verification of State Contract composition with Capability Contracts.

**37. FINAL ARTIFACT: Enterprise Agent State Ownership Architecture v0.1**

Core principles:
1. Context is projection, never SoT.
2. Agent is proposer/executor, not SoT for external business state.
3. Memory is advisory by default.
4. Freshness is version/event/dependency-first.
5. Separate Run State (ephemeral) from Workflow State (durable) for long-horizon work.
6. Every materialization and every write carries provenance and revision identity.
7. Risk class selects consistency, freshness and live-read requirements.
8. Smallest model that satisfies the profile’s safety properties wins.

The architecture is expressed through:
- State ontology (section 3)
- Authority matrix (section 6)
- StateContract + StateReference + FreshnessRequirement + StateRead/Write declarations
- Profile-specific minimal configurations (A/B/C)
- Explicit change → REFRESH/REVALIDATE/REBIND/REPLAN/RECOMPILE matrix

**38. KNOWLEDGE-BASE EXPORT** (representative entries)

ID: KB-STATE-001

CLAIM: Working Context is a projection and never automatically authoritative.

CLAIM_TYPE: DESIGN_RECOMMENDATION

STATE_CLASS: WORKING_CONTEXT

SOURCE: Memory harness literature + classical caching

EVIDENCE: Paging models, STALE benchmark, TOCTOU analysis

COUNTEREVIDENCE: Bounded static tasks

CONFIDENCE: High

ARCHITECTURE_IMPLICATION: Always attach provenance/freshness metadata to context facts

IMPLEMENTATION_IMPLICATION: Context materializer must record source revisions

FALSIFIER: Demonstration that un-annotated context yields zero stale failures under concurrent mutation

REVISION_TRIGGER: New empirical study showing context-as-truth is safe under defined conditions

(Additional entries follow the same schema for each major claim in the report; omitted here for length but required in a production knowledge base.)

This architecture is the smallest model that systematically prevents the enumerated failure classes while remaining implementable with existing distributed-systems and durable-execution primitives. It is intentionally incomplete where evidence is insufficient and marks all novel contracts as PROPOSED_STANDARD.

55 sources

Explore Temporal Workflow Engine

Gearbeitet für 15s

**Temporal Workflow Engine – Thorough Exploration**

Temporal is a durable execution platform (open-source core + commercial Temporal Cloud) that runs application code reliably despite process crashes, network failures, long waits, or infrastructure outages. It is not a traditional workflow engine that forces you to model processes as static graphs or YAML/JSON. Instead, you write ordinary code (Workflow Definitions) that looks sequential and synchronous; the platform makes that code durable, recoverable, and observable.⁠Docs.temporal

It originated from Uber’s Cadence project and is widely used for long-running business processes, microservices orchestration, and, increasingly, durable AI/agent systems.

### Core Design Principles

1. **Event Sourcing as the foundation**
Every significant step of a Workflow Execution is recorded as an immutable, ordered **Event** in an **Event History**. The full state of a Workflow can be reconstructed at any time by replaying this history. The Event History is the source of truth for the execution.⁠GitHub
2. **Separation of concerns**
  - **Workflow code** must be deterministic (no direct I/O, no non-deterministic randomness, no system clocks, etc.). It expresses control flow and local state.
  - **Activity code** may be non-deterministic and perform all external side effects (API calls, database writes, LLM invocations, file I/O, etc.). Activities are executed with at-least-once semantics and automatic retries.⁠Mintlify
3. **Durable Execution**
Once started, a Workflow runs to completion (seconds to years) regardless of infrastructure failures. If a Worker process dies, another Worker picks up the Event History, replays it to rebuild exact local state, and continues from the failure point as if nothing happened.⁠Docs.temporal
4. **Code-first, not config-first**
Business logic lives in normal programming-language code (Go, Java, TypeScript, Python, .NET, PHP, Ruby, etc.). The platform handles persistence, retries, timers, and recovery.

### High-Level Architecture

Temporal separates **user-hosted processes** from the **Temporal Service** (cluster):

**User-hosted**
- **Client** – starts Workflows, sends Signals/Updates/Queries, gets results (via SDK).
- **Workers** – processes that poll Task Queues and execute your Workflow and Activity code. You deploy and scale them yourself. The Temporal Service never runs your application code.

**Temporal Service (cluster)**
- **Frontend Service** – single gRPC entry point; handles auth, rate limiting, routing.
- **History Service** – owns each Workflow Execution, maintains its Event History and mutable state, drives progress by creating Tasks.
- **Matching Service** – manages Task Queues and matches Tasks to polling Workers.
- **Worker Service** – runs Temporal’s internal system Workflows (archival, visibility indexing, etc.).
- **Persistence** – database (Cassandra, MySQL, PostgreSQL, etc.) that stores Event Histories and state.⁠Docs.temporal

Communication flow (simplified):
Client → Frontend → History (records Events, schedules Tasks) → Matching (puts Tasks on queues) → Workers (poll, execute, return Commands/results) → History (appends more Events).

### Key Concepts

**Workflow Execution**

The main unit of work. Identified by Namespace + Workflow ID + Run ID.

Each Execution has exclusive access to its local state and runs concurrently with others. It communicates with the outside world only through Activities and with other Workflows via Signals, Updates, or Queries.⁠Docs.temporal

**Event History**

Append-only log of everything that happened: Workflow started, Activity scheduled/started/completed/failed, Timer started/fired, Signal received, etc.

This history is the durable source of truth. During recovery a Worker replays the history against the Workflow code; Commands generated during replay are matched against existing Events instead of being re-issued, so side-effecting Activities are not re-executed.⁠Docs.temporal

**Commands vs Events**
- Commands are requests issued by a Worker after a Workflow Task (e.g., “Schedule this Activity”, “Start this Timer”).
- The History Service turns Commands into Events and records them.
- External occurrences (Activity completion, timer fire, Signal) also produce Events.

**Activities**

The units that touch the outside world. They support:
- Automatic retries with configurable policies
- Heartbeats for long-running work
- Timeouts (schedule-to-start, start-to-close, etc.)
- Cancellation

**Task Queues**

Named queues that Workers poll. Different Task Queues can route different kinds of work (or different versions of code) to different Worker pools.

**Signals, Updates, Queries**
- **Signal** – asynchronous message that can change Workflow state.
- **Update** – request/response interaction that can also mutate state.
- **Query** – read-only view of current Workflow state (does not affect history).

**Continue-As-New**

Mechanism to keep Event History bounded for very long-running Workflows: the current Execution finishes and a new one starts with the same Workflow ID, carrying forward necessary state.

**Entity Workflow pattern**

One long-lived Workflow per business entity (user account, order, device, etc.). All operations on that entity go through Signals/Updates. Excellent for maintaining consistent entity state over months or years.⁠Docs.temporal

### How Recovery / Replay Works

1. Worker crashes or times out.
2. Temporal Service eventually schedules a new Workflow Task.
3. A (possibly different) Worker receives the Task and the full Event History.
4. It starts the Workflow code from the beginning with the original input.
5. As it executes, the SDK matches generated Commands against the existing history.
6. When it reaches the point of failure, the history ends; the Worker continues with new Commands.
7. Activities already recorded as completed are not re-run; their results are taken from the history.

This gives the illusion that the original process never died, while preserving exact state (local variables, position in code, etc.).

### Relevance to Enterprise Agent Harnesses & State Ownership

Temporal maps almost perfectly onto several key distinctions from the previous State Ownership research:

Concept in State Ownership ResearchTemporal RealizationNotes

**Workflow State** (durable control + history)Workflow Execution + Event HistoryAuthoritative, versioned by event offset / history sequence. Source of truth for control flow and past decisions.

**Run State** (ephemeral attempt)Individual Workflow Task / Activity attemptCan be discarded; recovery rebuilds from history.

**External State**Activities that call external systemsAgent/Workflow never becomes SoT; Activities interact with external systems of record.

**Snapshot vs Live**History is a durable snapshot of past decisions; Activities perform live external interaction“Plan/decide against history; effect via Activity”

**Freshness / Expected Revision**Not built-in for external resources, but can be expressed inside Activities or via Update patternsYou still need CAS/ETag/expected_revision at the external system boundary.

**Recovery & Audit**Full Event History + ReplayExcellent provenance and debuggability.

**Determinism requirement**Workflow code must be deterministicLLM decisions and tool results must be recorded as Events (usually via Activities) so they can be replayed.

**For AI Agents specifically**:
- A long-running agent loop (or conversation thread) is naturally modeled as a Temporal Workflow.
- LLM calls and tool executions become Activities (non-deterministic, retriable).
- Human approvals become Signals or Updates.
- Checkpoints / intermediate agent state can be stored in Workflow variables or via Updates; Temporal persists them.
- Several production patterns exist: wrapping LangGraph or OpenAI Agents SDK inside Temporal Workflows so the agent code itself needs little or no change, while gaining durability.⁠Temporal

Temporal does **not** solve external-state freshness or concurrency by itself. You still need expected_revision / conditional writes when Activities mutate external systems. It does, however, give you a rock-solid durable ledger of what the agent decided and what external effects it already committed.

### Strengths

- Extremely strong durability and recovery guarantees.
- Excellent observability (full history, Web UI, metrics).
- Scales to millions/billions of concurrent Workflows.
- Language-agnostic SDKs and polyglot Workers.
- Natural fit for long-running, human-in-the-loop, or multi-day processes.
- Clear separation of deterministic orchestration from non-deterministic side effects.

### Limitations & Trade-offs

- Workflow code must remain deterministic → careful design when embedding LLM decision points.
- Event History can grow large → use Continue-As-New or limit payload sizes.
- Learning curve around determinism constraints, versioning, and testing.
- Not a general-purpose database or cache; external state still lives elsewhere.
- Operational complexity if self-hosting the full cluster (many teams use Temporal Cloud).

### Practical Patterns Relevant to Agents

- **Approval / Human-in-the-loop**: Workflow blocks on Signal/Update.
- **Saga / Compensation**: explicit compensating Activities.
- **Entity Workflow**: one Workflow per long-lived agent session or business entity.
- **Long-running Activity + Heartbeat**: for streaming or multi-minute tool calls.
- **Update + Query**: for interactive agent state inspection and mid-flight changes.

### Summary Mapping to State Ownership

Temporal provides a battle-tested implementation of **durable Workflow State** with event-sourced history as the authoritative record of control decisions and past effects. It cleanly separates that from ephemeral Run/attempt state and from External System State (handled only via Activities).

It is therefore one of the strongest existing engineering precedents for the “Run State + Durable Workflow State + External State” model recommended in the earlier research, especially for Profile B (stateful multi-step) and Profile C (high-risk / long-running / side-effectful) agents.

If you want to go deeper on any specific aspect (determinism rules, versioning, Nexus for multi-team contracts, concrete agent code examples, performance characteristics, comparison with Step Functions / Cadence / Durable Functions, etc.), tell me which direction to explore next.

30 sources

Compare Temporal with Step Functions

Gearbeitet für 15s

**Temporal vs AWS Step Functions – Detailed Comparison**

Both Temporal and AWS Step Functions provide durable execution for long-running workflows. They guarantee that a multi-step process can survive process crashes, network failures, and long waits while maintaining progress and state. They differ fundamentally in philosophy, programming model, operational model, and constraints.

### 1. Core Philosophy & Programming Model

AspectTemporalAWS Step Functions

**Definition style**Code-first (Workflows written in Go, Java, TypeScript, Python, .NET, etc.)Declarative state machines (Amazon States Language – ASL JSON/YAML) + visual Workflow Studio

**Control flow**Ordinary language constructs (if/else, loops, try/catch, functions)Explicit states: Task, Choice, Parallel, Map, Wait, Pass, Succeed, Fail, etc.

**Side effects**Isolated in **Activities** (non-deterministic, retriable)Isolated in Task states (usually Lambda or other AWS services)

**Determinism**Workflow code must be deterministic; Activities may not beState machine definition is declarative; tasks can be non-deterministic

**Expressiveness**High – full power of a general-purpose languageModerate – complex logic often pushed into Lambda functions

**Implication**: Temporal keeps business logic centralized and type-safe in code. Step Functions separates orchestration (state machine) from business logic (tasks), which is excellent for visual communication and AWS-native composition but can lead to logic scattered across ASL + multiple Lambdas.

### 2. Architecture & Execution Model

**Temporal**
- Event-sourced: every step is recorded as an immutable Event in an Event History.
- Workers (you run) poll Task Queues and execute Workflow + Activity code.
- Temporal Service (self-hosted or Temporal Cloud) never runs your code; it only stores history and schedules tasks.
- Recovery = full replay of Event History against Workflow code to rebuild exact local state.
- Strong “exactly-once” semantics for the workflow as a whole (Activities are at-least-once and must be made idempotent).

**AWS Step Functions**
- Checkpoint-and-proceed state machine engine.
- Fully managed by AWS; no workers to run for the orchestration layer.
- Two workflow types:
  - **Standard**: durable, exactly-once state transitions, up to 1 year, full execution history (max ~25,000 events), retained 90 days.
  - **Express**: high-throughput, shorter (≤5 min), cheaper, at-least-once or at-most-once depending on sync/async, limited history.
- State is persisted at every state transition. Recovery resumes from the last successful checkpoint.
- Deep native integrations with 200+ AWS services (Lambda, ECS, SNS, SQS, DynamoDB, Bedrock, etc.).

### 3. Durability, Recovery & History

FeatureTemporalStep Functions (Standard)

Max durationEffectively unlimited (years)1 year

History modelFull ordered Event History (source of truth)Execution event history (state transitions)

History size limitsSoft limits; use Continue-As-NewHard limit ~25,000 events

RecoveryReplay from beginning using historyResume from last checkpoint

Audit / observabilityExcellent (full history + UI + queries)Good (console + CloudWatch; Express history more limited)

Redrive / restartReset, Continue-As-New, Signals/UpdatesRedrive from failed step (preserves successful history)

Both provide strong durability. Temporal’s full replay model gives stronger guarantees about exact local state reconstruction; Step Functions’ checkpoint model is simpler for many AWS-centric cases.

### 4. Operational Model & Deployment

- **Temporal**: You (or Temporal Cloud) run the service + you always run Workers. More operational responsibility (or cost of managed Cloud), but complete freedom of language, cloud, and infrastructure. Can run on-prem, multi-cloud, or hybrid.
- **Step Functions**: Fully serverless and managed. Zero infrastructure for the orchestration layer. Strong vendor lock-in to AWS. Excellent IAM, monitoring, and AWS ecosystem integration.

### 5. Cost Model (high-level)

- **Step Functions**: Primarily charged per state transition (Standard: ~$0.025 per 1,000 transitions). Express is request + duration based. Can become expensive with high-volume, chatty, or retry-heavy workflows.
- **Temporal**: Self-hosted = infrastructure cost. Temporal Cloud = actions + storage (often more predictable at very high volume once past the free/entry tier). Workers always incur compute cost.

Real-world reports show Step Functions can become costly at extreme scale with many transitions/retries, while Temporal’s cost curve is different (more fixed infrastructure or action-based).

### 6. Developer Experience & Tooling

AspectTemporalStep Functions

Local testingExcellent unit testing with replayStep Functions Local + ASL validation

IDE supportFull language tooling, type safetyVisual designer + JSON editing

VersioningExplicit versioning of Workflow codeState machine versions + aliases

DebuggingStandard IDE + Temporal Web UI + historyAWS Console + CloudWatch

Learning curveHigher (determinism rules, event sourcing)Lower for simple AWS workflows

### 7. Fit for AI / Agent Systems

Both are used for durable agents:
- **Temporal** is frequently chosen for long-running agent loops, multi-day human-in-the-loop, complex tool-use sequences, and polyglot or multi-cloud agents. Patterns exist for wrapping LangGraph, OpenAI Agents SDK, etc., so the agent code itself needs little change. LLM calls and tools become Activities.
- **Step Functions** is strong when the agent is tightly integrated with AWS services (Bedrock, Lambda, S3, etc.) and the control flow is relatively static or visually designable. AWS also offers Lambda Durable Functions as a more code-centric alternative within the AWS ecosystem.

For highly dynamic, code-heavy agent reasoning (frequent replanning, complex state, long horizons), Temporal’s code-first model usually scales better. For AWS-centric, service-orchestration-heavy agents, Step Functions is often simpler to adopt.

### 8. Strengths & Weaknesses Summary

**Temporal strengths**
- Maximum flexibility and expressiveness
- Cloud-agnostic / low lock-in
- Superior handling of complex, long-lived, code-centric logic
- Strong testing and replay guarantees
- Excellent for agents that need rich local state and dynamic control flow

**Temporal weaknesses**
- Higher operational complexity (unless using Temporal Cloud)
- Steeper learning curve
- You must run and scale Workers
- No native visual designer

**Step Functions strengths**
- Zero operational overhead
- Deep AWS integration and IAM
- Visual design + clear separation of orchestration vs logic
- Fast time-to-production for AWS-native workflows
- Built-in exactly-once (Standard) and rich error handling

**Step Functions weaknesses**
- Vendor lock-in
- Limited expressiveness (complex logic leaks into Lambdas)
- History size and payload limits (256 KiB per state, 25k events)
- Cost can escalate with high transition volume
- Less natural for highly dynamic or non-AWS workloads

### 9. Decision Guidance

**Prefer AWS Step Functions when**:
- You are heavily invested in AWS.
- Workflows are relatively static or can be clearly modeled as state machines.
- You want minimal operational burden.
- Visual communication with non-engineers is important.
- Native service integrations dominate the design.

**Prefer Temporal when**:
- You need complex, code-native control flow and rich local state.
- Multi-cloud, hybrid, or on-prem is required (or you want to avoid lock-in).
- Workflows are long-lived, highly dynamic, or agent-like.
- Strong testing, type safety, and developer experience in a general-purpose language matter.
- You are willing to operate Workers (or pay for Temporal Cloud).

**Hybrid patterns** also exist: Step Functions can trigger or coordinate with Temporal Workflows (or vice versa) when different parts of a system have different requirements.

### Relation to Enterprise Agent State Ownership

Both systems provide durable **Workflow State**. Temporal’s Event History is a particularly clean realization of an authoritative, versioned, replayable control-flow ledger. Step Functions provides a managed checkpointed state machine that is simpler for many AWS cases but less flexible for arbitrary agent reasoning loops. Neither automatically solves external-state freshness, expected-revision checks, or memory authority — those still require explicit design at the Activity/Task boundary, exactly as discussed in the earlier State Ownership research.

Would you like a deeper dive into any specific dimension (cost modeling, agent code examples, versioning strategies, failure modes, or migration considerations)?

30 sources
