# engineering-constitutions

`engineering-constitutions` is a simple public repository for Entgra-wide engineering constitutions, coding standards, security requirements, documentation standards, AI-assisted development instructions, and technology-specific guidance.

It is intentionally documentation-first. Projects can copy or reference the modules they need and use `.entgra-constitution.yml` to record which constitution modules apply to them.

## Repository Structure

- `constitutions/universal/`: guidance that applies across Entgra projects.
- `constitutions/languages/`: language-specific guidance.
- `constitutions/frameworks/`: framework-specific guidance.
- `constitutions/platforms/`: platform and delivery guidance.
- `constitutions/middleware/`: middleware modules, including CXF and OSGi guidance.
- `schema/entgra-constitution.schema.json`: JSON Schema for project selection files.
- `templates/entgra-constitution.yml`: reusable starter project configuration.
- `examples/`: sample project configurations and project-owned instruction files.

## How Modules Work

Each constitution module is a Markdown file with YAML front matter:

```markdown
---
id: universal.security
title: Security Requirements
category: universal
version: 1.0.0
status: active
owners:
  - security-team
applies_to:
  - all
depends_on:
  - universal.engineering-principles
---
```

Use stable module IDs. When a module depends on another module, list it in `depends_on` so project teams can see the relationship.

## Project Configuration

Projects can add `.entgra-constitution.yml` at the project root:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/entgra/engineering-constitutions/main/schema/entgra-constitution.schema.json
schema_version: 1

constitution:
  release: "1.0.1"

project:
  name: "sample-java-service"
  description: "Example Java REST service"
  repository: "https://github.com/entgra/sample-java-service"

modules:
  required:
    - universal.engineering-principles
    - universal.code-quality
    - universal.testing
    - universal.documentation
    - universal.security
    - universal.api-design
    - universal.ai-assisted-development
    - language.java
  optional:
    - framework.spring
    - platform.docker
    - platform.github-actions

project_instructions:
  path: "PROJECT_INSTRUCTIONS.md"
```

The config is a lightweight declaration of which shared modules apply to a project. It does not generate files by itself.

## What Is The JSON Schema?

`schema/entgra-constitution.schema.json` is a machine-readable contract for `.entgra-constitution.yml`.

Editors and CI tools can use it to catch mistakes such as:

- Missing required fields.
- Unsupported `schema_version`.
- Invalid module ID shape.
- Duplicate module IDs in one list.
- Absolute or parent-directory paths for `project_instructions.path`.
- Unknown extra fields in the configuration.

The schema validates the shape of a configuration file. It does not validate whether a module ID exists in this repository; that can be checked manually during review.

## Project-Specific Instructions

Shared constitutions describe organization-wide expectations. Project teams should keep local instructions in `PROJECT_INSTRUCTIONS.md`, for example:

- Runtime versions.
- Local build and test commands.
- Compatibility constraints.
- Migration rules.
- Service ownership and operational notes.

Keeping project-specific instructions separate makes shared constitution changes safer and easier to review.

## Adding A Module

1. Add a Markdown file under the right `constitutions/` directory.
2. Use complete YAML front matter.
3. Choose a stable, globally unique module ID.
4. Keep the guidance concise, practical, and verifiable.
5. Avoid duplicating universal rules in technology-specific modules.
6. Update examples when the new module should be demonstrated.

## Updating A Module

Use semantic versioning for constitution releases:

- Patch: wording corrections and non-material clarifications.
- Minor: new optional guidance or backward-compatible rules.
- Major: material changes to mandatory engineering requirements.

Update `CHANGELOG.md` for material changes.

## CXF And OSGi

`constitutions/middleware/cxf.md` defines CXF/JAX-RS WAR guidance for Carbon-based deployments.

`constitutions/middleware/osgi.md` defines OSGi component guidance for Carbon-based service, DAO, transaction, datasource, tenant-context, feature-packaging, and lifecycle behavior.
