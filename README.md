# Design Intelligence Infrastructure

This project blueprints the foundation of a governed, observable, and agent-ready design system's architecture, NOT to build an entire enterprise design system.

There are 5 main sections:
1. The Knowledge Base, which documents everything internally
2. The Design System, made of 6 categories labelled A-F
3. The Artifact Registry, a machine-readable central index, connecting the 6 categories
4. Agent and Human Orchestration, which defines agentic roles, permissions, and human-in-the-loop steps
5. Core + Domain Extensions, defining the core library vs. the independently governed design system extensions for a specific client's data application, for example

## Obsidian as the Knowledge and Relationship Base
A human-readable knowledge graph containing decisions, relationships, architecture, context, and ownership

## Six Interconnected Domains
All 6 domains share the same artifact identities and change history. The registry connects the (A) design artifacts in Figma and (B) code, contracts, and documentation owned by Git.

A. Design: Figma, Tokens, Assets

B. Code: React, Storybook Tests

C. Documentation: Guides, Examples, API

D. Contracts: Schemas, Rules, Validation

E. Provenance: Ownership, History, Reviews

F. Metrics: Adoption, Usage, Outcomes

## Artifact Registry
IDs, schemas, relationships, versions, owners, validation
Implement a simple registry package containing a JSON manifest for every component.

Example:
```
{
  "$schema": "./schemas/artifact.schema.json",
  "id": "ds.component.button",
  "name": "Button",
  "type": "component",
  "version": "1.0.0",
  "status": "stable",

  "ownership": {
    "team": "design-systems",
    "maintainer": "user-123",
    "approvers": [
      "user-456"
    ]
  },

  "relationships": {
    "implements": [
      "ds.token.color.action.primary",
      "ds.token.spacing.component"
    ],
    "governedBy": [
      "ds.contract.component",
      "ds.contract.accessibility"
    ],
    "documentedBy": [
      "ds.documentation.component.button"
    ]
  },

  "references": {
    "design": {
      "platform": "figma",
      "file": "figma-file-id",
      "node": "figma-node-id"
    },
    "implementation": {
      "package": "@system/components",
      "path": "packages/components/src/button"
    },
    "explorer": {
      "platform": "ladle",
      "story": "Button"
    }
  },

  "provenance": {
    "createdBy": "user-123",
    "createdAt": "2026-11-22"
  },

  "metrics": {
    "usageTracking": "contextual"
  }
}
```

Notes:
- Ownership is separate from provenance.
- Relationships are explicit and reference other stable artifact IDs.
- References can accommodate multiple platforms.
- Explorer replaces the Storybook-specific field with a Ladle-compatible reference.
- Metrics specifies an instrumentation policy rather than simply a Boolean.
- Complete contribution history is NOT stored in manifest, which should belong in Git and change records. Rather, the manifest contains current metadata and reference to the authoritative history.

Registry Architecture:
```
packages/
├── registry/
│   ├── artifacts/
│   │   ├── components/
│   │   │   └── button.json
│   │   ├── tokens/
│   │   ├── patterns/
│   │   └── assets/
│   │
│   ├── schemas/
│   │   ├── artifact.schema.json
│   │   ├── component.schema.json
│   │   ├── token.schema.json
│   │   └── provenance.schema.json
│   │
│   ├── relationships/
│   └── index.ts
```
Make the registry consumable through a small TypeScript API. That API becomes the foundation for the control plane, agent tools, and Obsidian synchronization.
```
registry.get("ds.component.button");
registry.list({ type: "component" });
registry.getRelationships("ds.component.button");
registry.validate("ds.component.button");
```

## Agent and Human Orchestration
Discover -> Propose -> Validate -> Review -> Publish -> Measure

More on this later.

### 4 Agentic Roles
1. Discoverer: finds existing components, tokens, patterns, and documentation. Identifies opportunities for reuse.
2. Interpretor: translates approved design definitions into code, stories, and documentation
3. Validator: checks contracts, token compliance, accessibility, test results, and missing documentation.
4. Transistor: analyzes usage, identifies redundant components, and recommends improvements based on adoption and outcome metrics.

## Core + Domain Extensions
Shared foundations, governed contributions, and product-specific extensions


░░░░▒▒▒▒▓▓▓▓█████

## The 6 Domains

### A. Design
Visual language and design artifacts such as tokens in Figma as a source of truth

#### Index
**00 – Getting Started:** Principles, conventions, changelog

**01 – Foundations:** Color, typography, spacing, grid, elevation

**02 – Tokens & Variables:** Primitives, semantic tokens, component tokens

**03 – Components:** Buttons, inputs, alerts, cards, navigation

**04 – Patterns:** Forms, tables, page layouts, workflows

**05 - Assets:** Icons, illustrations, logos

**06 – Deprecated:** Archived components and migration notes

### Component Scope
Start with 6 components:
1. Button
2. Text Input
3. Checkbox
4. Select
5. Alert
6. Card

#### Token Architecture
1. Primitive tokens: raw values
2. Semantic tokens: meaning and intent
3. Component tokens: component-specific application

