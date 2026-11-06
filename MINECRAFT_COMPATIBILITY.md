# TNE Minecraft compatibility

## Installation

Use `TNE-Economy-1.0.98.jar` with **RiddlesLabsCore 0.6.21 or newer within 0.6.x**. There is one TNE Minecraft JAR and one Core JAR for the supported range; no extra compatibility plugin is needed. TNECORE, the HTTP backend, needs no update for the system account identity migration.

| Server range targeted | Java runtime | Build evidence |
| --- | --- | --- |
| Paper 1.20 through 1.21.x | 21 or a newer runtime supported by that server | Production compilation against 1.20; static API inspection across representative releases |
| Folia 1.20.1 through 1.21.x | 21 or a newer runtime supported by that server | Production compilation against Folia 1.20.1; static API inspection |
| Paper 26.1.2 through 26.2 | 25 | Production compilation against 26.2; static inspection of 26.1.2 and 26.2 |
| Folia 26.1.2 through 26.2 | 25 | Production compilation against Folia 26.2; shared API inspection |

The plugin and included TNE libraries contain Java 21 bytecode. **Java 17 is not supported**, including on early Minecraft 1.20 servers. Builds use JDK 25 with `--release 21`. A server's own newer Java requirement still applies.

The range includes available Paper/Folia releases, not hypothetical Folia 1.20.0 or unpublished 26.1.0 builds. It does not add Spigot/Bukkit runtime support or promise compatibility with every fork. Native Paper descriptors and Folia support remain enabled.

## Historical 1.0.79 compatibility implementation

- Commands register through Core's version-aware runtime command service. TNE no longer links the modern Brigadier `BasicCommand` or `CommandSourceStack` types.
- Command registration uses Core's global-thread check. Core supplies the main-thread fallback on older Paper and the appropriate Folia schedulers.
- Menu listeners use Core's inventory-view adapter across the class/interface change in Minecraft 1.21.
- Setup and owner transfers use Core native dialogs where supported and Core private chat input on older servers. Forms collect fields step by step, then show a review and require confirmation. Currency lists are paginated rather than truncated.
- Keep, replace and clear actions are explicit. Secrets are never echoed in the review or included in TNE form logs. On older servers, replacement values are entered through Core's captured private chat input; owners can instead edit the existing configuration files. This is not encryption or isolation from other server plugins that observe chat events.
- Invalid input can be retried within the original form deadline. Cancellation, disconnect, permission loss, stale sessions and shutdown cannot submit an unfinished form. Owner transfers retain a separate financial confirmation, original request IDs and uncertain-outcome receipt handling.
- Cosmetic bundle tooltip handling is gated by the server's item-component capability. Older clients keep the ordinary tooltip. Item names and lore remain intact.
- Potion assets use the available base-potion API. Legacy/new names and long/strong variants preserve the requested effect. Unsupported effects fail explicitly; money-backed items are never silently replaced with a different asset.
- Website/wallet/update HTTP responses retain the same byte limits using Java 21-compatible response handling. URLs, authentication, payloads and pairing identities are unchanged.
- Release metadata uses the plugin's Java target rather than assuming Java 25. Signing and publication procedures remain in place; this local build has not been published.

## Historical 1.0.79 preservation and boundaries

This release does not migrate or reset configuration, languages, balances, SQL data, scopes, funding, TNE-Controller files or pairing. Public portable API sources, financial engine sources and SQL migration resources remain unchanged. Vault and VaultUnlocked providers keep their existing contracts and startup ordering. Choose optional integration plugin versions that support the server and Java runtime you run.

The backport does not make newer Minecraft content exist on old servers. The website item catalog reflects the running server. A saved request for an item/effect unavailable there is rejected or left for existing recovery handling, rather than substituted. Do not downgrade a Minecraft world or live inventory to an older server based on this plugin's compatibility range.

Synchronous Vault calls still occupy their caller during SQL work. Existing funding, unsupported bank/account operations, STARTUP consumer ordering and uncertain-commit limitations remain documented in [Vault compatibility](VAULT_COMPATIBILITY.md).

## Historical 1.0.79 evidence

Production-only compilation uses `-Dmaven.test.skip=true -DskipITs=true` and excludes the testkit module. Independent output directories avoid reusing classes from a different API baseline.

Static class-file inspection checks direct server references against Paper 1.20, 1.20.1, 1.20.6, 1.21, 1.21.4, 1.21.11, 26.1.2 and 26.2, plus Folia 1.20.1 and 26.2. It also checks class/interface invocation changes, packaged Java versions, provided API exclusions and unchanged SQL/API/financial sources. Reflection boundaries are reviewed against API declarations separately.

**No tests, test compilation, server runs, gameplay, live database calls or performance measurements were performed.** Compilation and static evidence are not runtime certification. Actual-server validation remains with the owner. The local release bundle includes compilation logs, API reference inspection and preservation evidence.

## Current upgrade

Stop the server normally, retain the existing data/configuration folders, and replace the TNE JAR. Keep Core 0.6.21 or newer within 0.6.x, or install it if the server has an earlier build. For 1.0.98, follow the coordinated account-migration instructions in [System account identities](SYSTEM_ACCOUNT_IDENTITIES.md) before starting any shared database writer. Remove duplicate older plugin JARs from the active plugins directory. Do not replace TNECORE or erase pairing files. Use the Java version required by both the server and the plugin.

## Local verification

Full unfiltered Maven clean verify passed on JDK 25: 919 tests discovered, 902 passed, 17 skipped, zero failures/errors. The system-account migration ran against local MariaDB 11.4 and H2, including exact balance preservation, replay, collisions and late-failure rollback. Skipped checks retain their existing disabled/environment requirements; live MySQL 8, Redis and gameplay/performance qualification are not claimed.

Compilation passed against Paper 1.20, Folia 1.20.1, Paper 26.2 and Folia 26.2. Static archive checks confirmed Java 21 bytecode, matching public API classes, and no bundled Core/Vault/server API or test classes. Website role-mapping checks, type checking and production build passed. No GitHub Actions were run.
