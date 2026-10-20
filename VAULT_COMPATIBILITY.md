# TNE 1.0.73 - Vault and VaultUnlocked compatibility

This supersedes the 1.0.70 caller-worker restriction. Supported synchronous
methods execute directly from ordinary caller threads. Consumers need no
TNE-specific executor access, source change, or asynchronous rewrite to call a
synchronous interface. These are local unpublished builds, with compilation and
static evidence only. Actual-server validation belongs to the owner.

TNE owns accounts, money, supply/funding, transactions, SQL, ledger, receipts,
audit, outbox and recovery. Java 25 and RiddlesLabsCore 0.6.1 are required.
Core remains independent infrastructure. Upgrade Core to 0.6.1: it now publishes
generic services during STARTUP and defers optional player/telemetry initialization.
Core 0.6.0 is insufficient for this startup-ordering release.
Region Manager and Trails were not modified.

## Installation and interfaces

| Optional host | Interfaces registered by TNE |
|---|---|
| Neither host installed | Existing native TNE API; economy remains usable |
| Classic Vault (VaultAPI 1.7 ABI) | `net.milkbowl.vault.economy.Economy` |
| VaultUnlocked 2.20.1, published API 2.20 | Classic Economy, native `net.milkbowl.vault2.economy.Economy`, and its `AsyncEconomy` view |
| Older VaultUnlocked without AsyncEconomy 2.20 | Classic ABI only; native provider deliberately not registered |
| Vault and VaultUnlocked together | Unsupported duplicate installation |

Install **Vault OR VaultUnlocked**, not both. The inspected ordinary
VaultUnlocked-2.20.1.jar declares its plugin name as Vault and supplies both
classic and native APIs. Its descriptor declares Folia support; the inspected
upstream classic Vault descriptor does not. On Folia use VaultUnlocked's
Folia-capable distribution for the classic facade. TNE does not make a
Paper-only third-party consumer Folia-compatible.

Compile/provided dependencies remain VaultAPI 1.7, VaultUnlockedAPI 2.20 from
the official CodeMC repository, and Core Paper API 0.6.1. None of these API
classes, or Bukkit/Paper APIs, is bundled into TNE. API2 2.20 is the exact
inspected ABI; future incompatible ABI revisions are not promised. The upstream
Gradle publication and actual downloaded artifact say 2.20; the stale Maven
source POM still says 2.19. TNE uses the published 2.20 artifact.

## Operations and semantics

| Capability | Classic Vault / classic facade | Native sync and async API2 |
|---|---|---|
| Player account existence and creation | Supported, including OfflinePlayer UUIDs | Supported UUID player accounts |
| Balance and affordability | Supported | Supported, per available currency |
| Deposit/withdraw | Supported funded payments | Supported funded payments |
| Atomic transfer | No such classic method; separate calls remain independent | Supported as one TNE ledger transaction |
| Absolute balance setter | No such method | Supported through an atomic funded delta |
| canDeposit/canWithdraw | No such methods | Supported read-only preflight checks |
| Formatting, names, precision | Configured default currency | Default or exact explicit currency |
| Account display-name rename | No such method | Supported for registered player accounts |
| Classic banks | NOT_IMPLEMENTED; bank support false | Not exposed as invented bank functionality |
| Shared accounts/membership/ownership | Unsupported | Support false; false/empty results |
| Arbitrary non-player UUID creation | Not applicable | Unsupported |
| Account deletion | Not exposed | Unsupported |

All 43 classic, 72 API2 Economy and 46 AsyncEconomy method signatures have typed
explicit implementations/overrides, including deprecated overloads and upstream
default set/transfer methods. Overrides do not inherit an unsafe separate
withdraw/deposit/refund sequence.

