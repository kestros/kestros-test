# kestros-test

[![GitHub release](https://img.shields.io/github/v/release/kestros/kestros-test)](https://github.com/kestros/kestros-test/releases)

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
Test repository for validating the Kestros agent pipeline end-to-end.

## Troubleshooting

This section covers common issues encountered when building, testing, or deploying `kestros-test`.

---

### Bundle fails to activate after deployment

**Symptom**

After deploying the bundle via the Felix Web Console, it appears in the bundle list with a `Resolved` or `Installed` state instead of `Active`.

**Cause**

One or more OSGi package imports declared in the bundle manifest cannot be satisfied by the currently installed bundles. This typically happens when a dependency bundle is missing, at the wrong version, or has not yet activated.

**Solution**

1. Open the Felix Web Console at `http://localhost:8080/system/console/bundles`.
2. Click the bundle name and inspect the **Imported Packages** section for any packages shown as unsatisfied (missing or in red).
3. Install and start the missing dependency bundle.
4. If all dependencies are present, try refreshing the package wiring:

   ```bash
   curl -u admin:<password> -X POST \
     "http://localhost:8080/system/console/bundles/<bundle-id>" \
     -d "action=refreshPackages"
   ```

5. If the bundle still does not activate, check the Sling error log for `BundleException` or `ClassNotFoundException` entries.

---

### Maven build fails with `package does not exist` errors

**Symptom**

Running `mvn clean install` produces compilation errors such as `package io.kestros.xxx does not exist` or `cannot find symbol`.

**Cause**

The local Maven repository does not contain one or more Kestros dependency artifacts. This can happen on a fresh clone or after a dependency version bump in `pom.xml`.

**Solution**

1. Ensure your local Maven repository is populated from the Kestros artifact repository or your organisation's proxy:

   ```bash
   mvn clean install -U
   ```

   The `-U` flag forces Maven to check remote repositories for updated snapshots.

2. If building behind a corporate proxy or firewall, confirm that `~/.m2/settings.xml` contains the correct proxy and mirror configuration pointing to the artifact repository hosting `io.kestros` artifacts.

3. If the missing artifact is a locally-developed Kestros module, build and install it first:

   ```bash
   cd /path/to/dependency-module
   mvn clean install -DskipTests
   ```

---

### Unit tests fail with `ResourceResolverFactory` not bound

**Symptom**

Test execution fails with errors such as `NullPointerException` on a `ResourceResolverFactory` reference, or OSGi service injection is not happening in test context.

**Cause**

Tests that rely on OSGi service injection (e.g., using Mockito or AEM Mocks) need the `ResourceResolverFactory` context to be set up explicitly. Without it, the injected factory reference remains `null`.

**Solution**

1. Use a Sling Mocks context in your test:

   ```java
   @ExtendWith(SlingContextExtension.class)
   class MyServiceTest {
       private final SlingContext context = new SlingContext(ResourceResolverType.JCR_MOCK);
       // ...
   }
   ```

2. Register any required services and resources on the context before invoking the class under test.

3. If you are testing a service that extends `BaseServiceResolverService`, mock `getServiceResourceResolver()` rather than the factory directly to avoid the full OSGi lifecycle.

---

### `mvn clean install` succeeds locally but CI fails with test errors

**Symptom**

Tests pass on a developer machine but fail in CI with assertion errors or timeout exceptions.

**Cause**

Environment-sensitive tests may behave differently under CI conditions — for example, tests that rely on a local Sling instance, hardcoded ports, or system-time-dependent assertions.

**Solution**

1. Check whether any tests directly reference `localhost` or a fixed port number. Replace these with configurable constants or system property lookups:

   ```java
   String host = System.getProperty("sling.host", "localhost");
   ```

2. Review tests that use `Thread.sleep` or `Date`/`Calendar` directly. Replace time-dependent logic with injectable clocks or mocks.

3. Run the full test suite with CI-equivalent JVM settings locally to reproduce the failure:

   ```bash
   mvn clean test -Dmaven.test.failure.ignore=false
   ```

---

### Deploying a new version does not replace the previous bundle

**Symptom**

After installing an updated version of the bundle, the Felix console still shows the old version running. Changes to code or configuration are not reflected.

**Cause**

The old bundle version was not uninstalled before the new one was installed, or the bundle symbolic name changed between versions, resulting in two bundles being installed simultaneously.

**Solution**

1. In the Felix Web Console, locate the old bundle by its symbolic name (`io.kestros.kestros-test`).
2. Stop and uninstall the old version before deploying the new one.
3. Alternatively, use the `action=update` parameter when posting to the Felix console — this replaces the running bundle in place:

   ```bash
   curl -u admin:<password> \
     -F "action=install" \
     -F "bundlestart=true" \
     -F "bundlefile=@target/kestros-test-0.1.0-SNAPSHOT.jar" \
     "http://localhost:8080/system/console/bundles"
   ```

   Replace `<password>` and the port with the values for your local instance.
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

## Usage Examples

The following examples demonstrate common patterns for testing OSGi components and Sling models in the Kestros framework using JUnit and the Sling Mocks testing utilities.

### 1. Setting up a Sling resource context for a unit test

Use `AemContext` (or `SlingContext`) to build an in-memory Sling repository and register resources without a running OSGi container.

```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyComponentTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testResourceType() {
        context.build()
            .resource("/content/mypage", "sling:resourceType", "kestros/components/my-component")
            .commit();

        assertEquals(
            "kestros/components/my-component",
            context.resourceResolver()
                .getResource("/content/mypage")
                .getResourceType()
        );
    }
}
```

### 2. Adapting a Sling resource to a Kestros model and asserting properties

Register a Sling model class in the test context and verify that property values are read correctly from JCR content.

```java
import io.kestros.commons.structuredsling.BaseComponent;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class BaseComponentModelTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testGetTitle() {
        context.addModelsForClasses(BaseComponent.class);
        context.build()
            .resource("/content/my-component",
                "jcr:title", "My Component Title",
                "sling:resourceType", "kestros/components/base")
            .commit();

        BaseComponent model = context.resourceResolver()
            .getResource("/content/my-component")
            .adaptTo(BaseComponent.class);

        assertEquals("My Component Title", model.getTitle());
    }
}
```

### 3. Registering and invoking an OSGi service in the test context

Register a mock OSGi service implementation against its interface and verify that a component under test calls it correctly.

```java
import io.kestros.tasks.api.services.TaskService;
import org.apache.sling.testing.mock.osgi.junit.OsgiContext;
import org.junit.Rule;
import org.junit.Test;
import static org.mockito.Mockito.*;

public class TaskServiceConsumerTest {

    @Rule
    public final OsgiContext context = new OsgiContext();

    @Test
    public void testServiceIsInvokedOnActivation() {
        TaskService mockService = mock(TaskService.class);
        context.registerService(TaskService.class, mockService);

        TaskServiceConsumer consumer = context.registerInjectActivateService(
            new TaskServiceConsumer()
        );

        verify(mockService, atLeastOnce()).getActiveTaskCount();
    }
}
```

### 4. Testing a Sling servlet using a mock request and response

Construct a mock `SlingHttpServletRequest` and `SlingHttpServletResponse` to exercise a servlet's `doGet` logic without an HTTP server.

```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.apache.sling.testing.mock.sling.servlet.MockSlingHttpServletRequest;
import org.apache.sling.testing.mock.sling.servlet.MockSlingHttpServletResponse;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyServletTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testDoGetReturnsJson() throws Exception {
        MockSlingHttpServletRequest request =
            new MockSlingHttpServletRequest(context.resourceResolver(), context.bundleContext());
        request.setQueryString("id=TEST-001");

        MockSlingHttpServletResponse response = new MockSlingHttpServletResponse();

        new MyServlet().doGet(request, response);

        assertEquals("application/json", response.getContentType());
    }
}
```

### 5. Asserting JCR child node structure after a service write operation

After a service method creates JCR nodes, verify the resulting tree structure using the resource resolver in the test context.

```java
import org.apache.sling.api.resource.Resource;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertNotNull;

public class StoryCreationServiceTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testStoryNodeCreatedWithNotesChild() {
        context.build()
            .resource("/content/kestros-tasks/stories/TEST-001",
                "title", "My First Story",
                "status", "todo")
            .resource("/content/kestros-tasks/stories/TEST-001/notes")
            .commit();

        Resource story = context.resourceResolver()
            .getResource("/content/kestros-tasks/stories/TEST-001");
        Resource notes = story.getChild("notes");

        assertNotNull("Story resource must exist", story);
        assertNotNull("Notes child node must be present", notes);
    }
}
```

## Usage Examples

The following examples demonstrate common patterns for testing OSGi components and Sling models in the Kestros framework using JUnit and the Sling Mocks testing utilities.

### 1. Setting up a Sling resource context for a unit test

Use `AemContext` (or `SlingContext`) to build an in-memory Sling repository and register resources without a running OSGi container.

```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyComponentTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testResourceType() {
        context.build()
            .resource("/content/mypage", "sling:resourceType", "kestros/components/my-component")
            .commit();

        assertEquals(
            "kestros/components/my-component",
            context.resourceResolver()
                .getResource("/content/mypage")
                .getResourceType()
        );
    }
}
```

### 2. Adapting a Sling resource to a Kestros model and asserting properties

Register a Sling model class in the test context and verify that property values are read correctly from JCR content.

```java
import io.kestros.commons.structuredsling.BaseComponent;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class BaseComponentModelTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testGetTitle() {
        context.addModelsForClasses(BaseComponent.class);
        context.build()
            .resource("/content/my-component",
                "jcr:title", "My Component Title",
                "sling:resourceType", "kestros/components/base")
            .commit();

        BaseComponent model = context.resourceResolver()
            .getResource("/content/my-component")
            .adaptTo(BaseComponent.class);

        assertEquals("My Component Title", model.getTitle());
    }
}
```

### 3. Registering and invoking an OSGi service in the test context

Register a mock OSGi service implementation against its interface and verify that a component under test calls it correctly.

```java
import io.kestros.tasks.api.services.TaskService;
import org.apache.sling.testing.mock.osgi.junit.OsgiContext;
import org.junit.Rule;
import org.junit.Test;
import static org.mockito.Mockito.*;

public class TaskServiceConsumerTest {

    @Rule
    public final OsgiContext context = new OsgiContext();

    @Test
    public void testServiceIsInvokedOnActivation() {
        TaskService mockService = mock(TaskService.class);
        context.registerService(TaskService.class, mockService);

        TaskServiceConsumer consumer = context.registerInjectActivateService(
            new TaskServiceConsumer()
        );

        verify(mockService, atLeastOnce()).getActiveTaskCount();
    }
}
```

### 4. Testing a Sling servlet using a mock request and response

Construct a mock `SlingHttpServletRequest` and `SlingHttpServletResponse` to exercise a servlet's `doGet` logic without an HTTP server.

```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.apache.sling.testing.mock.sling.servlet.MockSlingHttpServletRequest;
import org.apache.sling.testing.mock.sling.servlet.MockSlingHttpServletResponse;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyServletTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testDoGetReturnsJson() throws Exception {
        MockSlingHttpServletRequest request =
            new MockSlingHttpServletRequest(context.resourceResolver(), context.bundleContext());
        request.setQueryString("id=TEST-001");

        MockSlingHttpServletResponse response = new MockSlingHttpServletResponse();

        new MyServlet().doGet(request, response);

        assertEquals("application/json", response.getContentType());
    }
}
```

### 5. Asserting JCR child node structure after a service write operation

After a service method creates JCR nodes, verify the resulting tree structure using the resource resolver in the test context.

```java
import org.apache.sling.api.resource.Resource;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertNotNull;

public class StoryCreationServiceTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testStoryNodeCreatedWithNotesChild() {
        context.build()
            .resource("/content/kestros-tasks/stories/TEST-001",
                "title", "My First Story",
                "status", "todo")
            .resource("/content/kestros-tasks/stories/TEST-001/notes")
            .commit();

        Resource story = context.resourceResolver()
            .getResource("/content/kestros-tasks/stories/TEST-001");
        Resource notes = story.getChild("notes");

        assertNotNull("Story resource must exist", story);
        assertNotNull("Notes child node must be present", notes);
    }
}
```
Test repository for validating the Kestros agent pipeline end-to-end.

## Contributing

Contributions to `kestros-test` follow the same standards as all Kestros repositories. The guidelines below mirror the system-wide standards defined in `kestros-claude/standards/`.

### Pull Request Process

1. **Branch naming** — all branches must follow the convention `{agent-id}/TASK-NNN` (e.g. `dev-readme-06/TEST-026`). Feature branches created outside of the task system use `feature/short-description`.
2. **Branch from `main`** — always create your branch from `main`. Never branch from another feature branch.
3. **One branch per task, one PR per task** — scope each branch and PR to a single task. Do not batch multiple task IDs into a single PR.
4. **PR title format** — use `[kestros-test] Brief description of change` (e.g. `[kestros-test] Add Contributing section to README`).
5. **Target branch** — all PRs target `main`.
6. **Required reviewers** — at minimum, a Lead Developer review is required before merge. QA sign-off is required before the task moves to `pending-approval`.
7. **No AI references** — commit messages must never reference Claude, Anthropic, or any AI tool. Conventional prefixes only: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`.

### Code Style

- Follow the formatting conventions established in `kestros-claude/standards/`.
- Java code uses standard Maven project structure. Keep package declarations consistent with `io.kestros.test`.
- Commit messages use the format `[artifact-id]: action, brief result` — imperative mood, under 72 characters on the first line.
- Do not commit build artifacts, IDE config files, or generated output. Stage only the files relevant to the change.
- Keep scope contained: only modify files that are directly required by the task.

### Testing

Before submitting a PR, the following must pass locally:
Test repository for validating the Kestros agent pipeline end-to-end.

## FAQ

**Q: What is this repository for?**

`kestros-test` is a synthetic test repository used to validate the Kestros multi-agent pipeline end-to-end. It does not contain production application code. Instead, it serves as a controlled environment for testing agent workflows, story lifecycle transitions, PR creation, review gates, and deployment automation. All stories created against this repo are test stories marked with `[TEST]` in their titles.

**Q: How do I run the tests?**

Tests are run using Maven from the repository root:

```bash
mvn clean install
```

This compiles the source, runs all unit tests, and packages the bundle. A PR may not be submitted if `mvn clean install` fails.

For changes that affect API endpoints or servlet behaviour, run the smoke test against your local Sling instance after deploying:
This compiles all source, executes unit tests, and packages the bundle. For integration tests that require a running Sling instance, deploy the bundle to your local dev instance first, then run the smoke test:

```bash
bash scripts/smoke-test.sh http://localhost:9000
```

All tests must pass. Zero failures, zero skipped. Do not submit a PR with a failing test suite.

## Contributing

Contributions to `kestros-test` follow the same standards as all Kestros repositories. The guidelines below mirror the system-wide standards defined in `kestros-claude/standards/`.

### Pull Request Process

1. **Branch naming** — all branches must follow the convention `{agent-id}/TASK-NNN` (e.g. `dev-readme-06/TEST-026`). Feature branches created outside of the task system use `feature/short-description`.
2. **Branch from `main`** — always create your branch from `main`. Never branch from another feature branch.
3. **One branch per task, one PR per task** — scope each branch and PR to a single task. Do not batch multiple task IDs into a single PR.
4. **PR title format** — use `[kestros-test] Brief description of change` (e.g. `[kestros-test] Add Contributing section to README`).
5. **Target branch** — all PRs target `main`.
6. **Required reviewers** — at minimum, a Lead Developer review is required before merge. QA sign-off is required before the task moves to `pending-approval`.
7. **No AI references** — commit messages must never reference Claude, Anthropic, or any AI tool. Conventional prefixes only: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`.

### Code Style

- Follow the formatting conventions established in `kestros-claude/standards/`.
- Java code uses standard Maven project structure. Keep package declarations consistent with `io.kestros.test`.
- Commit messages use the format `[artifact-id]: action, brief result` — imperative mood, under 72 characters on the first line.
- Do not commit build artifacts, IDE config files, or generated output. Stage only the files relevant to the change.
- Keep scope contained: only modify files that are directly required by the task.

### Testing

Before submitting a PR, the following must pass locally:

```bash
mvn clean install
```

This compiles the source, runs all unit tests, and packages the bundle. A PR may not be submitted if `mvn clean install` fails.

For changes that affect API endpoints or servlet behaviour, run the smoke test against your local Sling instance after deploying:

```bash
bash scripts/smoke-test.sh http://localhost:9000
```

All tests must pass. Zero failures, zero skipped. Do not submit a PR with a failing test suite.
## License

This project is licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

Copyright © 2024 Kestros

## License

This project is licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

Copyright © 2024 Kestros
4. After installing, verify the active version in the console and refresh the browser to clear any cached UI resources.

---

### JCR content nodes from test setup are not cleaned up between runs

**Symptom**

Subsequent test runs produce unexpected results because JCR nodes created by a previous test still exist under `/content/` or `/apps/`.

**Cause**

Tests that write to a shared JCR instance (rather than an in-memory mock) do not always clean up their created nodes on failure or exception, leaving stale content.

**Solution**

1. Prefer `ResourceResolverType.JCR_MOCK` or `ResourceResolverType.RESOURCERESOLVER_MOCK` in unit tests so content is never persisted to a real repository.

2. For integration tests that require a real Sling instance, implement a `@AfterEach` or `@AfterAll` teardown that deletes created nodes:

   ```java
   @AfterEach
   void cleanup() throws Exception {
       try (ResourceResolver resolver = resolverFactory.getAdministrativeResourceResolver(null)) {
           Resource node = resolver.getResource("/content/test-data");
           if (node != null) {
               resolver.delete(node);
               resolver.commit();
           }
       }
   }
   ```

3. If running against the shared dev instance, prefix all test content paths with a unique agent or session ID to avoid collisions with other active agents.

## Troubleshooting

This section covers common issues encountered when building, testing, or deploying `kestros-test`.

---

### Bundle fails to activate after deployment

**Symptom**

After deploying the bundle via the Felix Web Console, it appears in the bundle list with a `Resolved` or `Installed` state instead of `Active`.

**Cause**

One or more OSGi package imports declared in the bundle manifest cannot be satisfied by the currently installed bundles. This typically happens when a dependency bundle is missing, at the wrong version, or has not yet activated.

**Solution**

1. Open the Felix Web Console at `http://localhost:8080/system/console/bundles`.
2. Click the bundle name and inspect the **Imported Packages** section for any packages shown as unsatisfied (missing or in red).
3. Install and start the missing dependency bundle.
4. If all dependencies are present, try refreshing the package wiring:

   ```bash
   curl -u admin:<password> -X POST \
     "http://localhost:8080/system/console/bundles/<bundle-id>" \
     -d "action=refreshPackages"
   ```

5. If the bundle still does not activate, check the Sling error log for `BundleException` or `ClassNotFoundException` entries.

---

### Maven build fails with `package does not exist` errors

**Symptom**

Running `mvn clean install` produces compilation errors such as `package io.kestros.xxx does not exist` or `cannot find symbol`.

**Cause**

The local Maven repository does not contain one or more Kestros dependency artifacts. This can happen on a fresh clone or after a dependency version bump in `pom.xml`.

**Solution**

1. Ensure your local Maven repository is populated from the Kestros artifact repository or your organisation's proxy:

   ```bash
   mvn clean install -U
   ```

   The `-U` flag forces Maven to check remote repositories for updated snapshots.

2. If building behind a corporate proxy or firewall, confirm that `~/.m2/settings.xml` contains the correct proxy and mirror configuration pointing to the artifact repository hosting `io.kestros` artifacts.

3. If the missing artifact is a locally-developed Kestros module, build and install it first:

   ```bash
   cd /path/to/dependency-module
   mvn clean install -DskipTests
   ```

---

### Unit tests fail with `ResourceResolverFactory` not bound

**Symptom**

Test execution fails with errors such as `NullPointerException` on a `ResourceResolverFactory` reference, or OSGi service injection is not happening in test context.

**Cause**

Tests that rely on OSGi service injection (e.g., using Mockito or AEM Mocks) need the `ResourceResolverFactory` context to be set up explicitly. Without it, the injected factory reference remains `null`.

**Solution**

1. Use a Sling Mocks context in your test:

   ```java
   @ExtendWith(SlingContextExtension.class)
   class MyServiceTest {
       private final SlingContext context = new SlingContext(ResourceResolverType.JCR_MOCK);
       // ...
   }
   ```

2. Register any required services and resources on the context before invoking the class under test.

3. If you are testing a service that extends `BaseServiceResolverService`, mock `getServiceResourceResolver()` rather than the factory directly to avoid the full OSGi lifecycle.

---

### `mvn clean install` succeeds locally but CI fails with test errors

**Symptom**

Tests pass on a developer machine but fail in CI with assertion errors or timeout exceptions.

**Cause**

Environment-sensitive tests may behave differently under CI conditions — for example, tests that rely on a local Sling instance, hardcoded ports, or system-time-dependent assertions.

**Solution**

1. Check whether any tests directly reference `localhost` or a fixed port number. Replace these with configurable constants or system property lookups:

   ```java
   String host = System.getProperty("sling.host", "localhost");
   ```

2. Review tests that use `Thread.sleep` or `Date`/`Calendar` directly. Replace time-dependent logic with injectable clocks or mocks.

3. Run the full test suite with CI-equivalent JVM settings locally to reproduce the failure:

   ```bash
   mvn clean test -Dmaven.test.failure.ignore=false
   ```

---

### Deploying a new version does not replace the previous bundle

**Symptom**

After installing an updated version of the bundle, the Felix console still shows the old version running. Changes to code or configuration are not reflected.

**Cause**

The old bundle version was not uninstalled before the new one was installed, or the bundle symbolic name changed between versions, resulting in two bundles being installed simultaneously.

**Solution**

1. In the Felix Web Console, locate the old bundle by its symbolic name (`io.kestros.kestros-test`).
2. Stop and uninstall the old version before deploying the new one.
3. Alternatively, use the `action=update` parameter when posting to the Felix console — this replaces the running bundle in place:

   ```bash
   curl -u admin:<password> \
     -F "action=install" \
     -F "bundlestart=true" \
     -F "bundlefile=@target/kestros-test-0.1.0-SNAPSHOT.jar" \
     "http://localhost:8080/system/console/bundles"
   ```

4. After installing, verify the active version in the console and refresh the browser to clear any cached UI resources.

---

### JCR content nodes from test setup are not cleaned up between runs

**Symptom**

Subsequent test runs produce unexpected results because JCR nodes created by a previous test still exist under `/content/` or `/apps/`.

**Cause**

Tests that write to a shared JCR instance (rather than an in-memory mock) do not always clean up their created nodes on failure or exception, leaving stale content.

**Solution**

1. Prefer `ResourceResolverType.JCR_MOCK` or `ResourceResolverType.RESOURCERESOLVER_MOCK` in unit tests so content is never persisted to a real repository.

2. For integration tests that require a real Sling instance, implement a `@AfterEach` or `@AfterAll` teardown that deletes created nodes:

   ```java
   @AfterEach
   void cleanup() throws Exception {
       try (ResourceResolver resolver = resolverFactory.getAdministrativeResourceResolver(null)) {
           Resource node = resolver.getResource("/content/test-data");
           if (node != null) {
               resolver.delete(node);
               resolver.commit();
           }
       }
   }
   ```

3. If running against the shared dev instance, prefix all test content paths with a unique agent or session ID to avoid collisions with other active agents.

## Troubleshooting

This section covers common issues encountered when building, testing, or deploying `kestros-test`.

---

### Bundle fails to activate after deployment

**Symptom**

After deploying the bundle via the Felix Web Console, it appears in the bundle list with a `Resolved` or `Installed` state instead of `Active`.

**Cause**

One or more OSGi package imports declared in the bundle manifest cannot be satisfied by the currently installed bundles. This typically happens when a dependency bundle is missing, at the wrong version, or has not yet activated.

**Solution**

1. Open the Felix Web Console at `http://localhost:8080/system/console/bundles`.
2. Click the bundle name and inspect the **Imported Packages** section for any packages shown as unsatisfied (missing or in red).
3. Install and start the missing dependency bundle.
4. If all dependencies are present, try refreshing the package wiring:

   ```bash
   curl -u admin:<password> -X POST \
     "http://localhost:8080/system/console/bundles/<bundle-id>" \
     -d "action=refreshPackages"
   ```

5. If the bundle still does not activate, check the Sling error log for `BundleException` or `ClassNotFoundException` entries.

---

### Maven build fails with `package does not exist` errors

**Symptom**

Running `mvn clean install` produces compilation errors such as `package io.kestros.xxx does not exist` or `cannot find symbol`.

**Cause**

The local Maven repository does not contain one or more Kestros dependency artifacts. This can happen on a fresh clone or after a dependency version bump in `pom.xml`.

**Solution**

1. Ensure your local Maven repository is populated from the Kestros artifact repository or your organisation's proxy:

   ```bash
   mvn clean install -U
   ```

   The `-U` flag forces Maven to check remote repositories for updated snapshots.

2. If building behind a corporate proxy or firewall, confirm that `~/.m2/settings.xml` contains the correct proxy and mirror configuration pointing to the artifact repository hosting `io.kestros` artifacts.

3. If the missing artifact is a locally-developed Kestros module, build and install it first:

   ```bash
   cd /path/to/dependency-module
   mvn clean install -DskipTests
   ```

---

### Unit tests fail with `ResourceResolverFactory` not bound

**Symptom**

Test execution fails with errors such as `NullPointerException` on a `ResourceResolverFactory` reference, or OSGi service injection is not happening in test context.

**Cause**

Tests that rely on OSGi service injection (e.g., using Mockito or AEM Mocks) need the `ResourceResolverFactory` context to be set up explicitly. Without it, the injected factory reference remains `null`.

**Solution**

1. Use a Sling Mocks context in your test:

   ```java
   @ExtendWith(SlingContextExtension.class)
   class MyServiceTest {
       private final SlingContext context = new SlingContext(ResourceResolverType.JCR_MOCK);
       // ...
   }
   ```

2. Register any required services and resources on the context before invoking the class under test.

3. If you are testing a service that extends `BaseServiceResolverService`, mock `getServiceResourceResolver()` rather than the factory directly to avoid the full OSGi lifecycle.

---

### `mvn clean install` succeeds locally but CI fails with test errors

**Symptom**

Tests pass on a developer machine but fail in CI with assertion errors or timeout exceptions.

**Cause**

Environment-sensitive tests may behave differently under CI conditions — for example, tests that rely on a local Sling instance, hardcoded ports, or system-time-dependent assertions.

**Solution**

1. Check whether any tests directly reference `localhost` or a fixed port number. Replace these with configurable constants or system property lookups:

   ```java
   String host = System.getProperty("sling.host", "localhost");
   ```

2. Review tests that use `Thread.sleep` or `Date`/`Calendar` directly. Replace time-dependent logic with injectable clocks or mocks.

3. Run the full test suite with CI-equivalent JVM settings locally to reproduce the failure:

   ```bash
   mvn clean test -Dmaven.test.failure.ignore=false
   ```

---

### Deploying a new version does not replace the previous bundle

**Symptom**

After installing an updated version of the bundle, the Felix console still shows the old version running. Changes to code or configuration are not reflected.

**Cause**

The old bundle version was not uninstalled before the new one was installed, or the bundle symbolic name changed between versions, resulting in two bundles being installed simultaneously.

**Solution**

1. In the Felix Web Console, locate the old bundle by its symbolic name (`io.kestros.kestros-test`).
2. Stop and uninstall the old version before deploying the new one.
3. Alternatively, use the `action=update` parameter when posting to the Felix console — this replaces the running bundle in place:

   ```bash
   curl -u admin:<password> \
     -F "action=install" \
     -F "bundlestart=true" \
     -F "bundlefile=@target/kestros-test-0.1.0-SNAPSHOT.jar" \
     "http://localhost:8080/system/console/bundles"
   ```

   Replace `<password>` and the port with the values for your local instance.
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

## Usage Examples

The following examples demonstrate common patterns for testing OSGi components and Sling models in the Kestros framework using JUnit and the Sling Mocks testing utilities.

### 1. Setting up a Sling resource context for a unit test

Use `AemContext` (or `SlingContext`) to build an in-memory Sling repository and register resources without a running OSGi container.

```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyComponentTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testResourceType() {
        context.build()
            .resource("/content/mypage", "sling:resourceType", "kestros/components/my-component")
            .commit();

        assertEquals(
            "kestros/components/my-component",
            context.resourceResolver()
                .getResource("/content/mypage")
                .getResourceType()
        );
    }
}
```

### 2. Adapting a Sling resource to a Kestros model and asserting properties

Register a Sling model class in the test context and verify that property values are read correctly from JCR content.

```java
import io.kestros.commons.structuredsling.BaseComponent;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class BaseComponentModelTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testGetTitle() {
        context.addModelsForClasses(BaseComponent.class);
        context.build()
            .resource("/content/my-component",
                "jcr:title", "My Component Title",
                "sling:resourceType", "kestros/components/base")
            .commit();

        BaseComponent model = context.resourceResolver()
            .getResource("/content/my-component")
            .adaptTo(BaseComponent.class);

        assertEquals("My Component Title", model.getTitle());
    }
}
```

### 3. Registering and invoking an OSGi service in the test context

Register a mock OSGi service implementation against its interface and verify that a component under test calls it correctly.

```java
import io.kestros.tasks.api.services.TaskService;
import org.apache.sling.testing.mock.osgi.junit.OsgiContext;
import org.junit.Rule;
import org.junit.Test;
import static org.mockito.Mockito.*;

public class TaskServiceConsumerTest {

    @Rule
    public final OsgiContext context = new OsgiContext();

    @Test
    public void testServiceIsInvokedOnActivation() {
        TaskService mockService = mock(TaskService.class);
        context.registerService(TaskService.class, mockService);

        TaskServiceConsumer consumer = context.registerInjectActivateService(
            new TaskServiceConsumer()
        );

        verify(mockService, atLeastOnce()).getActiveTaskCount();
    }
}
```

### 4. Testing a Sling servlet using a mock request and response

Construct a mock `SlingHttpServletRequest` and `SlingHttpServletResponse` to exercise a servlet's `doGet` logic without an HTTP server.

```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.apache.sling.testing.mock.sling.servlet.MockSlingHttpServletRequest;
import org.apache.sling.testing.mock.sling.servlet.MockSlingHttpServletResponse;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyServletTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testDoGetReturnsJson() throws Exception {
        MockSlingHttpServletRequest request =
            new MockSlingHttpServletRequest(context.resourceResolver(), context.bundleContext());
        request.setQueryString("id=TEST-001");

        MockSlingHttpServletResponse response = new MockSlingHttpServletResponse();

        new MyServlet().doGet(request, response);

        assertEquals("application/json", response.getContentType());
    }
}
```

### 5. Asserting JCR child node structure after a service write operation

After a service method creates JCR nodes, verify the resulting tree structure using the resource resolver in the test context.

```java
import org.apache.sling.api.resource.Resource;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertNotNull;

public class StoryCreationServiceTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testStoryNodeCreatedWithNotesChild() {
        context.build()
            .resource("/content/kestros-tasks/stories/TEST-001",
                "title", "My First Story",
                "status", "todo")
            .resource("/content/kestros-tasks/stories/TEST-001/notes")
            .commit();

        Resource story = context.resourceResolver()
            .getResource("/content/kestros-tasks/stories/TEST-001");
        Resource notes = story.getChild("notes");

        assertNotNull("Story resource must exist", story);
        assertNotNull("Notes child node must be present", notes);
    }
}
```

## Usage Examples

The following examples demonstrate common patterns for testing OSGi components and Sling models in the Kestros framework using JUnit and the Sling Mocks testing utilities.

### 1. Setting up a Sling resource context for a unit test

Use `AemContext` (or `SlingContext`) to build an in-memory Sling repository and register resources without a running OSGi container.

```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyComponentTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testResourceType() {
        context.build()
            .resource("/content/mypage", "sling:resourceType", "kestros/components/my-component")
            .commit();

        assertEquals(
            "kestros/components/my-component",
            context.resourceResolver()
                .getResource("/content/mypage")
                .getResourceType()
        );
    }
}
```

### 2. Adapting a Sling resource to a Kestros model and asserting properties

Register a Sling model class in the test context and verify that property values are read correctly from JCR content.

```java
import io.kestros.commons.structuredsling.BaseComponent;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class BaseComponentModelTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testGetTitle() {
        context.addModelsForClasses(BaseComponent.class);
        context.build()
            .resource("/content/my-component",
                "jcr:title", "My Component Title",
                "sling:resourceType", "kestros/components/base")
            .commit();

        BaseComponent model = context.resourceResolver()
            .getResource("/content/my-component")
            .adaptTo(BaseComponent.class);

        assertEquals("My Component Title", model.getTitle());
    }
}
```

### 3. Registering and invoking an OSGi service in the test context

Register a mock OSGi service implementation against its interface and verify that a component under test calls it correctly.

```java
import io.kestros.tasks.api.services.TaskService;
import org.apache.sling.testing.mock.osgi.junit.OsgiContext;
import org.junit.Rule;
import org.junit.Test;
import static org.mockito.Mockito.*;

public class TaskServiceConsumerTest {

    @Rule
    public final OsgiContext context = new OsgiContext();

    @Test
    public void testServiceIsInvokedOnActivation() {
        TaskService mockService = mock(TaskService.class);
        context.registerService(TaskService.class, mockService);

        TaskServiceConsumer consumer = context.registerInjectActivateService(
            new TaskServiceConsumer()
        );

        verify(mockService, atLeastOnce()).getActiveTaskCount();
    }
}
```

### 4. Testing a Sling servlet using a mock request and response

Construct a mock `SlingHttpServletRequest` and `SlingHttpServletResponse` to exercise a servlet's `doGet` logic without an HTTP server.

```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.apache.sling.testing.mock.sling.servlet.MockSlingHttpServletRequest;
import org.apache.sling.testing.mock.sling.servlet.MockSlingHttpServletResponse;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyServletTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testDoGetReturnsJson() throws Exception {
        MockSlingHttpServletRequest request =
            new MockSlingHttpServletRequest(context.resourceResolver(), context.bundleContext());
        request.setQueryString("id=TEST-001");

        MockSlingHttpServletResponse response = new MockSlingHttpServletResponse();

        new MyServlet().doGet(request, response);

        assertEquals("application/json", response.getContentType());
    }
}
```

### 5. Asserting JCR child node structure after a service write operation

After a service method creates JCR nodes, verify the resulting tree structure using the resource resolver in the test context.

```java
import org.apache.sling.api.resource.Resource;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import static org.junit.Assert.assertNotNull;

public class StoryCreationServiceTest {

    @Rule
    public final SlingContext context = new SlingContext();

    @Test
    public void testStoryNodeCreatedWithNotesChild() {
        context.build()
            .resource("/content/kestros-tasks/stories/TEST-001",
                "title", "My First Story",
                "status", "todo")
            .resource("/content/kestros-tasks/stories/TEST-001/notes")
            .commit();

        Resource story = context.resourceResolver()
            .getResource("/content/kestros-tasks/stories/TEST-001");
        Resource notes = story.getChild("notes");

        assertNotNull("Story resource must exist", story);
        assertNotNull("Notes child node must be present", notes);
    }
}
```
4. After installing, verify the active version in the console and refresh the browser to clear any cached UI resources.

---

### JCR content nodes from test setup are not cleaned up between runs

**Symptom**

Subsequent test runs produce unexpected results because JCR nodes created by a previous test still exist under `/content/` or `/apps/`.

**Cause**

Tests that write to a shared JCR instance (rather than an in-memory mock) do not always clean up their created nodes on failure or exception, leaving stale content.

**Solution**

1. Prefer `ResourceResolverType.JCR_MOCK` or `ResourceResolverType.RESOURCERESOLVER_MOCK` in unit tests so content is never persisted to a real repository.

2. For integration tests that require a real Sling instance, implement a `@AfterEach` or `@AfterAll` teardown that deletes created nodes:

   ```java
   @AfterEach
   void cleanup() throws Exception {
       try (ResourceResolver resolver = resolverFactory.getAdministrativeResourceResolver(null)) {
           Resource node = resolver.getResource("/content/test-data");
           if (node != null) {
               resolver.delete(node);
               resolver.commit();
           }
       }
   }
   ```

3. If running against the shared dev instance, prefix all test content paths with a unique agent or session ID to avoid collisions with other active agents.

## Troubleshooting

This section covers common issues encountered when building, testing, or deploying `kestros-test`.

---

### Bundle fails to activate after deployment

**Symptom**

After deploying the bundle via the Felix Web Console, it appears in the bundle list with a `Resolved` or `Installed` state instead of `Active`.

**Cause**

One or more OSGi package imports declared in the bundle manifest cannot be satisfied by the currently installed bundles. This typically happens when a dependency bundle is missing, at the wrong version, or has not yet activated.

**Solution**

1. Open the Felix Web Console at `http://localhost:8080/system/console/bundles`.
2. Click the bundle name and inspect the **Imported Packages** section for any packages shown as unsatisfied (missing or in red).
3. Install and start the missing dependency bundle.
4. If all dependencies are present, try refreshing the package wiring:

   ```bash
   curl -u admin:<password> -X POST \
     "http://localhost:8080/system/console/bundles/<bundle-id>" \
     -d "action=refreshPackages"
   ```

5. If the bundle still does not activate, check the Sling error log for `BundleException` or `ClassNotFoundException` entries.

---

### Maven build fails with `package does not exist` errors

**Symptom**

Running `mvn clean install` produces compilation errors such as `package io.kestros.xxx does not exist` or `cannot find symbol`.

**Cause**

The local Maven repository does not contain one or more Kestros dependency artifacts. This can happen on a fresh clone or after a dependency version bump in `pom.xml`.

**Solution**

1. Ensure your local Maven repository is populated from the Kestros artifact repository or your organisation's proxy:

   ```bash
   mvn clean install -U
   ```

   The `-U` flag forces Maven to check remote repositories for updated snapshots.

2. If building behind a corporate proxy or firewall, confirm that `~/.m2/settings.xml` contains the correct proxy and mirror configuration pointing to the artifact repository hosting `io.kestros` artifacts.

3. If the missing artifact is a locally-developed Kestros module, build and install it first:

   ```bash
   cd /path/to/dependency-module
   mvn clean install -DskipTests
   ```

---

### Unit tests fail with `ResourceResolverFactory` not bound

**Symptom**

Test execution fails with errors such as `NullPointerException` on a `ResourceResolverFactory` reference, or OSGi service injection is not happening in test context.

**Cause**

Tests that rely on OSGi service injection (e.g., using Mockito or AEM Mocks) need the `ResourceResolverFactory` context to be set up explicitly. Without it, the injected factory reference remains `null`.

**Solution**

1. Use a Sling Mocks context in your test:

   ```java
   @ExtendWith(SlingContextExtension.class)
   class MyServiceTest {
       private final SlingContext context = new SlingContext(ResourceResolverType.JCR_MOCK);
       // ...
   }
   ```

2. Register any required services and resources on the context before invoking the class under test.

3. If you are testing a service that extends `BaseServiceResolverService`, mock `getServiceResourceResolver()` rather than the factory directly to avoid the full OSGi lifecycle.

---

### `mvn clean install` succeeds locally but CI fails with test errors

**Symptom**

Tests pass on a developer machine but fail in CI with assertion errors or timeout exceptions.

**Cause**

Environment-sensitive tests may behave differently under CI conditions — for example, tests that rely on a local Sling instance, hardcoded ports, or system-time-dependent assertions.

**Solution**

1. Check whether any tests directly reference `localhost` or a fixed port number. Replace these with configurable constants or system property lookups:

   ```java
   String host = System.getProperty("sling.host", "localhost");
   ```

2. Review tests that use `Thread.sleep` or `Date`/`Calendar` directly. Replace time-dependent logic with injectable clocks or mocks.

3. Run the full test suite with CI-equivalent JVM settings locally to reproduce the failure:

   ```bash
   mvn clean test -Dmaven.test.failure.ignore=false
   ```

---

### Deploying a new version does not replace the previous bundle

**Symptom**

After installing an updated version of the bundle, the Felix console still shows the old version running. Changes to code or configuration are not reflected.

**Cause**

The old bundle version was not uninstalled before the new one was installed, or the bundle symbolic name changed between versions, resulting in two bundles being installed simultaneously.

**Solution**

1. In the Felix Web Console, locate the old bundle by its symbolic name (`io.kestros.kestros-test`).
2. Stop and uninstall the old version before deploying the new one.
3. Alternatively, use the `action=update` parameter when posting to the Felix console — this replaces the running bundle in place:

   ```bash
   curl -u admin:<password> \
     -F "action=install" \
     -F "bundlestart=true" \
     -F "bundlefile=@target/kestros-test-0.1.0-SNAPSHOT.jar" \
     "http://localhost:8080/system/console/bundles"
   ```

4. After installing, verify the active version in the console and refresh the browser to clear any cached UI resources.

---

### JCR content nodes from test setup are not cleaned up between runs

**Symptom**

Subsequent test runs produce unexpected results because JCR nodes created by a previous test still exist under `/content/` or `/apps/`.

**Cause**

Tests that write to a shared JCR instance (rather than an in-memory mock) do not always clean up their created nodes on failure or exception, leaving stale content.

**Solution**

1. Prefer `ResourceResolverType.JCR_MOCK` or `ResourceResolverType.RESOURCERESOLVER_MOCK` in unit tests so content is never persisted to a real repository.

2. For integration tests that require a real Sling instance, implement a `@AfterEach` or `@AfterAll` teardown that deletes created nodes:

   ```java
   @AfterEach
   void cleanup() throws Exception {
       try (ResourceResolver resolver = resolverFactory.getAdministrativeResourceResolver(null)) {
           Resource node = resolver.getResource("/content/test-data");
           if (node != null) {
               resolver.delete(node);
               resolver.commit();
           }
       }
   }
   ```

3. If running against the shared dev instance, prefix all test content paths with a unique agent or session ID to avoid collisions with other active agents.
All tests must pass with zero failures and zero skipped before a PR is submitted.

**Q: How do I reset test data?**

Test data in the shared dev instance lives under `/content/kestros-tasks/stories/TEST-NNN/`. To reset:

1. Log in to the Felix Content Browser at `http://192.168.86.216:9000/system/console/jcrresolver`.
2. Navigate to `/content/kestros-tasks/stories/`.
3. Delete any `TEST-NNN` nodes you want to reset.

Alternatively, if your test code created content programmatically, use the JCR admin API or a teardown script. Prefer `JCR_MOCK` resolvers in unit tests to avoid persisting data to the shared instance at all.

**Q: How do I add new test stories?**

New test stories are created via the kestros-tasks API:

```bash
curl -s -u admin:kestros-dev000 -X POST "http://192.168.86.216:9000/api/stories/create" \
  -d "title=[TEST] My new test story&description=Description here&status=todo&board=test"
```

Use the `board=test` parameter so the story appears on the test board rather than the main product board. Follow the `[TEST]` title prefix convention so test stories are clearly identifiable and can be bulk-cleaned up later.

**Q: Who do I contact with questions?**

For questions about the Kestros agent pipeline, standards, or this repository, post a message in the `#kestros-dev` Slack channel or open a discussion post on the relevant story via the task board at `http://192.168.86.216:9000/ui/board.html`. For urgent issues, DM Danny directly via Slack (`@danny`). Agent-specific questions can also be filed as feedback in `kestros-claude/agent-feedback/`.

**Q: Why does `main` only have an empty initial commit?**

The `kestros-test` repository is built up incrementally by agents executing test stories. Each section of the README — Overview, Installation, Configuration, Usage Examples, Troubleshooting, Contributing, License, and this FAQ — is added by a separate story and PR. This mirrors the real Kestros agent workflow: each agent works on a scoped branch, opens a PR, and the changes are merged in sequence. The empty initial commit is intentional — it gives every agent branch a common base to branch from.

**Q: Are there any branch or commit message rules I need to follow?**

Yes. All branches must follow the pattern `{agent-id}/TASK-NNN`. Commit messages use conventional prefixes (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`) and must never reference AI tools, Claude, or Anthropic. PRs always target `main`. See the [Contributing](#contributing) section for the full PR checklist.

## FAQ

**Q: What is this repository for?**

`kestros-test` is a synthetic test repository used to validate the Kestros multi-agent pipeline end-to-end. It does not contain production application code. Instead, it serves as a controlled environment for testing agent workflows, story lifecycle transitions, PR creation, review gates, and deployment automation. All stories created against this repo are test stories marked with `[TEST]` in their titles.

**Q: How do I run the tests?**

Tests are run using Maven from the repository root:

```bash
mvn clean install
```

This compiles all source, executes unit tests, and packages the bundle. For integration tests that require a running Sling instance, deploy the bundle to your local dev instance first, then run the smoke test:

```bash
bash scripts/smoke-test.sh http://localhost:9000
```

All tests must pass with zero failures and zero skipped before a PR is submitted.

**Q: How do I reset test data?**

Test data in the shared dev instance lives under `/content/kestros-tasks/stories/TEST-NNN/`. To reset:

1. Log in to the Felix Content Browser at `http://192.168.86.216:9000/system/console/jcrresolver`.
2. Navigate to `/content/kestros-tasks/stories/`.
3. Delete any `TEST-NNN` nodes you want to reset.

Alternatively, if your test code created content programmatically, use the JCR admin API or a teardown script. Prefer `JCR_MOCK` resolvers in unit tests to avoid persisting data to the shared instance at all.

**Q: How do I add new test stories?**

New test stories are created via the kestros-tasks API:

```bash
curl -s -u admin:kestros-dev000 -X POST "http://192.168.86.216:9000/api/stories/create" \
  -d "title=[TEST] My new test story&description=Description here&status=todo&board=test"
```

Use the `board=test` parameter so the story appears on the test board rather than the main product board. Follow the `[TEST]` title prefix convention so test stories are clearly identifiable and can be bulk-cleaned up later.

**Q: Who do I contact with questions?**

For questions about the Kestros agent pipeline, standards, or this repository, post a message in the `#kestros-dev` Slack channel or open a discussion post on the relevant story via the task board at `http://192.168.86.216:9000/ui/board.html`. For urgent issues, DM Danny directly via Slack (`@danny`). Agent-specific questions can also be filed as feedback in `kestros-claude/agent-feedback/`.

**Q: Why does `main` only have an empty initial commit?**

The `kestros-test` repository is built up incrementally by agents executing test stories. Each section of the README — Overview, Installation, Configuration, Usage Examples, Troubleshooting, Contributing, License, and this FAQ — is added by a separate story and PR. This mirrors the real Kestros agent workflow: each agent works on a scoped branch, opens a PR, and the changes are merged in sequence. The empty initial commit is intentional — it gives every agent branch a common base to branch from.

**Q: Are there any branch or commit message rules I need to follow?**

Yes. All branches must follow the pattern `{agent-id}/TASK-NNN`. Commit messages use conventional prefixes (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`) and must never reference AI tools, Claude, or Anthropic. PRs always target `main`. See the [Contributing](#contributing) section for the full PR checklist.

## FAQ

**Q: What is this repository for?**

`kestros-test` is a synthetic test repository used to validate the Kestros multi-agent pipeline end-to-end. It does not contain production application code. Instead, it serves as a controlled environment for testing agent workflows, story lifecycle transitions, PR creation, review gates, and deployment automation. All stories created against this repo are test stories marked with `[TEST]` in their titles.

**Q: How do I run the tests?**

Tests are run using Maven from the repository root:

```bash
mvn clean install
```

This compiles the source, runs all unit tests, and packages the bundle. A PR may not be submitted if `mvn clean install` fails.

For changes that affect API endpoints or servlet behaviour, run the smoke test against your local Sling instance after deploying:
This compiles all source, executes unit tests, and packages the bundle. For integration tests that require a running Sling instance, deploy the bundle to your local dev instance first, then run the smoke test:

```bash
bash scripts/smoke-test.sh http://localhost:9000
```

All tests must pass. Zero failures, zero skipped. Do not submit a PR with a failing test suite.

## Contributing

Contributions to `kestros-test` follow the same standards as all Kestros repositories. The guidelines below mirror the system-wide standards defined in `kestros-claude/standards/`.

### Pull Request Process

1. **Branch naming** — all branches must follow the convention `{agent-id}/TASK-NNN` (e.g. `dev-readme-06/TEST-026`). Feature branches created outside of the task system use `feature/short-description`.
2. **Branch from `main`** — always create your branch from `main`. Never branch from another feature branch.
3. **One branch per task, one PR per task** — scope each branch and PR to a single task. Do not batch multiple task IDs into a single PR.
4. **PR title format** — use `[kestros-test] Brief description of change` (e.g. `[kestros-test] Add Contributing section to README`).
5. **Target branch** — all PRs target `main`.
6. **Required reviewers** — at minimum, a Lead Developer review is required before merge. QA sign-off is required before the task moves to `pending-approval`.
7. **No AI references** — commit messages must never reference Claude, Anthropic, or any AI tool. Conventional prefixes only: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`.

### Code Style

- Follow the formatting conventions established in `kestros-claude/standards/`.
- Java code uses standard Maven project structure. Keep package declarations consistent with `io.kestros.test`.
- Commit messages use the format `[artifact-id]: action, brief result` — imperative mood, under 72 characters on the first line.
- Do not commit build artifacts, IDE config files, or generated output. Stage only the files relevant to the change.
- Keep scope contained: only modify files that are directly required by the task.

### Testing

Before submitting a PR, the following must pass locally:

```bash
mvn clean install
```

This compiles the source, runs all unit tests, and packages the bundle. A PR may not be submitted if `mvn clean install` fails.

For changes that affect API endpoints or servlet behaviour, run the smoke test against your local Sling instance after deploying:

```bash
bash scripts/smoke-test.sh http://localhost:9000
```

All tests must pass. Zero failures, zero skipped. Do not submit a PR with a failing test suite.
All tests must pass with zero failures and zero skipped before a PR is submitted.

**Q: How do I reset test data?**

Test data in the shared dev instance lives under `/content/kestros-tasks/stories/TEST-NNN/`. To reset:

1. Log in to the Felix Content Browser at `http://192.168.86.216:9000/system/console/jcrresolver`.
2. Navigate to `/content/kestros-tasks/stories/`.
3. Delete any `TEST-NNN` nodes you want to reset.

Alternatively, if your test code created content programmatically, use the JCR admin API or a teardown script. Prefer `JCR_MOCK` resolvers in unit tests to avoid persisting data to the shared instance at all.

**Q: How do I add new test stories?**

New test stories are created via the kestros-tasks API:

```bash
curl -s -u admin:kestros-dev000 -X POST "http://192.168.86.216:9000/api/stories/create" \
  -d "title=[TEST] My new test story&description=Description here&status=todo&board=test"
```

Use the `board=test` parameter so the story appears on the test board rather than the main product board. Follow the `[TEST]` title prefix convention so test stories are clearly identifiable and can be bulk-cleaned up later.

**Q: Who do I contact with questions?**

For questions about the Kestros agent pipeline, standards, or this repository, post a message in the `#kestros-dev` Slack channel or open a discussion post on the relevant story via the task board at `http://192.168.86.216:9000/ui/board.html`. For urgent issues, DM Danny directly via Slack (`@danny`). Agent-specific questions can also be filed as feedback in `kestros-claude/agent-feedback/`.

**Q: Why does `main` only have an empty initial commit?**

The `kestros-test` repository is built up incrementally by agents executing test stories. Each section of the README — Overview, Installation, Configuration, Usage Examples, Troubleshooting, Contributing, License, and this FAQ — is added by a separate story and PR. This mirrors the real Kestros agent workflow: each agent works on a scoped branch, opens a PR, and the changes are merged in sequence. The empty initial commit is intentional — it gives every agent branch a common base to branch from.

**Q: Are there any branch or commit message rules I need to follow?**

Yes. All branches must follow the pattern `{agent-id}/TASK-NNN`. Commit messages use conventional prefixes (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`) and must never reference AI tools, Claude, or Anthropic. PRs always target `main`. See the [Contributing](#contributing) section for the full PR checklist.

## FAQ

**Q: What is this repository for?**

`kestros-test` is a synthetic test repository used to validate the Kestros multi-agent pipeline end-to-end. It does not contain production application code. Instead, it serves as a controlled environment for testing agent workflows, story lifecycle transitions, PR creation, review gates, and deployment automation. All stories created against this repo are test stories marked with `[TEST]` in their titles.

**Q: How do I run the tests?**

Tests are run using Maven from the repository root:

```bash
mvn clean install
```

This compiles all source, executes unit tests, and packages the bundle. For integration tests that require a running Sling instance, deploy the bundle to your local dev instance first, then run the smoke test:

```bash
bash scripts/smoke-test.sh http://localhost:9000
```

All tests must pass with zero failures and zero skipped before a PR is submitted.

**Q: How do I reset test data?**

Test data in the shared dev instance lives under `/content/kestros-tasks/stories/TEST-NNN/`. To reset:

1. Log in to the Felix Content Browser at `http://192.168.86.216:9000/system/console/jcrresolver`.
2. Navigate to `/content/kestros-tasks/stories/`.
3. Delete any `TEST-NNN` nodes you want to reset.

Alternatively, if your test code created content programmatically, use the JCR admin API or a teardown script. Prefer `JCR_MOCK` resolvers in unit tests to avoid persisting data to the shared instance at all.

**Q: How do I add new test stories?**

New test stories are created via the kestros-tasks API:

```bash
curl -s -u admin:kestros-dev000 -X POST "http://192.168.86.216:9000/api/stories/create" \
  -d "title=[TEST] My new test story&description=Description here&status=todo&board=test"
```

Use the `board=test` parameter so the story appears on the test board rather than the main product board. Follow the `[TEST]` title prefix convention so test stories are clearly identifiable and can be bulk-cleaned up later.

**Q: Who do I contact with questions?**

For questions about the Kestros agent pipeline, standards, or this repository, post a message in the `#kestros-dev` Slack channel or open a discussion post on the relevant story via the task board at `http://192.168.86.216:9000/ui/board.html`. For urgent issues, DM Danny directly via Slack (`@danny`). Agent-specific questions can also be filed as feedback in `kestros-claude/agent-feedback/`.

**Q: Why does `main` only have an empty initial commit?**

The `kestros-test` repository is built up incrementally by agents executing test stories. Each section of the README — Overview, Installation, Configuration, Usage Examples, Troubleshooting, Contributing, License, and this FAQ — is added by a separate story and PR. This mirrors the real Kestros agent workflow: each agent works on a scoped branch, opens a PR, and the changes are merged in sequence. The empty initial commit is intentional — it gives every agent branch a common base to branch from.

**Q: Are there any branch or commit message rules I need to follow?**

Yes. All branches must follow the pattern `{agent-id}/TASK-NNN`. Commit messages use conventional prefixes (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`) and must never reference AI tools, Claude, or Anthropic. PRs always target `main`. See the [Contributing](#contributing) section for the full PR checklist.