### B. Code
Functional implementations and testing using React, TypeScript, and Ladle, version-controlled via Git. Ladle is the development environment but component catalog is generated from the artifact registry to avoid having the explorer and registry independently define which components exist.

Each component should have:
- TypeScript implementation
- Design tokens and dependencies
- Figma reference
- Ladle stories
- Accessibility tests
- Interaction tests
- Visual regression coverage
- Usage documentation
- Registry manifest
- Contract metadata

#### Testing
| Tool         |     Responsibility                                            |
|--------------|---------------------------------------------------------------|
| Ladle                                     |  Isolated rendering and interactive examples |
| Vitest                                    |  Unit and component logic tests  |
| Playwright                                |  Browser-level interaction tests |
| axe-core                                  |  Automated accessibility checks  |
| Optional Chromatic or screenshot tooling  |  Visual regression               |

Component Structure:
```
packages/components/
├── src/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.stories.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.a11y.test.tsx
│   │   └── index.ts
│   │
│   ├── TextInput/
│   ├── Checkbox/
│   ├── Select/
│   ├── Alert/
│   └── Card/
│
└── package.json
```

### C. Documentation
Human and machine-readable guidance as close to the source as possible in Markdown/Storybook via Git

#### Surfaces and their Responsibilities:
1. Obsidian: Knowledge graph, relationships, decisions, navigation
2. Git Markdown: Canonical technical documentation
3. Ladle: Interactive implementation documentation

##### Obsidian's Vault, Bases, and Graph View
Use **Bases** for structured component, ownership, and contract views in Markdown files and properties and **Graph View** for relationships. Use 2 kinds of notes to prevent competing sources of truth between Obsidian and Git:
1. Canonical notes: Authored and maintained by humans, such as architectural decisions, governance policies, and system principles.
2. Generated notes: Created from registry manifests, such as component metadata, ownership, and current versions.

Each vault contains:
- Architecture notes
- Component relationship notes
- Design decision records
- Governance policies
- Contribution workflows
- Ownership and team info
- Metrics definitions
- Agent instructions and operational context

```
design-system/
├── knowledge/
│   ├── .obsidian/
│   ├── 00-System/
│   ├── 01-Design/
│   ├── 02-Code/
│   ├── 03-Documentation/
│   ├── 04-Contracts/
│   ├── 05-Provenance/
│   ├── 06-Metrics/
│   └── 07-Agents/
```

Example Obsidian component note
```
---
id: ds.component.button
type: component
status: stable
version: 1.0.0
owner: design-systems
maintainer: user-123
figma: https://figma.com/...
source: packages/components/src/Button
ladle: /?story=button
---

# Button

## Purpose
Triggers a primary or secondary action.

## Usage
- Use for explicit user actions.
- Prefer one primary action per context.

## Avoid
- Using buttons for navigation.
- Using buttons as decorative elements.

## Relationships

- Design: [[Button - Figma]]
- Tokens: [[Color Interactive]]
- Contract: [[Button Contract]]
- Implementation: [[Button - React]]
- Ownership: [[Design Systems Team]]

## Accessibility
Keyboard activation, visible focus,
appropriate accessible name.
```

Each component page should answer:
- what is it?
- when should it be used?
- when shouldn't it be used?
- what variants and states exist?
- what are its accessibility requirements?
- which tokens does it consume?
- where is its Figma definition?
- where is its code implementation?
- who owns it?
- what version is it?
- what changes have occurred?

The agent-ready addition would be an AGENTS.md file at the repository root, explaining the architecture, package structure, naming conventions, component creation process, validation commands, and rules agents must follow. Smaller contextual instruction would be provided in relevant packages.

### D. Contracts
Rules governing consistency and behavior in JSON/YAML schemas, tests, and validation via Git

Five contract types:
1. Identity (JSON Schema)
2. Design-to-code (Token and reference checks)
3. Quality's minimum requirements, standards (Tests, axe, lint)
4. Changes Processes in proposals, approvals, and versioning
5. Relationship integrity for agent-readiness, as in:
   a. Every stable component must have a valid design reference
   b. Every stable component must have an implementation.
   c. Every implementation must have a registry entry.
   d. Every component must reference valid tokens.
   e. Every deprecated component must have a migration reference.
   f. Every artifact must have an owner.
   g. Every relationship must resolve to an existing artifact ID.

### E. Provenance
Ownership, authorship, reviews, and change history with Git + metadata

Roles:
- Creator: original creator
- Contributor: who changed it
- Maintainer: who currently maintains it
- Approver: who authorized its release
- Consumer: who uses it

Demo this with 2 simulated contributors and one AI agent showing ownership transfer and reviewed change.

### F. Metrics
Usage, adoption, and outcomes in an analytics database

#### Level I: System Health
- Number of components.
- Number of documented components.
- Percentage of components with Figma references.
- Percentage using design tokens.
- Accessibility pass rate.
- Visual regression pass rate.
- Deprecated component count.
- Average time to resolve component issues.

