# Server owner economy controls

Requires TNE-Economy 1.0.19, TNECORE 1.0.21 and the updated website. Select **Server console → your server**. The overview remains the entry point; the server tools operate on that installation's local SQL economy. Central issuance, onboarding reserves and identity approval remain developer controls.

## Set up funding

1. Finish database and currency setup. Open **Funding and system accounts**.
2. Choose a currency and review Issuance, Treasury, Rewards, Vault settlement, Conversion settlement and Conversion liquidity. Configured fee/tax accounts also appear. Existing account IDs and balances are retained.
3. Use **Issue supply here** on Treasury to create the initial local supply. Nothing is minted automatically. The issuance account's negative balance records the offsetting supply.
4. Use **Transfer / reward** to fund operating accounts or pay a registered player. The source must have sufficient funds. Use its UUID from the account card; registered player UUIDs appear in Players and balances.
5. Fund Vault settlement before plugins make Vault deposits. Vault withdrawals return funds there. Fund the target currency in Conversion liquidity before players convert.
6. For a fixed circulating pool, open **Currency rules and conversions**, select **Pre-funded**, save and restart after the server confirms staging. New supply from admin/console/plugin ledger grants is then rejected. Existing funded transfers and retirement remain available. Manual issuance is the compatible default and can be restored through the reviewed config process.

Account labels and purposes can be edited. Role privileges are fixed; changing a label cannot give an account issuance privileges. **Retire supply** removes an amount from a funded operating account and returns its offset to issuance. Local issuance never creates central TNE backing or changes an operator's central reserve.

## Rules and conversions

Edit display names, symbols, permission prefixes, minimum/maximum transaction amounts, conversion pairs, fee/tax fractions and receiving account UUIDs. A 0.02 fee is 2%. Each conversion direction needs its own rate. Existing IDs and decimals cannot be redefined through guided setup.

Changes use the encrypted local setup channel, configuration revision check, database test, file backups and save/restart confirmation already used by server setup. Manual YAML remains supported in `economy.yml`:

```yaml
economy:
  currencies:
    coins:
      display-name: Coins
      symbol: C
      permission: tne.currency.coins
      min-transaction: '0'
      max-transaction: ''
      funding-policy: MANUAL_ISSUANCE
```

`PRE_FUNDED` disables supply creation at the SQL commit boundary. Currency permissions and transaction bounds apply where a command invokes currency access policy; the web owner's administrative actions retain their explicit ownership authorization. Vault's API does not reliably identify the plugin calling it, so this release does not invent per-plugin attribution or budgets.

## Day-to-day controls

- **Players and balances:** player/system classification, saved system labels, cached player names, balances and account history.
- **Ledger / Transaction receipt:** individual balancing entries, source, correlation, actor/reason/operation metadata and reversal through compensating entries.
- **Payment requests:** scope/message details and cancellation of pending requests.
- **Money notes:** invalidate an issued note and return its verified original backing to its issuer. Already redeemed/invalidated notes cannot refund again. Unverifiable legacy backing is rejected for manual investigation.
- **Balance and item actions / Asset recovery:** balance adjustments, item/XP delivery, and reversals remain available. A failed delivery may be retried directly only when the server proves its effect was `NOT_APPLIED`. `REVIEW_REQUIRED` operations show an attempt ID and failure details; the owner must inspect actual assets and record an evidence reference before confirming received or missing. The latter queues a new claimed attempt. **Asset decisions** shows the durable reconciliation audit.
- **Audit / Owner action history / Command audit:** committed financial records, durable owner request outcomes, and new player/console command invocations. Command invocation alone does not prove success; use its financial receipt. Command audit begins after upgrading; it cannot reconstruct old commands.
- **Connection health / Sync event history:** local SQL/integration status and waiting/published event timestamps. Health remains readable when SQL is unavailable. Read views refresh every 15 seconds while open.

All owner operations are authenticated by TNECORE, limited to configured local currencies/scope, and journaled atomically with their writes. A saved request can be resent after a connection failure without repeating the operation. The note/payment state claim and financial transaction commit together, preventing stale cancellation/redemption from paying twice.

## Reviewed currency reset

