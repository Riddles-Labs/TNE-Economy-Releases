# In-game economy menus

TNE uses a consistent six-row inventory layout with framed edges, overview cards,
hover details, previous/next controls, and a navigation bar. No setup wizard is
part of these menus. Existing command aliases from `config.yml` still apply.

## Category navigation

Root menus show category cards. Lists, filters, and financial actions live inside
their own sections. The star at the top shows the current path; Back returns to
the parent section. Bank, Wallet, and authorized Admin shortcuts remain in the
footer, beside Refresh and Close. Lists use Previous/Next at the outer corners.

| Entry | Categories | Detail screens |
| --- | --- | --- |
| `/bank` or `/tnemybank` | Account Overview, Payment Requests, Transactions, Wallet | All-currency sunflower balance preview and paged balances; account identity/scope/version; Received Requests and Sent Requests; filtered history |
| `/wallet` | Balances, Transactions, Exchange, Wallet Details, Leaderboards | Paged currency balances; filtered history; conversion rates; account and scope information |
| `/tneadminmenu` | Transactions, Accounts, Currencies | Global ledger and account audits; account management; currency type and precision |
| Admin > Transactions | All Transactions, Account Audits, Find Receipt | Paged cross-account ledger; account selector and incoming/outgoing filters; exact ID lookup |
| Receipt | Ledger Entries, Transaction Context (Audit Context for admins) | Every debit/credit and scope; public or full operator metadata |

Admin Accounts opens a paged balance list; selecting an account opens its detail
screen with Grant +1, Debit -1, and Audit Transactions when permitted. No adjustment
buttons appear on category hubs or the account list. Audit selectors cover accounts
with balances in the default currency; direct lookup can inspect other accounts.

Received request lists align a request card with Accept and Decline below it.
Sent requests have Cancel beneath each card. Lists page independently, seven
requests at a time. Request details include requester, payer, amount, currency,
message, status, creation time, expiry, and request ID.

Accept, decline, cancel, conversion, and admin adjustment actions open a review
screen. Cancel and completion return to the relevant section. Each confirmation
accepts its confirm click once, preventing rapid duplicate submissions.

`/bank history` still opens personal history directly. History shows 21 committed
transactions per page, newest first, with All / Incoming / Outgoing filters across
currencies and scopes. A conversion can appear in both directional filters. Wallet
history remembers its Wallet parent; receipt Back restores the history page/filter
or the global ledger page. Players can only open transactions they participated in.

`/exchange` opens the exchange list directly. Rates and 1-unit conversions remain
available through Wallet > Exchange. Use `/tneconvert` for custom amounts and
conversions requiring rounding/adjustment confirmation. Rates are base rates before
fees and rounding, not final quotes.

Direct admin shortcuts remain available:

- `/tneadminmenu <player-name|account-uuid>` opens an account audit.
- `/tneadminmenu receipt <transaction-uuid>` opens an operator receipt.
- Find Receipt supplies an editable chat command so the administrator can enter a
  full transaction ID without an inventory text-entry wizard.

`tne.admin.menu` controls the admin overview. `tne.transaction.info` controls
transaction views; `tne.transaction.info.others` additionally controls cross-account
audits and operator receipts. `tne.admin.adjust` controls adjustments. Navigation
and adjustment controls are hidden when their permission is absent, and permissions
are checked again when opening protected views or executing an adjustment.
Authorized operator audits can read historical records for accounts that no longer
have player wallet eligibility; ordinary player history retains its eligibility check.

Operator receipts include every stored metadata field plus the idempotency key.
Player receipts show public context fields only: `reason`, `message`, `operation`,
`adapter`, `reversalOf`, `requestId`, `paymentRequestId`, `originalTransactionId`,
`originalSourceNamespace`, `action`, `senderName`, `recipientName`,
`sourcePlayer`, and `targetPlayer`. Other metadata stays in the operator view.

New GUI admin adjustments persist `actorUuid`, `actorName`, `reason`, action, target,
and displayed ledger version in the existing transactional audit/ledger metadata.
Their server log includes actor UUID, correlation ID, target, amount, and transaction
ID. Accepted GUI payment requests also capture actor identity and log the
correlation ID linking the request to its committed transfer.
Historical records only contain metadata captured when they were committed;
missing actor details are not inferred. These menus browse committed ledger
transactions, not a separate feed of failed commands or uncommitted attempts.

History uses SQL filtering and `LIMIT/OFFSET` paging rather than loading the entire
account ledger. Each page reads one additional record to decide whether Next should
appear. New transactions arriving during browsing can shift offset page boundaries;
refresh returns the current page's latest committed data.

## Player-head leaderboards (1.0.53)

Wallet and Bank include **Leaderboards** with per-currency rankings and an
**Overall Average Rank** category. `/tnetop gui` opens the same menu. Pages show
21 named player heads, balances or rank scores, and retain their parent menu.
See [commands, permissions and scoring rules](LEADERBOARDS.md).

## Verification

Automated checks cover filtered SQL paging, player isolation, operator permission
checks, complete multi-page receipt metadata, history return state, menu slot
uniqueness, and single-use confirmation. On a Paper/Folia test server, inspect the
layout with a normal player and an admin, exercise payment request confirmations,
and check shift-click/drag cancellation and refresh/back behavior.

## Compact tooltips and readable accounts (1.0.58)

Every inventory item name and lore line wraps at 32 visible characters. Long IDs
continue onto the next line without losing characters. Colors and events survive
wrapping. Wallet bag icons omit vanilla bundle contents so Minecraft does not add
an Empty preview. Wallet shortcut status follows the current menu.

Account fields show player names and saved account labels for the exact currency
and scope; known system accounts show their role. Historical names are a fallback
when the server has no current name. Unknown accounts keep their UUID. Admin
receipt entries retain an Account ID detail, while transaction IDs remain visible.

The bank balance card previews up to seven configured currencies plus an overflow
count. Click it for the complete balance list, 21 currencies per page. Each uses
its own decimal precision; Refresh retains the page and Back returns to Bank.

## Currency icons (1.0.59)

Currency and balance cards use SUNFLOWER in Wallet, Bank, the admin currency
catalog and leaderboard selection. If the running API lacks a usable sunflower
item, the cosmetic icon falls back to GOLD_NUGGET, then GOLD_INGOT, then PAPER.
The plugin’s Java/API requirements still apply.
