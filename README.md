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

```bash
mvn clean install
```

This compiles the source, runs all unit tests, and packages the bundle. A PR may not be submitted if `mvn clean install` fails.

For changes that affect API endpoints or servlet behaviour, run the smoke test against your local Sling instance after deploying:

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
