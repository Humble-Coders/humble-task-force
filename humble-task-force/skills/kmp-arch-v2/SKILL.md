---
name: kmp-arch-v2
description: KMP Architecture Guide v2 — Native UI on Android/iOS/Desktop with maximal shared Kotlin business logic, SKIE Swift bridging, shared i18n, and the battle-tested conventions from Aromex-KMP. Use for any KMP project set up in this style — writing features, reviewing code, or scaffolding a new project.
---

# KMP Architecture v2 — Native UI + Maximal Shared Logic

You are working in a Kotlin Multiplatform project built in the Aromex-KMP style. Apply this
architecture strictly for all code you write or review. This supersedes the v1 `kmp-arch` guide.

## Architecture: 3 Platforms, 1 Brain

```
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
│ Android (Compose)│  │  iOS (SwiftUI)   │  │ Desktop (Compose JVM)│
│ ui/<feature>/    │  │ ui/ + viewmodel/ │  │ ui/<feature>/        │
│  Screen + VM     │  │  View + VM       │  │  Screen + VM         │
│ data/ (repos)    │  │ repository/      │  │ data/ (repos)        │
└────────┬─────────┘  └────────┬─────────┘  └──────────┬───────────┘
         └─────────────────────┼───────────────────────┘
              ┌────────────────▼─────────────────┐
              │        sharedLogic (KMP)          │
              │ model/ repository/ usecase/ util/ │
              │ i18n/ data/(Ktor impls) config/   │
              └───────────────────────────────────┘
```

**No shared UI. No expect/actual. No DI framework. Native SDKs stay native.**

The v1 rule "shared = models + interfaces + use cases" has grown. Shared now owns **everything
that is a decision, a calculation, or a projection** — the platforms own only pixels, native
SDKs, and state plumbing. If two platforms could ever disagree about a number, a rule, or a
label, the code that produces it belongs in `sharedLogic`.

---

## Module Structure (real names — match them)

```
project-root/
├── sharedLogic/src/commonMain/kotlin/com/<org>/<app>/
│   ├── model/         # Data classes + enums + DERIVED LOGIC on them (see below)
│   ├── repository/    # Interfaces for platform-implemented repos
│   ├── usecase/       # Business operations + pure domain engines
│   ├── data/          # SHARED implementations where the transport is multiplatform
│   │                  #   (Ktor HTTP clients for a backend API). Native-SDK repos are NOT here.
│   ├── i18n/          # Strings.kt (keys) + EnglishStrings.kt (map) + LocalizationRegistry
│   ├── config/        # Shared constants (collection names, reserved ids, base URLs)
│   └── util/          # Money, date stamping, pure helpers
├── sharedLogic/src/commonTest/…    # Domain tests, organized by feature folder
├── sharedLogic/src/jvmTest/…       # JVM-only guard tests (e.g. ThrowsAnnotationGuardTest)
│
├── androidApp/src/main/kotlin/com/<org>/<app>/
│   ├── MainActivity.kt
│   ├── navigation/    # Route enum + the state-var router composable
│   ├── data/          # Backend*Repository impls (Firebase Android SDK), token brokers
│   ├── util/          # Platform utils (e.g. BalanceRefreshBus)
│   └── ui/<feature>/  # Screen + ViewModel + UiState TOGETHER per feature
│                      #   ui/components/ for shared widgets, ui/theme/ for design tokens
│
├── iosApp/iosApp/
│   ├── iOSApp.swift
│   ├── config/        # AppConfig.swift (base URLs)
│   ├── navigation/    # Root router (Splash/Login/Home switch)
│   ├── repository/    # Backend*Repository.swift impls (Firebase iOS SDK)
│   ├── viewmodel/     # @MainActor ObservableObject VMs (one file per feature)
│   └── ui/            # SwiftUI views + ui/components/
│
└── desktopApp/src/main/kotlin/com/<org>/<app>/
    ├── main.kt
    ├── navigation/    # Section enum + router
    ├── data/          # Backend*Repository impls (Firestore Admin/REST + token broker)
    └── ui/<feature>/  # Screen + ViewModel per feature
```

