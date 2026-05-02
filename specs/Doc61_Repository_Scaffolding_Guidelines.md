# Document 61 – Repository Scaffolding Guidelines

## Purpose

This document defines the directory structure and scaffolding guidelines for the AionFM platform repositories (`aionfm-ui`, `aionfm-api`, `aionfm-sdk`).  Consistent scaffolding simplifies onboarding, encourages best practices and ensures that new repositories follow a standard pattern.

## Audience

DevOps and maintainers responsible for creating and maintaining the platform repositories.

## General Structure

For each repository, adopt the following top‑level layout:

```
repo-name/
├── README.md
├── LICENSE
├── .gitignore
├── docs/
├── src/ (or server/, client/, depending on context)
├── tests/
├── scripts/
├── CI configuration files (e.g. .github/workflows)
└── examples/
```

### README.md

* Provide an overview of the repository’s purpose, key features, and how it fits into the AionFM ecosystem.
* Include instructions for building, running and testing the project.
* Explain how to contribute and where to find further documentation.

### LICENSE

* Use an open‑source license consistent across all repositories (e.g. MIT or Apache‑2.0).

### docs/

* Include markdown documents and diagrams explaining architecture, API usage and design decisions specific to the repository.
* Reference relevant documents from the AionFM specification.

### src/ or server/client

* For Rust projects (API, SDK), place code under `src/`.  Organize modules logically (see Doc53 and Doc54).
* For UI projects, use directories like `client/` for front‑end code and `public/` for static assets.  Use frameworks (e.g. React) with established conventions (e.g. `src/components`, `src/pages`).

### tests/

* Include unit and integration tests as described in Doc55.  Mirror the directory structure of `src/` for clarity.
* Use fixture data under `tests/fixtures`.

### scripts/

* Provide helper scripts for setup, linting, testing, building and deployment.  Use shell scripts or cross‑platform tools like `just` or `make`.

### CI configuration

* Place CI configuration (e.g. GitHub Actions workflows) in `.github/workflows/`.  Define jobs for linting, testing and building (Doc55).

### examples/

* Include sample applications or notebooks demonstrating typical usage.  For the SDK, provide example code to perform forecasts.  For the UI, include example dashboards.

## Bootstrapping

* Use scaffolding tools or templates to create consistent boilerplate across repositories.  Provide a script (e.g. `scripts/create_repo.sh`) that copies the standard structure and fills in names.
* Encourage contributions via pull requests.  Include a `CONTRIBUTING.md` explaining coding standards, branching strategy and review process.

## Version Control Practices

* Follow semantic versioning in tags and release notes.  Use `CHANGELOG.md` to record changes.
* Use conventional commits for commit messages (e.g. `feat: add new endpoint`, `fix: correct error handling`).
* Protect main branches and require code review and passing CI before merging.

## Maintaining Consistency

* Use shared code style and linting rules across repositories (e.g. Rustfmt for Rust, Prettier for JavaScript/TypeScript).
* Centralize shared configurations (e.g. code style, commit hooks) in a common location if multiple repositories exist within a monorepo.
