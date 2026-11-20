# Spec: Agents Context Modularization v0.0.1

This spec normalizes how a repository organizes its context artifacts: the markdown instructions and other files, traditionally not considered
code, that together with code and tests give AI agents the context they need to work on it: domain knowledge, project perimeter and
principles, team conventions and self-service platform tools.
It does so in a way that is as agent-agnostic and as reproducible as possible, and consistent with the capabilities of the agents consuming it.

## Dependencies

- https://github.com/Uqido/spec-specification-repository/tree/v1.0.1

## Adoption Impact

This specification defines a common structure for the context artifacts of a repository, in order to:
- Give agents a single, vendor-agnostic entrypoint to the repository context, so that, given a clear goal, they work autonomously: neither
  stuck on missing context nor taking decisions the context would have ruled out
- Promote a single, minimal and complete responsibility per repository: a repository whose context cannot be stated as one such responsibility
  does not qualify for adoption, so overlapping or fragmented responsibilities surface as soon as the context is written
- Promote a single notion of context in which context artifacts are part of the codebase: they follow the repository's division of
  responsibilities and are modularized with the same principles as code, while code and tests, as living documentation, speak the same
  language as the context artifacts
- Enable automated consumption of the context of other adopting repositories, by both humans and agents, through shared conventions and
  support tools, with no repository-specific integration
- Apply information hiding across repositories: a repository makes sense on its own, as if no other repository and its context existed, while
  still consulting the context of related repositories to better understand the outcome its own responsibility serves
- Load only the context a task needs, keeping token cost and context window usage proportional to the outcome/domain rather than to the repository
- Detect broken references or unindexed artifacts automatically, before an agent acts on an incomplete context
- Allow both humans and AI systems to know exactly which context artifacts an agent could rely on at any given commit
- Enable the definition of the project artifacts agents need, as typical concerns with suggested names and techniques, so that the same
  concern is found and read the same way across repositories; how effective they are depends on their content, which is the adopting
  project's responsibility. For example, artifacts that:
  - Let agents and new team members use domain terms with the single meaning the team gives them, since ambiguity is cognitive load paid at
    every read and change of the code
  - Let agents verify their own hypotheses and changes with the project's tools—build, test, lint and any other declared one—and report the
    evidence produced, without asking how to run them
  - Let agents autonomously follow the team's workflow, from issue to pull request, up to the Definition of Done, so that review feedback
    can focus on the outcome of a change rather than on process
  - Ground feedback and decisions in the project's perimeter and principles: feedback on an agent's proposal (e.g. AID) cites the persona or
    principle at stake, and open alternatives become an explicit choice when it is due, rather than a cost paid later for a decision nobody took

## Normative Content

Rules are identified by a code (e.g. `M4`) so that reviews, human or AI, can cite them.

### Definitions

- **Context**: whatever an agent or a human relies on to understand and change the repository: code, tests and context artifacts alike.
  Code and tests are living documentation: they speak the same domain language as the context artifacts of their module, and follow the
  same modularization principles.
- **Context artifact**: a file of the codebase not traditionally considered code, such as markdown instructions, glossaries, templates or
  schemas. Context artifacts are code: versioned, reviewed and modularized together with the rest of the codebase.
- **Indexed context artifact**: a context artifact marked as such by the frontmatter of M1, whose `name` matches its path, or, when its
  format cannot hold a frontmatter, by being listed with `name` and `description` in an index. The rules of this spec on frontmatter,
  indexes and references apply to indexed context artifacts only; non-indexed ones (e.g. `README.md`, a changelog, existing documentation)
  are context as code is, and are referred to the same way.
- **Context modularization**: the application of the codebase's modularization principles (separation of concerns, minimal responsibility,
  open-closed, information hiding) to context artifacts.
- **Entrypoint**: the `AGENTS.md` indexed context artifact at the repository root.
- **Index**: an indexed context artifact listing the indexed context artifacts of one directory level; the entrypoint is the root index, `core.md` files are
  all the others.
