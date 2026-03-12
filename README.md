# kestros-test

[![GitHub release](https://img.shields.io/github/v/release/kestros/kestros-test)](https://github.com/kestros/kestros-test/releases)

<<<<<<< HEAD
## Overview

kestros-test is a synthetic test pipeline repository used to validate the Kestros multi-agent workflow end-to-end. It exercises the full agent lifecycle, from BA scoping and Lead Developer guidelines through Developer implementation, QA review, and final Danny approval. The stories tracked on the `test` board are synthetic and do not represent real product work — they exist solely to verify that each stage of the pipeline functions correctly and that agents hand off work as expected.

## Installation

### Prerequisites

- **Java 11** or higher
- **Maven 3.6** or higher
- Git

### Steps

1. **Clone the repository**

   ```bash
   git clone https://github.com/kestros/kestros-test.git
   cd kestros-test
   ```

2. **Build the project**

   ```bash
   mvn clean install
   ```

   This compiles the source, runs all tests, and packages the bundle.

3. **Deploy (optional)**

   If you are deploying to a local Apache Sling instance, install the generated bundle via the Felix Web Console:

   ```bash
   curl -u admin:<password> \
     -F "action=install" \
     -F "bundlestart=true" \
     -F "bundlefile=@target/kestros-test-0.1.0-SNAPSHOT.jar" \
     "http://localhost:8080/system/console/bundles"
   ```

   Replace `<password>` and the port with the values for your local instance.
=======
Test repository for validating the Kestros agent pipeline end-to-end.

## Configuration

`kestros-test` is a Maven OSGi bundle project. The following configuration options are available at build time via Maven system properties or `pom.xml` overrides.

### Build Properties

| Name | Type | Default | Description |
|---|---|---|---|
| `project.version` | String | `0.1.0-SNAPSHOT` | Bundle version published to the OSGi container. Override with `-Dproject.version=<ver>` on the Maven command line or via a CI release plugin. |
| `maven.test.skip` | Boolean | `false` | Skip test execution during the build. Set to `true` to build the bundle without running unit tests (`-Dmaven.test.skip=true`). |
| `maven.compiler.source` | String | JVM default | Java source compatibility level used by the compiler plugin. Override in `pom.xml` `<properties>` to target a specific JDK version. |
| `maven.compiler.target` | String | JVM default | Java bytecode target compatibility level. Should match `maven.compiler.source`. |

### Bundle Packaging

| Property | Value | Description |
|---|---|---|
| `groupId` | `io.kestros` | Maven group identifier. Determines the OSGi bundle's symbolic name prefix. |
| `artifactId` | `kestros-test` | Maven artifact identifier. Combined with `groupId` to form the OSGi `Bundle-SymbolicName`. |
| `packaging` | `bundle` | Produces an OSGi bundle JAR via the `maven-bundle-plugin`. |

### Example: Override version at build time

```bash
mvn clean package -Dproject.version=1.0.0
```

### Example: Skip tests during a local build

```bash
mvn clean package -Dmaven.test.skip=true
```

### Example: Target a specific Java version

Add to `pom.xml` `<properties>`:

```xml
<maven.compiler.source>11</maven.compiler.source>
<maven.compiler.target>11</maven.compiler.target>
```

> **Note:** This repository is a pipeline validation harness. Configuration options will expand as the module evolves beyond test-harness scope.
>>>>>>> a5de553 (docs: add Configuration section to README)

## Configuration

`kestros-test` is a Maven OSGi bundle project. The following configuration options are available at build time via Maven system properties or `pom.xml` overrides.

### Build Properties

| Name | Type | Default | Description |
|---|---|---|---|
| `project.version` | String | `0.1.0-SNAPSHOT` | Bundle version published to the OSGi container. Override with `-Dproject.version=<ver>` on the Maven command line or via a CI release plugin. |
| `maven.test.skip` | Boolean | `false` | Skip test execution during the build. Set to `true` to build the bundle without running unit tests (`-Dmaven.test.skip=true`). |
| `maven.compiler.source` | String | JVM default | Java source compatibility level used by the compiler plugin. Override in `pom.xml` `<properties>` to target a specific JDK version. |
| `maven.compiler.target` | String | JVM default | Java bytecode target compatibility level. Should match `maven.compiler.source`. |

### Bundle Packaging

| Property | Value | Description |
|---|---|---|
| `groupId` | `io.kestros` | Maven group identifier. Determines the OSGi bundle's symbolic name prefix. |
| `artifactId` | `kestros-test` | Maven artifact identifier. Combined with `groupId` to form the OSGi `Bundle-SymbolicName`. |
| `packaging` | `bundle` | Produces an OSGi bundle JAR via the `maven-bundle-plugin`. |

### Example: Override version at build time

```bash
mvn clean package -Dproject.version=1.0.0
```

### Example: Skip tests during a local build

```bash
mvn clean package -Dmaven.test.skip=true
```

### Example: Target a specific Java version

Add to `pom.xml` `<properties>`:

```xml
<maven.compiler.source>11</maven.compiler.source>
<maven.compiler.target>11</maven.compiler.target>
```

> **Note:** This repository is a pipeline validation harness. Configuration options will expand as the module evolves beyond test-harness scope.
