# Gradle Build Tool & Modern Build Automation

## 1. What is Gradle?

**Gradle** is an open-source build automation tool designed for multi-language and large-scale software development.

Unlike Maven—which primarily uses declarative XML configuration (`pom.xml`)—Gradle uses a programmatic **Domain-Specific Language (DSL)** based on **Groovy** or **Kotlin**.

### Language Support

- **Maven**: Primarily designed for Java/JVM projects, but can support other languages through plugins.
- **Gradle**: Supports multiple languages and ecosystems, including Java, Kotlin, Groovy, Scala, C++, Swift, and Android projects.

---

## 2. Configuration: Groovy DSL vs Kotlin DSL

Gradle build scripts use a **DSL (Domain-Specific Language)** rather than XML.

The two commonly used Gradle DSLs are:

- **Groovy DSL** → `build.gradle`
- **Kotlin DSL** → `build.gradle.kts`

### Groovy DSL vs Kotlin DSL

| Feature         | Groovy DSL (`build.gradle`) | Kotlin DSL (`build.gradle.kts`)                                |
| :-------------- | :-------------------------- | :------------------------------------------------------------- |
| **Typing**      | Dynamically typed           | Statically typed                                               |
| **IDE support** | Good                        | Generally stronger type-aware autocomplete and error detection |
| **Refactoring** | Less type-safe              | Better type safety and refactoring support                     |
| **Syntax**      | More concise/flexible       | More explicit, standard Kotlin syntax                          |

### Syntax Comparison

#### Groovy DSL (`build.gradle`)

```groovy
plugins {
    id 'java'
    id 'application'
}

group = 'com.telusko'
version = '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
}

// Customize archive output name
tasks.named('jar') {
    archiveBaseName.set('my-custom-app')
}
```

#### Kotlin DSL (`build.gradle.kts`)

```kotlin
plugins {
    java
    application
}

group = "com.telusko"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
}

// Customize archive output name
tasks.named<Jar>("jar") {
    archiveBaseName.set("my-custom-app")
}
```

---

## 3. Gradle Project Files

A typical Gradle project can contain:

```text
my-project/
 ├── build.gradle                  # Build logic, dependencies & plugins
 ├── settings.gradle               # Root project name & subproject definitions
 ├── gradlew                       # Linux / macOS wrapper shell script
 ├── gradlew.bat                   # Windows wrapper batch script
 └── gradle/
      └── wrapper/
           ├── gradle-wrapper.jar         # Downloads declared Gradle distribution
           └── gradle-wrapper.properties  # Version and download URL metadata
```

### `build.gradle`

This is where we configure how the project should be built, such as:

- Plugins
- Dependencies
- Repositories
- Compilation settings
- Testing
- Packaging
- Custom tasks

### `settings.gradle`

This file configures the Gradle build itself and is particularly important for identifying/configuring projects and subprojects in multi-project builds.

---

## 4. The Gradle Wrapper (`gradlew`)

The **Gradle Wrapper** is the recommended way to execute a Gradle build.

It allows a project to use a specific Gradle version without requiring developers to install that version of Gradle globally.

### Wrapper Components

```text
my-project/

├── gradlew                  # Linux / macOS shell script
├── gradlew.bat              # Windows batch script
└── gradle/
    └── wrapper/
        ├── gradle-wrapper.jar
        └── gradle-wrapper.properties
```

### How the Wrapper Works

```text
Developer runs ./gradlew build
(or gradlew.bat build on Windows)

                    │
                    ▼

Does the configured Gradle distribution
already exist in Gradle User Home?

        ├── Yes ──► Use the cached Gradle distribution
        │
        └── No  ──► Download the configured Gradle distribution,
                    cache it, and execute the build
```

### Key Advantages

- **Version Pinning**: The project specifies the Gradle version it should use, avoiding differences between developers' global Gradle installations.
- **No Manual Gradle Installation**: Developers and CI/CD systems can use the wrapper instead of installing Gradle separately.
- **CI/CD Ready**: A CI environment can execute the Gradle Wrapper with a compatible Java runtime/JDK.

