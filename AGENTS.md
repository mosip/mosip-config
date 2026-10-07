# AGENTS.md

## Repository Overview

**mosip-config** is the central configuration repository for the MOSIP platform. It contains the property files, JSON schemas, business-rule scripts, and routing definitions consumed by MOSIP's various microservices. There is no application code here — this repo works together with a **Spring Cloud Config Server** so that MOSIP services fetch their configuration dynamically at runtime, without rebuilding or redeploying.

Each service points its Spring Cloud Config client at this repo with a specific `spring.cloud.config.name` (matching a file in this repo) and `spring.cloud.config.label` (a git branch/tag in this repo). Different deployments/environments can therefore point at different branches of this same repository. See the [MOSIP Config Server Setup Guide](https://docs.mosip.io/1.2.0/modules/registration-processor/registration-processor-developers-guide#environment-setup) for how the config server itself is set up and started.

## Technology Stack

- No build toolchain — this is a flat collection of configuration artifacts, not a compiled/packaged project.
- **Spring Boot `.properties` files** (`*-default.properties`) — one per MOSIP microservice, named to match that service's `spring.cloud.config.name` and served for the `default` Spring profile.
- **JSON** — schemas (`*-schema.json`), mapping files (`*-mapping.json`), and structured config (`mosip-context.json`, `CredentialType.json`, etc.)
- **JSON-LD** (`*.jsonld`) — verifiable credential contexts (`cred-v1.jsonld`, `vccontext*.jsonld`, `odrl.jsonld`)
- **MVEL scripts** (`*.mvel`) — business rules evaluated by kernel/registration-processor services (`applicanttype.mvel`, `credentialdata.mvel`, `identity-data-formatter.mvel`)
- **Apache Camel routes** (`registration-processor-camel-routes-*-default.xml`) — routing definitions for registration-processor packet flows (new, update, lost, deactivate, reactivate, CRVS variants, etc.)
- **XSD** (`mosip-cbeff.xsd`) — CBEFF biometric data schema
- **TOML** (`websub-service.toml`, `websub-consolidator.toml`) — websub service configuration

## Build & Test Commands

This repo contains configuration artifacts — `.properties`, `.json`, `.jsonld`, `.xml`, `.xsd`, and `.toml` files — consumed by MOSIP services via the Spring Cloud Config Server. Note that `.mvel` files (business-rule scripts) and the Camel routes (`.xml`) are **executable at runtime**, not passive data, so changes to them carry the same risk as a code change even though there is no build, package, or automated test step in this repo. A change takes effect the next time a consuming service refreshes its config from the Spring Cloud Config Server pointed at this repo's branch — for `.mvel` and Camel route changes, manually verify the logic (e.g. against the consuming service locally) before merging, since nothing here will catch a broken rule or route automatically.

## Configuration

- File naming convention: `<spring.cloud.config.name>-default.properties` — e.g. `id-authentication-default.properties` backs the `id-authentication` service's `default` profile. `digitalcard-template.properties` is the one exception without a `-default` suffix.
- **Secrets are never committed as literal values, in any served artifact** — not just `*-default.properties`, but also `digitalcard-template.properties`, the `.json`/`.jsonld` files, the Camel-route `.xml` files, and the `.toml` files. The vast majority of `*-default.properties` files (e.g. `admin-default.properties`, `application-default.properties`, `kernel-default.properties`, `id-authentication-default.properties`, `id-repository-default.properties`, `esignet-default.properties`, `idp-default.properties`, `mock-abis-default.properties`, `partner-management-default.properties`, `registration-processor-default.properties`, `pre-registration-default.properties`, `print-default.properties`, and others) carry an explicit header comment stating that certain properties (client secrets, credentials, etc.) **must be supplied via environment variables at the config-server/helm level** and must NOT be hardcoded here. When adding a new property or field that holds a secret — in a `.properties`, `.json`/`.jsonld`, `.xml`, or `.toml` file alike — follow the same pattern: reference it as `${...}` (or that format's equivalent placeholder syntax) and document in a comment/field description that the real value is injected via an environment variable at the config-server/helm level, never as a literal in this repo.
- `logs/` is gitignored — it's a local scratch directory, not part of the served configuration.

## Project Structure Notes

This repo is intentionally flat — there are no subdirectories of source code, only a single directory of config artifacts at the root (plus `logs/`, which is gitignored). Files group by the service or concern they configure:

- `kernel-default.properties`, `application-default.properties` — cross-cutting/shared MOSIP kernel config
- `id-authentication-*-default.properties`, `id-repository-default.properties` — ID Authentication (IDA) service family
- `registration-processor-default.properties` + `registration-processor-camel-routes-*-default.xml` + `registration-processor-*.json` — Registration Processor and its packet-processing Camel routes
- `idp-default.properties`, `idp-binding-default.properties`, `idp-claims-mapping.json`, `esignet-default.properties`, `signup-default.properties` — identity provider / eSignet / signup services
- `resident-default.properties` + `resident-ui-*-schema.json` — Resident services and Resident UI form schemas
- `partner-management-default.properties`, `pms-migration-utility-default.properties` + `auth-policy-schema.json`, `data-share-policy-schema.json`, `misp-policy-schema.json` — partner-management-services config and policy schema definitions
- `compliance-toolkit-default.properties` — mosip-compliance-toolkit
- `mock-abis-default.properties`, `mock-identity-system-default.properties`, `mock-mv-default.properties` — mock/test-double services used in dev and CI environments
- `websub-consolidator.toml`, `websub-service.toml` — websub pub/sub service config

## Development Workflow

- This repo has **no CI workflows** (`.github/workflows` does not exist) — nothing here is built, tested, or linted automatically on push.
- Because a config-server deployment pins to a specific branch (`label`), a change only affects services whose config client is configured with that branch as its label — check which environments/services actually consume the branch you're changing before assuming a fix is "live" everywhere.

## Pull Request Guidelines

- Never introduce a literal secret, password, or client credential value into any served artifact — `*-default.properties`, `digitalcard-template.properties`, `.json`/`.jsonld`, `.xml` (including Camel routes), or `.toml` alike — follow the existing `${...}` (or format-equivalent) + environment-variable-override pattern documented above.
- If a property is renamed or removed, check whether any other file in this repo cross-references it (e.g. shared kernel properties referenced from multiple service files) before merging.
- Sign off commits (`git commit -s`) per MOSIP contribution conventions.

## Repository-Specific Considerations

- This is mostly a **config repository, not a code repository** — there is no `pom.xml`, `package.json`, or build pipeline. For plain config (`.properties`, `.json`, `.xsd`, `.toml`), "correctness" means valid syntax (parses cleanly) and values consistent with what the consuming service expects, not passing tests. The `.mvel` scripts and Camel-route `.xml` files are the exception: they are executable logic, so review them for correctness the way you would review code, and manually validate the change against the consuming service before merging — there is no CI check that will catch a broken rule or route.
- Because many services (`kernel`, `id-authentication`, `registration-processor`, etc.) have very large property files (tens of thousands of lines), search for the specific property key you intend to change rather than reading the whole file, and double-check you're editing the correct service's file — several services share similarly named properties.