Feature parity is the default: a feature ships on all three platforms (desktop may lag behind
by explicit decision, tracked in an audit doc — never silently).

---

## What Lives in Shared — the Expanded Contract

### `model/` — data classes WITH their derived logic
Models are not dumb bags. Put on the model everything that answers a question about it:

```kotlin
data class SaleCommission(
    val payeeEntityId: String = "",
    val amount: String = "0",          // money is ALWAYS a decimal string
    val paidCash: String = "0",
    val paidBank: String = "0",
    val paidCashRegisterId: String? = null,
    val paidBankRegisterId: String? = null,
) {
    val paidNow: String get() = Money.add(paidCash.ifBlank { "0" }, paidBank.ifBlank { "0" })

    /** Why this can't be recorded, or null. THE single source of truth — see Blocking Reasons. */
    fun blockingReason(saleCustomerEntityId: String): SaleCommissionBlock? { … }
}
```

Also in `model/`: **fold/projection functions** that turn raw backend rows into what screens
render (e.g. `RegisterLedgerEvent.fold(rows)`, `SaleSummary.registerSplit(registers)`). One
shared projection means three platforms can never disagree about a figure.

- Default values on ALL fields (wire tolerance + SKIE ergonomics).
- Enums own their wire mapping: `fromWire(value: String?)` companions with a safe fallback.
- Reserved/sentinel ids (walk-in customer, default registers) are shared constants, never
  re-typed per platform.

### `repository/` — interfaces for what platforms must implement
```kotlin
interface SalesRepository {
    suspend fun recordSale(record: SaleRecord): String
    fun observeSales(session: UserSession): Flow<List<SaleSummary>>
}
```
- All one-shot methods `suspend`; live data as `Flow<…>` (SKIE turns these into Swift
  `AsyncSequence`).
- Accept/return shared models only.

### `data/` — shared IMPLEMENTATIONS when the transport is multiplatform
New in v2: if the backend is plain HTTP, implement the client ONCE in commonMain with **Ktor**
(e.g. `KtorAccountLedgerRepository`). Only SDK-bound repos (Firebase, platform speech/camera)
are implemented per platform. Rule of thumb: *HTTP+JSON → shared Ktor impl; native SDK →
per-platform impl.*

### `usecase/` — operations AND pure domain engines
- `VerbNounUseCase(repositoryInterfaces…)` with a single `execute(…)`. Validation lives here
  and throws typed exceptions.
- Pure engines as objects/classes with no repos at all: money correction planning
  (`BalanceCorrection.forParty/forRegister` — "type the FINAL balance, we compute the posting"),
  facet/filter engines (`PickerFacets`), calculators, date-range presets. These exist so the
  platforms never hand-roll arithmetic or filtering.

### `i18n/` — ALL user-facing strings
```kotlin
object Strings { const val sales_tab_commission = "sales_tab_commission" /* "Commission" */ }
// EnglishStrings.kt: Strings.sales_tab_commission to "Commission",
```
Platforms render via their localization helper (`strings(key)` in Compose, `loc.t(key)` in
Swift). **Never hard-code user-facing text in a platform layer.** Parameterized strings use
`{0}`-style placeholders resolved by the platform helper.

### `util/Money` — the money contract (memorize this)
- Money is a **decimal string** end to end. Never Double, never Long cents.
- `Money.subtract(a, b)` is defined for non-negative operands and **clamps at zero** — correct
  for prices/discounts, WRONG for signed balances. For signed work use `signOf`/`abs`/`compare`/
  `signedSubtract` and combine explicitly (see `BalanceCorrection.difference` for the pattern).
- Normalize before comparing ("1200" == "1200.00"); blank/unparseable input reads as "0", never
  as a crash and never as a huge number.

---