Player existence is queried from TNE SQL, rather than inferred from a balance
query or a worker-thread check. Explicit creation uses TNE's existing local
zero-balance wallet initializer in an owned SQL transaction and updates the
existing registration cache after commit. It accepts offline UUIDs, including
players absent from the online directory. Privileged/system, issuance, market
and plugin-owned identities cannot be converted into player wallets. A
concurrent/uncertain account creation can be verified by a fresh account query;
this does not replay a payment. Existing accounts are not renamed by creation.

Native creation names and renameAccount change display labels using TNE's
existing owner-label storage. They do not rename Minecraft identities or assert
website authentication. Labels must contain 1–64 printable characters. Name
maps enumerate registered player accounts; missing labels use the immutable
Minecraft name snapshot, then UUID text. Deprecated classic name overloads
resolve known, unambiguous Minecraft identities from that snapshot. They do not
invent an online-mode UUID from an unresolvable name or perform a blocking
profile/network lookup. An unresolved/ambiguous name fails; the existing
OfflinePlayer/UUID interfaces can identify offline accounts directly.

An unknown account has no spendable funds: hasAccount/has return false;
mutations require explicit account creation first. A balance query for a truly
empty unknown account returns zero. Existing nonzero stored funds without valid
registration raise an explicit registration error instead of being hidden by a
zero response. SQL errors and unavailable providers do not become zero balances.

Amounts must be finite (classic double), nonnegative, representable in TNE's
DECIMAL(38,18) storage, and exact at the selected currency's precision. Classic
conversion uses BigDecimal.valueOf and UNNECESSARY rounding; excess fractional
precision is rejected, never silently rounded. Native inputs remain BigDecimal.
Nonzero mutations/deltas obey the currency's minimum/maximum transaction bounds.
Formatting alone uses HALF_UP display rounding and configured currency formatting.
Classic double outputs retain that protocol's binary precision limitations.

Zero deposits, withdrawals and transfers validate accounts/currency/scope and
return successful no-ops, with current authoritative balances and no ledger
transaction or notification. An absolute setter already at its requested balance
also returns a no-op. Positive transfers to the same account are rejected by
TNE's existing distinct-account rule. A setter reports the actual delta moved,
not an assertion that new money was issued.

Preflight returns the checked amount and current player balance, checks currency,
precision/bounds, registration, source funding and projected storage range,
and reserves/changes no funds. Its success is not a guarantee that a subsequent
payment will succeed after another transaction changes the account or policy.
It does not consume grants, create receipts, or execute-and-roll-back a payment.

## Normal calling contexts and latency

Synchronous queries, registration, payments, transfers, setters, preflight and
renaming run **directly on the caller**, whether that is a Paper tick, Folia
entity/region/global tick, or an ordinary external worker. No TNE worker identity
is required. No financial code waits for a scheduler callback, joins a future,
reads inventories/experience/menus, or accesses mutable player/world state.
UUIDs, immutable configuration and identity snapshots are used. Concurrent Folia
regions use independent SQL units/connections and the same stable database lock
order and TNE financial boundary.

**This bridge is blocking.** SQL/network latency occupies the caller and can
stall the tick or region that calls it. Place SQL near the Minecraft server and
account for slow queries/contention. Owners explicitly accepted this immediate
result contract. TNE does not advertise a synchronous call as nonblocking.

Each runtime binding shares a bounded compatibility connection source across
both providers: 32 concurrent connections, up to 5 seconds waiting for capacity,
5 seconds network connect timeout, 10 seconds socket/read timeout, 5 seconds
per JDBC statement and 5 seconds for InnoDB and metadata locks. Connections use
the same database URL location, credentials and schema; timeout/replay URL options
are overridden only in the derived bridge source, with auto-reconnect and
transaction replay disabled. Stored configuration is not rewritten. The configured runtime/wallet source has the separate bootstrap limits described
below; other independently configured connections retain their settings. These are per-stage limits,
not a whole-operation deadline: multiple statements, DNS, driver cancellation,
connection setup and multiple configured hosts can add latency. Some JDBC
metadata work relies on the socket timeout. No server runtime latency was measured.

