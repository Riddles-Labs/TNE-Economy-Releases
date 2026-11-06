# System account identities — 1.0.98

The three built-in Vault and exchange accounts now use deterministic full UUIDs. Their identity is stable across restarts and servers sharing the same ledger. Each account still has independent balances for every currency and scope; this does not combine currencies or economies.

| Account role | Previous reserved ID | Current default ID |
| --- | --- | --- |
| Vault Checking | `00000000-0000-0000-0000-000000000001` | `f8513102-6047-3276-9f13-90545beaea99` |
| Currency Exchange Receipts | `00000000-0000-0000-0000-000000000002` | `76ac6899-b975-32d7-9359-ce8b8e5ba422` |
| Currency Exchange Payouts | `00000000-0000-0000-0000-000000000003` | `427b71f2-47cc-36ef-aee2-85376299d92e` |

The former `…002` account receives the source currency surrendered during conversion. The former `…003` account supplies the target currency paid to the player. They are distinct from Currency Mint, which records issuance and retirement. Custom account IDs configured by owners remain unchanged. TNE Checking, Server Savings, Server Rewards and issuance identities are not replaced by this migration.

## Upgrade existing servers

Stop every Minecraft server and proxy writer sharing the database, take a database/configuration backup, and upgrade them together. Do not mix old and new versions against this ledger. Migration refuses recent active nodes, conflicting destination identities, unsafe pending operations and unsupported nontransactional tables rather than guessing or merging balances. A recently crashed node can require its heartbeat to expire before cutover.

SQL bootstrap migrates structured account references in a transaction before financial readiness. Currency balances and ledger versions are checked for preservation. Historical signed items and hashed evidence payloads remain unchanged; durable old/new identity records explain their original IDs. The new commit boundary rejects retired IDs rather than creating replacement empty accounts.

Only after SQL migration and health verification succeeds does the plugin update the known Vault/exchange configuration references. Original files are retained as `config.yml.before-system-account-ids.bak` and `economy.yml.before-system-account-ids.bak` when changed. Each replacement is atomic. If saving a configuration file fails, financial startup pauses; correct file access and restart to resume. Do not restore only the database or only the configuration from a pre-upgrade backup.

## Native integration API

Use `com.cmriddles.thenewereconomy.api.account.SystemAccountIds` from API 1.0.98:

- `VAULT_CHECKING`: default Vault integration account.
- `CONVERSION_RECEIPTS`: default exchange source-currency receipt account.
- `CONVERSION_PAYOUTS`: default exchange target-currency payout account.
- `migrations()`: immutable previous-to-current identity map for evidence tooling.
- `canonical(UUID)`: translates only the three reserved historical IDs; custom UUIDs are returned unchanged.

`AccountId` itself never rewrites UUIDs. The mapping helper grants no spending authority. Do not overwrite signatures, receipt hashes or historical JSON to change displayed IDs. Do not hardcode these defaults where an owner may configure another account. Native shared-account payments should use the owner-authorized `client.checking()` service, which routes through separate funded TNE Checking and supplies integration attribution.

Minecraft runtime requirements remain Paper 1.20+ / published Folia 1.20.1+ through the documented 26.2 baseline, RiddlesLabsCore 0.6.21+ within 0.6.x, and Java 21 or the newer Java runtime required by the server. Build tooling uses JDK 25 with Java 21 bytecode.
