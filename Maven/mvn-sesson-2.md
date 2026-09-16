````markdown
## 1. The Effective POM

Maven operates on inheritance. Even if your `pom.xml` has only 20 lines, Maven actually executes against a much larger, fully resolved XML called the **Effective POM**.

### What Makes Up the Effective POM?

```text
Super POM (Maven's built-in root POM defaults)
       +
Parent POM (if using multi-module or Spring Boot Starter Parent)
       +
Your Project's pom.xml
       ↓
==================================================
              Effective POM
 (The actual runtime configuration Maven executes)
==================================================
```
````

- **Super POM**: Maven's default base POM bundled inside Maven core. It specifies default directory structures (`src/main/java`), default plugin versions (compiler, surefire, jar), and default repository bindings.
- **Why it matters**: It explains why Maven can compile and package your code even when you haven't declared a single `<plugin>` in your `pom.xml`.

### How to View Your Effective POM

Run this command in your project root to see every inherited plugin, configuration, and default:

```bash
mvn help:effective-pom

```

---

## 2. The Three Maven Lifecycles

Maven has **three built-in lifecycles**. Each runs completely independently of the others:

| Lifecycle     | Responsibility                                                          | Common Command               | Frequency of Use         |
| ------------- | ----------------------------------------------------------------------- | ---------------------------- | ------------------------ |
| **`default`** | Main lifecycle handling compilation, testing, packaging, and deployment | `mvn package`, `mvn install` | Daily / Constant         |
| **`clean`**   | Removes build artifacts and deletes the `target/` directory             | `mvn clean`                  | Frequent                 |
| **`site`**    | Generates project documentation, reports, and site pages                | `mvn site`                   | Rare / Automated CI only |

---

## 3. Deep Dive: Key Lifecycle Phases in Order

The `default` lifecycle consists of sequential phases. Running any later phase automatically triggers all prior phases:

```text
validate → compile → test → package → verify → install → deploy

```

### 1. `validate`

- **When**: The very first phase of the default lifecycle.
- **What it does**: Ensures the project is correct and all necessary information is available.
- **Checks**:
- `pom.xml` exists in the project root and is well-formed XML.
- Required GAV coordinates (`groupId`, `artifactId`, `version`) are present.
- Project directory conventions are met.

### 2. `verify`

- **When**: Runs _after_ `package` and _before_ `install`.
- **What it does**: Inspects the generated package (`.jar` or `.war`) to ensure it meets quality criteria and passes integration tests.
- **Primary Use Cases**:
- **Integration Tests**: Running tests against the packaged application (e.g., via the `maven-failsafe-plugin`).
- **Code Quality / Coverage**: Running tools like JaCoCo, SonarQube, or Checkstyle to verify minimum code coverage thresholds.

### 3. `install`

- **Command**: `mvn install`
- **What it does**:

1. Executes `validate` → `compile` → `test` → `package` → `verify`.
2. Copies the resulting `.jar` or `.war` into your local machine's cache: `~/.m2/repository/`.

- **Why use it**: Allows other local projects on your machine to depend on this artifact without uploading it anywhere.

### 4. `deploy`

- **Command**: `mvn deploy`
- **What it does**:

1. Executes all preceding phases through `install`.
2. Uploads the finalized artifact and its `pom.xml` to a configured remote repository manager (e.g., Sonatype Nexus, JFrog Artifactory).

- **Why use it**: Enables other team members, environments, and CI/CD pipelines to pull and reuse your built artifacts.

---

## 4. Phase Execution Comparison

| Command       | validate | compile | test | package | verify | install | deploy |
| ------------- | -------- | ------- | ---- | ------- | ------ | ------- | ------ |
| `mvn compile` | ✅       | ✅      | ❌   | ❌      | ❌     | ❌      | ❌     |
| `mvn test`    | ✅       | ✅      | ✅   | ❌      | ❌     | ❌      | ❌     |
| `mvn package` | ✅       | ✅      | ✅   | ✅      | ❌     | ❌      | ❌     |
| `mvn verify`  | ✅       | ✅      | ✅   | ✅      | ✅     | ❌      | ❌     |
| `mvn install` | ✅       | ✅      | ✅   | ✅      | ✅     | ✅      | ❌     |
| `mvn deploy`  | ✅       | ✅      | ✅   | ✅      | ✅     | ✅      | ✅     |

---

# Quick Recall

- **Effective POM** → Super POM + Parent POM + your `pom.xml` combined (`mvn help:effective-pom`).
- **3 Lifecycles** → `default` (build/deploy), `clean` (delete `target/`), `site` (documentation).
- **`validate`** → First phase; checks project structure and `pom.xml` integrity.
- **`verify`** → Runs post-packaging; runs integration tests and quality checks (JaCoCo/Sonar).
- **`install`** → Copies the artifact to `~/.m2/repository` for local reuse.
- **`deploy`** → Publishes the artifact to a remote repository (e.g., Nexus, Artifactory) for team/production reuse.

```

---

### Feedback on Raw Notes & Mistakes Clarified

1. **Clarification on the Effective POM origin**:
   - In your notes, you mentioned: *"It has all the plugins that we don't mention, that is, all the plugins that are required."*
   - To be technically precise for interviews: Maven achieves this through the **Super POM** (which acts like `java.lang.Object` for all POMs). Any plugin you don't explicitly declare comes from the Super POM defaults.

2. **Integration Testing: `test` vs `verify`**:
   - Remember this distinction:
     - The **`test`** phase runs **Unit Tests** on bare class files (handled by `maven-surefire-plugin`).
     - The **`verify`** phase runs **Integration Tests** on the bundled JAR/WAR (handled by `maven-failsafe-plugin`).

3. **Deploy Destination**:
   - You noted: *"mvn deploy is used to deploy our generated build into the Nexus. This is the remote repository."*
   - This is conceptually correct, but keep in mind **Nexus** is just one product (by Sonatype). Other common enterprise remote repositories include **JFrog Artifactory** and **AWS CodeArtifact**. Maven interacts with all of them using the standard `<distributionManagement>` tag inside `pom.xml`.

<FollowUp label="Want to move on to Spring Core and the IoC Container next?" query="Let's start Spring Framework: explain Spring Core, Inversion of Control (IoC), and the Bean Factory vs ApplicationContext."/>

```