- **Module**: a unit of the codebase with a single responsibility; its context artifacts follow the same boundaries as its code.
- **Concern**: a kind of knowledge the context covers (see [Context Concerns](#context-concerns)), implemented by one or more modules.
  A concern applies only where it is meaningful for the repository's responsibility: e.g. a project overview or architecture principles
  apply to a repository representing a system (such as the one holding the product's infrastructure as code), not to the repository of one
  of its services.
- **Technique**: a concrete notation used to implement a concern. This spec suggests a default technique for some concerns; an adopter may
  replace it, as long as the concern is still covered.

### Repository Responsibility

- **R1**: a repository adopting this spec **will** have a clear, minimal and complete responsibility; this spec does not verify it, but its
  rules rely on it. For example, and not exhaustively, changes recurrently requiring coordinated changes in another repository, or modules
  whose purpose cannot be traced back to the repository's responsibility, are signs that it does not hold.

### Entrypoint

- **E1**: `AGENTS.md` at the repository root **shall** be the single entrypoint and the root index of the indexed context artifacts.
- **E2**: every indexed context artifact **shall** be reachable from the entrypoint through a chain of `[[name]]` references (see M4).
- **E3**: vendor-specific instruction files (e.g. `CLAUDE.md`, `.github/copilot-instructions.md`, `.cursor/rules/`) **shall** not hold
  context; when an agent does not read the entrypoint natively, its vendor-specific file **shall** contain only a reference to the
  entrypoint (e.g. `@AGENTS.md`).
- **E4**: indexed context artifacts **shall** be composable through `[[name]]` and indexes alone. Vendor-specific mechanisms (e.g. import directives,
  path-scoped rules, vendor frontmatter fields) **should** only be added on top, so that removing them leaves the context unchanged.

### Language

- **L1**: context artifacts **shall** be written in English.
- **L2**: domain terms may be localized when that is the language the team, users or stakeholders actually use (e.g. a user persona whose
  domain name is in the local language); each localized term **shall** have an entry in the domain language concern (C2).

### Modularization

- **M1**: every markdown indexed context artifact **shall** open with a YAML frontmatter holding at least:
  ```yaml
  ---
  name: path/relative/to/repository/root   # no extension; the target of [[name]] references
  description: one line stating what the artifact is for, enough to pick it out of a set without reading it
  ---
  ```
  An indexed context artifact whose format cannot hold this frontmatter **shall** have its `name` and `description` stated in the index
  listing it. A frontmatter holding `name` and `description` whose `name` does not match its path is not the one of this rule: support tools
  **should** report it as a likely mistake.
- **M2**: an indexed context artifact **should** declare `depends_on: [name, ...]` only when it cannot be understood without the listed artifacts.
- **M3**: additional frontmatter fields (e.g. a `kind` taxonomy) are allowed; each one **shall** be defined in the index of the directory level
  where it is first used, and applies to that level and below.
- **M4**: indexed context artifacts **shall** refer to each other only through `[[name]]`, where `name` is the target's `name`; never
  through file paths, header anchors or markdown links. Every `[[name]]` **shall** resolve to an existing indexed context artifact.
  Indexed context artifacts refer to non-indexed ones and to code through repository-relative paths, and to other repositories as stated in
  [External References](#external-references).
- **M5**: every directory containing indexed context artifacts, other than the root, **shall** contain a `core.md` index listing each
  indexed context artifact of that level and of its subdirectories, each with its description.
- **M6**: whenever indexed context artifacts are meant to be loaded together, the index where the composition happens **shall** state the exact order
  of concatenation (e.g. "general artifact first, then the specialization").
- **M7**: an artifact specializing a more general one **will** state only its delta (scope, omissions, emphasis) and reference the general
  artifact for everything else; what it does not mention holds unchanged.
- **M8**: context modularization **will** follow the same responsibilities as the code, according to the open-closed and the minimal
  responsibility principles. A boundary that does not produce desirable friction, in the reverse Conway sense, is a failed split: the
  modules **will** be merged back. Where context artifacts live, alongside the code they concern or apart from it, is a design choice left
  to the adopter.

### External References

- **X1**: indexed context artifacts **shall** refer to other repositories only through `[[url]]` or `[[url#name]]`, where `url` points to a version
  of the repository, a tag or a commit, never a branch:
  - `[[url]]` is a *repository reference*: it refers to the repository as a whole, i.e. to its responsibility, and resolves to its
    entrypoint, or to its `README.md` when the repository does not adopt this spec;
  - `[[url#name]]` is an *artifact reference*: it refers to the indexed context artifact with that `name` in that repository, at that version;
    it therefore applies only to repositories adopting this spec.

  For example, `[[https://github.com/Uqido/acme-system/tree/v2.1.0]]` refers to the system a service belongs to, while
  `[[https://github.com/Uqido/acme-system/tree/v2.1.0#conventions/workflow]]` refers to the workflow that system defines for all its
  services. The dependencies of a spec on other specs are repository references too, which additionally imply the adoption of the
  referenced specs.
- **X2**: every external reference **shall** resolve at the version it states.
- **X3**: repository references **should** be preferred to artifact references: an artifact reference couples the referring artifact to the
  internal structure of the referenced repository.
- **X4**: `depends_on` **shall** list only artifacts of the same repository: since its responsibility is complete (R1), a repository is
  understood without resolving its external references.

### Authoring

- **W1**: a context artifact **will** state only what is true now; the history of how its text evolved belongs to version control, not to the
  artifact.
- **W2**: context artifacts **will** not restate what an agent can cheaply derive from code and tests (e.g. directory listings, function
  signatures, behavior already specified by a test).
- **W3**: a technique **should** cite its source and state what it does not cover.

### Context Concerns

The concerns below are typical, not exhaustive: an adopter may cover other concerns, following the same rules. Each concern applies only where
it is meaningful for the repository's responsibility (see [Definitions](#definitions)). Where it applies:
- the module covering it **should** cover what the concern states;
- the module **should** be named as suggested; when several modules cover the concern, the suggested name **should** be the root directory
  holding them, with its index.

Suggested techniques are informative: an adopter may replace them (see [Definitions](#definitions)).

- **C1 Project overview**: the module **should** state the project's purpose, main modules, stack, external dependencies and constraints.
  - *Suggested name*: `project_overview`
  - *Applies to*: a repository representing the project as a system of services, hence typically the one holding its infrastructure as
    code.
- **C2 Domain language**: the module **should** define the terms that cannot be derived from the code: the language found in user stories,
  in stakeholder conversations and among the team, each with a single meaning within the repository.
  - *Suggested name*: `domain_language`
  - *Suggested technique*: a glossary of term/definition entries, i.e. the ubiquitous language of Domain-Driven Design (Eric Evans,
    *Domain-Driven Design*, 2003).
  - *Applies to*: any repository whose code embodies domain concepts, a service as well as a system; rarely to purely technical ones (e.g.
    generic libraries or tooling).
- **C3 Conventions and workflow**: the module **should** state how work flows from an issue to a merged change: branch naming, commit
  conventions, pull request process and Definition of Done, plus any project-wide coding convention not enforced by tooling.
  - *Suggested name*: `conventions`
  - *Applies to*: any repository where changes are delivered through a team workflow.
- **C4 Platform tools**: the module **should** list the self-service tools an agent can use to validate hypotheses, produce evidence or
  create artifacts, including build, test and lint whenever the project has them.
  - *Suggested name*: `platform_tools`
  - *Suggested technique*: one record per tool, stating purpose (the question it answers or the artifact it produces), invocation (the
    exact command to run it), preconditions (what must hold before running it: environment, credentials, services) and evidence (what
    output counts as a result and how to read it).
  - *Applies to*: any repository whose code can be built, tested, run or inspected by an agent.
- **C5 Perimeter**: the module **should** state the scope of the project in terms of who it serves and what change it is meant to produce.
  - *Suggested name*: `perimeter`
  - *Suggested techniques*:
    - personas, each with `domain_name`, `triggering_context`, `current_behavior`, `goal` and `value_chain_position`; `domain_name` is a
      domain language term (C2), not a narrative name;
    - impacts and outcomes in the Output/Outcome/Impact vocabulary (Josh Seiden, *Outcomes Over Outputs*, 2019);
    - acceptance criteria as Given/When/Then scenarios (Behaviour-Driven Development, Gherkin notation).
  - *Applies to*: a repository representing the project as a system of services; to a service repository only when it serves personas of
    its own.
- **C6 Principles**: the module **should** list the adopted principles that bound every subsequent technical decision: a principle narrows
  the set of admissible alternatives, it never picks among them, and it is revised only on evidence that contradicts it.
  - *Suggested name*: `principles`
  - *Suggested technique*: TOGAF's Principles Catalog: each principle has Name, Statement, Rationale and Implications, and is
    Understandable, Robust, Complete, Consistent and Stable.
  - *Applies to*: a repository representing the project as a system of services, where principles bound decisions across services; to a
    service repository only for principles specific to it.
