# Console and in-game reporting

TNE-Economy 1.0.54 uses one MiniMessage reporting path for console output and player/admin messages. Paper/Folia renders components using the console's supported colors, with real INFO/WARN/ERROR severity preserved. File logs remain readable. In-game reports use a consistent TNE prefix, readable labels and intact click/hover actions.

Edit `messages.yml` templates under `reports.*` to customize chat prefixes, console categories, fields, service states, startup, storage, failures and update notices. Missing keys use built-in defaults. Existing command templates still apply. Dynamic values are inserted literally, so a player name or remote string cannot inject MiniMessage formatting. Known credential fields in key/value reports are redacted.

In `config.yml`:

```yaml
logging:
  colors: true
updates:
  notify: true
  enabled: false
```

`logging.colors: false` removes console colors without losing detail. `updates.notify` checks the fixed signed release feed and alerts administrators with `tne.admin.updates`; it does not download a JAR. `updates.enabled` opts into verified downloads and normal-restart installation. Set both update options false to disable checks. Never replace or reload a running plugin JAR in place.

Startup and `/tne setup status` show labeled platform, Java, server identity and service states. Command failures show the problem, explanation and correlation reference. Audit reports retain operation, actor, amount and transaction identifiers. GUI error details use individual lore lines rather than multiline item names.

Connection reports identify the affected channel and distinguish timeout, connection refusal, DNS, TLS and HTTP authentication/server failures. First failures are visible immediately; repeated warnings are limited to once per minute. Recovery includes elapsed interruption time and failed checks. Local balances are not changed by a reporting outage. Core health and authenticated website/setup channels have separate connection states.

With TNECORE 1.0.45, the public health response supplies its running version and opaque process identity. The plugin compares these with its last observation and reports a confirmed core version change, a possible restart/backend switch, or normal reconnection accurately. An older core reports its version as unavailable. This metadata never authorizes executable downloads; plugin updates still require the fixed signed release feed. A core update does not imply that a Minecraft plugin update exists.

In 1.0.55 the startup console report again uses its original 72-column box, including developer `cmriddles`, runtime/integration sections and complete guidance. `messages.reports.startup.box` supplies MiniMessage color templates; preserve `<line>` or `<prefix>/<value>/<suffix>` for alignment. Console banners go directly to the native component logger without an extra category prefix.

In 1.0.56 currency display labels use initial capitals consistently (`coins`/`COINS`/`cOiNs` → `Coins`, with `TNE` preserved), including styled money and notifications. Stored IDs and permission/action keys remain unchanged. `%tne_currency_name%` supplies the default plural display name; `%tne_currency%` continues to supply its raw ID.
