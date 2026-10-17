# TNE native API example

A complete, read-only Paper/Folia plugin. `/tneexamplebalance [currency]` reads
your own balance, using the server's default currency when omitted. It does not
modify balances.

## Build

1. Install **JDK 25** and Maven.
2. Download `TNE-API-1.0.62.jar` from
   [TNE 1.0.62](https://github.com/SlyRiddles/TNE-Economy-Releases/releases/tag/v1.0.62),
   or use the copy supplied in the developer SDK.
3. In the directory containing the API JAR, run:

   ```shell
   mvn install:install-file -Dfile=TNE-API-1.0.62.jar -DgroupId=com.cmriddles -DartifactId=the-newer-economy-api -Dversion=1.0.62 -Dpackaging=jar -DgeneratePom=true
   ```

4. In this example's directory (the directory containing its `pom.xml`), run:

   ```shell
   mvn clean package
   ```

5. Install `target/tne-api-example-1.0.0.jar` alongside the normal
   `TNE-Economy-1.0.62.jar` in your Paper/Folia server's `plugins` directory.
   Start the server and run `/tneexamplebalance` as a player.

The example compiles against Paper's published `1.21.8-R0.1-SNAPSHOT` API,
matching TNE's build dependency. Its runtime target is TNE's supported
Paper/Folia 26.1+ server, with Java 25. This compile coordinate is not a promise
that TNE supports older Minecraft servers.

## Adapt it

Change the package, main class, plugin name, and command to suit your plugin.
Keep TNE as a `provided` dependency; do not shade its API into your JAR. The
`depend: [TheNewerEconomy]` entry makes TNE's classes available at runtime.

The command looks up `TneApi` for each request because service availability can
change. TNE starts storage asynchronously, so even a dependent plugin must
check readiness. Balance work is already asynchronous; the callback schedules
player interaction through the player's entity scheduler. If a player retires
before that task runs, no reply is sent.

For transfers, decimal amounts, idempotency, audit attribution, and failure
handling, read the API guide: `docs/API.md` in the source checkout or `README.md`
beside the `example` directory in the SDK ZIP.
