# Startup ordering design - TNE 1.0.73 / Core 0.6.1

Written after source inspection and before implementation on 2026-10-20.
Preserve the working copies and all prior changes. No consumer/server edits.

## Loader finding

Paper 26.1.2 and 26.2 LoadOrderTree constructs load order edges from each
provider configuration. CraftServer.enablePlugins filters that ordered plugin
list by PluginLoadOrder; STARTUP finishes before world loading, POSTWORLD runs
after world creation. The corresponding Folia 26.1.x/26.2.x patches preserve
these calls and replace global-thread detection; lifecycle startup is not an
entity/region/global tick ownership grant. Paper descriptor dependencies with
load BEFORE order the named dependency and permit classpath access separately.
Legacy depend/softdepend use their named host. Paper chooses paper-plugin.yml
before plugin.yml. Both API hosts inspected declare STARTUP. VaultUnlocked
2.20.1 is named Vault and contains both classic and API2 interfaces.

With TNE POSTWORLD, Vault -> TNE and Vault -> Consumer leave TNE/Consumer unordered.
A loadbefore Vault edge contradicts TNE's Vault dependency and creates a cycle.
There is no wildcard dependency edge for every Vault consumer. Neither consumer
JAR modification nor speculative plugin-manager manipulation is acceptable.

## Chosen supported implementation

1. Release independent Core 0.6.1 with load STARTUP. Its configuration, pure
   runtime/services, module ownership, bounded workers, listener/integration
   registries and PaperCore publication require no worlds or economy provider.
   Publish these normally during enable; keep /rl lifecycle command declaration.
   Defer initial player observations and optional FastStats startup to ServerLoad
   and the global scheduler. PAPI remains optional; its existing availability
   registry activates it when that plugin enables, even if POSTWORLD.
2. Release TNE 1.0.73 with load STARTUP in both descriptors, retaining required
   Core BEFORE and optional Vault/VaultUnlocked BEFORE with classpath access.
   Require Core API >=0.6.1,<0.7.0 so old POSTWORLD Core fails clearly before SQL.
3. Initial TNE enable loads existing local configuration and cached website
   settings; uses the existing bounded synchronous SQL/migration/health/wallet/
   funding path exactly once; constructs virtual economy/notifier metadata;
   binds native TNE and classic/API2 providers. No gameplay service or tick
   callback is necessary to finish financial startup.
4. Register an owned ServerLoadEvent listener before bootstrap. It starts UI,
   command map, player/wallet session listeners, optional integrations, cache
   backfill, website/update work and telemetry on the global scheduler. Later
   accepted website setup uses the same runtime publication boundary and defers
   gameplay binding until that gate is open. Generation checks suppress stale
   callbacks. It never runs another migration/funding bootstrap for UI setup.

## Audited early/late operations

Early: local YAML/journal/cache reads; configuration metadata; SQL history and
health; wallet registration/system policy loading; receipt-backed setup funding;
PaperVirtualEconomyService; notifier construction (no delivery); registrar.
Core names for deprecated Vault string lookups use offline identity metadata:
DedicatedServer sets PlayerList and playerDataStorage before STARTUP; inspected
CraftServer.getOfflinePlayers and CraftOfflinePlayer name reads use that storage,
not a loaded world's live state. No inventory/location/world data is requested.

Late: wallet.start (icon/online entity enrollment and website sync), command map
registration, menus/dialogs/item stack/note adapters, account/session listeners,
commerce effects, notification inbox draining, recovery/heartbeat/outbox/Redis,
PAPI expansion, update watchers, owner website pollers, telemetry contexts and
leaderboard backfill. Entity/region ownership rules are unchanged thereafter.

## Exact guarantee and limits

For healthy configured economies, a POSTWORLD consumer using classic Vault,
VaultUnlocked classic facade, or native API2 gets API class identity from its
normal host dependency/classpath declaration and a ready TNE provider from the
completed STARTUP phase. No TNE dependency, source/descriptor/JAR change, retry
or executor access is required. API2 async remains its bounded completion view.

A STARTUP-phase consumer with only a Vault edge remains unordered relative to
TNE. Supporting every such consumer is impossible using these supported phase/
dependency mechanisms without an additional edge or earlier-lifecycle economy
architecture. Do not claim it is solved. A POSTWORLD consumer with no declarations
is still not promised API class access or platform compatibility. onLoad lookups,
incorrect VaultUnlocked plugin identity, Paper OMIT/classpath-isolated metadata,
missing SQL/setup/host, incompatible hosts and provider conflicts have explicit
limits. No known-plugin loadbefore list or consumer rewrite is introduced.

## Preservation and evidence

Retain all money/domain/API/provider implementation, migrations, receipts,
identities, config paths, translations, funding and controller/authentication.
Retain setup-only readiness failures and accepted transaction outcomes. Inspect
Core's other consumers for API-range/phase consequences without modifying them.
Compile/package against both published API baselines with tests and test
compilation skipped; inspect final archives and compare preserved resources.
Runtime startup/SQL/platform validation belongs exclusively to the owner.

Source snapshots/hashes: target/ordering-1.0.73/official-sources.json and saved
Paper 26.1.2/26.2 sources, Folia 26.1.x/26.2.x patches, pinned VaultUnlocked
803d09f69c8dd2d61753d4a1a95510a65da714e9 and published 2.20.1 host descriptor.