In **Settings and controls**, prepare a currency reset review, run `tneweb local-reset <review-id>` in the Minecraft **server console**, then apply that review on the website. In-game players cannot approve it. Reviews expire after ten minutes; any intervening balance/version change invalidates the snapshot. Close issued notes, pending requests and unfinished deliveries first. Interactive reset is limited to 5,000 balance rows.

Reset appends one balancing transaction that sets the chosen currency/scope's balances to zero. It preserves original ledger history, other currencies, local configuration, server pairing, and central player identities/reserves. The pre-reset amounts remain in the reset's ledger entries. This replaces the old global factory-reset behavior for server owners.

## Upgrade order

Restart TNECORE to install 1.0.21. Deploy the website's latest `main`. Wait for **UPDATE READY** on Minecraft, then restart to install TNE-Economy 1.0.19. Source pushes alone do not deploy the live website.

## Guided funding setup (TNE-Economy 1.0.20 / TNECORE 1.0.22)

Server setup ends with **Funding setup**, after configuration is loaded and the database is connected. Existing servers show **Finish funding setup** until their owner reviews each configured currency. Existing account IDs, balances, pairing and transaction history are retained.

1. Choose whether each currency uses the funded Rewards pool. For the default currency, choose whether you use Vault payouts. Vault is initially suggested for the default currency; the owner must confirm the plan. These choices record intended use; they do not enable or disable integrations.
2. Review current Treasury, Vault settlement, Rewards and Conversion liquidity balances. Issue local supply only with an explicit amount and confirmation, or transfer existing Treasury funds. Pre-funded currency policy blocks issuance; change the policy deliberately in Currency rules and conversions before issuing any new supply.
3. Confirm the plan. The Minecraft server verifies that selected payout accounts have positive balances. Configured conversion targets always require positive liquidity. The review itself creates no currency and survives restarts.
4. Test a small payout through the actual integration. Configure reward plugins/commands to spend the selected account; recording a Rewards plan does not reconfigure third-party plugins. A positive balance confirms basic funding, not that every future payout can be covered.

Vault deposits spend settlement funds. Console/admin grants issue local supply when manual issuance is allowed. Funded transfers spend existing balances. Local issuance never adds central TNE backing or draws central reserve funds.

For wallet transfers/manual grants only, optional payout features can be left off and the plan reviewed without creating supply. New currencies or changes to required account roles/conversion targets require another review. If a selected account is later depleted, **Replenish funding** appears. Runtime health is reported separately; missing funding review does not pause an existing working server.

The owner overview counts an economy ready only with fresh runtime health and completed funding review. Reports normally arrive every 15 seconds; the funding step checks every 5 seconds. Older plugins show an update prompt rather than completed funding. Deploy the website and update TNECORE before installing this plugin release because older TNECORE versions reject new setup-report fields.

## Multiple payment methods and starting-funds planner (plugin 1.0.21 / core 1.0.23)

Select all intended methods independently: Vault plugins, direct purchase grants (such as Tebex), admin grants, custom rewards from a separate account, and player transfers. These are recorded intentions; they do not install/configure integrations or change command permissions or currency supply policy. Direct grants require manual issuance. Ordinary reward plugins using Vault should be counted only in the Vault payout budget.

Planning inputs are per currency: expected daily active players, coverage days (1–30), buffer percentage (0–100), daily payout per player for each selected pool, and expected returns to that same pool (0–100%). Seven days and a 20% buffer are editable starting assumptions; monetary payout amounts remain blank. A fresh count of players with economy transactions during the last 24 hours can be copied explicitly; it is not all daily visitors or a forecast. The plugin does not read third-party payout configuration.

The calculator uses exact minor units at the currency precision. Gross daily payouts = expected players × daily payout per player. Net daily outflow = gross × (100 − return percentage)/100, rounded up. Suggested starting balance = max(one gross payout day, net daily outflow × days) × (100 + buffer)/100, rounded up. Existing funds are subtracted to suggest only a top-up. Inputs exceeding supported precision or amount bounds do not produce a suggestion.

Below-estimate funding warns about needing to refill before the chosen period and requires acknowledgment. Funding above twice the estimate asks the owner to check the larger amount. Both allow deliberate continuation. A changed amount or refreshed current balance resets acknowledgment. Zero required balances still fail the Minecraft-side review. A missing estimate does not invent a minimum; owners may explicitly acknowledge choosing their own funded balance. Existing savings are offered before creating new local currency.