## The Blocking-Reason Pattern (single source of refusal)

For any action that can be refused (confirm a sale, record a correction), the shared model/use
case exposes ONE function returning a typed reason-or-null:

```kotlin
val commissionBlock: SaleCommissionBlock? get() = commission?.blockingReason(customerId)
val canConfirm: Boolean get() = … && commissionBlock == null
```

- The disabled button, the inline notice, AND the use case's thrown error all read the SAME
  function. A greyed-out button and a server refusal must never drift apart.
- Platforms map the typed reason to a localized string in one `when`/`switch` per platform —
  that mapping is the ONLY per-platform part.
- UI-only gates that shared can't know about (e.g. "mode not chosen yet" on a tri-state control)
  are checked in the screen layer BEFORE delegating, with their own string.

## No Silent Defaults (money-critical inputs)

Anything that decides where money goes or which period it lands in starts **unanswered**, not
defaulted: registers (`registerChoice: String? = null`), sale dates (`saleDate: Long? = null`),
either/or money modes (`payNow: Boolean? = null` — tri-state, nothing preselected). The wire
form may use null-means-default (**choice vs wire split**: UI state holds the full chosen id;
`registerIdForWire` narrows a default to null) — but the *user* must have answered.

---

## SKIE — the Swift Bridge (read before writing any Swift against shared)

SKIE (`co.touchlab.skie`) generates the Swift-facing API. Its rules shape every Swift call site:

1. **No Kotlin default parameters surface.** Swift must pass EVERY argument, in declaration
   order. Adding a field to a shared data class breaks every Swift full-initializer call site —
   grep for them when you add fields.
2. **No `copy()` on data classes in Swift.** To "copy with one change", rebuild through the full
   initializer (see `withInvoice(_:_:)`-style private helpers) — keep one helper per rebuild site.
3. **`suspend` functions** surface as `async` — but a same-named overload set may expose the
   member `__`-prefixed (e.g. `__querySales(query:)`). Check the generated header before guessing.
4. **`@Throws(…)` is mandatory** on every public shared `suspend fun` that can throw — an
   undeclared Kotlin exception TERMINATES the iOS process instead of surfacing to Swift. Keep a
   `ThrowsAnnotationGuardTest` (jvmTest source scan) that fails the build when a public suspend
   fun with `require/check/error/throw` lacks `@Throws`. `CancellationException` must be listed
   and rethrown, never swallowed.
5. **Naming translations:** Kotlin enum cases translate (`NEW` → `.theNew`, `TODAY` → `.today`);
   `description` becomes `description_`; companions are `Type.companion.member`; Kotlin `object`
   is `Type.shared`; nested types flatten (`BalanceCorrection.Plan` → `BalanceCorrectionPlan`).
6. **Boxed primitives:** nullable Kotlin `Long`/`Int` cross as `KotlinLong`/`KotlinInt` — wrap
   (`KotlinLong(longLong:)`) and unwrap explicitly. `Int32` for Kotlin `Int` fields in
   initializers.
7. **Errors in Swift:** Kotlin exceptions arrive as `NSError`; recover the typed exception via
   the bridging helper (e.g. `ns.kotlinException as? AlreadySoldException`) — never string-match
   error messages.
8. `Flow` surfaces as `AsyncSequence`: consume with `for try await x in flow { … }` inside a
   stored `Task`, cancel the task in `deinit`/rebind.
9. Disable SKIE analytics in `sharedLogic/build.gradle.kts` (`skie { analytics { enabled.set(false) } }`).

**Kotlin/Native test caveat:** backticked test names must not contain commas — they compile on
JVM but fail native test targets. Keep shared test names comma-free.

---

## Platform Repositories

- Naming: `Backend*Repository` for mirrored data repos; `Android*/IOS*/Desktop*Repository` for
  platform capabilities (auth, speech, scanner).