Optional notification and logging work is dispatched separately after commit.
The existing notifier schedules gameplay messages on the player's entity
scheduler. A rejected notification queue, offline/retired player, or shutdown
cannot replace the acknowledged payment result or cause a second payment.
There is no physical-item/inventory transfer hidden inside a virtual payment.

## Native asynchronous calls and lifecycle

VaultUnlocked Economy is synchronous; its separate Economy.async() view exposes
AsyncEconomy and supportsAsync() advertises it. Every async operation submits to
Core's bounded TNE financial lane. It does not execute blocking SQL on the
calling tick thread, nor implement async by wrapping an already-completed sync
call on that caller. There is no financial dependency on an HTTP request.

Queue rejection/readiness failure is an exceptional future with no operation
accepted. Financial work accepted before provider closure retains its captured
runtime/SQL source and drains normally, returning the actual result. A returned
future is a caller-owned copy: cancellation/completion does not interrupt or
roll back persistence. Accepted-work admission markers are cleared before user
continuations run, so a late continuation cannot start a new call on a closed
provider. Consumers remain responsible for their own gameplay thread ownership.

Synchronous admission checks readiness only and release their short monitor
before SQL starts; no Java monitor is held while waiting for SQL or a gameplay
callback. Started operations retain their context through close/rebinding.
Unregistration/consumer cleanup never waits for SQL on a tick thread or closes
another in-flight call's connection. Forced process/thread termination cannot
promise delivery of a return value; database recovery remains authoritative.

Providers track optional host/service changes, decline an interface already owned
by another plugin, and unregister on close/rebinding. Closing one interface does
not prevent cleanup of another. TNE native API availability does not depend on
either optional host. Core still requires no economy provider to start.

## Startup discovery and immediate operations (1.0.73)

This release moves the audited financial bootstrap to STARTUP and separates it
from world/UI setup. It fixes the 1.0.72 gap for ordinary POSTWORLD host-only
consumers without changing consumer JARs, source, descriptors or calling threads.
The [design recorded before implementation](STARTUP_ORDERING_DESIGN.md) contains
the source audit and the same-phase limitation.

Configured initial startup now follows one synchronous enable path:

1. Required Core 0.6.1 completes its own enable and publishes PaperCore through
   Bukkit's service manager. TNE registers its Core consumer and bounded lanes.
2. TNE reads its existing local files/setup journal and any saved website economy
   snapshot. It does not fetch a website snapshot to make startup ready. Existing
   identity, SQL URL/credentials, account/currency IDs and pairing are retained.
3. The enable thread runs exactly one SqlStorageBootstrapPlan directly, applies
   only missing migrations using the existing history/checksums, and verifies SQL
   health. It attaches the existing wallet store/registration set and processes
   the existing idempotent setup.yml funding requests before publication.
4. TNE constructs only its virtual economy and notification metadata services,
   binds its native financial API, and immediately
   registers the classic and/or API2 provider supported by the already enabled
   optional host, unless another plugin owns that interface. API2 async access is
   the view returned by Economy.async(), not a different Bukkit service class.
5. TNE returns from STARTUP onEnable with READY_BEFORE_ENABLE_RETURN for a healthy,
   configured binding. The server loads worlds, then enables ordinary POSTWORLD
   consumers. They discover a ready provider using their existing host dependency.
6. ServerLoadEvent, which follows POSTWORLD enable, queues one owned global
   callback for TNE's UI/commands/player services and optional integrations.
   Those reuse the captured runtime and never rerun migrations, funding or
   provider registration. Native finance is already usable before that callback;
   commerce presentation/physical effects become available at this later gate.

There is no future get/join, tick/entity callback, or later plugin-enable event
in the required SQL/bootstrap completion chain. PaperStorageBootstrapper uses a
direct executor only for initial enable. The later first-website-setup path uses
the bounded financial lane and publishes through the global scheduler, guarded
by its binding generation. These are alternative paths for a binding, not two
initializers. Close never holds a callback monitor while waiting for SQL.

