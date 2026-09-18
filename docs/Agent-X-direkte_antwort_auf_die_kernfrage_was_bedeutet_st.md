**Direkte Antwort auf die Kernfrage „Was bedeutet STATE an jeder Pipeline-Stelle?"**

StageWas dort *State* ist

Taskgar keiner — nur Direktiven/Inputs (typisierte `DIRECTIVE`, keine Authority)

Capability SelectionRequirements-Typen (State-Typ-Klassen), keine Instanzen

Capability Contract`state_reads/state_writes` als **Patterns + Requirements**, nie konkrete IDs/Revisions

IR / Manifestcompile-time Obligations + **runtime-gebundene Slots**: `expected_revision`, `pre_effect_refresh[]`, `effect_id_derivation`, `conflict_policy`

Runtime/State Refreshfrische-evidenzierte Bindings (`mode`, `source_revision`, `observed_at`) — Refresh nur für die effect-affected Closure

Preconditionsatomare Vergleiche gegen die Quelle (CAS/`If-Match`), nicht „wir haben gerade gelesen"

AuthorizationPolicy-Revision + Consistency-Token, am Effect-Boundary; unknown ⇒ fail-closed bei hohem Blast-Radius

Executionidempotenter Effekt mit stabilem `effect_id`

ValidationPostconditions (Effekt verifiziert, nicht angenommen)

Commit/RecoveryEffect-Ledger; `UNKNOWN_COMMIT` wird rekonziliiert, nie als Fehler behandelt

**Die fünf belastbaren Kernergebnisse:** (1) State ist ein *adressierbares Binding* (Identität+Quelle+Revision|UNKNOWN+Beobachtung), kein Wert — die acht Quality-Gate-Gleichungen werden dadurch schema-*unrepräsentierbar* statt disziplinabhängig. (2) Authority ist **vier orthogonal** Dimensionen (epistemisch / Storage / Write / Entscheidungsverantwortung). (3) Die 10er-Ontologie reduziert sich auf **4 Primitive + 2 Projektionen + 2 Registries** (+ Effect-Ledger); Semantic Memory → Derived, Episodic → Log-View, Artifact → externer Subtyp, Run → Attempt-View. (4) Policy State wird aus *Mechanik*-Gründen separiert (self-referential, new-enemy-Problem, fail-closed). (5) Freshness ist eine **Relation** mit vier Modi; „älter als X" ist Detektor/Fallback, keine Definition.

**Wichtigste Non-Findings (anti-sycophantisch):** Es gibt **keine agentenspezifische Messung**, dass Pre-Effect-Refresh Fehlerraten senkt (H5/H14 bleiben Design-Empfehlung ohne Effektstärke); **keinen externen Standard** für State/Freshness-Contracts (H18 bleibt OPEN, §25–28 sind `PROPOSED_STANDARD` mit Falsifizierern pro Feld); **keine Evidenz** für Event Sourcing als Agent-Default (H17 bestätigt *negativ*); **keine Kalibrierung** für „Memory-Confidence" als Entscheidungsinput. Zusätzlich: Ob Modelle Freshness-Marker überhaupt beachten, ist ungeprüft — Gating gehört daher in die Runtime, nicht in den Prompt.

**Beigelegt und maschinell verifiziert:** `state-ownership.kb.jsonl` (50 Claims SOK-001…050, 12 Pflichtfelder, 0 Abweichungen), `sources.jsonl` (33 Quellen, in-session verifiziert vs. „recalled classics" getrennt markiert), 4 JSON-Schemas + `validate_examples.py` (4 Positiv-, **7 Negativfälle** — u. a. „value ohne Revisionsevidenz", „TIME_BOUNDED ohne Bound", „missing on_unknown" werden nachweislich abgelehnt), Governance-Ledger mit offenen Blockern (u. a. BLOCKER-001: Profile C ist ohne Authz-Service mit Consistency-Tokens nicht schließbar; BLOCKER-002: keine Harness vorhanden, um die A–E-Ablations zu fahren), sowie `ADR-006` als PROPOSED mit unbenanntem Decider.

Wenn du willst, kann ich als nächsten Schritt (a) die §35.2-Ablations als lauffähigen Test-Harness-Skelett (pytest) anlegen oder (b) `StateContract`/`StateReference` in einen konkreten Capability-Contract-Prototyp (z. B. Jira-Issue-Edit mit `expected_revision`) gießen.
