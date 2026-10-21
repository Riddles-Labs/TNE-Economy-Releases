# Owner account and receipt reporting

This website build supports the extended owner-console reports from TNE 1.0.76
and TNECORE 1.0.46. It preserves the existing authenticated, installation-scoped
`columns` / `rows` / `hasMore` transport. No new endpoint or permission is introduced.

## Display behavior

- Ledger, receipt entries and server-account cards/details display registered
  account names, account type, owning integration, stable account key and owner
  namespace. Direct report columns take precedence over cached directory labels.
- Plugin directory identity can resolve a UUID in a historical currency/scope;
  balances stay separate by UUID, currency and scope. Known system-role fallbacks
  and older custom scoped labels continue to work.
- An explicit `UNKNOWN` identity stays unknown. Business metadata, UUID appearance
  and transaction source never establish account ownership. Plugin and unknown
  accounts do not appear as player wallets or offer player-report/grant controls.
- Receipts show the exact stored idempotency key and source namespace with separate
  copy buttons. Recovery requires the installation context as well as both values.
  Copying or viewing these values does not retry an operation.
- Each receipt page reads its first nonempty `Metadata` cell. All original business
  keys/values are shown as escaped text and can be copied as the reported JSON.
  Credential redaction remains the plugin's reporting responsibility. Historical
  settlement snapshots remain distinct from current registry ownership.
- Native payment lists use the recorded integration actor as a human title when
  available. Receipts additionally show recorded integration/region-action context.
  The original source namespace and `native_post` operation remain visible in
  receipt technical fields; stored values are unchanged.
- Ledger, receipt and account-activity timestamps explicitly show UTC and retain
  the exact reported UTC timestamp, including sub-millisecond precision.
- CSV export preserves exact decimal strings, metadata and recovery keys without
  duplicating an existing `Account name` column.

Older reports remain usable. Missing metadata/key fields are explicitly unavailable;
an empty stored metadata object is distinguished from a missing report. Malformed
metadata is reported as undecodable and remains available in the raw CSV export.
New Region Manager metadata does not backfill historical transactions.

The overview's short recent-activity snapshot does not transport full business
metadata. Open its receipt for business details and recovery identity. This change
does not add reversal previews or change reversal execution/scope.

## Upgrade and verification

1. Install backend **TNECORE 1.0.46 first**, then plugin **TNE 1.0.76**. Older backend
   validation can reject the extended receipt/ledger/account columns.
2. Publish the complete website `dist/` output through the existing hosting process,
   retaining its hashed assets, routing and security policy. No new configuration
   or credentials are needed. Source publication to GitHub and a successful local
   build do not establish that the live hosting deployment has completed.
3. Independent **RiddlesLabsCore 0.6.1 / compatible 0.6.x** needs no replacement.

Focused UI tests cover identity precedence, unknown UUIDs, current account renames,
historical scopes, metadata snapshots/pagination, recovery-key copying, exact
amounts, CSV columns, safe escaping, older reports and timestamp precision. Existing
reporting, history and portal UI tests also run. TypeScript checking and the production
Vite build (including the existing source/dist item-icon integrity checks) are local.
No Minecraft server, live database query, financial write or deployment is performed.
Actual owner-site/server behavior still requires the owner's validation.

Latest local verification: **100 tests passed, zero failures**, including nine
new owner-reporting cases; TypeScript checking and production packaging passed.
Logs: `target-owner-reporting-tests.log`, `target-owner-reporting-typecheck.log`,
`target-owner-reporting-build.log` (included in the local release bundle).