> The Gradle Wrapper is used to **build and execute Gradle tasks**. It is not required to run an already-built JAR.

---

## 5. Gradle vs Maven: Execution & Performance

| Feature                | Apache Maven                                                                                | Gradle                                                                                         |
| :--------------------- | :------------------------------------------------------------------------------------------ | :--------------------------------------------------------------------------------------------- |
| **Configuration**      | XML (`pom.xml`)                                                                             | Groovy / Kotlin DSL                                                                            |
| **Execution Model**    | Lifecycle phases and plugin goals                                                           | Task-based execution using a task dependency graph                                             |
| **Build Optimization** | Supports incremental compilation and other optimizations depending on configuration/plugins | Strong incremental-build, caching, and task-avoidance capabilities                             |
| **Daemon**             | No equivalent persistent build daemon by default                                            | Gradle Daemon keeps a background JVM process available for builds                              |
| **Build Performance**  | Depends on project and configuration                                                        | Can benefit significantly from incremental builds, caching, parallelism, and the Gradle Daemon |

### Why Can Gradle Be Faster?

1. **Incremental Builds**

   Gradle tracks task inputs and outputs. If a task's inputs and outputs have not changed, Gradle can mark the task as `UP-TO-DATE` and skip executing it.

2. **Incremental Compilation**

   For supported compilation tasks, Gradle can compile only affected portions of the source code instead of recompiling everything unnecessarily.

3. **Build Cache**

   When enabled, Gradle can reuse outputs from previous builds through a local or remote build cache, avoiding repeated work.

4. **Gradle Daemon**

   The Gradle Daemon is a long-lived JVM process that can remain running between builds. This avoids repeatedly starting a new JVM and allows Gradle to benefit from JVM/JIT optimizations.

---

## 6. Common Gradle Commands

Gradle tasks correspond to individual units of work.

It is generally recommended to use the Gradle Wrapper for consistency:

```bash
# Compile code, run tests, and produce build artifacts
./gradlew build

# Windows
gradlew.bat build

# Execute the application directly
# Requires the Application plugin
./gradlew run

# Remove the build/ output directory
./gradlew clean

# Run unit tests
./gradlew test

# List available Gradle tasks
./gradlew tasks
```

---

## 7. Build Artifacts

Running:

```bash
./gradlew build
```

typically compiles the source code, runs the configured tests, and produces the configured build artifacts.

For a typical Java Gradle project:

```text
my-project/

└── build/

    ├── classes/       # Compiled bytecode
    ├── reports/       # Test/report output
    └── libs/          # Packaged JAR/WAR artifacts
```

The `build/` directory contains generated build outputs.

### Running a JAR

If the generated JAR is executable, it can be run using Java:

```bash
java -jar build/libs/my-app.jar
```

**Maven or Gradle is not required to run the already-built JAR.**

The required Java runtime/JDK depends on how the application was built.

### WAR and Tomcat

For a traditional WAR-based web application, the WAR can be deployed to an external servlet container such as Tomcat.

For example:

```text
my-app.war
       │
       ▼
Tomcat/webapps/
```

`java -jar` is generally used to execute an executable JAR, not to deploy a traditional WAR to an external Tomcat server.

Spring Boot applications can also package an embedded server and commonly run using:

```bash
java -jar application.jar
```

---

## 8. Maven Wrapper vs Gradle Wrapper

Gradle is not the only build tool with a wrapper.

Gradle provides:

```text
gradlew
gradlew.bat
```

Maven provides:

```text
mvnw
mvnw.cmd
```

Both wrappers allow a project to use a specified build-tool version without requiring developers to manually install that build tool globally.

Therefore:

> A Maven project does not necessarily require Maven to be manually installed when the Maven Wrapper is included.

---

## 9. Archive Name

Gradle can customize the name of generated archives such as JAR files.

For example, with the Groovy DSL:

```groovy
tasks.named('jar') {
    archiveBaseName.set('my-custom-app')
}
```

This changes the base name of the generated archive.

