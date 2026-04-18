---
description: Add a Spring Modulith modularity test to a Java project
argument-hint: (optional) space-separated module names to exclude, e.g. "util helper"
---

You are tasked with adding a Spring Modulith `ModularityTest` to a Java project. Follow these steps carefully.

## Step 1: Discover Project Structure

1. Find the main application class:
   - Search for a class annotated with `@SpringBootApplication` or `@Modulithic`
   - Extract its fully-qualified package name — this is the **base package**
   - Note the class name (e.g., `MyApplication`)

2. List all **top-level subdirectories** under `src/main/java/<base-package>/` — each is a potential Spring Modulith module

3. Locate the test root: `src/test/java/<base-package>/`

## Step 2: Check Spring Modulith Dependencies

Open `build.gradle` (Gradle) or `pom.xml` (Maven) and check:

### Gradle
- **BOM** in `dependencyManagement.imports`: `org.springframework.modulith:spring-modulith-bom:<version>`
- **Runtime**: `implementation 'org.springframework.modulith:spring-modulith-starter-core'`
- **Test**: `testImplementation 'org.springframework.modulith:spring-modulith-test'`

If any are missing, add them. Use the latest stable version from the BOM (default: `1.2.2` if unknown).

### Maven
- **BOM** in `<dependencyManagement>`: `spring-modulith-bom`
- **Runtime**: `spring-modulith-starter-core`
- **Test**: `spring-modulith-test` with `<scope>test</scope>`

### Main application class
- If `@Modulithic` is absent from the main application class, add it (import: `org.springframework.modulith.Modulithic`).

## Step 3: Check for Existing Test

Check if `ModularityTest.java` already exists at `src/test/java/<base-package>/ModularityTest.java`.

- **If it exists**: show its current content and ask the user what they want to change. Do NOT overwrite silently.
- **If it does not exist**: proceed to Step 4.

## Step 4: Determine Module Exclusions

If the user provided arguments (module names), treat them as exclusions directly.

Otherwise, present the discovered modules and use `AskUserQuestion` to ask:

> "I found these modules: [list]. Do any need to be excluded from modularity verification?
>
> Modules typically need exclusion when they:
> - Use `@Autowired` field injection that violates module boundaries
> - Are shared utilities intentionally used by multiple modules (cross-module DI)
>
> Enter module names to exclude (comma-separated), or leave blank for none."

## Step 5: Generate ModularityTest.java

Create `src/test/java/<base-package>/ModularityTest.java` using the template below.

### Rules
- Javadoc on the class **must be in Russian**
- Only include the `DEFAULT_EXCLUSIONS` field and its imports if there are actual exclusions
- Use `ApplicationModules.Filters.withoutModules(...)` for the exclusion predicate
- Add a comment per excluded module explaining **why** it is excluded (ask the user if unknown)
- **DO NOT run the test** — tell the user to run it manually with `./gradlew test` (or `mvn test`) when ready

### Template (no exclusions)

```java
package <base.package>;

import org.junit.jupiter.api.Test;
import org.springframework.modulith.core.ApplicationModules;

/**
 * Тест для проверки модульной архитектуры приложения.
 * <p>
 * Использует Spring Modulith для верификации границ между модулями
 * и обеспечения соблюдения принципов модульного дизайна.
 * </p>
 */
public class ModularityTest {

    ApplicationModules modules = ApplicationModules.of(<MainApp>.class);

    @Test
    void verifiesModularStructure() {
        modules.verify();
    }
}
```

### Template (with exclusions)

```java
package <base.package>;

import com.tngtech.archunit.base.DescribedPredicate;
import com.tngtech.archunit.core.domain.JavaClass;
import org.junit.jupiter.api.Test;
import org.springframework.modulith.core.ApplicationModules;

/**
 * Тест для проверки модульной архитектуры приложения.
 * <p>
 * Использует Spring Modulith для верификации границ между модулями
 * и обеспечения соблюдения принципов модульного дизайна.
 * </p>
 */
public class ModularityTest {

    // <module1> - <reason for exclusion>
    // <module2> - <reason for exclusion>
    static final DescribedPredicate<JavaClass> DEFAULT_EXCLUSIONS =
            ApplicationModules.Filters.withoutModules("<module1>", "<module2>");

    ApplicationModules modules = ApplicationModules.of(<MainApp>.class, DEFAULT_EXCLUSIONS);

    @Test
    void verifiesModularStructure() {
        modules.verify();
    }
}
```

## Step 6: Report to User

After generating the file, tell the user:
- What was created or modified (file paths)
- Which modules were detected
- Which modules were excluded and why
- How to run the test: `./gradlew test` or `mvn test` (do NOT run it yourself)
- Any dependency changes made to `build.gradle` / `pom.xml`
