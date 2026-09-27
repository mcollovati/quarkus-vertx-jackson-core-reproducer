# quarkus-vertx-jackson-core-reproducer

Reproducer for a native build failure in Quarkus 3.33.3.3 (and 3.40.0).

The application uses `quarkus-undertow` and Jackson 3
(`tools.jackson.core:jackson-core`), but not Jackson 2
(`com.fasterxml.jackson.core:jackson-core`). The native build fails with:

```
Error: Class initialization of io.vertx.core.json.Json failed. ...
Caused by: java.lang.NoClassDefFoundError: com/fasterxml/jackson/core/util/ByteArrayBuilder
	at io.quarkus.vertx.runtime.jackson.QuarkusJacksonFactory.codec(QuarkusJacksonFactory.java:30)
	at io.vertx.core.json.Json.<clinit>(Json.java:28)
```

`UndertowBuildStep.jacksonOnClasspath()` checks only the artifactId
`jackson-core`. Jackson 3 has the same artifactId, so Quarkus thinks that
Jackson 2 is present and initializes `io.vertx.core.json.Json` at build time.
On 3.33.0 this did not matter, because Jackson 2 core came in through
`vertx-codegen`.

## Steps to reproduce

You need GraalVM or Mandrel 25 (or a container runtime for
`-Dquarkus.native.container-build=true`).

1. Build with Quarkus 3.33.3.3 (the default). The build fails:

   ```
   ./mvnw clean package -Dnative
   ```

2. Build with Quarkus 3.33.0. The build succeeds:

   ```
   ./mvnw clean package -Dnative -Dquarkus.platform.version=3.33.0
   ```

3. Optional: build with Quarkus 3.33.3.3 and Jackson 2 core added (the
   workaround). The build succeeds:

   ```
   ./mvnw clean package -Dnative -Dwith-jackson2
   ```

If you remove the `tools.jackson.core:jackson-core` dependency from
`pom.xml`, the build also succeeds on 3.33.3.3.
