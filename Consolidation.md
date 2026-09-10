---
name: consolidate-design-docs
description: Merges the already-generated HLD and LLD documents from the frontend repo and backend repo into a single unified HLD and LLD, written for an Enterprise Architect audience. Does not re-analyze source code.
---

# Consolidate Design Documents

You are merging documentation that has **already been generated** — you are not
analyzing source code and you are not regenerating anything from scratch.

## Inputs (read-only)

Assume both repositories are checked out side by side in the current workspace:

- `frontend-repo/docs/HLD.md`
- `frontend-repo/docs/LLD.md`
- `backend-repo/docs/HLD.md`
- `backend-repo/docs/LLD.md`

If any of these files are missing, stop and report which one is missing rather
than guessing its contents or falling back to reading source code.

## Outputs

- `docs/HLD_consolidated.md`
- `docs/LLD_consolidated.md`

(Adjust these output paths if this skill is placed in a repo with an existing
`docs/` convention — keep whatever naming distinguishes the consolidated doc
from the individual per-repo docs.)

## Hard Rules

1. **Do not re-analyze code.** Your only inputs are the four markdown files
   listed above. If something seems missing or unclear, note it as a gap
   rather than inferring it from anything else.
2. **Do not simply concatenate.** A unified document should read as one
   coherent system description, not "frontend doc" + "backend doc" stapled
   together.
3. **Preserve accuracy.** Do not alter or reinterpret factual claims from the
   source documents — only reorganize, merge, and de-duplicate them.
4. **Update, don't rewrite**, on subsequent runs — if a consolidated doc
   already exists, only touch the sections whose source material changed.
5. **When merging Mermaid diagrams, re-validate syntax — do not just splice
   text together.** Apply the full Mermaid Syntax Rules from the
   generate-design-docs skill (node label quoting, edge label quoting, no
   reserved words like `end` as IDs, valid arrow syntax, one statement per
   line, balanced sequence-diagram activations, etc.) to the merged result —
   a diagram that was valid in each source doc can still become invalid after
   merging. In particular:
   - If the frontend and backend diagrams each define a node with the same
     ID but different meaning, rename one before merging — colliding IDs
     silently overwrite each other in Mermaid.
   - If both source diagrams use the same participant alias in sequence
     diagrams for different actors, rename one before merging.
   - After merging, mentally re-parse the combined diagram line by line
     against every rule in the generate-design-docs skill's Mermaid Syntax
     Rules section — do not assume validity just because both inputs were
     individually valid.

## Merge Logic

### HLD merge (`docs/HLD_consolidated.md`)
- **Executive Overview / Objective**: write one combined overview describing
  the system as a whole (frontend + backend together), not two separate ones.
- **Architecture Description**: combine both Mermaid diagrams into a single
  `flowchart TB` showing the frontend and backend as connected layers/systems,
  with the connection between them clearly labeled (protocol, e.g. HTTPS/REST,
  gRPC, WebSocket). Do not just place two diagrams side by side — merge them.
- **Core Workflows / Data Flow**: merge into unified end-to-end workflows that
  span frontend → backend where applicable (e.g. "user submits form" → "API
  receives request" → "data persisted").
- **Data Protection**: combine into one section covering the full data
  lifecycle across both systems — where data enters (frontend), how it
  travels, and where it's ultimately stored (backend).
- **Security Requirements**: merge into one section; if frontend and backend
  have different auth mechanisms, describe both and how they relate (e.g.
  frontend obtains a token that the backend validates).
- **Integrations**: list all integrations from both docs in one table; note
  which side (frontend/backend) owns each integration.
- **Environment Variables & Secrets Inventory**: combine into one inventory,
  labeled by which repo/service each variable belongs to.
- **Change Log**: append a new entry noting this is a consolidation run and
  which source docs it pulled from (include their last-updated dates if
  present).
- Sections unique to only one side (e.g. a frontend-only "Key Features"
  section with no backend equivalent) should be kept, clearly scoped.

### LLD merge (`docs/LLD_consolidated.md`)
- **Module/Component Breakdown**: organize under two top-level groups,
  Frontend and Backend, but add a short "Cross-Cutting" note wherever a
  frontend module directly depends on a backend module or endpoint.
- **Sequence Diagrams**: prefer merged, end-to-end sequence diagrams for
  workflows that cross both systems, over keeping separate frontend-only and
  backend-only diagrams for the same workflow.
- **Data Models / Schemas**: combine into one section; flag where a
  frontend-side type and a backend-side schema represent the same entity.
- Everything else (Error Handling, Configuration, Known Limitations, Change
  Log): merge straightforwardly, keeping frontend/backend attribution where it
  aids clarity.

## Process

1. Read all four source files in full.
2. Read the existing consolidated docs, if present, to determine what's
   already reflected.
3. Identify what's new or changed in either source doc since the last
   consolidation.
4. Produce or update the two consolidated documents per the merge logic above.
5. Write the files to disk in this repo's `docs/` folder (create it if
   needed).
6. If there are no actual changes compared to what's already committed, stop
   here — do not create a branch or PR.
7. If there are changes, open a pull request in this repo:
   - Create a new branch named `docs/consolidated-update-<YYYYMMDD-HHMM>` off
     the default branch.
   - Commit with a message like
     `docs: automated consolidated HLD/LLD update <date>`.
   - Push and open a PR against the default branch, titled
     `docs(hld): automated consolidated update <date>`, labeled `automerge`,
     with a body noting which source docs (and their last-updated dates) were
     used.
   - Never push directly to the default branch — always go through a PR.
