# TNE-Economy releases

Official compiled plugin downloads for Paper and Folia. Download the latest TNE-Economy jar from [Releases](https://github.com/SlyRiddles/TNE-Economy-Releases/releases/latest).

TNE-Economy 1.0.11 and newer download verified updates directly from this repository and install them on a normal server restart. The source code remains in a separate private repository. No GitHub token is required by server owners.

Each release includes the jar, a SHA-256 checksum and tne-economy-latest.json used by the updater.

## Unified Wallet and Bank (1.0.61)

`/wallet`, `/bank`, `/tnemybank` and configured bank aliases open the same Wallet
menu. Currency flowers remain at the top. Account Overview, Payment Requests,
Leaderboards and Transactions share a centered row, with Exchange centered below.
One Wallet footer shortcut replaces the separate Bank and Wallet links. Details,
balances, requests, history and exchange return to the same player menu.

The existing `tne.menu.wallet`, `tne.menu.bank` and `tne.menu.mybank` nodes are
alternative entry grants. Deny all three to deny shared-menu entry. Financial,
transaction, exchange, leaderboard and admin permissions still apply separately.
`/wallet history` and `/bank history` share history; `/wallet web` linking remains.

Install 1.0.61 while Minecraft is stopped and restart normally. No core update or
migration is required. See [in-game menus](IN_GAME_MENUS.md).

## Separate bank currency flowers (1.0.60)

`/bank` and `/tnemybank` now show each server currency as its own sunflower across
the top row, from left to right. Each flower shows that currency’s name and balance
with its own precision. Seven fit on a row page; Previous/Next browse the rest and
Refresh keeps the current page. All Balances opens the full currency list.

Hover text now wraps between words and keeps long names and IDs whole. My Bank
keeps one Wallet shortcut in the footer. Leaderboard Player ID details require
`tne.admin.account.inspect`; regular players see names, ranks and balances.

Account Overview keeps account identity, scope and ledger version below the flowers.
The existing material fallbacks and compact hover formatting remain included.
Install 1.0.60 while Minecraft is stopped and restart normally. No core update or
migration is required. See [in-game menus](IN_GAME_MENUS.md).

## Sunflower currency icons (1.0.59)

Currency and balance cards now use sunflowers across Wallet, Bank, admin currencies
and leaderboard selection. If SUNFLOWER is missing or is not a usable item on the
running API, icons fall back to GOLD_NUGGET, then GOLD_INGOT, then PAPER. The
plugin’s declared Java/API requirements still apply.

Install 1.0.59 while Minecraft is stopped and restart normally. No core update or
database migration is required. Compact hover text and readable account names
remain included. See [in-game menus](IN_GAME_MENUS.md).

## Compact menus and readable accounts (1.0.58)

Wallet bags keep their icon and TNE details without Minecraft’s misleading Empty
contents preview. Their status follows the wallet screen. All menu item names and
hover text wrap at 32 visible characters, preserving full values and MiniMessage
colors instead of cutting them off.

Ledger, history, receipts and account menus show player names and existing account
labels for the exact currency and scope. Known system accounts show their role;
unknown identities retain a UUID fallback. Admin receipt entries retain account IDs
and chat receipt names can copy the original UUID. Transaction IDs and stored
ledger data stay intact.

The gold-bar Balances card in `/bank` and `/tnemybank` previews configured server
currencies, with an overflow count and a complete paged list (21 per page). Each
currency uses its own precision; Back, Refresh and pagination stay within Bank.
See [in-game menus](IN_GAME_MENUS.md).

Install 1.0.58 while Minecraft is stopped and restart normally. No core update or
database migration is required. Automated checks do not certify live client
rendering or gameplay qualification; this remains a production-testing beta.

## Folia leaderboard completion crash fix (1.0.57)

Fixes leaderboard autocomplete reading `args[0]` when Paper/Folia supplies an empty
argument list. This exception could escape into a region tick. Root completions
now handle both zero arguments and an empty first argument correctly, retaining
permission filtering and currency capitalization. Versions 1.0.55 and 1.0.56 both
contain the reported bug; install 1.0.57 while Minecraft is stopped, then restart.

The shared adapter also contains unexpected runtime exceptions from TNE's bound
completers, returns no suggestions on failure and throttles repeated warnings.
The release retains the currency display and startup-banner fixes. No core update,
balance reset, re-pairing or database migration is required.

## Consistent currency capitalization (1.0.56)

Currency names use initial capitals regardless of input case: `coins`, `COINS`
and `cOiNs` show as `Coins`; `TNE` remains uppercase. Balances/payments, Wallet/Bank
and exchange menus, leaderboards, receipts/history, money notes, notifications,
Vault names and display placeholders share the formatting. MiniMessage colors,
gradients, numeric precision and click actions are retained.

Currency IDs, permissions, database keys, menu payloads and note metadata are
unchanged. `%tne_currency_name%` supplies the default display name;
`%tne_currency%` remains the raw ID. The boxed startup banner is retained.
Install 1.0.56 and restart normally. No additional core update or migration is needed.

## Restored startup banner (1.0.55)

Restores the organized boxed TNE startup console report, with version, developer
`cmriddles`, platform/Java, node/server identity, setup mode, integration states,
economy/SQL/TNECORE connections, pairing, heartbeat, outbox and Redis. Original
section borders, aligned columns and status/setup guidance are preserved with
MiniMessage colors. Install 1.0.55 and restart normally; no additional core update
or re-pairing is needed. Other detailed reports and the Folia menu fix are retained.

## MiniMessage reports and Folia menu fix (1.0.54)

TNE-owned Paper/Folia console logs and in-game messages now share MiniMessage
formatting, readable fields, error references and clearer startup/service reports.
Connection warnings distinguish timeouts and rejected requests, limit repeated
warnings and report recovery details. A timeout does not claim that the core updated.
TNECORE 1.0.45 supplies actual version/process metadata for confirmed core version
changes and possible restart/backend-switch notices. Older cores remain compatible.

`logging.colors: true` controls console colors. `updates.notify: true` checks signed
releases and alerts administrators without downloading a JAR. `updates.enabled`
remains the separate opt-in for verified downloads and installation on normal restart.
Templates live under `messages.reports` in messages.yml; existing custom values remain.
See [report formatting and configuration](REPORTING.md).

Fixes the CHAIN icon crash on Folia 26.2 by resolving IRON_CHAIN/CHAIN across versions.
Unavailable cosmetic icons fall back safely instead of preventing the menu opening.
Install plugin 1.0.54 while Minecraft is stopped, then restart. Core 1.0.45 only needs
to be installed/restarted for the new core identity reports; no re-pairing is required.

## Player-head leaderboards (1.0.53)

Use `/tnetop <currency>` for currency balances, `/tnetop all` for an equal-weight
average percentile rank, and `/tnetop gui` for the player-head menu. Wallet and
Bank also have Leaderboards categories. Rankings are paged, use each currency's
precision, exclude system accounts, and respect currency permissions.

The overall value is a 0–100 rank score, not combined money: each visible currency
counts equally, tied balances share midpoint percentiles, and missing balances
count as zero. A tied or single-player currency contributes a neutral 50.
Existing `/tnetop 25 compact` syntax remains available. Install the signed 1.0.53
server JAR and restart Minecraft; no TNECORE update or re-pairing is required.

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
