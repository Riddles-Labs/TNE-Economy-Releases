# Player leaderboards

Available in TNE-Economy 1.0.53. The configured command name defaults to `/tnetop`.

| Command | Result |
| --- | --- |
| `/tnetop` | Top ten balances in the configured default currency |
| `/tnetop coin` | Top ten balances in `coin` |
| `/tnetop gems 25 compact` | Top 25 gem balances using compact number formatting |
| `/tnetop all` | Overall average percentile ranks across currencies you can view |
| `/tnetop list` | Available currency IDs and menu help |
| `/tnetop gui` | Currency selector and overall-rank category |
| `/tnetop gui gems` | Player-head leaderboard for gems |
| `/tnetop gui all 2` | Second overall-rank page |
| `/tnetop currency all` | The actual currency named `all`, rather than the overall board |

Chat limits are 1–100; GUI pages show 21 players with Previous/Next. Existing
`/tnetop 25 compact` and `/tnetop compact` syntax still uses the default currency.
`full` selects ordinary formatted values. GUI numeric currency IDs work directly;
use `currency <id>` to disambiguate reserved IDs such as `all`, `gui`, or `compact`.
The explicit prefix also works after `gui`. Tab completion lists configured,
permitted currencies. Console senders can use the chat commands.

Wallet and Bank have a **Leaderboards** category. The selector lists currencies
and **Overall Average Rank**. Each ranking card uses the player's UUID as the
head owner, shows their name, position and balance/score, and retains the selected
currency's own precision. Offline names and textures use Minecraft's available
profile data; an unknown name falls back to the UUID. No blocking skin-web-service
lookup runs while constructing an inventory. Back restores the Bank or Wallet
parent; Refresh and pagination retain the selected leaderboard.

## Overall rank score

Money values from different currencies are not added or averaged. Instead, each
permitted currency produces a percentile rank on a 0–100 scale. Each currency
contributes equally to the final average, regardless of its denominations,
precision or exchange rate. This is a ranking score, not spendable money.

For a population of N players, let R be a player's descending competition rank
and T the number of players tied at that balance. Their currency score is:

```text
100 * (N - R - (T - 1) / 2) / (N - 1)
```

Ties share their midpoint percentile. A single-player population or a currency
where everyone ties contributes a neutral 50. Missing currency balances count as
zero; negative balances rank below zero. A configured currency with no balances
therefore contributes 50 to everyone and does not change their relative order.
The eligible population consists of non-system accounts with at least one stored
balance in a selected currency, in the installation's configured default scope.
Only the viewer's permitted, scope-compatible currencies participate; viewers
with different currency access can have different overall rankings. Up to 100
currencies can participate in the overall calculation.

For example, a player at the 100th percentile in coins and 0th in gems has an
average score of 50. A player at the 75th percentile in both has a score of 75,
even if coin balances are millions and gem balances are single digits.

Scores and equal per-currency balances use account UUID as a stable ordering
tiebreaker. Displayed positions are ordered list positions; equal scores retain
the same underlying score. Displayed scores round to two decimal places.

## Access and data

`tne.money.top` controls commands and menu access. Each currency also requires
its configured balance permissions and scope access. Negative Bukkit permissions
are resolved before loading data. Menus recheck permissions before showing a
loaded view. Issuance, Vault settlement, registered admin/wallet system accounts,
and federated market accounts are excluded before ranking and pagination.

SQL reads and average calculations run on workers. Database window functions
rank the full eligible population, not a truncated top-player sample. SQL returns
only the requested page, plus one row when the GUI needs to detect Next. Each
player can have one in-flight menu request; obsolete responses do not replace a
newly opened inventory. GUI heads and player-name reads run on the player's
scheduler. Refresh reads current committed data; new transactions can shift page
boundaries. These rankings do not issue currency or change balances.

Automated checks cover H2 and the configured disposable MySQL/MariaDB fixtures,
permissions, exclusions, ties, missing balances, currency precision, stable page
boundaries, command compatibility, head UUIDs and parent navigation. Live
Paper/Folia visual/gameplay qualification remains an operator beta check.
