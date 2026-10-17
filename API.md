# TNE API for plugin developers

Use the native Java API to read player balances, transfer money, or credit and
debit players from your Paper/Folia plugin. You do not need TNECORE credentials,
HTTP requests, database access, or a TNE command to do this.

The entry point is `TneApi`. Get it from Bukkit's `ServicesManager`, then create
one client using your plugin's name. Balance and money operations are
asynchronous and return a typed result.

Requires **TNE 1.0.62 or later and Java 25**. This guide covers the native
Paper/Folia API. For the separate backend HTTP API, see
[TNECORE API documentation](https://github.com/SlyRiddles/TNECORE/blob/main/docs/API.md).

## 1. Add the API dependency

Download `TNE-API-1.0.62.jar` or the complete `TNE-Developer-SDK-1.0.62.zip` from
[the 1.0.62 release](https://github.com/SlyRiddles/TNE-Economy-Releases/releases/tag/v1.0.62).
The SDK includes sources, Javadoc, and a complete example plugin. The API JAR is
a compile dependency; install the normal TNE server JAR on your server.

There is currently no public TNE Maven repository. Install the downloaded API
JAR into your local Maven repository:

```shell
mvn install:install-file -Dfile=TNE-API-1.0.62.jar -DgroupId=com.cmriddles -DartifactId=the-newer-economy-api -Dversion=1.0.62 -Dpackaging=jar -DgeneratePom=true
```

Then add this to your plugin's `pom.xml`:

```xml
<dependency>
    <groupId>com.cmriddles</groupId>
    <artifactId>the-newer-economy-api</artifactId>
    <version>1.0.62</version>
    <scope>provided</scope>
</dependency>
```

Use `provided` and **do not shade or relocate the TNE API** into your plugin.
TNE provides these classes at runtime; a second copy can prevent service lookup.
Your CI needs the same dependency installation step or your own private artifact
repository.

For Gradle, you can instead put the downloaded JAR in your project's `libs`
directory and add this to `build.gradle.kts`:

```kotlin
dependencies {
    compileOnly(files("libs/TNE-API-1.0.62.jar"))
}
```

Keep your normal Paper API dependency too. The TNE JAR belongs on the compile
classpath only; the server supplies it at runtime.

## 2. Declare TNE as a dependency

In `plugin.yml`:

```yaml
name: MyPlugin
version: 1.0.0
main: example.MyPlugin
api-version: "26.1"
folia-supported: true
depend: [TheNewerEconomy]
```

`depend` makes TNE load before your plugin and makes the API classes visible.
It does not guarantee that TNE's asynchronous storage initialization has finished.
Check availability when you handle a request. Only declare `folia-supported` if
your whole plugin follows Folia's threading rules.

If your plugin uses `paper-plugin.yml` instead, declare its server dependency:

```yaml
dependencies:
  server:
    TheNewerEconomy:
      load: BEFORE
      required: true
      join-classpath: true
```

Here `BEFORE` means TNE must load before your plugin, and `join-classpath` gives
your plugin access to TNE's API classes. See Paper's
[dependency documentation](https://docs.papermc.io/paper/dev/getting-started/paper-plugins/)
and [plugin interoperability reference](https://docs.papermc.io/paper/reference/paper-plugins/).
The complete example uses `plugin.yml`; you do not need both descriptors.

## 3. Get the API and read a balance

Imports used below:

```java
import com.cmriddles.thenewereconomy.api.EconomyClient;
import com.cmriddles.thenewereconomy.api.TneApi;
import java.math.BigDecimal;
```

Inside your `JavaPlugin`, on the player's owning thread:

```java
TneApi api = getServer().getServicesManager().load(TneApi.class);
if (api == null || !api.isAvailable()) {
    player.sendMessage("The economy is starting or unavailable. Please try again shortly.");
    return;
}

EconomyClient economy = api.client(getName());
economy.balance(player.getUniqueId()).whenComplete((result, error) -> {
    if (!isEnabled()) return;
    // TNE completes asynchronously. Schedule player interaction on their owning thread.
    player.getScheduler().run(this, task -> {
        if (error != null) {
            player.sendMessage("Unable to read your balance. Please try again shortly.");
        } else if (!result.isSuccess()) {
            player.sendMessage("Unable to read your balance: " + result.failure().code());
        } else {
            var balance = result.value();
            player.sendMessage(balance.amount().toPlainString() + " " + balance.currency());
        }
    }, null);
});
```

`balance(uuid)` uses the configured default currency. Use
`balance(uuid, "coin")` to select a currency by its ID. UUIDs work for offline
players too; looking up a username is your plugin's responsibility.

The [complete balance-command example](plugin-api-example/README.md) includes
the POM, Java class, plugin metadata, lifecycle handling, and build instructions.

## 4. Transfer money

Use a decimal string to create money. `new BigDecimal("12.50")` is exact; avoid
`new BigDecimal(12.50)` and converting money through `double`.

```java
// Save this ID with your business operation BEFORE submitting the transfer.
String operationKey = "order:" + persistedOrderId;

economy.transfer(buyerUuid, sellerUuid, "coin", new BigDecimal("12.50"), operationKey)
        .whenComplete((result, error) -> {
            if (error != null) {
                // Outcome may be unknown. Reconcile the operation; do not invent a new key.
                return;
            }
            if (!result.isSuccess()) {
                // Inspect result.failure().code(); do not grant the purchased item here.
                return;
            }
            var transaction = result.value();
            // Persist transaction.transactionId() against the order.
            // Deliver any reward exactly once, on its correct Paper/Folia thread.
        });
```

This fragment demonstrates the money operation; it is not a complete shop.
Your plugin must persist order state and make reward delivery idempotent.
A successful transfer does not make an inventory edit atomic with the ledger.
Disconnects or a server restart can happen between those two actions.

Every write requires an **idempotency key**. Use one unique, persistent key for
each logical operation. Reuse that same key only when recovering that exact
operation with the same players, currency, amount, and scope. A retry does not
guarantee a second `SUCCESS` response: it can return `CONFLICT`. Do not assume
`CONFLICT` proves your specific operation succeeded, and do not create a fresh
key to bypass it. Keep your own durable operation and transaction records for
reconciliation. Two independent purchases must use different keys.
Keys share one namespace per integration across all currencies and mutation
types, so do not reuse an order's payment key for its later refund.
Keys contain 1 to 128 ASCII letters, digits, dots, underscores, colons or hyphens;
spaces are invalid. A persisted order ID or UUID with a short operation prefix
works well, such as `order:123:payment`.

### Recover a lost response

Use the original integration client and saved key to look up its committed
receipt. This reads the ledger; it does not submit a second payment:

```java
economy.transaction(operationKey).whenComplete((result, error) -> {
    if (error != null || !result.isSuccess()) {
        // Keep the order pending and try reconciliation again later.
        return;
    }
    var receipt = result.value(); // Optional<Transaction>
    if (receipt.isPresent()) {
        var transaction = receipt.orElseThrow();
        // Match this to your saved operation; record transaction.transactionId().
        // Resume your own exactly-once delivery workflow.
    } else {
        // No committed receipt was visible. The original request may still be in flight.
        // Keep the saved key; do not invent a replacement key or assume failure.
    }
});
```

`transaction(key)` searches only this integration's namespace. A successful
empty result means no committed receipt was found at the time of the read; it
does not prove the original request failed. A `CONFLICT` on a write is also not
proof that its arguments match your intended operation. Your durable order
record remains the source of the players, amount, currency, and intended action.

## Common operations

All methods below belong to `EconomyClient`. Default-currency overloads omit
the `currency` argument.

| Task | Call | Successful value |
| --- | --- | --- |
| Read default balance | `balance(playerUuid)` | `Balance` |
| Read selected balance | `balance(playerUuid, currency)` | `Balance` |
| Pay another player | `transfer(fromUuid, toUuid, currency, amount, key)` | `Transaction` |
| Credit a player | `deposit(playerUuid, currency, amount, key)` | `Transaction` |
| Debit a player | `withdraw(playerUuid, currency, amount, key)` | `Transaction` |
| Recover a committed receipt | `transaction(key)` | `Optional<Transaction>` |

`amount` is a positive `BigDecimal`. Currency precision is enforced; the API
does not silently round an invalid amount. Read `CurrencyInfo.scale()` to learn
how many decimal places a currency supports.

A `Balance` is a snapshot, not a reservation. Decide whether a payment succeeded
from the transfer or withdrawal result, rather than checking a balance first.
`Transaction.amount()` is the amount moved, not the player's resulting balance.

`deposit` and `withdraw` are privileged integration operations. A deposit debits
TNE's configured payout pool and credits the player; a withdrawal does the
reverse. Deposits can fail if that pool lacks funds. These are balanced, audited
ledger transactions, not unbacked money creation.

The server operator trusts installed plugins with this access; the API does not
automatically check a player's command permissions. Your plugin must authorize
its own actions. Never expose an unrestricted player-controlled deposit command.

### Fund the payout pool before testing deposits

The account UUID is configured in `economy.yml` under
`integrations.vault.settlement-account-id` (default:
`00000000-0000-0000-0000-000000000001`). Native deposits use that same account
identity, with a separate balance for each currency in the configured scope.
Do not assume that funding the default currency funds every other currency.

The owner can open **Server console > your server > Funding setup** on the
website, select the currency, and transfer existing Treasury funds to the
**Vault settlement** account. If local manual issuance is explicitly allowed,
an administrator with `tne.admin.adjust` can alternatively issue a reviewed
amount to that configured account:

```text
/tneadminadjust grant <configured-payout-account-uuid> coin 1000.00 native-api-funding:coin:1
```

Replace the UUID, currency, amount, and unique operation key deliberately; the
command name can be changed in configuration. A grant creates local currency
supply and remains subject to the currency's supply policy; `PRE_FUNDED` blocks
issuance. It is not needed if existing funds can be transferred. See
[owner funding controls](SERVER_OWNER_CONTROLS.md#guided-funding-setup-tne-economy-1020--tnecore-1022)
for the funding review and policy details. Funding a Rewards account alone does
not fund the native API's payout account.

## Currency names and scopes

```java
var defaultCurrency = api.defaultCurrency();
String id = defaultCurrency.id();             // e.g. "coin": use in API calls
String display = defaultCurrency.pluralName(); // e.g. "Coins": use in your UI
int decimals = defaultCurrency.scale();

for (var currency : api.currencies()) {
    // Configured virtual currencies usable through this API.
}
```

Currency IDs are case-insensitive and normalize to lowercase. Display names are
separate, with initial capitals. Do not use a display name, symbol, or formatted
balance as an identifier. The native money API does not deliver item currencies,
XP, or physical money notes.

The client uses `api.defaultScope()`, TNE's configured server scope. `scope()`
shows that value. There is no scope override in this API; this is not a
remote-server or federated settlement API.

## Results, readiness, and threading

Money and balance methods return `CompletionStage<Result<T>>`:

1. Check an exceptional completion (`error != null`). Unexpected failures and
   shutdown can happen. An exception is not proof that a write did not commit.
2. Otherwise check `result.isSuccess()`.
3. Read `result.value()` only on success. Read `result.failure()` only on failure.

Useful failure codes:

| Code | Meaning / next step |
| --- | --- |
| `VALIDATION_ERROR` | Check UUIDs, positive amount, precision, or key. |
| `CURRENCY_UNAVAILABLE` | Check the currency ID and configured currency type. |
| `INSUFFICIENT_FUNDS` | The required debit cannot be funded. |
| `ACCOUNT_RESTRICTED` | An account policy prevents the operation. |
| `PERMISSION_DENIED` | TNE policy rejected the operation. |
| `CONFLICT` | Conflicting/repeated operation; reconcile using your saved records. |
| `DEPENDENCY_UNAVAILABLE` | Storage/runtime is starting, stopping, or unavailable. |
| `INTERNAL_ERROR` | Record the failure's correlation ID for investigation. |

The failure also provides `message()` and `correlationId()`. Log these for
diagnosis; keep player-facing messages short. Do not parse English error messages
to decide whether a transaction succeeded. Invalid operation inputs return a
validation result; `client(integrationName)` itself rejects blank, oversized, or
control-character-containing names synchronously.

`isAvailable()` is a quick readiness check, not a guarantee that the next request
will succeed. Currency metadata is cached and can be read without waiting for
storage. The provider is unregistered when TNE stops. Look up the service again
after a lifecycle change; treat an old client as unavailable.

Do not call `.join()` or blocking `.get()` on Paper's main thread or any Folia
global, region, or entity thread. The API schedules its own storage work; you can
call it from a command handler without creating your own database thread.
Completion callbacks can run on a worker thread. Use the relevant Paper/Folia
scheduler for inventory changes, messages, entities, and world access. The example
uses `player.getScheduler().run(...)`, which also handles the player retiring.

## What is recorded

Create your client with `api.client(getName())` so writes are attributed to your
declared integration in TNE's transaction and audit records. This is attribution,
not plugin authentication. Use a stable plugin name; do not put player input or
an order ID in the integration name. The per-operation idempotency key identifies
the individual action.

Writes go through TNE's existing transaction engine, balance checks, immutable
double-entry ledger, audit records, and configured storage. This API does not
provide a balance setter or direct SQL access. It does not require your plugin
to construct ledger entries, balancing UUIDs, storage services, or commands.

The API classes to depend on are under `com.cmriddles.thenewereconomy.api`.
Platform and application implementation classes are internal integration details;
do not cast the provider or depend on `Paper*` classes.
