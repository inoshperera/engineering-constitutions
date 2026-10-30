# Constitution Catalogue

Universal rules remain flat. Languages and frameworks use one folder per technology, with shared rules at `<technology>/<technology>.md` and version-specific locations at `<technology>/<version>/<technology>.md`.

Directory placement does not change a released module's ID or metadata. Select existing modules using their current IDs in `.entgra-constitution.yml`; the configuration schema and example selections are unchanged.

## Universal

| Module ID | Constitution |
|---|---|
| `universal.engineering-principles` | [Engineering principles](universal/engineering-principles.md) |
| `universal.ai-assisted-development` | [AI-assisted development](universal/ai-assisted-development.md) |
| `universal.security` | [Security](universal/security.md) |
| `universal.code-quality` | [Code quality](universal/code-quality.md) |
| `universal.testing` | [Testing](universal/testing.md) |
| `universal.documentation` | [Documentation](universal/documentation.md) |
| `universal.api-design` | [API design](universal/api-design.md) |
| `universal.database-changes` | [Database changes](universal/database-changes.md) |
| `universal.dependency-management` | [Dependency management](universal/dependency-management.md) |
| `universal.observability` | [Observability](universal/observability.md) |

## Languages

| Module ID | Shared constitution | Version navigation |
|---|---|---|
| `language.java` | [Java](languages/java/java.md) | [Java versions](languages/java/README.md) |
| `language.kotlin` | [Kotlin](languages/kotlin/kotlin.md) | [Kotlin versions](languages/kotlin/README.md) |
| `language.javascript` | [JavaScript](languages/javascript/javascript.md) | [JavaScript](languages/javascript/README.md) |
| `language.typescript` | [TypeScript](languages/typescript/typescript.md) | [TypeScript versions](languages/typescript/README.md) |
| `language.go` | [Go](languages/go/go.md) | [Go](languages/go/README.md) |
| `language.shell` | [Shell](languages/shell/shell.md) | [Shell](languages/shell/README.md) |

## Frameworks

| Module ID | Shared constitution | Version navigation |
|---|---|---|
| `framework.spring` | [Spring](frameworks/spring/spring.md) | [Spring versions](frameworks/spring/README.md) |
| `framework.react` | [React](frameworks/react/react.md) | [React versions](frameworks/react/README.md) |
| `framework.react-native` | [React Native](frameworks/react-native/react-native.md) | [React Native versions](frameworks/react-native/README.md) |


## Platforms

| Module ID | Constitution |
|---|---|
| `framework.android` | [Android](platforms/android/android.md) |
| `middleware.cxf` | [Carbon CXF webapps](platforms/carbon/cxf-webapps/cxf-webapps.md) |
| `middleware.osgi` | [Carbon OSGi components](platforms/carbon/osgi-components/osgi-components.md) |

## Middleware

[WSO2 API Manager](middleware/wso2-api-manager/wso2-api-manager.md), [WSO2 Identity Server](middleware/wso2-identity-server/wso2-identity-server.md), and [Apache Synapse](middleware/apache-synapse/apache-synapse.md) have reserved placeholder files. None is a selectable module.

## Delivery

| Module ID | Constitution |
|---|---|
| `platform.docker` | [Docker](delivery/docker/docker.md) |
| `platform.kubernetes` | [Kubernetes](delivery/kubernetes/kubernetes.md) |
| `platform.github-actions` | [GitHub Actions](delivery/github-actions/github-actions.md) |

## Version Placeholders

The Java 17/21, Kotlin 2, TypeScript 5, Spring 5/6, React 18/19, and React Native 0.82 directories reserve locations for reviewed version-specific differences. They do not add approved rules, establish supported versions, or define selectable module IDs. Existing projects continue to use the shared modules until version-specific guidance is approved.

A technology version in a directory name is separate from the constitution revision recorded in a module's front matter. Universal and shared guidance remain the baseline; approved version-specific documents should contain only relevant differences and explicitly identify any approved overrides.

## Relocated Files

All existing files retain their original contents. Existing README files, changelog, configuration, and ownership rules are unchanged. Older references embedded in those files still use the previous paths; this table records their current destinations.

| Previous path | Current file |
|---|---|
| `constitutions/languages/java.md` | [constitutions/languages/java/java.md](languages/java/java.md) |
| `constitutions/languages/kotlin.md` | [constitutions/languages/kotlin/kotlin.md](languages/kotlin/kotlin.md) |
| `constitutions/languages/javascript.md` | [constitutions/languages/javascript/javascript.md](languages/javascript/javascript.md) |
| `constitutions/languages/typescript.md` | [constitutions/languages/typescript/typescript.md](languages/typescript/typescript.md) |
| `constitutions/languages/go.md` | [constitutions/languages/go/go.md](languages/go/go.md) |
| `constitutions/languages/shell.md` | [constitutions/languages/shell/shell.md](languages/shell/shell.md) |
| `constitutions/frameworks/spring.md` | [constitutions/frameworks/spring/spring.md](frameworks/spring/spring.md) |
| `constitutions/frameworks/react.md` | [constitutions/frameworks/react/react.md](frameworks/react/react.md) |
| `constitutions/frameworks/android.md` | [constitutions/platforms/android/android.md](platforms/android/android.md) |
| `constitutions/middleware/cxf.md` | [constitutions/platforms/carbon/cxf-webapps/cxf-webapps.md](platforms/carbon/cxf-webapps/cxf-webapps.md) |
| `constitutions/middleware/osgi.md` | [constitutions/platforms/carbon/osgi-components/osgi-components.md](platforms/carbon/osgi-components/osgi-components.md) |
| `constitutions/platforms/docker.md` | [constitutions/delivery/docker/docker.md](delivery/docker/docker.md) |
| `constitutions/platforms/kubernetes.md` | [constitutions/delivery/kubernetes/kubernetes.md](delivery/kubernetes/kubernetes.md) |
| `constitutions/platforms/github-actions.md` | [constitutions/delivery/github-actions/github-actions.md](delivery/github-actions/github-actions.md) |
| `constitutions/frameworks/react-native.md` | [constitutions/frameworks/react-native/react-native.md](frameworks/react-native/react-native.md) |
