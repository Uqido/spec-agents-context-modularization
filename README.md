# Spec: Agents Context Modularization v0.0.1

This spec normalizes how a repository organizes its context artifacts: the markdown instructions and other files, traditionally not considered
code, that together with code and tests give AI agents the context they need to work on it: domain knowledge, project perimeter and
principles, team conventions and self-service platform tools.
It does so in a way that is as agent-agnostic and as reproducible as possible, and consistent with the capabilities of the agents consuming it.

## Dependencies

- https://github.com/Uqido/spec-specification-repository/tree/v1.0.1

## Adoption Impact

This specification defines a common structure for the context artifacts of a repository, in order to:
- Give agents a single, vendor-agnostic entry point to the repository context, so that, given a clear goal, they work autonomously: neither
  stuck on missing context nor taking decisions the context would have ruled out
- Promote a single, minimal and complete responsibility per repository: a repository whose context cannot be stated as one such responsibility
  does not qualify for adoption, so overlapping or fragmented responsibilities surface as soon as the context is written
- Promote a single notion of context in which instructions are part of the codebase: context artifacts follow the repository's division of
  responsibilities and are modularized with the same principles as code, while code and tests, as living documentation, speak the same
  language as the instructions
- Enable automated consumption of the context of other adopting repositories, by both humans and agents, through shared conventions and
  support tools, with no repository-specific integration
- Apply information hiding across repositories: a repository makes sense on its own, as if no other repository and its context existed, while
  still consulting the context of related repositories to better understand the outcome its own responsibility serves
- Load only the context a task needs, keeping token cost and context window usage proportional to the outcome/domain rather than to the repository
- Detect broken references, unindexed artifacts automatically, before an agent acts on a stale or incomplete context
- Allow both humans and AI systems to know exactly which context an agent could rely on at any given commit
- Enable the definition of the project artifacts agents need, in a known place and shape; how effective they are depends on their content,
  which is the adopting project's responsibility. For example, artifacts that:
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
  Code and tests are living documentation: they speak the same domain language and follow the same modularization principles as context
  artifacts.
- **Context artifact**: a file of the codebase not traditionally considered code, such as markdown instructions, glossaries, templates or
  schemas. Context artifacts are code: versioned, reviewed and modularized together with the rest of the codebase.
- **Context modularization**: the application of the codebase's modularization principles (separation of concerns, minimal responsibility,
  open-closed, information hiding) to context artifacts.
- **Entrypoint**: the `AGENTS.md` context artifact at the repository root.
- **Index**: a context artifact listing the context artifacts of one directory level; the entrypoint is the root index, `core.md` files are
  all the others.
