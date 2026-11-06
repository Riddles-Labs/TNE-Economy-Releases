# The Newer Economy

Economy plugin for Paper and Folia, with multi-currency accounts, payments,
transaction history, in-game menus and a public integration API.

## Downloads

[Download TNE 1.0.98](https://github.com/Riddles-Labs/TNE-Economy-Releases/releases/download/v1.0.98/TNE-Economy-1.0.98.jar)
· [All releases](https://github.com/Riddles-Labs/TNE-Economy-Releases/releases)

Server owners need **TNE-Economy-1.0.98.jar** and **RiddlesLabsCore 0.6.21 or newer within 0.6.x**.
API and SDK downloads are for plugin developers and do not belong in the server's
`plugins` folder.

### Requirements and installation

- Target platforms: Paper 1.20–26.2 and available Folia releases from 1.20.1–26.2.
- Use Java 21 for older servers, or Java 25 where the server requires it.
- Install RiddlesLabsCore 0.6.21 or newer within 0.6.x alongside TNE.
- Back up your server, stop it normally, replace the old TNE JAR and restart.
  Keep existing configuration and database files, and only one TNE JAR.

TNE creates player wallets automatically. In-game economy access does not require
website signup or a live TNECORE connection. Website wallet linking, reports and
website management use the separate TNECORE service.

This is a beta release. Local verification on JDK 25 discovered 919 tests:
902 passed, 17 skipped, zero failures or errors. Migration checks ran against
isolated MariaDB 11.4 and H2, covering exact balances, replay, collisions and
late-failure rollback. Compilation passed against Paper 1.20, Folia 1.20.1,
Paper 26.2 and Folia 26.2. These checks do not certify live gameplay on every
server version, live MySQL 8, Redis or performance. No GitHub Actions ran.
See [Minecraft compatibility](MINECRAFT_COMPATIBILITY.md).

## Upgrading to 1.0.98

**Back up your economy database and configuration. Stop every Minecraft server,
proxy and other writer sharing that database, and upgrade all writers together.**
Start one upgraded server first to complete migration, then start the others.
Do not run old and new versions together. If a recently stopped node still has
an active heartbeat, allow it to expire before retrying.

Vault Checking and the two exchange accounts now use stable full UUIDs. Existing
balances and structured history migrate together; custom configured IDs and
owner nicknames stay intact. Historical signed items and immutable exported
evidence retain their original identities. Follow the
[account identity migration guide](SYSTEM_ACCOUNT_IDENTITIES.md) for recovery
and configuration backup details.

The former account ending **0002** is **Currency Exchange Receipts**: it receives
the source currency surrendered during a conversion. **Currency Exchange Payouts**
(formerly 0003) supplies the destination currency. Neither is Currency Mint.
TNECORE's separate central ledger is unchanged; this migration requires no
TNECORE replacement.

## Plugin developers

Start with the [API guide](API.md) and [example plugin](plugin-api-example/README.md).
The API provides accounts, payments, receipts, reservations, recurring payments,
refunds, invoices and durable events.

Maven coordinate: `com.cmriddles:the-newer-economy-api:1.0.98`.
Use `provided` or `compileOnly`; do not bundle or relocate the API classes.

The [public Maven repository](https://raw.githubusercontent.com/Riddles-Labs/TNE-Economy-Releases/main/maven/)
and [release downloads](https://github.com/Riddles-Labs/TNE-Economy-Releases/releases/tag/v1.0.98)
provide the API binary, API sources, Javadoc and SDK with examples.

- [Workflows and scheduled payments](API_WORKFLOWS.md)
- [Access controls and refunds](API_ACCESS.md)
- [Invoices, quotes and money notes](API_COMMERCE.md)
- [Events and extension points](API_EXTENSIONS.md)
- [Vault compatibility](VAULT_COMPATIBILITY.md)

## Download contents

`TNE-Economy-<version>.jar` is the server plugin. `TNE-API-*` and
`TNE-Developer-SDK-*` are integration resources. `.sha256` files verify downloads;
`tne-economy-latest.json` is the signed manifest used by TNE's updater.

The plugin implementation is maintained in a private repository. Public source
JARs contain only the integration API. GitHub's automatic **Source code** archives
contain this download repository's documentation, API artifacts and example code;
they do not contain the plugin implementation.
