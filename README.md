# TNE-Economy releases

Official compiled plugin downloads for Paper and Folia. Download the latest TNE-Economy jar from [Releases](https://github.com/SlyRiddles/TNE-Economy-Releases/releases/latest).

TNE-Economy 1.0.11 and newer download verified updates directly from this repository and install them on a normal server restart. The source code remains in a separate private repository. No GitHub token is required by server owners.

Each release includes the jar, a SHA-256 checksum and tne-economy-latest.json used by the updater.

## Category menus and transaction audits (1.0.52)

Adds six-row Bank, Wallet and Admin menus with category cards, detail views,
breadcrumbs, Back/Refresh controls and pagination. Admin > Transactions opens the
global ledger, account audits and receipt lookup. Players can filter their history
and inspect their own receipts; administrators can inspect the complete audit
context. Financial actions have a confirmation screen.

This is an operator production-testing beta; manual live gameplay qualification
remains incomplete. The server JAR is built and tested by a persistent VM runner,
then independently rebuilt and compared before offline signing and publication.
The GUI overhaul first appears in **1.0.52**; 1.0.51 retains the earlier menus.
Wait for UPDATE READY, then restart Minecraft, or stop the server and replace the
old JAR manually. Keep only one TNE JAR and preserve the plugin data folder.
No TNECORE update or re-pairing is required for these menus.

## Detailed server-owner controls (1.0.23)

Restores local issuance, treasury and reward funding, Vault settlement and conversion reserves, account labels, currency rules/conversion settings, receipts, payment cancellation, money-note refunds, command/owner audit and console-approved currency resets. Pre-funded supply blocks new issuance while allowing funded payouts. Existing balances and pairing remain intact; no currency is issued automatically.

Use TNECORE 1.0.23 and deploy the updated website. Restart TNECORE first, then restart Minecraft after UPDATE READY. Open your server console and choose **Funding and system accounts** to review the accounts and funding workflow.

## Player wallet dashboard (1.0.18)

Adds server icons, currency catalogs and durable transaction history for player wallets. With TNECORE 1.0.23 and the updated website, players get server wallet cards, a global currency collection, recent and complete transaction history, scoped stats and saved organization. Existing balances and pairing remain intact. History catches up after updating; older unrecorded post-transaction balances remain unknown. Restart TNECORE and deploy the website, then restart Minecraft after UPDATE READY.

## Wallet registration compatibility (1.0.17)

Fixes the plugin rejecting numeric registration-link expiry values and adds useful wallet failure details in the Minecraft console. Credentials and registration links are excluded from diagnostics. TNECORE 1.0.23 also fixes responses for older installed plugins. Restart TNECORE, install the plugin update after UPDATE READY, then retry /tne register wallet. No website change is needed.

## Faster website console (1.0.16)

Use TNECORE 1.0.18 and the latest website. The plugin now answers website reads through a live connection instead of waiting for the setup polling cycle. Concurrent player and item queries run together. Visited website views remain visible during background refreshes. Restart Minecraft after UPDATE READY to install this release.

## Matching developer and owner action screens (1.0.15)

With TNECORE 1.0.17 and the latest website, server owners use the same Balance and item actions screen as developers, including player search, Minecraft item icons/categories, multiple selected stacks, previews, delivery status, and confirmation. Owner data comes from the selected installation. Up to 20 item stacks, including potion variants, are queued together in one transaction; retries cannot duplicate them.

## Server status and owner controls (1.0.14)

TNE-Economy 1.0.14 reports Minecraft version and local event counts about every 30 seconds. With TNECORE 1.0.16 and the latest website, the developer server directory uses the installation connection for online status, SQL health, and plugin version.

Server owners can review and submit local balance adjustments, transaction reversals, item/XP deliveries, and failed-delivery recovery from their own server console. Requests are scoped to the owned installation and saved with a stable ID; reconnecting or resending cannot apply them twice. Player targets must be registered locally. Local grants do not issue central TNE backing.

Restart TNECORE to update it, deploy the latest website, then restart Minecraft once the verified plugin update is ready.

## Versioned installed filenames

Starting with 1.0.13, automatic updates install as `TNE-Economy-<version>.jar`. Paper/Folia applies the verified update and renames the old JAR during startup. Older pending TNE downloads are removed while unrelated plugins' updates are preserved.

When upgrading from 1.0.12 or earlier, the old updater initially keeps the old filename. After 1.0.13 starts, wait for **FILENAME UPDATE READY**, then restart once more to correct it. Future updates change the version and filename together in one restart. No TNECORE or website update is needed.

## Install the updater fix once

Versions before 1.0.11 can announce an update but reject downloading it because Paper/Folia reports API `26.1.0` while the manifest uses `26.1`. Version 1.0.11 fixes that comparison.

If you are stuck on an older version, stop Minecraft, replace the existing TNE plugin JAR with the latest release, and start Minecraft. Keep your plugin data/config folder and only one installed TNE JAR. Future compatible updates download automatically and install on restart. With the default Bukkit setting, pending updates go in `plugins/update`; `plugins/TheNewerEconomy/updates` holds state and backups.

## Live update notifications

TNE-Economy 1.0.8+ connects to TNECORE 1.0.11+ for live release signals. Publishing a stable release runs the **Notify Minecraft servers** workflow, which wakes connected plugins through TNECORE. Owners see current/newest versions and a restart instruction after the verified download. The workflow can also be run manually after upgrading TNECORE.

The signal contains no release instructions or credentials. Plugins independently verify this repository's latest manifest and JAR. Periodic checks remain available if the live connection is unavailable. Source commits without a published JAR do not announce an installable update.
