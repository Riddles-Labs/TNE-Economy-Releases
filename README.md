# The Newer Economy

Economy plugin for Paper and Folia, with multi-currency accounts, payments,
transaction history, in-game menus and a public integration API.

## Downloads

[Download TNE 1.0.79](https://github.com/Riddles-Labs/TNE-Economy-Releases/releases/download/v1.0.79/TNE-Economy-1.0.79.jar)
· [All releases](https://github.com/Riddles-Labs/TNE-Economy-Releases/releases)

Server owners need **TNE-Economy-1.0.79.jar** and **RiddlesLabsCore 0.6.17**.
API and SDK downloads are for plugin developers and do not belong in the server's
`plugins` folder.

### Requirements and installation

- Target platforms: Paper 1.20–26.2 and available Folia releases from 1.20.1–26.2.
- Use Java 21 for older servers, or Java 25 where the server requires it.
- Install RiddlesLabsCore 0.6.17 alongside TNE.
- Back up your server, stop it normally, replace the old TNE JAR and restart.
  Keep existing configuration and database files, and only one TNE JAR.

TNE creates player wallets automatically. In-game economy access does not require
website signup or a live TNECORE connection. Website wallet linking, reports and
website management use the separate TNECORE service.

This is a beta release. Automated checks pass; live gameplay validation across
the target server versions is still in progress.

## Plugin developers

Start with the [API guide](API.md) and [example plugin](plugin-api-example/README.md).
The API provides accounts, payments, receipts, reservations, recurring payments,
refunds, invoices and durable events.

Maven coordinate: `com.cmriddles:the-newer-economy-api:1.0.79`.
Use `provided` or `compileOnly`; do not bundle or relocate the API classes.

The [public Maven repository](https://raw.githubusercontent.com/Riddles-Labs/TNE-Economy-Releases/main/maven/)
and [release downloads](https://github.com/Riddles-Labs/TNE-Economy-Releases/releases/tag/v1.0.79)
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