The immediate classic balance path is: service lookup -> PaperClassicVaultProvider
-> PaperVaultBackend.requireAvailable -> bounded JDBC read of that captured,
initialized runtime. A payment instead enters TNE's existing prepared transaction
boundary, captures response balances under its locks, commits and returns the
standard response. Neither path requires the global scheduler, a live Player,
or a TNE-worker caller. Native API2 sync uses the same backend; API2 async submits
to the bounded financial lane and completes with the actual transaction result.
A ServiceRegisterEvent listener can call the provider within registration too:
SQL, wallet registration and funding have already completed at that point.

Website/pairing requests, update/release checks, telemetry, known-account
leaderboard backfill, notification draining, recovery/heartbeat/outbox/Redis,
wallet player enrollment, command maps, menus, dialogs and physical-note adapters
start after ServerLoad on the global scheduler. Actual player effects retain
entity ownership and region effects retain region ownership. UI initialization
failure does not unbind the initialized financial service. Queued UI bindings
carry generation/runtime checks; shutdown clears pending work and closes owned
scheduling. No tick callback is awaited by initial financial enable.

Core 0.6.1 likewise enables its independent infrastructure in STARTUP. Its owned
registries, storage/work facilities and module scopes do not require worlds.
Initial player observations and optional FastStats contexts are deferred. Core's
existing availability registry activates optional PAPI only when PAPI is enabled,
even when PAPI uses POSTWORLD. Required TNE infrastructure stays independent of
any economy provider. Core's public API/ABI used by other consumers is preserved.

### Timeouts and setup-only failures

The configured runtime/wallet connection source uses a 5-second connection
capacity wait (32 slots), 5-second driver connect timeout, 35-second socket
read timeout, 30-second JDBC statement timeout and 15-second MySQL/MariaDB row
and metadata lock waits. The bridge derives its own shared 32-slot source with
its existing 5-second statements/locks/connect and 10-second socket limit.
Auto-reconnect and driver transaction replay remain disabled. These affect only
derived connection options, not saved URLs, data locations or credentials.

These are individual JDBC/driver limits, not a hard whole-startup deadline:
connections, finite migration statements and funding requests add up. OS DNS,
multi-host connection attempts and driver cancellation can add latency. Startup
blocks deliberately during necessary SQL work. Slow migrations can time out;
no background migration is abandoned so another bootstrap can race it, and no
uncertain payment is automatically retried. Preserve schema history and funding
receipts after a timeout; MySQL DDL can commit independently of a surrounding
transaction. Diagnose the failure before attempting recovery.

Missing economy configuration, disabled SQL, migration/checksum/permission/
authentication failures, health failures or failed configured funding leave
SETUP_ONLY / FAILED_SETUP_ONLY. TNE retains setup, wizard and health access where
configuration loading permits it, leaves the native API unavailable, and registers
no Vault financial provider. No fallback economy, fabricated balance or successful
payment is supplied. Credential-free SQLState/vendor-code diagnostics and the
existing migration mismatch guidance are retained. Malformed configuration or
an unrecoverable service-construction error can still disable TNE; correct saved
configuration and restart. SQL/funding corrections should also be followed by a
normal restart; automatic retry of uncertain setup payments is not introduced.

A fresh website-managed install without a saved economy snapshot remains setup
only until an accepted snapshot arrives. Consumers doing a single lookup during
that first setup cannot receive a working provider; finish setup and restart
normally. A late API host enables reconciliation, but is likewise outside the
configured, normally ordered startup guarantee. Closed cached references reject
new work; already accepted financial operations retain their captured outcome.

### Supported discovery ordering

