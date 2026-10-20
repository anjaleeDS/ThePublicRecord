# The Public Record

**Don't take their word for it. Follow the evidence.**

The Public Record is a proposed free, nonpartisan, citation-first civic research platform for California residents. It will help people research **candidates and serving elected officials**, their documented public statements and actions, election measures, and relevant public records.

## Initial geography
- Santa Clara County
- San Mateo County
- San Francisco City and County

The November 2026 election is an initial dataset and validation opportunity, **not an end date** for the service. The longer-term product covers accountability between elections.

## Status
**Planning / pre-implementation.** This repository contains design documentation, not a working production application. Do not claim candidate coverage, validated sources, a live AI backend, or published services until they exist.

## Source-of-truth documents
- [Full product and technical specification](THE_PUBLIC_RECORD_SPEC.md)
- [Technical architecture](ARCHITECTURE.md)
- [Codex working instructions](AGENTS.md)

If documents disagree, raise the conflict for product-owner review instead of silently overriding requirements.

## Proposed implementation
Next.js / TypeScript for the site and API; PostgreSQL with pgvector for structured records and retrieval; Python ingestion workers; document/object storage for immutable evidence snapshots; later a read-only MCP interface for AI clients.

See `ARCHITECTURE.md` for trust boundaries and `THE_PUBLIC_RECORD_SPEC.md` for details and acceptance criteria.

## Development process
Track work in the Linear project **The Public Record**. Each implementation task should identify specification sections, acceptance criteria, tests, and dependencies. Implement one milestone at a time, keeping PRs small and reviewed.

**First milestone: PR-001, foundation only** — scaffold, development tooling, test/CI configuration, documentation, and safety guardrails. Do not add invented political data or ship AI answers in the foundation milestone.

## Public-interest safeguards
No political endorsements, candidate rankings, user profiling for political persuasion, uncited political factual answers, or invented source material. Clearly communicate geographic, temporal, and evidence coverage gaps. Source claims must remain attributable to the documents supporting them.

## Running locally
To be documented once PR-001 creates the application scaffold.
