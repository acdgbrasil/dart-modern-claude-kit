# RATIONALE — Why Each Decision Exists

This document records the *why* behind every opinionated choice in this kit. It exists because rules without justification become cargo cult — and cargo cult is worse than no rules at all.

If a rule below looks arbitrary or wrong, follow the **Source** link to understand the incident, constraint, or trade-off that motivated it. Then decide if your project shares that constraint. **You are expected to disagree with some of this.** That's healthy.

---

## Origin Story

This kit was extracted from a Flutter monorepo built for a Brazilian healthcare nonprofit (ACDG / Conecta Raros). The platform handles sensitive health data of patients with rare genetic diseases, runs on Web (WASM) + Desktop (native, no webview), and ships an offline-first experience for social workers in low-connectivity regions.

**Honest disclosure on adoption (2026-04-30):** the rules below were forged inside a single monorepo, by a single primary author (Gabriel Aderaldo). They have not yet been validated against other ACDG codebases, other Flutter teams, or any third-party project. The kit is published under the ACDG organization for stewardship continuity — not because it carries broad organizational consensus. Treat each rule as a strong default with one production data point, and stress-test it before adopting wholesale.

The architectural decisions here were forged in production. They are biased toward:

- **Strict layering** over flexibility (because hospital-grade audits)
- **Compile-time safety** over runtime cleverness (because patient records can't be wrong)
- **Predictable rebuild scope** over magic reactivity (because mobile + low-end desktop)
- **Tests against fakes** over mocks (because we got burned by mock/prod divergence)
- **Inside-out implementation** over UI-first (because schema changes are expensive)

If you are building a CRUD admin tool for an internal team of 5 people, you don't need most of this. If you are building anything that handles money, identity, or health — you probably do.

---

## The 5 Policies

### Policy 1 — MVVM + Logic Layer (the spine)

**Decision:** Every feature has 5 layers, in this order: Model → Service → Repository → UseCase → ViewModel → View.

**Why:**
- The official Flutter Architecture Guidelines recommend MVVM, but stop short of mandating UseCases. We mandate them because in practice, "thin ViewModel that delegates to Repository" decays into "fat ViewModel that orchestrates 3 Repositories with business logic baked in." A UseCase forces the orchestration to live somewhere named.
- The 5-layer rule is non-negotiable even when a feature looks simple. Today's "1 Repository call" feature becomes tomorrow's "validate, fetch, transform, persist, emit event" feature. Adding a UseCase later means refactoring the View — adding it now costs 12 lines.

**How to apply:**
- Always start by drafting the Model. Then write a failing test against the Repository's abstract interface. Then implement bottom-up.
- If you find yourself writing business logic in the ViewModel, stop and extract a UseCase.

**Source:** Flutter Architecture Guidelines (2025-2026), `references/layer_communication.md`

---

### Policy 2 — Result Pattern (errors as values)

**Decision:** All async operations return `Result<T>`. `throw` is forbidden in domain and application layers. `try/catch` is allowed *only* at adapter boundaries (Service layer), where it must convert the exception into `Result.error()`.

**Why:**
- Exceptions are invisible in function signatures. A function that returns `Future<User>` is lying — it actually returns `Future<User OR an unbounded set of exceptions you cannot enumerate`. `Result<T>` makes the failure mode part of the type.
- We had a real production incident where a `UseCase` swallowed a `FormatException` from `int.parse(idString)` (silent fallback to 0), causing patient data to be associated with the wrong record. After we migrated to `Result<T>`, that class of bug stopped appearing in code review because the unwrap is visible.
- `switch (result)` with sealed-class exhaustiveness lets the compiler catch missed error branches. `try/catch` does not.

**How to apply:**
- Service: wrap the network call in `try/catch`, return `Result.ok(...)` or `Result.error(e)`.
- Repository: pass through or transform Results. Never `throw`.
- UseCase: orchestrate Results, combine via `map` / `flatMap`.
- ViewModel: unwrap with exhaustive `switch`. The Command pattern handles the error UI state.
- View: never sees a Result — only finished state.

**Anti-pattern we banned:** `result.valueOrNull!`. If you need the value, `switch` it. If you need a default, use `result.fold(...)`.

**Source:** `references/patterns.md` § "Tratamento de Erros com Objetos Result"

---

### Policy 3 — Encapsulation Policy (H1–H9)

**Decision:** Nine rules about how and when to use private (`_`) members, composition over inheritance, and where to draw class boundaries.

**Why each rule exists:**

- **H1 — `_` only on injected deps, file-local helpers, real invariants.** Privacy as default makes refactoring impossible — every `_field` becomes a god-class secret. Default to public; make private only when you have a *reason*.
- **H2 — Equatable on every value object.** Manual `==` / `hashCode` always rots. We had 3 separate bugs from `hashCode` going out of sync with `==` after `copyWith` was added. `Equatable` makes them automatic.
- **H3 — Composition over inheritance.** Inheritance shares behavior; composition shares boundaries. A `Repository extends BaseRepository extends BaseAuth extends BaseLog` chain is unmaintainable. A `Repository` that holds an `Auth` and a `Log` member can be tested independently.
- **H4 — Single Responsibility per class.** If you can't describe the class in one sentence without "and", split it.
- **H5 — No god-interfaces.** A `Repository` interface with 17 methods serving 4 use cases means 4 interfaces. Split by client, not by data type.
- **H6 — No `_field` private collections.** A private list/map that escapes via a getter is a lie. Either expose it as `UnmodifiableListView` or split the class.
- **H7 — Sealed class for variations.** A sealed class with 3 subclasses is checked at compile time. A `String type` field with magic values is checked at runtime — by your users.
- **H8 — Extension type for brand types.** `String userId` is identical to `String email` to the compiler. `extension type UserId(String)` makes them distinct without runtime cost.
- **H9 — Local UI state is local.** A `selectedItem` that lives in a `ViewModel` survives navigation. A `selectedItem` in a `StatefulWidget` resets on back. Pick the lifetime that matches the data.

**How to apply:** Read `references/encapsulation_policy.md` — it has counter-examples for each rule with the production incident that motivated it.

**Source:** `references/encapsulation_policy.md`

---

### Policy 4 — Pattern Matching Policy (P1–P5)

**Decision:** Five patterns for using Dart 3 advanced features (records, sealed classes, if-case, switch expressions). Includes one carefully-bounded exception (P2b) where `try/catch` is allowed in domain.

**Why each pattern exists:**

- **P1 — State Matrix with Record + switch.** When a ViewModel has 3 booleans (`loading`, `error`, `empty`), they have 8 combinations and most are nonsensical. A sealed class state with 4 cases makes invalid states unrepresentable.
- **P2 — `if-case` for surgical validation.** `if (input case String s when s.length > 0)` reads better than nested `if`s. Use it when destructuring + guarding in one place.
- **P2b (the exception that proves the rule) — `try/catch` over generated `fromJson`.** When a DTO has 10+ fields, hand-validating each one in domain doubles the line count and hides the actual logic. Allowed *only* at adapter boundary, *only* over `fromJson`, *always* with `catch (e, st)` + observability log + a private `_XxxParseError` type. `catch (_)` is rejected in code review — you must have stack trace and reason. Documented because a category of bug (silently empty patient records) was caused by `catch (_)`.
- **P3 — Tear-offs in `map`/chain.** `users.map(User.fromJson)` over `users.map((u) => User.fromJson(u))`. Less noise.
- **P4 — `Never` for unreachable branches.** `default: throw StateError(...)` rots when you add a case. `default: _exhaustive(value)` where `_exhaustive` returns `Never` makes the compiler catch it.
- **P5 — No sealed-class downcast.** `if (result is Ok<T>) return result.value` works but bypasses exhaustiveness. Use `switch` or combinators (`map`, `flatMap`, `combineWith`) — the compiler enforces handling all cases.

**Source:** `references/pattern_matching_policy.md`

---

### Policy 5 — Concurrency & Performance Policy (C1–C3)

**Decision:** Three rules about isolates, class modifiers, and FFI horizon.

**Why:**

- **C1 — `Isolate.run` to unblock main thread.** Heavy parse / hash / image work on the UI thread causes jank. `Isolate.run` for >16ms work is non-negotiable on mobile / low-end desktop.
- **C2 — Class modifiers as architectural firewall.** `final class` (no inheritance), `interface class` (no extension), `sealed class` (no extension outside library), `base class` (no `implements`). These prevent accidental coupling. We use `final` by default for value objects, `sealed` for state, `interface` for ports, `base` rarely.
- **C3 — FFI + Native Assets as horizon, not present.** We have no hot path that needs FFI today. Documented so future-us doesn't reach for it without re-evaluating. (Likely candidate: cryptographic signature validation if the platform federates.)

**Mental model bonus:** The same person who understands `actor` in Swift understands `Isolate.run` in Dart. Both are message-passing concurrency. We picked these primitives partly because the team also reads Vapor backend code.

**Source:** `references/concurrency_and_performance_policy.md`

---

### Policy 6 — Agent Testing Policy (UI as a stable API for AI agents)

**Decision:** Every interactive widget in the design system exposes a `semanticId` (e.g. `EnvButton.primary`). Per-feature `.md` specs in `packages/<feature>/specs/` document the state matrix.

**Why:**
- We are writing software in 2026 — Claude Code, MCP servers, browser-driving agents are part of the development loop. If the UI is identified by visual position or label text, agent-driven tests break every time the designer moves a button. Stable IDs decouple the test contract from the visual representation.
- The `.md` per-feature specs serve as both human documentation and AI ground truth — the test writer agent reads the spec, not the implementation. (This is REGRA #2 of the source project: never write a test that mirrors the implementation; tests assert intention.)

**Source:** `references/agent_testing_policy.md`

---

## Atomic Design + Selectors/Connectors

**Decision:** Page > Organism > Molecule > Atom hierarchy. ViewModels are *never* passed to Atoms or Molecules. Use a Selector (data) + Connector (callback) at the Organism level.

**Why:**
- Passing a ViewModel down 4 levels of widgets means every widget is coupled to the ViewModel's full surface. Refactor one getter name → 5 widgets break.
- `ListenableBuilder(listenable: viewModel, ...)` at the Page level means the entire screen rebuilds on every `notifyListeners()`. Surgical reactivity (one `ListenableBuilder` per visually-changing region) keeps frame budget under 16ms.
- Selectors expose only the primitive the child needs. Connectors expose only the action. The Atom doesn't know a ViewModel exists.

**Anti-pattern we banned:** `_build*()` helper methods inside a widget. Each `_buildHeader()` should be its own `StatelessWidget` class. Why: `_build*` rebuilds with the parent; a sibling `StatelessWidget` doesn't.

**Source:** `references/selectors_connectors.md`, `references/ui_layer.md`

---

## Repository Naming — No "Impl" Suffix

**Decision:** Implementations are named by *strategy*, never by `Impl`. So `HttpPatientRepository`, `DriftPatientRepository`, `InMemoryPatientRepository`, `FakePatientRepository` — never `PatientRepositoryImpl`.

**Why:**
- `Impl` tells you nothing. `Http`, `Drift`, `InMemory`, `Fake` tell you everything. When DI wiring fails in production, the strategy name in the stack trace tells you instantly which implementation was loaded.
- It also forces you to *think* about whether there will ever be more than one implementation. If the answer is "no" — congratulations, you don't need the abstract class either. The naming pressure surfaces premature abstractions.

**Source:** `references/data_layer.md`

---

## Fakes, Not Mocks

**Decision:** Tests use Fake implementations of abstract classes (`FakePatientRepository extends PatientRepository`). Magic mocks (`when(...).thenReturn(...)`) are forbidden.

**Why:**
- Mocks are coupled to the call sequence — they break on internal refactors that don't change behavior. Fakes are coupled to the contract — they break only when the contract changes.
- A Fake can hold realistic state (`_patients = [...]`), so multiple test cases share setup naturally. A mock requires re-stubbing every call.
- Real production incident: a mocked `Repository` returned a fixed list, but the production migration changed the field order. The mocked test passed; production crashed at first request. After we migrated to fakes (which round-trip through real serialization), this class of bug disappeared.

**Source:** `references/tests.md`, plus the `feedback_no_test_cheating` memory of the source project

---

## REGRA #2 — No Test Cheating

**Decision:** When a test is red, you must verbalize 4 points before touching the test or the implementation:
1. **Intention** — what was this test trying to prove?
2. **Failure** — what's the exact failure output?
3. **Verdict** — is it the implementation, the expectation, or design ambiguity?
4. **Options** — at least 2 paths to resolve, with trade-offs.

Then *wait for the developer to decide*. Never silently change the test to match a buggy implementation.

**Why:**
- Real incident from the source project (2026-04-28): I (Claude, in an earlier session) moved a test from a route that was returning HTTP 500 to a route that returned 404, because the 404 case made the test pass. The test was *trying* to prove that `/team/people` was a valid endpoint. By moving it, I hid the topology leak the test was exposing. The user caught me and added this rule.
- Test cheating is the silent compounding debt. One cheated test today → ten tomorrow → no one trusts the suite.

**Anti-patterns banned:**
- Moving a test to a passing case without documenting why the original case was abandoned
- Replacing `expect(404)` with `expect(anyOf(404, 500))` to "be flexible"
- Adding `try/catch` in implementation just to make a test pass
- Rewriting a test to assert what the implementation does rather than what the contract demands

**Allowed exception:** the test was demonstrably wrong (typo, broken fixture). Fix it and document in the commit.

**Source:** Source project's `CLAUDE.md` REGRA #2

---

## Inside-Out Implementation

**Decision:** Always implement Model → Service → Repository → UseCase → ViewModel → View. Never start from the View.

**Why:**
- Starting from the View means you design the API to fit the screen. The screen changes 5x; the API changes once. You'll refactor the API every time.
- Starting from the Model forces you to think about the *data*, which is the most expensive thing to change. When the data shape is stable, layers above iterate cheaply.
- This is the same lesson as "design the database before the UI" from 30 years of CRUD app history. Flutter doesn't change physics.

**Source:** `references/recomendations.md`

---

## Models Are Schemas (No Business Logic in Models)

**Decision:** Models contain only `final` fields, `copyWith`, and `Equatable.props`. No methods like `Patient.isMinor()` or `Patient.canVote()`. All logic lives in UseCases or BFF.

**Why:**
- A method on a Model encodes a business rule in the wrong place. `Patient.isMinor()` returns `true` based on age — but in some jurisdictions, "minor" depends on context (criminal vs. medical consent). The rule belongs to the UseCase that needs it.
- Models cross network boundaries (frontend, BFF, backend can all share the same DTO). Methods don't serialize. Stick to data.
- The corollary: if you find yourself wanting `model.something()`, write a free function or a UseCase method instead.

**Source:** `references/patterns.md`, source project's ADR-010 (model immutability)

---

## What This Kit Does NOT Mandate

The source project has many more decisions that are *project-specific*, not universal. They are explicitly **excluded** from this kit:

- **Drift over Isar** — Source project picked Drift after Isar's Swift Package Manager incompatibility blocked iOS. Your project may not have that constraint.
- **Riverpod over Provider/get_it** — Source project picked Riverpod for the stub+override DI pattern. The kit shows examples in Riverpod but doesn't enforce it.
- **Split-Token OIDC over plain JWT** — Source project's threat model required this; yours might not.
- **Per-feature `.pipeline/` folders** — Source project uses a multi-agent fail-first pipeline with literal folder structure. You can adopt the agents without the folder.
- **Specific package names** (`package:core`, `package:core_contracts`, `package:design_system`) — these are source-project packages. The kit's agents and skills reference them generically (`package:equatable`, `<your-design-system>`).

If you want to adopt any of these, you can — but read the source project's `handbook/architecture/DECISIONS.md` first to understand *why*, not just *what*.

---

## How to Disagree With This Kit

This is a starting point, not a constitution. To override a rule for your project:

1. Add a project-level `CLAUDE.md` that explicitly contradicts the rule, with your reasoning.
2. Disable individual skills/agents you don't want via `/plugin disable <name>`.
3. Fork this repo and remove what doesn't apply.

The worst outcome is silently following a rule you don't believe in. The second-worst is silently breaking a rule because you didn't read the rationale. This document exists to make the third option — informed disagreement — possible.
