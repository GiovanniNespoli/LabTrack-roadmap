# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A study project: learning software architecture by building **LabTrack**, an internal system for reserving and lending lab assets (Orion Labs, ~350 assets, up to 80 requests/day). The repo follows the roadmap in `labtrack-roteiro-de-arquitetura-aplicada.pdf`, which is the source of truth for chapters, deadlines and required evidence.

There is no application code yet — chapter 01 is documentation only. Build/lint/test commands will come with chapter 02 (Docker Compose, REST API, CI pipeline); update this file when they exist.

Reading the PDF: `pdftoppm` is not installed, so the Read tool cannot render it. Extract text with `python3 -c "import pypdf; [print(p.extract_text()) for p in pypdf.PdfReader('labtrack-roteiro-de-arquitetura-aplicada.pdf').pages]"`.

## Roadmap structure

- One folder per chapter: `step01/`, `step02/`, … Each chapter has a deadline and a list of **"Evidências que você entrega"** (deliverables) in the PDF — check that list before calling a chapter done.
- Every chapter follows the same cycle: read → draw (C4/UML/flow) → write an ADR → implement the smallest vertical slice → test and demo → defend the decision in five minutes.
- Roadmap rule: every technology must solve an observable problem in the product or live in an isolated lab with simulated data (e.g. SOAP, Hadoop, microfrontends). Don't bend the LabTrack domain to fit a technology.
- If a chapter doesn't fit its deadline, cut feature width, not evidence (ADR, test, demo).

## Git workflow (one branch per chapter)

- Each roadmap chapter has its own branch named after its folder: `step01`, `step02`, … `step13`. All work for a chapter is committed on that branch, inside its `stepNN/` folder.
- When the chapter's evidence is complete, merge into `main` with `git merge --no-ff stepNN` so every chapter appears as its own merge bubble in the history.
- **Never delete a step branch after merging** (neither locally nor on the remote); they are kept as a permanent record of each chapter.
- Repository-wide files (`README.md`, `CLAUDE.md`, the roadmap PDF) may be committed directly on `main`.
- Start a new step branch from the up-to-date `main`, after the previous step was merged.

## Chapter 01 documents (`step01/`)

Documents were deliberately consolidated to few files; add to an existing one before creating a new one.

- `visao.md` — product vision + stakeholder matrix + pains/goals/metrics; ends with links to every document in the step.
- `requisitos.md` — RF, RNF and business rules (RB-01..RB-05) with states and Given/When/Then acceptance criteria.
- `riscos.md` — probability × impact risk matrix (R-01..R-05).
- `backlog.md` — user stories HU-01..HU-16 with priority and RF/RB/RNF references.
- `jornada.excalidraw` — swimlane user journey (Solicitante, Gestor, Técnico, Sistema).
- `adr/ADR-NNN-<slug>.md` — one file per ADR: Contexto, Opções, Decisão, Consequências, Como reverter.

## Documentation conventions

- Write in Brazilian Portuguese with accents, lowercase kebab-case filenames, lines wrapped at ~80 chars.
- Use stable IDs and cross-reference them across documents: `RF-NN`, `RNF-NN`, `RB-NN`, `CA-NN.N` (acceptance criterion), `R-NN` (risk), `HU-NN` (story), `ADR-NNN`. When renaming or merging files, update relative links in the other documents.
- Domain vocabulary: Solicitante (pesquisador/instrutor), Gestor do laboratório, Técnico de manutenção, Administrador; journey is consultar → solicitar → aprovar → retirar → devolver → manter.
- Entity states (keep in English, as in the roadmap):
  - Ativo: `AVAILABLE, RESERVED, LOANED, IN_MAINTENANCE, RETIRED`
  - Empréstimo: `REQUESTED, APPROVED, READY_FOR_PICKUP, CHECKED_OUT, RETURNED, OVERDUE, REJECTED, CANCELLED`

## Architecture decisions so far

From ADR-001 (`step01/adr/ADR-001-monolito-modular.md`):

- **Modular monolith backend** (Node.js + TypeScript + PostgreSQL) and a **separate Next.js frontend**, in one monorepo (planned: `apps/web`, `apps/api`, `packages/contracts`).
- Backend modules: `catalog`, `loans`, `maintenance`, `audit`, `identity`, each with `domain / application / infra / http` layers. Modules talk only through their `index.ts` public API — never import another module's `domain`/`infra` or touch its tables.
- Business rules and state transitions live in the backend domain, never in the UI.
- RB-01 (one active loan per asset) and RB-04 (immutable audit trail) are the highest-risk rules: enforce them with an atomic transaction plus a database constraint, and cover them with concurrency/audit tests.
- Don't design microservices yet; later chapters (07, 10) revisit extraction via queues/events.
