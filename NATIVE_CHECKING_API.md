# Native TNE Checking

Each virtual currency has its own funded TNE Checking account in the active economy scope. It is separate from Vault Checking, Savings, Rewards, exchange accounts and the Currency Mint. Existing balances are not moved automatically.

## Owner approval

Add the exact installed plugin name to `economy.yml`, then restart TNE normally:

```yaml
integrations:
  native-checking:
    allowed-plugins:
      - MyGamePlugin
```

The default empty list approves no plugins. Approval allows that plugin to spend the shared checking balance for configured virtual currencies; approve only trusted integrations. It does not grant minting rights. Java plugins share one JVM: this is an integration permission boundary, not a sandbox for hostile code.

## Developer usage

From your own plugin class (or a class loaded by the same plugin), obtain the registered `TneApi` and call:

```java
EconomyClient client = api.clientForPlugin(this);
client.checking().pay(AccountRef.player(playerUuid), "gold",
    new BigDecimal("25.00"), "Quest completion: first mine", stableOperationKey)
    .thenAccept(result -> {
        // Inspect success/failure. Schedule player/world access on the appropriate
        // Paper/Folia owner context. Retry uncertain outcomes with the SAME key.
    });
```

The destination must already exist. Use the existing account API and player lifecycle to resolve it. The payout uses TNE's normal atomic ledger, insufficient-funds checks, receipt, provenance and idempotency path. Receipts record the real plugin name, platform-bound attribution and required reason. No fallback to Vault, Savings, Rewards or issuance occurs.

`client.checking().account("gold")` exposes the deterministic account reference, not permission to spend it. Balance/history reads and incoming transfers can use that reference. Generic `post`, transfer, reversal and lifecycle adapters cannot debit TNE Checking; the approved explicit `checking().pay()` path is required. Existing owned plugin accounts remain available for integrations needing isolated funds.

`api.client("name")` remains supported for existing integrations and their accounts. Its self-declared name does not authorize shared checking. Platform-bound clients cannot be obtained by supplying another plugin's name or instance from a different plugin classloader. Do not share an authorized client with other plugins.

Native checking payout approval is currently a per-plugin, all-configured-virtual-currencies permission, not a per-currency budget. Scheduled and reserved shared-checking payouts are intentionally unsupported; developers must make explicit approved payouts when due or use their own funded plugin account for lifecycle workflows.

Legacy native `deposit` and `withdraw` method signatures remain, but now use TNE Checking and require the same approved platform client. Existing integrations using `client(String)` must migrate to `clientForPlugin(this)` for these convenience methods. No existing funds are silently borrowed from Vault. Dedicated-account `pay`, `charge` and transfers retain their existing behavior.
