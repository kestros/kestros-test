## Usage

Add `kestros-test` as a test-scoped dependency in your Maven project, then extend the provided base test classes:

```java
import org.junit.Test;
import static org.junit.Assert.*;

public class MyComponentTest /* extends KestrosBaseTest once added */ {

    @Test
    public void testExample() {
        // Example: assert your component logic here
        assertTrue(true);
    }
}
```

> **Note:** This repository is a test harness for pipeline validation. Base class names and imports will be updated here as the module evolves.