Both TNE descriptors and Core's Paper descriptor now declare STARTUP. Paper's
PluginFileType selects paper-plugin.yml before plugin.yml; TNE's legacy metadata
is retained for the Bukkit descriptor path, but the complete product requires a
Paper/Folia server and the Paper-only Core plugin. This is not Spigot support.

There are two distinct rules, both necessary:

- The dependency graph orders Core and the optional API host BEFORE TNE's plugin
  load/enable within STARTUP. TNE keeps the legal API classpath joins. Core does
  not depend on TNE, Vault or VaultUnlocked. Consumers use their existing host
  declarations for API class access; a Paper join-classpath false/absent edge
  cannot be repaired by economy registration.
- CraftServer.enablePlugins filters the ordered list by phase. DedicatedServer
  finishes STARTUP before loading worlds; MinecraftServer enables POSTWORLD
  afterward. Thus TNE's ready service exists before every ordinary POSTWORLD
  Vault consumer's onEnable, even if that consumer has no TNE dependency and
  appears before TNE in the complete topological list. Folia's inspected target
  patches retain this phase sequence. Global/entity/region ownership is still
  handled independently; STARTUP itself is not a region ownership grant.

No loadbefore Vault edge is added: that would contradict TNE's host dependency.
No known-consumer list, wildcard edge, lookup interception, forced enable/reload,
consumer repackaging or third-party descriptor modification is used.

| Existing consumer / host | API access rule | Provider ordering result |
|---|---|---|
| Paper server, legacy consumer with depend/softdepend Vault, default/POSTWORLD phase; classic Vault host | Host exports classic VaultAPI 1.7 ABI through normal legacy classloading | Ready classic provider from completed TNE STARTUP phase |
| Paper server, same consumer with VaultUnlocked 2.20.1 host | Same classic interface identity from host named Vault | Same phase guarantee; no second host needed |
| VaultUnlocked API2 sync/async consumer, legacy descriptor, depend/softdepend Vault, POSTWORLD | Host named Vault supplies API2; async is Economy.async() | Ready native provider/view from TNE STARTUP |
| Paper descriptor consumer, server dependency Vault, load BEFORE, join-classpath true, POSTWORLD | Explicit host classpath access | Host and TNE complete STARTUP before consumer enable; no TNE dependency needed |
| Folia consumer using either descriptor, POSTWORLD, valid Vault host declarations | VaultUnlocked 2.20.1 declares Folia support and supplies classic/API2 | Same loader/phase guarantee; consumer must independently support Folia |
| Custom/repackaged API host itself enables POSTWORLD | API classes may load, but host readiness occurs too late | Outside the guarantee; supported inspected hosts are STARTUP |
| STARTUP consumer already ordered after TNE through an existing valid dependency chain | Normal host classpath access | Ready provider after TNE in the same phase |
| STARTUP consumer with ONLY a Vault edge | API may be accessible, but host -> TNE and host -> consumer are sibling edges | No generic order between TNE and consumer; NOT guaranteed |
| No declarations, onLoad lookup, Paper OMIT or isolated API classpath | Missing relevant contract/edge or wrong lifecycle | No guarantee; phase ordering does not invent legal API access |
| Consumer hard-depends VaultUnlocked but actual host is named Vault | Dependency name does not match the inspected host; no provides alias | Loader failure belongs to host/consumer metadata; TNE cannot fix it |

Normal/default plugin phase is POSTWORLD. Existing correct host declarations
remain sufficient for that normal case; owners do not add a TNE dependency.
Original upstream classic Vault does not declare Folia support. On Folia use the
inspected Folia-capable VaultUnlocked host for both classic and native consumers,
and install Vault OR VaultUnlocked, never both. TNE does not make a Paper-only
consumer Folia-compatible.

