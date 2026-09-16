````markdown
# Java Day 3 — Maven Fundamentals, Lifecycles & Architecture

## 1. What is Maven & Why Do We Need It?

**Apache Maven** is an open-source build automation and dependency management tool primarily used for Java projects.

### The Build Process

Building software is not just compiling source code. A complete build includes:

```text
Source Code (.java)
       ↓  (Compile)
Bytecode (.class)
       ↓  (Test execution via JUnit/TestNG)
Test Results Verified
       ↓  (Package)
Artifact Produced (.jar / .war)
```
````

### The Problem: Life Before Build Tools

Without Maven, developers had to manage builds manually:

1. **Manual Dependency Lookup**: Find, download, and track external `.jar` files (Spring, Hibernate, JDBC drivers).
2. **Classpath Hell**: Manually configure IDE build paths (e.g., Eclipse `.classpath` file or IntelliJ module settings).
3. **Transitive Dependency Nightmare**: If Library A depends on Library B version 2.0, but Library C depends on Library B version 1.0, resolving conflicts manually was brittle and error-prone.
4. **Non-Standard Folder Structures**: Every developer had their own directory conventions, making onboarding difficult.

Maven eliminates this friction by standardizing project layout, automating the lifecycle, and resolving external dependencies automatically.

---

## 2. JAR vs WAR

| Feature                | JAR (Java ARchive)                                                                    | WAR (Web Application ARchive)                                                                        |
| ---------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Extension**          | `.jar`                                                                                | `.war`                                                                                               |
| **Primary Use**        | Standalone applications, desktop tools, reusable libraries, Spring Boot runnable apps | Traditional enterprise web applications                                                              |
| **Internal Structure** | Compiled `.class` files, resource assets, and `META-INF/MANIFEST.MF`                  | Servlets, JSPs, static web assets (HTML/CSS/JS), `WEB-INF/web.xml`, and `WEB-INF/lib` (bundled JARs) |
| **Deployment Target**  | Executed directly via JVM (`java -jar app.jar`)                                       | Deployed into an external Servlet Container / Application Server (Tomcat, WildFly)                   |

---

## 3. Standard Maven Project Structure

Maven enforces **Convention over Configuration**. If you keep files in standard locations, you do not need to configure build paths.

```text
my-app/
 ├── pom.xml                               # Project Configuration
 └── src/
      ├── main/
      │    ├── java/                      # Application source code (.java)
      │    └── resources/                 # Config files (application.properties, logback.xml)
      └── test/
           ├── java/                      # Unit test source code
           └── resources/                 # Test-specific configuration assets

```

Compiled output is placed in an auto-generated directory:

```text
my-app/
 └── target/
      ├── classes/                        # Compiled .class files
      ├── test-classes/                   # Compiled test .class files
      └── my-app-1.0.0.jar                # Packaged artifact

```

---

## 4. Maven Coordinates (GAV)

Every artifact in the Maven universe is uniquely identified by coordinate properties defined in `pom.xml`.

```xml
<groupId>com.telusko</groupId>
<artifactId>telusko-app</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>

```

- **`groupId`**: Identifies the organization, team, or umbrella company. Follows the reverse domain name convention (e.g., `com.tcs`, `org.springframework`).
- **`artifactId`**: The specific project or sub-module name. This forms the base name of the final generated file (e.g., `telusko-app.jar`).
- **`version`**: The current build revision of the artifact.
- `SNAPSHOT`: Active development build (mutable; Maven checks for upstream updates).
- `RELEASE` (or concrete numbers like `1.0.0`): Immutable production build.

- **`packaging`**: The packaging target format (`jar`, `war`, `pom`). Defaults to `jar` if omitted.

---

## 5. Maven Archetypes

An **archetype** is a project template that sets up the standard folder skeleton and a base `pom.xml`.

### Creating a Standalone Java App

```bash
mvn archetype:generate \
  -DgroupId=com.telusko \
  -DartifactId=telusko-app \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DarchetypeVersion=1.4 \
  -DinteractiveMode=false

```

### Creating a Web Application

```bash
mvn archetype:generate \
  -DgroupId=com.telusko \
  -DartifactId=telusko-web-app \
  -DarchetypeArtifactId=maven-archetype-webapp \
  -DarchetypeVersion=1.4 \
  -DinteractiveMode=false

```

---

## 6. Maven Lifecycle, Phases & Goals

A **lifecycle** is an ordered sequence of phases. Executing any phase automatically runs all preceding phases in that lifecycle.

### Core Build Phases (In Order)

```text
validate → compile → test → package → verify → install → deploy

```

- **`validate`**: Checks whether the project structure and `pom.xml` are complete and correct.
- **`compile`**: Converts `.java` files from `src/main/java` into `.class` files inside `target/classes`.
- **`test`**: Compiles `src/test/java` and executes unit tests (skips packaging if tests fail).
- **`package`**: Bundles compiled classes and resources into the configured artifact (`.jar` or `.war`) inside `target/`.
- **`install`**: Copies the generated artifact into your machine's **local repository** (`~/.m2/repository`) for use as a dependency in other local projects.
- **`deploy`**: Uploads the final artifact to an external/enterprise **remote repository**.

### Clean Lifecycle

Runs independently of the default build lifecycle:

```bash
mvn clean

```

Deletes the entire `target/` directory to guarantee an unpolluted compile.

### Common CLI Command Patterns