- **Hand-mapped documents.** Firestore doc ↔ shared model mapping is explicit per platform
  (`toSaleSummary()`, `saleData(record)`), field-for-field. THE classic parity bug is one
  platform's mapper missing a field the others write — a whole feature silently drops data at
  the database boundary while all UI-level tests pass. When shared gains a field: update **every
  platform's read AND write mapper in the same change**, and test at the mapper level.
- Wire conventions are shared knowledge, not per-platform invention: null-means-default ids,
  `syncStatus` lifecycles (`PENDING → SYNCED | FAILED` stamped by backend triggers), idempotent
  `sourceId` conventions (`sale_<id>:commission_accrue`, `entity_<id>:opening`) so retries can
  never double-post.
- Repos that own network clients (Ktor `HttpClient`, listeners) expose `close()`; desktop VMs
  call it in `dispose()`, iOS in `deinit`. A repo held by a ViewModel is a fresh instance per
  bind — never a singleton.
- Multi-tenant: repos take the resolved backend config (`FirebaseClientConfig`) — never a
  global app instance — via a platform app-factory keyed by config.

## ViewModels

**The `bind(session, config)` pattern (all platforms):** VMs are constructed empty and wired in
`bind`, which (a) early-returns when `uid` AND `config` are unchanged, (b) rebuilds repos/use
cases fresh on a real change (tenant safety — a cached use case must never replay company A's
data into company B), (c) resets per-tenant state. One-time collectors guard with a flag.

**Android** — `AndroidViewModel` + `StateFlow<UiState>`:
- UiState is an immutable data class whose **derived rules are computed vals on the state class**
  (`canConfirm`, `commissionStarted`, `commission: SaleCommission?`). This makes business wiring
  unit-testable with plain constructors — the test suite instantiates UiState and asserts gates.
- Mutations via `_uiState.update { it.copy(…) }`; a `recomputed()` extension re-derives
  dependent totals after cart-like changes.

**iOS** — `@MainActor ObservableObject` + `@Published`:
- Stored `Task`s for stream consumption; cancel on rebind and in `deinit`.
- Guard async result application against staleness (re-check the id the result belongs to is
  still the open one before writing it into state).

**Desktop** — plain class + injected `CoroutineScope(SupervisorJob() + Dispatchers.Default)`:
- `dispose()` closes repos AND cancels the scope (a leaked scope keeps Firestore listeners
  alive across sign-out). The router calls `dispose()` when tearing a section down.
- Rethrow `CancellationException` in every `runCatching` failure branch — a superseded refresh
  must never render as an error banner.