The STARTUP sibling case is an architectural limit of supported dependency and
phase mechanisms. There is no earlier supported enable phase before STARTUP.
Moving before the host would lose API ordering; registering in onLoad would
precede the enabled Core consumer/service lifecycle and is not this implementation.
The preserved Core.register contract requires an installed, enabled module
owner. Covering every STARTUP sibling through onLoad would therefore require a
new generic prepared-module lifecycle: pre-enable ownership/admission, deferred
platform facilities, enable-failure rollback and shutdown of never-enabled
owners. That architecture is not implemented or claimed by this phase correction.
A finite loadbefore list is not universal, and modifying third-party plugins or
loader internals is excluded. If an affected consumer already offers an option
to defer economy hooking until world/server readiness, that is the least disruptive
available option; otherwise it requires a separately supported consumer release
or an upstream host/provider bootstrap contract. No such setting/release is
assumed to exist. This build does not claim universal STARTUP compatibility, and
does not present adding a TNE dependency as the fix for ordinary consumers.

Missing local/cached configuration, SQL/migration/funding failure, absent or
incompatible API host, duplicate hosts and another economy provider remain
explicit failure/conflict states. Providers do not claim success or fabricate
balances. This release targets a normal full server startup; hot enabling/reloading Core
or TNE after the startup ServerLoad event is not a supported substitute for a
restart. Optional host reconciliation and asynchronous first-setup binding
within the running TNE lifecycle remain supported. Setup/health commands become
available at the later world/UI gate;
finish first setup or repair SQL and restart normally before one-shot discovery.

/tnehealth reports initial financial readiness, configured state, SQL limits,
Core/TNE STARTUP versus POSTWORLD discovery, world/UI lifecycle state, interface
host/registration timestamps, selected currency/scope and conflicts. Startup logs
mark STARTUP finance completion separately from STARTED/DEGRADED_AFTER_SERVER_LOAD.

Official source snapshots inspected on 2026-10-20 are recorded, with URLs and
SHA-256 hashes, in the 1.0.73 evidence. They include Paper ver/26.1.2 and ver/26.2
CraftServer, DedicatedServer/MinecraftServer patches and LoadOrderTree; Folia
ver/26.1.x and ver/26.2.x region-threading patches; descriptor/classpath metadata;
legacy provider and loading strategy; and the published VaultUnlocked 2.20.1
host plus its source pinned at 803d09f69c8dd2d61753d4a1a95510a65da714e9.