- **Module**: a unit of the codebase with a single responsibility; its context artifacts follow the same boundaries as its code.
- **Concern**: a kind of knowledge the context covers (see [Context Concerns](#context-concerns)), implemented by one or more modules.
  A concern applies only where it is meaningful for the repository's responsibility: e.g. a project overview or architecture principles
  apply to a repository representing a system (such as the one holding the product's infrastructure as code), not to the repository of one
  of its services.
- **Technique**: a concrete notation used to implement a concern. This spec suggests a default technique for some concerns; an adopter may
  replace it, as long as the concern's rule is still satisfied.

### Repository Responsibility

- **R1**: a repository adopting this spec **will** have a clear, minimal and complete responsibility; this spec does not verify it, but its
  rules rely on it. For example, and not exhaustively, changes recurrently requiring coordinated changes in another repository, or modules
  whose purpose cannot be traced back to the repository's responsibility, are signs that it does not hold.

### Entrypoint

- **E1**: `AGENTS.md` at the repository root **shall** be the single entrypoint and the root index of the context artifacts.
- **E2**: every context artifact **shall** be reachable from the entrypoint through a chain of `[[name]]` references (see M4).
- **E3**: vendor-specific instruction files (e.g. `CLAUDE.md`, `.github/copilot-instructions.md`, `.cursor/rules/`) **shall** not hold
  context; when an agent does not read the entrypoint natively, its vendor-specific file **shall** contain only a reference to the
  entrypoint (e.g. `@AGENTS.md`).
- **E4**: context artifacts **shall** be composable through `[[name]]` and indexes alone. Vendor-specific mechanisms (e.g. import directives,
  path-scoped rules, vendor frontmatter fields) **should** only be added on top, so that removing them leaves the context unchanged.

### Language

- **L1**: context artifacts **shall** be written in English.
- **L2**: domain terms may be localized when that is the language the team, users or stakeholders actually use (e.g. a user persona whose
  domain name is in the local language); each localized term **shall** have an entry in the domain language concern (C2).

### Modularization

- **M1**: every markdown context artifact **shall** open with a YAML frontmatter holding at least:
  ```yaml
  ---
  name: path/relative/to/repository/root   # no extension; the target of [[name]] references
  description: one line stating what the artifact is for, enough to pick it out of a set without reading it
  ---
  ```
  A context artifact whose format cannot hold this frontmatter **shall** have its `name` and `description` stated in the index listing it.
- **M2**: a context artifact **should** declare `depends_on: [name, ...]` only when it cannot be understood without the listed artifacts.
- **M3**: additional frontmatter fields (e.g. a `kind` taxonomy) are allowed; each one **shall** be defined in the index of the directory level
  where it is first used, and applies to that level and below.
- **M4**: context artifacts **shall** refer to each other only through `[[name]]`, where `name` is the target's `name`; never through file
  paths, header anchors or markdown links. Every `[[name]]` **shall** resolve to an existing context artifact. Context artifacts refer to
  code through repository-relative paths.
- **M5**: every directory containing context artifacts, other than the root, **shall** contain a `core.md` index listing each context artifact
  of that level and of its subdirectories, each with its description.
- **M6**: whenever context artifacts are meant to be loaded together, the index where the composition happens **shall** state the exact order
  of concatenation (e.g. "general artifact first, then the specialization").
- **M7**: an artifact specializing a more general one **will** state only its delta (scope, omissions, emphasis) and reference the general
  artifact for everything else; what it does not mention holds unchanged.
- **M8**: context modularization **will** follow the same responsibilities as the code, according to the open-closed and the minimal
  responsibility principles. A boundary that does not produce desirable friction, in the reverse Conway sense, is a failed split: the
  modules **will** be merged back. Where context artifacts live, alongside the code they concern or apart from it, is a design choice left
  to the adopter.

### Authoring

- **W1**: a context artifact **will** state only what is true now; the history of how its text evolved belongs to version control, not to the
  artifact.
- **W2**: context artifacts **will** not restate what an agent can cheaply derive from code and tests (e.g. directory listings, function
  signatures, behavior already specified by a test).
- **W3**: a technique **should** cite its source and state what it does not cover.

### Context Concerns

Each concern below applies only where it is meaningful for the repository's responsibility (see [Definitions](#definitions)); where it
applies, its rule holds as stated.

- **C1 Project overview** **shall** state the project's purpose, main modules, stack, external dependencies and constraints.

  *Applies to*: a repository representing the project as a system of services, hence typically the one holding its infrastructure as code.
- **C2 Domain language** **shall** define the terms that cannot be derived from the code: the language found in user stories, in stakeholder
  conversations and among the team. Each term has a single meaning within the repository; the default technique is a glossary of
  term/definition entries, i.e. the ubiquitous language of Domain-Driven Design (Eric Evans, *Domain-Driven Design*, 2003).

  *Applies to*: any repository whose code embodies domain concepts, a service as well as a system; rarely to purely technical ones (e.g.
  generic libraries or tooling).
- **C3 Conventions and workflow** **shall** state how work flows from an issue to a merged change: branch naming, commit conventions,
  pull request process and Definition of Done, plus any project-wide coding convention not enforced by tooling.

  *Applies to*: any repository where changes are delivered through a team workflow.
- **C4 Platform tools** **shall** list the self-service tools an agent can use to validate hypotheses, produce evidence or create artifacts,
  including build, test and lint whenever the project has them. Each tool **shall** state:
  - purpose: the question it answers or the artifact it produces;
  - invocation: the exact command or entry point;
  - preconditions: what must hold before running it (environment, credentials, services);
  - evidence: what output counts as a result and how to read it.

  *Applies to*: any repository whose code can be built, tested, run or inspected by an agent.
- **C5 Perimeter** **should** state the scope of the project in terms of who it serves and what change it is meant to produce. Default
  techniques:
  - personas, each with `domain_name`, `triggering_context`, `current_behavior`, `goal` and `value_chain_position`; `domain_name` is a
    domain language term (C2), not a narrative name;
  - impacts and outcomes in the Output/Outcome/Impact vocabulary (Josh Seiden, *Outcomes Over Outputs*, 2019);
  - acceptance criteria as Given/When/Then scenarios (Behaviour-Driven Development, Gherkin notation).

  *Applies to*: a repository representing the project as a system of services; to a service repository only when it serves personas of its
  own.
- **C6 Principles** **should** list the adopted principles that bound every subsequent technical decision: a principle narrows the set of
  admissible alternatives, it never picks among them. A principle is revised only on evidence that contradicts it. The default technique is
  TOGAF's Principles Catalog: each principle has Name, Statement, Rationale and Implications, and is Understandable, Robust, Complete,
  Consistent and Stable.

  *Applies to*: a repository representing the project as a system of services, where principles bound decisions across services; to a
  service repository only for principles specific to it.
- **C7 Bounded contexts** **should** extend C2 where the same term carries different meanings in different parts of the system, naming each
  bounded context and stating, as a context map, how terms translate across their boundaries.

  *Applies to*: a repository spanning several bounded contexts, typically the one representing the system of services, whose context map
  relates the services' languages.