**Cross-screen refresh:** when one screen's write moves numbers another screen shows, signal via
a tiny platform bus (Android/Desktop: `object BalanceRefreshBus { MutableSharedFlow<Unit> }`;
iOS: a `Notification.Name`). Writers emit on success; showing screens collect and re-pull
forced+silent. When the write syncs to a backend asynchronously, **await the sync verdict (poll
the doc's `syncStatus`, bounded) before the confirming UI dismisses**, so the user never sees a
stale figure "after" a confirmed action.

---

## Data & Caching — bounded vs unbounded

**Bounded collections** (contacts, products, in-stock units, registers — fits in memory):
live-observe via `Flow` streams into VM state once per bind; all filter/search/facet work is
client-side against the cache. Filtering engines (facets, cross-filters) live in **shared**, not
per-platform. "The observed lists won't re-emit because a form cleared — never wipe caches when
resetting a form."

**Unbounded collections** (sales history): server-side query objects in shared
(`SalesQuery` with filters + cursor + direction), platform repos translate to real queries,
VM pages with an opaque cursor. Local quick-search filters *loaded* rows only; a submitted
search runs server-side. **A sort-direction flip is a server re-walk, never a reversal of loaded
pages.** Cursors carry full timestamp precision (seconds + nanos) so page boundaries never skip
same-millisecond rows.

**Fetch-on-demand data** (external ledger balances): explicit `loadBalances(force, silent)` with
a short freshness window; `force` bypasses it, `silent` suppresses error UI for background
refreshes. Never zero out shown figures on a failed refresh — stale-and-labeled beats fabricated
zero.

Simple UI-only filter enums may stay platform-side; anything with logic (cross-filtering,
pruning, counting) goes shared.

---

## Navigation (what actually works)

All three platforms use a **state-variable router**, not a nav framework:
- Android: `Route` enum + `rememberSaveable` current route + `AnimatedContent` switch, inside a
  `ModalNavigationDrawer`. ONE global `BackHandler`: drawer open → close it; non-home authed
  route → home; home → system exits. Screen-local BackHandlers compose innermost and win.
- iOS: root switch (Splash/Login/Home), then an `AppScreen` state switch inside Home with a
  `go(_:)` funnel. `NavigationStack` only INSIDE features.
- Desktop: `Section` enum + sidebar; router calls `dispose()` on section teardown.
- Cross-screen deep links are **consume-once pending parameters** (`pendingSaleId`,
  `openRegisterId` + handled callback), never long-lived globals; the target VM accepts the id
  arriving before OR after its data loads.

---

## Testing & the Verification Matrix

- Shared: `commonTest` per feature folder; `jvmTest` for JVM-only guards (source-scanning tests
  like ThrowsAnnotationGuard). Test the engines (money edge cases incl. zero-crossing, folds,
  blocking reasons) — they are the highest-value tests in the codebase.
- Android: unit tests over UiState derived logic + VM behavior. Desktop: VM tests on
  `UnconfinedTestDispatcher` (remember: eager coroutines + virtual time — assert before
  `advanceUntilIdle()` when testing auto-dismiss timers).
- **The done-rule: a change that touches `sharedLogic` is NOT done until Android compiles+tests,
  Desktop compiles+tests, AND the iOS workspace `xcodebuild` succeeds.** Run all three; SKIE
  breakages only surface in the iOS build. State the results explicitly in the PR.
- Repository mappers get their own tests — VM-level suites pass right over a missing doc field.

## Adding a Feature — Order (updated)

1. `sharedLogic/model/` — models + derived logic + blocking reasons
2. `sharedLogic/repository/` — interface (or `data/` Ktor impl if plain HTTP)
3. `sharedLogic/usecase/` — use case with `@Throws` on public suspend funs
4. `sharedLogic/i18n/` — string keys + English values
5. Shared tests (commonTest)
6. Android `data/` mapper (read AND write) → `ui/<feature>/` VM+UiState → screen
7. iOS `repository/` mapper → `viewmodel/` → `ui/`
8. Desktop `data/` mapper → `ui/<feature>/` (or an explicit tracked deferral)
9. Navigation wiring per platform
10. Full verification matrix (Android + Desktop + iOS builds/tests)

## Hard Rules

**DO**
- Share every decision, calculation, projection, validation, and string.
- Money as decimal strings through the shared Money util; know the clamp-at-zero contract.
- One blocking-reason function per refusable action; platforms only localize it.
- `@Throws` on every throwing public shared suspend fun, enforced by a guard test.
- Update every platform's read AND write mappers when shared models gain fields.
- Fresh repos/use cases per `bind`; close/dispose what you open; rethrow CancellationException.
- Tri-state (nullable) for unanswered money-critical choices; choice-vs-wire split for defaults.
- Run the 3-platform verification matrix for any shared change.

**DON'T**
- No platform imports in sharedLogic (Ktor/kotlinx are fine — they ARE multiplatform).
- No expect/actual, no DI framework, no shared UI, no singleton repos.
- No Double for money, no `Money.subtract` on signed balances.
- No user-facing string literals in platform code.
- No silent defaults for registers, dates, or money modes.
- No skipping desktop silently — defer explicitly with an audit note, or ship it.
- No swallowing CancellationException; no treating a cancelled call as "offline".
- No trusting VM tests to prove persistence — mapper gaps are invisible to them.