- [Paper dependency/classpath declarations](https://docs.papermc.io/paper/dev/getting-started/paper-plugins/#dependency-declaration)
- [Default phase and legacy dependency contracts](https://docs.papermc.io/paper/dev/plugin-yml/)
- [Paper 26.1.2 enable-phase loop](https://github.com/PaperMC/Paper/blob/ver/26.1.2/paper-server/src/main/java/org/bukkit/craftbukkit/CraftServer.java)
- [Paper 26.2 enable-phase loop](https://github.com/PaperMC/Paper/blob/ver/26.2/paper-server/src/main/java/org/bukkit/craftbukkit/CraftServer.java)
- [Paper STARTUP call](https://github.com/PaperMC/Paper/blob/ver/26.1.2/paper-server/patches/sources/net/minecraft/server/dedicated/DedicatedServer.java.patch)
- [Paper POSTWORLD and ServerLoad calls](https://github.com/PaperMC/Paper/blob/ver/26.1.2/paper-server/patches/sources/net/minecraft/server/MinecraftServer.java.patch)
- [Pinned VaultUnlocked host source](https://github.com/TheNewEconomy/VaultUnlocked/blob/803d09f69c8dd2d61753d4a1a95510a65da714e9/VaultUnlocked/src/main/java/net/milkbowl/vault/Vault.java)

## Funding, currency and scope configuration

Keep existing stable IDs and account UUIDs. Classic and omitted-currency native
calls use `economy.default-currency.id` in economy.yml, pinned to that provider
binding. Native explicit-currency calls require the exact configured currency ID.
Unknown/removed/null explicit IDs fail; there is no first-currency/alias/default
fallback after lookup failure. Only virtual currencies allowed in the selected
scope are advertised. Restart after manual configuration changes.

All calls use `economy.default-scope.id`. The current loader defines that scope
as NETWORK, so world overloads consistently use the same network balance. They
do not create per-world money. A future WORLD/WORLD_GROUP binding requires an
explicit world to match that configured scope ID rather than silently routing
to a different scope.

`integrations.vault.settlement-account-id` in economy.yml is the funded TNE
system account used for legacy AND native deposits/withdrawals and setters.
Deposits/increasing setters transfer money from it to the player; withdrawals/
decreasing setters return money to it. Fund it in every currency that native
consumers will use, through existing setup.yml/website/owner funding tools.
It must be active and distinct from issuance accounts. Insufficient settlement
funds fail normally. No unlimited issuance, new supply, or silent currency/scope
change is introduced. Player-to-player native transfers need no settlement leg.

## Known versus uncertain outcomes

An additive prepared-operation path in TNE's existing DefaultTransactionEngine
owns one SQL unit of work: prepare under registration/account locks, reserve
idempotency, invoke the existing commit boundary (including supply/wallet checks,
ledger, projections, receipt history, audit, outbox and events), capture response
balances while those row locks are still held, then commit. Setters derive
their delta under the same sorted locks. No nonzero balance update bypasses that
boundary, and no new in-memory financial authority or transaction manager exists.
Other native TNE API behavior and the historical migration files are preserved.

Known successful commit returns standard SUCCESS with its captured balances.
No post-commit balance query is required. A capture failure happens before
commit and fails the operation. A connection-close failure after acknowledged
commit is diagnosed without replacing success. PaperVaultCommittedException is
retained only for binary compatibility with 1.0.70 and is not thrown by current
providers; ordinary consumers need no knowledge of it.

Known rejected operations return FAILURE with a pre-operation balance snapshot
when one was captured, otherwise explicitly unavailable balance placeholders.
Unavailable queries throw; boolean methods return false where their interface
has no structured failure result. No result depends on the caller's thread type.

If commit itself throws/times out/loses its response, the outcome is explicitly
UNCERTAIN in the standard failure message, with an operation key and unavailable
balance. TNE does not retry, refund, or assert rollback after this uncertainty.
Reconcile the persisted receipt/key before another attempt. Zero fields in an
uncertain/error response are unavailable placeholders, not successful snapshots.
Classic and API2 Vault calls have no caller idempotency-key parameter: a new
invocation gets a new key and arbitrary consumer retries cannot always be
deduplicated. Known completed payments include receipt and key in the response
message. Namespace is vault or vaultunlocked; API2 pluginName is bounded audit
attribution only and never changes business rules. Separate withdrawal and
deposit calls are independent transactions, never an atomic transfer.

## Genuine remaining capabilities and platform limitations

- TNE has no classic Vault bank/member/owner model. Bank methods return
  NOT_IMPLEMENTED and hasBankSupport is false.
- TNE integration accounts have TNE-generated IDs, namespace ownership and
  spending grants; they cannot safely be invented from an arbitrary caller UUID
  and an unauthenticated API2 pluginName. Non-player UUID creation and shared
  owner/member/permission methods remain unsupported (false/empty; shared flag
  false). Supporting that exact account model needs a durable identity/ownership
  mapping and an explicit TNE authorization design, not merely another provider
  flag. Existing TNE-native integration accounts/services remain unchanged.
- Deleting player accounts would erase/recreate identities still referenced by
  ledger/recovery records and automatic Minecraft wallet provisioning. TNE has
  no matching player-account deletion contract. Delete returns false. Display
  rename is implemented without deleting or changing an account identity.
- Deprecated string identities must be resolvable as described above. UUID
  creation does not assert Minecraft authentication or pair a website identity.
- Third-party Paper-only gameplay and incorrect Folia thread access remain that
  consumer's limitations. No TNE executor adaptation is required for supported
  ordinary synchronous financial calls. This is not a claim that every Vault
  plugin or every optional account capability is universally compatible.

## Compatibility, upgrade and verification

| Target | Java | Core / TNE | Static evidence | Runtime |
|---|---|---|---|---|
| Paper 26.1.2+ (26.1 family) | 25 | 0.6.1 / 1.0.73 | Main compile and API/archive inspection, build 53 | Owner validation pending |
| Folia 26.1.2+ (26.1 family) | 25 | 0.6.1 / 1.0.73 | Same API plus source ownership/entry-point audit | Owner validation pending |
| Paper 26.2 | 25 | 0.6.1 / 1.0.73 | Additional main compile and API/archive inspection, build 133 | Owner validation pending |
| Folia 26.2 | 25 | 0.6.1 / 1.0.73 | Same API plus source ownership/entry-point audit | Owner validation pending |

One deliverable JAR uses the older 26.1.2.build.53-stable compile baseline.
The same final JAR is inspected against both APIs. Stop normally, keep backups,
replace TNE with 1.0.73 and Core with required 0.6.1. Core must be upgraded from
0.6.0 to establish early infrastructure readiness. Keep existing config,
SQL locations, all data/account/currency/scope IDs, translations, TNE-Controller
and pairing credentials. No migration replay, data relocation, reset, import or
re-pairing is required. Keep one optional host, or neither. After the owner's
next startup inspect /tnehealth, /rl health tne and /rl tasks tne. These commands
were not executed during this work.

No tests, test compilation, harnesses, simulations, benchmarks, Minecraft
servers or bots were run. Maven package commands use
-Dmaven.test.skip=true -DskipITs=true without the quality profile or publisher.
Only compilation/packaging and static source/API/ZIP/class-file inspection are
evidence. Reports under docs/compatibility/TNE-1.0.73-* include preserved interface
implementations, startup call-path inspection, descriptors, class/member resolution,
SQL-resource preservation and artifact hashes. They do not prove live database
isolation, driver/lock/timeout behavior, classloader loading, shutdown timing,
third-party integration behavior, or actual-server Folia correctness.
No live database/server files were accessed, and nothing was pushed, published
or deployed. Diagnostic uploads remain optional/owner-controlled; no new
telemetry or financial/player data collection was added.

Region Manager's financial Folia safeguards remain unchanged. Its later native
TNE integration still needs durable intent/receipt reconciliation, continuation
context and idempotent owning-region effects. Making ordinary Vault calls work
does not complete that separate Region Manager feature.

## Primary contracts and connection documentation

- [Classic Vault API](https://github.com/MilkBowl/VaultAPI)
- [Pinned VaultUnlocked API source](https://github.com/TheNewEconomy/VaultUnlockedAPI/tree/e75d44060ce3c19bed229324f415ebfba70b35dd)
- [Native Economy](https://github.com/TheNewEconomy/VaultUnlockedAPI/blob/e75d44060ce3c19bed229324f415ebfba70b35dd/src/main/java/net/milkbowl/vault2/economy/Economy.java)
- [Native AsyncEconomy](https://github.com/TheNewEconomy/VaultUnlockedAPI/blob/e75d44060ce3c19bed229324f415ebfba70b35dd/src/main/java/net/milkbowl/vault2/economy/AsyncEconomy.java)
- [Published API 2.20](https://repo.codemc.io/repository/creatorfromhell/net/milkbowl/vault/VaultUnlockedAPI/2.20/)
- [VaultUnlocked 2.20.1](https://github.com/TheNewEconomy/VaultUnlocked/releases/tag/2.20.1)
- [MariaDB Connector/J connection options](https://mariadb.com/docs/connectors/mariadb-connector-j/about-mariadb-connector-j)
- [MySQL Connector/J networking](https://dev.mysql.com/doc/connector-j/en/connector-j-connp-props-networking.html)