Funding once is possible only if enough money returns to the account making payouts. Player transfers and store receipts do not automatically replenish that account. Vault withdrawals return to its settlement account; custom pools require their configured return path. With net outflows, any finite pool eventually depletes. The estimates are neither spending limits nor guarantees.

Choices and assumptions persist with the currency/scope funding review in SQL. No balances are created or changed by saving preferences. Existing reviews migrate without losing data; older review requests preserve saved planning preferences. Core advertises planner support, so an older core cannot silently drop these settings. Deploy Core 1.0.23 before plugin 1.0.21, and deploy the updated website.

## Prices, earnings and premium packages

Currency setup uses **Standard Currency** for everyday gameplay money and **Premium Currency** for a separate currency such as store tokens. Existing GAMEPLAY configuration values remain compatible. The labels appear in creation, setup review, funding and currency rules.

In funding step 2, enter expected daily earnings per player, a typical everyday purchase price and a larger purchase price. The comparison shows earning days needed if the player starts at zero and saves all earnings. These are owner estimates, not measured shop averages. An optional button copies the sum of Vault and separate reward estimates; do not count the same rewards in both. Shop prices do not increase the payout pool estimate automatically.

Premium currencies ask whether they are sold on the owner's website. Save up to five representative packages, each with a three-letter payment currency, positive price (up to two decimals) and amount delivered at the game currency's precision. For example, 25 USD gives 32,000 tokens. Include bonuses in the amount. Selecting Yes also selects website purchases as a payment method. Selecting No retains packages as inactive. Pricing records do not process purchases, mint money, configure delivery commands, or set TNE exchange rates. Configure the store integration separately.

The review saves estimates and packages per currency and economy scope, survives restarts, and preserves legacy payout plans. The plan is bounded to 512 characters; shorten unusually long amounts if the form reports that limit. Requires TNE-Economy 1.0.22, TNECORE 1.0.24 and the updated website. Update TNECORE first.

## Total daily spending

The planner now asks for average total spending per player per day: all routine purchases combined. Savings equal daily earnings minus daily spending; the larger purchase estimate uses these savings and rounds up to whole days. For example, 100 earned minus 60 spent leaves 40 saved each day; a 1,000 goal takes 25 days. Spending above earnings shows a daily shortfall. Missing spending remains unknown; enter 0 explicitly when appropriate.

Previously saved single-purchase prices are shown for reference with daily spending left blank, so existing plans are not silently reinterpreted. Save a new estimate using TNE-Economy 1.0.23, TNECORE 1.0.25 and the updated website. Premium packages and other planning choices are preserved. Daily spending does not imply money returns to a payout account.

## Shared funding guide and action progress

The funding guide asks for expected daily players, coverage days and buffer once per server. These apply to every currency. Each currency keeps its own payout amounts and methods; the owner reviews and saves each updated plan through the existing server action. Currency navigation shows which plans still need review. Shared and currency drafts are kept in this browser until all plans are saved, so returning after an interrupted request does not require re-entering estimates.

Each account's estimate, suggested top-up, current funds and funding controls appear together. Expandable question boxes explain player counts, separate reward accounts, payment methods, calculation examples and circulating funds. Earnings and spending comparisons are optional. Separate rewards are a server-owned account of the same local currency, funded by the owner; Vault reward plugins should be counted under Vault only.

Owner actions show sending, pending confirmation, reconnecting, confirmed or failed states, plus elapsed checking time and the latest response time. A network interruption is not reported as a failed transaction. Pending requests are restored using their saved request ID; resending the same request is offered when a connection error needs recovery. After a confirmed action, the guide waits for a new server report before enabling additional funding, and explains the delay instead of displaying an estimated balance as confirmed.

This is a website update using the existing TNE-Economy 1.0.23 and TNECORE 1.0.25 interfaces. Deploy the updated TNE-WEB-Panel main branch. Validation covers shared counts across two currencies, exact funding amounts, delayed responses, connection interruption, refresh recovery, failure messages, stale balance guards, saved plans and mobile layout.