```bash
mvn compile          # Compiles main source files
mvn test             # Compiles main + test files, then runs tests
mvn package          # Runs validation, compilation, tests, then packages JAR/WAR
mvn clean package    # Wipes target folder, re-runs tests, and creates a fresh package
mvn clean install    # Wipes target, tests, packages, and pushes artifact to local ~/.m2

```

---

## 7. Maven Repositories & Dependency Resolution Flow

### The Three Repository Types

1. **Local Repository**: Located on your development machine at `~/.m2/repository`. Caches downloaded artifacts to eliminate redundant network calls.
2. **Central Repository**: The official public registry (`https://repo.maven.apache.org/maven2/`) operated by Sonatype and the Apache Foundation.
3. **Remote Repository**: A custom, internally hosted server (e.g., JFrog Artifactory, Sonatype Nexus) used within organizations to host private internal libraries.

### Resolution Step-by-Step

When you declare a dependency in `pom.xml` and invoke a phase like `mvn compile`:

```text
Step 1: Maven inspects the Local Repository (~/.m2/repository)
        ├── Found? → Maven links it directly to project classpath.
        └── Not Found? → Proceed to Step 2.

Step 2: Maven queries Configured Remote Repositories / Central Repository
        ├── Found? → Downloads the .jar and its checksum metadata.
        │            Stores it in ~/.m2/repository for future builds.
        │            Links it to project classpath.
        └── Not Found? → Build Failure ("Could not resolve dependencies").

```

```text
[Your Project]
      │
      ▼ (Check)
[Local Repository: ~/.m2/repository]
      ├── Present? ─── Yes ──► (Attach to Build Classpath)
      └── No
           │
           ▼ (Fetch over HTTP/HTTPS)
[Maven Central / Enterprise Remote Repo]
           │
           ▼ (Save to cache)
[Local Repository: ~/.m2/repository] ──► (Attach to Build Classpath)

```

---

## 8. Anatomy of `pom.xml` & Build Plugins

The **Project Object Model (POM)** is the declarative XML specification Maven reads to manage dependencies and build behaviour.

### Declaring Dependencies

Dependencies are pulled from online indexes like [mvnrepository.com](https://mvnrepository.com):

```xml
<dependencies>
    <!-- JUnit 5 for testing -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.10.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>

```

### Build Plugins: Configuring the Compiler

Plugins execute the actual work of Maven lifecycle phases. The most critical build plugin is the **Maven Compiler Plugin**, which defines your Java language level:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
            </configuration>
        </plugin>
    </plugins>
</build>

```

- **`groupId` / `artifactId` / `version**`: Identifies the exact plugin version to execute.
- **`<configuration>`**: Passes settings directly into the plugin's underlying Mojo (Maven plain Old Java Object).
- **`<source>`**: Specifies the Java language specification of your source code (e.g., allows modern switch expressions or records if set to `17`).
- **`<target>`**: Specifies the target JVM bytecode version compatibility produced for execution.

---

# Quick Recall

- **Maven** → Declarative build and dependency automation framework.
- **JAR vs WAR** → JAR = Standalone JVM execution; WAR = Enterprise Servlet container deployment.
- **Convention over Configuration** → Java code lives in `src/main/java`; tests live in `src/test/java`.
- **GAV** → `groupId` (organization), `artifactId` (module name), `version` (revision).
- **`target/`** → Destination folder for all generated bytecode and packages.
- **`mvn clean`** → Deletes the `target/` directory.
- **`mvn compile`** → Compiles source files to `.class`.
- **`mvn test`** → Executes test suites.
- **`mvn package`** → Executes tests and creates `.jar` or `.war`.
- **Resolution Order** → Local (`~/.m2/repository`) → Remote (Enterprise) → Maven Central.

```

---

### Feedback on Raw Notes & Mistakes Clarified

1. **"war and var differences"**:
   - You asked for the difference between `war` and `var`.
   - **`WAR`** is a packaging format (**W**eb Application **Ar**chive) used for web applications.
   - **`var`** is **not** a packaging format. In Java, `var` is a reserved type name introduced in Java 10 for local variable type inference (e.g., `var name = "Shekhar";`). The valid packaging types are `jar`, `war`, `ear`, and `pom`.

2. **File name for build path without a build tool**:
   - In your notes, you asked: *"we have to include in our IDE in some file in build path (Please you mention the file name)"*.
   - In Eclipse, this file is named `.classpath` (stored in the project root alongside `.project`).
   - In IntelliJ IDEA, build paths and module dependencies are managed in `.iml` files and the `.idea/libraries/` folder.

3. **CLI Syntax with Angle Brackets**:
   - In your raw notes, you wrote: `mvn <compile>`, `mvn <test>`, `mvn <package>`.
   - In terminal shells, angle brackets `<>` are shell redirection operators and will cause command failures. The actual terminal commands must be run without brackets: `mvn compile`, `mvn test`, `mvn package`.

4. **Who checks the local repository? (Maven vs JVM)**:
   - You asked: *"whenever we need a dependency, our application or JVM or please explain exactly what it will what is that will look into local repository"*.
   - The **JVM does not check `.m2`**. The JVM knows nothing about Maven repositories; it only understands the classpath.
   - **Maven** checks the `.m2` directory during compilation/packaging, grabs the needed JARs, and constructs the classpath string for the compiler and runtime.

<FollowUp label="Want to cover Maven Dependency Scopes (compile, test, provided, runtime) next?" query="Explain Maven dependency scopes (compile, provided, runtime, test, system) with real-world examples."/>

```
