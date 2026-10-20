# The Public Record — Technical Architecture

**Status:** Proposed design, not a deployed implementation.  
**Canonical product requirements:** [THE_PUBLIC_RECORD_SPEC.md](THE_PUBLIC_RECORD_SPEC.md).

## 1. Architecture goals
Build a **source-first** civic research system, not a generic chatbot. Preserve provenance from original official/public documents to machine-extracted statements and any synthesized answer. Support statewide context and the initial Santa Clara, San Mateo, and San Francisco county pilot, including officials between elections.

No user-visible political factual claim should be returned without a traceable source. A URL alone is insufficient: store a stable document identity, publisher, publication/observation time, retrieved time, and a passage locator wherever available. Distinguish fact, quotation, allegation, analysis, and missing evidence.

## 2. Logical architecture

```text
Official government sites / public source feeds / approved reporting
                           |
                           v
        Scheduled ingestion workers (Python)
        discover -> fetch -> validate -> extract -> normalize
                           |
            +--------------+--------------+
            v                             v
   Immutable source files           Structured staging
   + hashes + fetch metadata        + deduplication queue
            |                             |
            +--------------+--------------+
                           v
                Entity resolution / review
        people, offices, elections, districts, organizations
                           |
                           v
               PostgreSQL + pgvector
        records, source locators, evidence, history, index
                           |
                +----------+----------+
                |                     |
                v                     v
         Read-only research API   Admin review/corrections
                |
        +-------+------------------+
        v                          v
  Next.js public website     Future read-only MCP
        |
        v
 Question -> context + entity check -> structured search
          -> document retrieval -> evidence assembly
          -> conditional LLM synthesis -> claim/citation check
          -> answer with citations, dates, caveats or abstention
```

## 3. Suggested components and responsibility
- **Web:** Next.js App Router, TypeScript, responsive and accessible design. Candidate/official profiles, ballot discovery, search, comparisons, evidence inspector, research questions, and public corrections mechanism.
- **API:** Validated read-only public endpoints for entities, records, questions, comparisons, source citations, and coverage. Separate privileged administrative endpoints behind authentication.
- **Database:** PostgreSQL; pgvector only where semantic retrieval materially helps. Structured joins are authoritative for identity, jurisdiction, dates, votes, and monetary values.
- **Workers:** Python ingestion adapters with checkpoints, backoff, idempotency, source terms review, and reconciliation reports.
- **Evidence storage:** Original immutable content/snapshots, hashes, metadata, extraction versions, and document/passage locators.
- **LLM orchestration:** A bounded retrieval and synthesis service that can abstain, labels statements by source type, and never generates a fabricated citation.
- **MCP adapter:** Later read-only tools that call the *same* audited research API as the web application; no independent truth store.

## 4. Data model and relationships
Refer to specification §7 for proposed schema. At minimum model:
- person, aliases, candidate participation, office, office tenure, jurisdiction, electoral district and boundary validity, election, contest, ballot measure;
- organization and campaign committee with stable external identifiers;
- source publisher, source document, retrieval snapshot/hash, publication date, locator, passage/excerpt;
- claims, promises, votes, legislative/administrative actions, finance filings/transactions, with explicit evidence links;
- ingestion run, extraction/reconciliation state, correction request and correction history.

**Identity safety:** Same-name matches require distinguishing attributes and ambiguity must be surfaced. Never infer an individual's position from party, associates, or proximity to a document.

**Temporal safety:** Store effective dates and retrieval dates separately. Do not overwrite old claims with new positions; represent changes over time.

## 5. Ingestion pipeline
1. Maintain a versioned **source register** (authority, jurisdiction, URL/feed, permitted use, refresh schedule, freshness and parser status).
2. Fetch data and retain source URL, retrieval timestamp, content checksum, response metadata, and unmodified snapshot where permitted.
3. Validate format and source; reject or quarantine failed runs rather than silently publishing partial outputs.
4. Extract text/tables while preserving page/section anchors. Keep extraction provenance and uncertainty.
5. Normalize public entities and date/monetary fields; queue ambiguous identities for human review.
6. Link evidence to original source passages; expose **coverage and last-updated status**.
7. Publish reviewed records to the public query layer; record corrections and reprocessing history.

Start with a small verified dataset from California election sources and **one county adapter**; add San Mateo and San Francisco after the ingestion interface is repeatable.

## 6. Retrieval and answer contract
Inputs: question, optional selected person/contest/jurisdiction, requested time period. Resolve ambiguous locations and names explicitly; a ZIP code is an entry point, not a guaranteed exact ballot.

Preferred processing sequence:
1. Validate scope and resolve subject.
2. Query structured records when answering exact facts.
3. Retrieve primary-source excerpts with locators; use secondary reporting only with clearly labeled attribution.
4. Assemble a set of evidence items with durable IDs.
5. Generate a bounded neutral response only from retrieved evidence.
6. Check that each substantive factual assertion is backed by a cited item and that the locator is meaningful.
7. Respond with citations, source dates, uncertainty, and known gaps; **abstain** or narrow the answer if support is insufficient.

Separate *what someone said* from *what they did*, and *what a donor disclosed* from any interpretation about influence. Do not produce candidate scores, rankings, endorsements, inferred intent, or electoral forecasts.

## 7. Example read-only API boundaries
These are design targets, not yet implemented:
- `GET /api/coverage`
- `GET /api/search?q=...`
- `GET /api/people/:id`
- `GET /api/contests/:id`
- `GET /api/sources/:id`
- `GET /api/evidence/:id`
- `POST /api/research/questions` (rate-limited, with evidence/citation contract)
- `POST /api/compare`

JSON responses for assertions should identify structured `evidence_ids`, stable `source_ids`, locator data, publication and retrieval timestamps, and an explicit `insufficient_evidence` outcome where appropriate. Exact wire shapes must be defined and tested before implementation.

## 8. Security, privacy, and operations
- No user accounts required for basic public access; collect minimal visitor data. Never use political research queries for voter persuasion profiles.
- Apply endpoint quotas/rate limits and cost controls; do not expose LLM keys or ingestion credentials to the browser.
- Sanitize source text and treat downloaded documents as untrusted inputs, including embedded prompt-injection instructions.
- Admin corrections and source approvals are authenticated, audited, and separate from public routes.
- Monitor freshness, failed fetches, unsupported assertions, citation link health, API cost, latency, and accessibility.
- Provide correction reporting, methodology and coverage pages, and source retractions/update handling.

## 9. Deployment boundaries
Develop locally first; use environment-specific credentials and migrations. Avoid paid services and production deployment without product-owner approval. CI should run lint, typecheck, tests, security checks, and build validation. No invented political fixtures in production.

## 10. Implementation order and open decisions
1. Foundation and contract tests.
2. Relational schema and evidence provenance.
3. Ingestion adapter for a small verified pilot.
4. Evidence viewer and research navigation.
5. Grounded AI Q&A with verification and evaluation.
6. Candidate/official comparison and finance.
7. Additional county connectors; hardened ops.
8. MCP distribution using the same query APIs.

**Open decisions:** final source licensing and ingestion frequency, hosting/cost constraints, scope of initial contests, exact county source mappings, administrative review workflow, question retention, evaluation release thresholds, and deployment authorization. Record decisions before irreversible design choices.

**Implementation tracker:** Linear project “The Public Record”; milestones and tickets to be created separately.