For example, instead of:

```text
build/libs/my-app-1.0-SNAPSHOT.jar
```

the generated artifact can have a base name such as:

```text
build/libs/my-custom-app-1.0-SNAPSHOT.jar
```

---

# Quick Recall

- **Gradle** → Build automation tool supporting multiple languages and ecosystems.
- **DSL** → Gradle build scripts can use Groovy (`build.gradle`) or Kotlin (`build.gradle.kts`).
- **`build.gradle`** → Groovy-based Gradle build configuration.
- **`build.gradle.kts`** → Kotlin-based Gradle build configuration.
- **`settings.gradle`** → Configures the Gradle build and identifies/configures projects and subprojects.
- **`gradlew` / `gradlew.bat`** → Gradle Wrapper scripts used to execute the project's configured Gradle version without manually installing Gradle.
- **`build/`** → Directory containing generated Gradle build outputs.
- **`build/libs/`** → Standard output directory for packaged JAR/WAR artifacts in a typical Java Gradle project.
- **`UP-TO-DATE`** → Gradle determines that a task's inputs and outputs have not changed, so the task can be skipped.
- **Incremental build** → Gradle can avoid rebuilding unaffected parts of a project.
- **Build Cache** → When enabled, Gradle can reuse task outputs from previous builds.
- **Gradle Daemon** → Long-lived JVM process that can improve build performance.
- **`archiveBaseName`** → Property used to customize the base name of generated archives.
- **`java -jar`** → Runs an executable JAR; Maven/Gradle is not required to run an already-built JAR.
- **Maven Wrapper** → Maven also provides `mvnw` / `mvnw.cmd`, so Maven does not always need to be manually installed.

---

## Feedback on Raw Notes & Mistakes Clarified

### 1. "To run a Maven JAR in another system, Maven has to be installed"

**Incorrect.**

Neither Maven nor Gradle is required to run an already-built executable JAR.

For example:

```bash
java -jar my-app.jar
```

Build tools are primarily used to:

- Compile source code
- Resolve dependencies
- Run tests
- Package applications
- Execute other build tasks

If someone needs to **build the source code**, they can use the project's wrapper:

- Gradle → `gradlew` / `gradlew.bat`
- Maven → `mvnw` / `mvnw.cmd`

---

### 2. "Gradle has a wrapper but Maven doesn't"

**Incorrect/outdated.**

Maven also has a wrapper:

```text
mvnw
mvnw.cmd
```

Therefore, both Maven and Gradle can provide a project-local mechanism for obtaining/using the required build-tool version.

---

### 3. "java -jar <build file> deploys it to Tomcat"

**Incorrect.**

`java -jar` runs an executable JAR.

A traditional WAR application intended for an external Tomcat server is deployed to the Tomcat server, commonly through its `webapps/` directory or another deployment mechanism.

---

### 4. "Maven compiles everything every time"

**Too broad and incorrect.**

Maven should not be described as always recompiling every source file.

Maven and its compiler/build plugins can perform incremental compilation depending on the configuration and build state.

Gradle provides extensive mechanisms for incremental builds, task avoidance, caching, and other build optimizations.

---

### 5. "Gradle only compiles newly added code"

**Too broad.**

Gradle supports incremental compilation and incremental builds, but it does not simply compile only newly added code in every situation.

Gradle determines what work needs to be performed based on task inputs, outputs, dependencies, and configuration.

---

### 6. `jar { archivesBaseName: "myname" }`

This is an older/outdated approach and should not be memorized as the modern Gradle syntax.

A modern Groovy DSL example is:

```groovy
tasks.named('jar') {
    archiveBaseName.set('myname')
}
```

---

### 7. Important distinction: Build vs Run

Remember this distinction:

```text
Source Code
    │
    ▼
Maven / Gradle
    │
    ├── Compile
    ├── Test
    ├── Package
    ▼
JAR / WAR
    │
    ▼
Java Runtime / Application Server
```

**Maven/Gradle → build the application**

**Java/Tomcat/etc. → run or host the built application**