#### Level II: Adoption and Usage
- Component usage across applications.
- Component usage across customers.
- Most frequently used variants.
- Most frequently used patterns.
- Adoption of new components.
- Percentage of product UI using system components.
- Number of custom implementations replacing standard components.

#### Level III: Product and Business Outcomes
Implement with PostHog to support analytics, funnels, retention, experiments, feature flags, and dashboards. However, avoid instrumenting every low-level component interaction to mitigate excessive event volume and noisy data. Establish 3 events categories:
1. `design_system.component_used` – meaningful adoption events.
2. `design_system.pattern_completed` – meaningful workflow events.
3. `product.outcome_completed` – business outcome events.

Example:
```
{
  "event": "design_system.component_used",
  "properties": {
    "artifact_id": "ds.component.button",
    "version": "1.0.0",
    "variant": "primary",
    "application_id": "customer-portal",
    "context": "account-creation"
  }
}
```

Product applications -> Analytics event pipeline -> Analytics storage -> Adoption + Outcomes

- product applications: React components emit structured events
- analytics event pipeline: component ID, variant, version, context, event
- analytics storage: events, applications, customers, experiments
- adoption: usage and coverage
- outcomes: conversion and retention

Examples:
| Component or pattern  |   Potential business metric        |
|-----------------------|------------------------------------|
| Form inputs           |   Form completion rate             |
| Buttons               |   Primary action conversion        |
| Navigation            |   Task completion                  |
| Search                |   Search success rate              |
| Data tables           |   Task efficiency                  |
| Onboarding patterns   |   Activation rate                  |
| Error messages        |   Recovery rate                    |
| Checkout components   |   Checkout conversion              |
--------------------------------------------------------------

## Agentic Interface for Orchestration

Make the registry API and CLI the first agent interfaces, then expose them through MCP when appropriate. Suggested bash commands:

```
ds list
ds inspect ds.component.button
ds search "form input"
ds validate ds.component.button
ds validate --all
ds relationships ds.component.button
ds provenance ds.component.button
ds metrics ds.component.button
```


### Tool vs. Action

list_artifacts: Discover system artifacts

get_artifact: Retrieve artifact metadata

get_relationships: Explore dependencies

get_tokens: Retrieve token definitions

validate_artifact: Run contracts

get_provenance: Inspect authorship and ownership

get_metrics: Retrieve adoption and outcome data

propose_change: Create a proposed change

## Repo Structure
```
design-intelligence/
│
├── apps/
│   ├── ladle/
│   └── portal/
│
├── packages/
│   ├── tokens/
│   │   ├── source/
│   │   └── generated/
│   │
│   ├── components/
│   │   └── src/
│   │       ├── Button/
│   │       ├── TextInput/
│   │       ├── Checkbox/
│   │       ├── Select/
│   │       ├── Alert/
│   │       └── Card/
│   │
│   ├── registry/
│   │   ├── artifacts/
│   │   ├── schemas/
│   │   ├── relationships/
│   │   └── index.ts
│   │
│   ├── contracts/
│   │   ├── identity/
│   │   ├── design-code/
│   │   ├── accessibility/
│   │   └── lifecycle/
│   │
│   ├── analytics/
│   │   ├── events/
│   │   ├── adapters/
│   │   └── metrics/
│   │
│   └── tooling/
│       └── cli/
│
├── docs/
│   ├── architecture/
│   ├── components/
│   ├── foundations/
│   ├── governance/
│   └── contribution/
│
├── knowledge/
│   ├── .obsidian/
│   ├── 00-System/
│   ├── 01-Design/
│   ├── 02-Code/
│   ├── 03-Documentation/
│   ├── 04-Contracts/
│   ├── 05-Provenance/
│   ├── 06-Metrics/
│   └── 07-Agents/
│
├── scripts/
│   ├── validate-registry.ts
│   ├── validate-tokens.ts
│   └── generate-knowledge.ts
│
├── .github/
│   ├── workflows/
│   ├── CODEOWNERS
│   └── PULL_REQUEST_TEMPLATE.md
│
├── AGENTS.md
├── package.json
└── README.md
```

## Use Case

Theoretically, this should work but would require a tyrant, engaged hive, and/or small group to sustainably maintain.

To summarize, Design Intelligence Infrastructure is a governed, interconnected system that transforms design-system artifacts into machine-readable, versioned, observable resources that humans and AI agents can discover, contribute to, validate, and improve.

A demonstration or example of this working in action would be:
- A designer changes a token in Figma. Then, an agent discovers the change and proposes a corresponding implementation. After a solution is reviewed and approved, Ladle renders the updated component. With the rendering available for comparison, the Validator(s) validates design-code compliance by way of the contracts and (a) Maintainer(s) reviews the change. Finally, Provenance records the contribution, relationships are updated across documentation such as the knowledge base, and metrics track usage, adoption, and such thanks to the Transistor. 

This means we'd have one coherent system with a traceable lifecycle, assuming the team overseeing this can sustain, steer, and comply with such a colossal effort... or a living, breathing nightmare. Successful groupings of and results from machine-human and human-to-human collaboration, e.g. catching false negatives, would be crucial.
