# The Public Record — Product Requirements and Technical Specification

**Version:** 1.0  
**Date:** 2026-10-19  
**Status:** Implementation baseline (proposed design, not an existing production system)  
**Audience:** Product owner, Codex coding agent, developers, researchers, and reviewers

## 0. Operating instructions for Codex

Read this entire specification before making structural changes. Implement one milestone at a time. For each milestone: propose a small implementation plan, write tests, implement, run lint/typecheck/tests, summarize changes and limitations, and stop at the milestone boundary. Do not populate production views with invented candidate names, positions, votes, finances, or citations. Build demo fixtures in a visibly labeled demo environment only. Never add uncited factual political claims to production answers. Do not deploy, acquire paid services, or make externally visible publications without explicit authorization.

**Definition of ready:** Confirmed scope and official sources, data rights/access strategy, schema and verification test fixtures.

**Definition of done:** Functionality, acceptance tests, accessibility checks, evidence provenance, and relevant docs committed; no known source-less assertions in user-facing political content.

## 1. Product vision

The Public Record is a free, neutral, source-first civic research application. A visitor can identify relevant California political jurisdictions, search candidates and current officials, explore claims/promises/actions/campaign finance/ballot measures, ask natural-language questions, and inspect the original public records behind the answer. The product is useful both during election season and between elections.

**Positioning:** “Don't take their word for it. Follow the evidence.”

**Principles:** Evidence over assertion; neutrality over persuasion; transparency over false confidence; accessibility; minimize data collection; equal standards for all people; explicit freshness and coverage boundaries; human correction process.

**Non-goals:** Endorsements, ratings, candidate scorecards, persuasion targeting, turnout targeting, predicting electoral outcomes, inferring candidate motives, suggesting how to vote, presenting correlation as causation, accepting anonymous unsourced allegations as facts, claiming complete coverage when partial.

## 2. Target users and jobs to be done

- **Everyday voter:** “What is on my ballot and what can I verify about each candidate?”
- **Curious resident:** “What did my local representatives do about housing or transit?”
- **Journalist/researcher:** “Find the meeting, vote, disclosure, or direct quote and its date.”
- **Community member:** “Compare what officials said with subsequent documented actions.”
- **Accessibility and multilingual users:** “Explain original source materials in accessible language, without distorting meaning.”

No account required for basic search or question answering.

## 3. Scope, geography, and release strategy

**Launch region:** Santa Clara County, San Mateo County, City and County of San Francisco. These are coverage goals, not a claim that every municipality, district, board or race is present on day one.

**Temporal domains:** 2026 general election candidate/measure data, past governmental actions where available, and current officeholders. Preserve all historical snapshots and election cycles, never silently overwrite past records.

**Delivery slices:**
1. **M0 — Foundation:** polished responsive frontend, navigable product, tested DB schema, demo fixtures separated from real data, deployment pipeline and observability.
2. **M1 — Verified directory:** load official state candidate list and one county election source, normalize offices/contests/candidates, sourced profiles, source freshness and coverage indicators.
3. **M2 — Evidence explorer:** ingest selected official meeting agendas/minutes/vote records and candidate statements; full-text search, citations, evidence viewer, manual curation tools.
4. **M3 — Grounded Q&A:** retrieval, claim-to-citation validation, refusal-to-infer behavior, uncertainty handling, research sessions, evaluation suite.
5. **M4 — Compare and measures:** side-by-side sourced comparisons; proposition explanations anchored in official analyses.
6. **M5 — Three-county expansion:** county-specific adapters, campaign finance, promises-to-actions exploration, consistency audits, public corrections workflow.
7. **M6 — Distribution:** indexable public profile pages, read-only public API, remote MCP server, ChatGPT plugin testing and submission.

**Important date:** General election November 3, 2026. A production-quality three-county service is not assumed ready by that date. Launch a narrow verified subset if ready; mark coverage accordingly.

## 4. Information architecture and routes

- `/` — homepage, search box, three-county coverage, product explainer, latest data status.
- `/explore` — browse jurisdictions, offices, officials, election cycles, topics.
- `/ballot` — ZIP entry, district confirmation, county election office links, relevant contests only when validated; NEVER claim ZIP uniquely identifies an exact ballot.
- `/people` — search/filter people.
- `/people/[slug]` — candidate/official profile, sources, role history, direct statements, relevant public actions, finance when available, update timestamp, coverage scope.
- `/contests/[id]` — roster from verified election source, equal presentation, links to profiles, compare.
- `/compare?people=...` — two or more candidates/officials, columns by topic, shared evidence formatting; no evaluation/score/winner.
- `/measures/[id]` — ballot text, what Yes/No does, official analysis, fiscal impacts and opposing attributed arguments.
- `/ask` — research conversation, sources panel, follow-up questions, visible evidence gaps.
- `/sources/[id]` — source metadata, original URL, document version, retrieved timestamp, excerpts, claim links.
- `/coverage` — jurisdiction/office/source coverage matrix and last-checked times.
- `/methodology` — sourcing policy, classification, limits, corrections process.
- `/corrections` — report issues and track public revisions.
- `/about`, `/privacy`, `/accessibility`.
- `/admin/*` — gated import queue, entity review, correction queue, citation QA, coverage dashboard.

## 5. Primary user journeys

**J1 Search with location:** User enters ZIP; app responds with possible jurisdictions where boundaries overlap; requires district confirmation or redirects to official personalized ballot lookup for exact ballot. No exact ballot claim unless district assignment is verified by suitable address/district data.

**J2 Research official:** Search name; resolve ambiguous people/roles; profile shows present/past offices with date ranges and sourced actions.

**J3 Ask a question:** User asks, “What did [official] do on housing?” Backend resolves person+time+jursidiction; retrieves vetted evidence; constructs answer sentence-by-sentence with citations; lists unverified areas. Clarifying questions only for essential ambiguity.

**J4 Compare:** Select two candidates in a specific contest; compare directly stated positions, documented actions, finance facts with identical time periods and limitations. Missing records labeled “Not located in indexed sources,” not “No record exists.”

**J5 Claim investigation:** Paste public quote/URL; capture context and speaker identity; decompose checkable propositions; seek corroborating/opposing sources; show what is documented, disputed, or unresolved. Never automatically classify a person's honesty or motive.

**J6 Proposition:** Locate an actual ballot measure by election and jurisdiction; reproduce legally appropriate official question where licensed; explain Yes and No, fiscal notes and uncertainties, cite state/county official text.

**J7 Corrections:** User submits citation/claim issue; queue stores report and affected answer/claim/source; curator reviews; correction log records before/after, rationale, reviewed date and notification where appropriate.

## 6. Functional requirements and acceptance criteria

### 6.1 Search and navigation (P0)
- Search people, roles, contests, measures, topics, sources, and places.
- Alias handling with disambiguation: same names, nickname/alternate formatting, changing office titles.
- Filters for county/city/district, election cycle, position, status, source type, date range.
- Every displayed political entity has a verified canonical ID and explicit as-of date.
- **Acceptance:** A candidate name plus a race yields only relevant verified records; ambiguous names trigger clear selection instead of silent merging.

### 6.2 Coverage transparency (P0)
- Display for each geography/entity: indexed source classes, missing categories, last indexed, last validated, update cadence.
- “This site covers selected races” banner when incomplete.
- **Acceptance:** Missing county-level records do not render as empty factual results without a coverage disclaimer.

### 6.3 Evidence viewer (P0)
- Every evidence item: title, publisher, URL, source type, original publication/event date, fetched time, content fingerprint, original excerpt, page/paragraph/timecode locator if available.
- Show archived or preserved snapshot when rights permit; avoid misrepresenting changed live URLs.
- Broken links flagged with alternate source search if available.
- **Acceptance:** Every quoted text passage corresponds to the original source and locator.

### 6.4 AI answer engine (P0, after evidence)
- Grounded on indexed, approved materials, not ungated model memory.
- Citations alongside each substantive political factual claim, not merely an end-of-answer bibliography.
- Separate attributed statements, observed actions, financial facts, independent reporting, analysis, allegations, corrections.
- User-visible limitations, date scope and coverage.
- No unsourced rankings, value judgments, endorsements, inferred motivations or electability forecasts.
- No source found -> explicit “I couldn't verify this in the sources currently indexed” and links to official research endpoints.
- Source conflicts -> show both with dates and explanation of why they differ when supported.
- **Acceptance:** Every generated factual statement has at least one supporting claim-source link; adversarial tests block fabricated URLs and misleading summaries.

### 6.5 Candidate/official comparison (P1)
- Side-by-side by user-selected topics; equal fields for each person.
- Preserve temporal equivalence (e.g., reporting dates and legislative sessions).
- Neutral alphabetic order or ballot order, labeled clearly. Never comparative scoring, winners, “better choice” statements.
- **Acceptance:** Not having a legislative record must not be shown as an adverse assessment of a newcomer.

### 6.6 Campaign finance (P1/P2)
- State records from California Secretary of State official finance system; federal records from FEC; local filings from respective local authorities where available.
- Distinguish direct campaign contributions, party contributions, PAC expenditures, independent expenditures; jurisdictions, committees, amounts, dates, reporting windows and amendments.
- Never claim money caused a specific policy vote based only on temporal proximity or donor identity.
- **Acceptance:** Sum of disclosures reconciles to imported filings where possible; raw transactions and query definitions are inspectable.

### 6.7 Privacy and accessibility (P0)
- No user accounts required. ZIP without persistently storing precise location. No individualized political profiles inferred or constructed.
- Retention limited to operational security, aggregate analytics; no ad-tech/political microtargeting.
- WCAG 2.2 AA target; keyboard flow, screen readers, contrast, reduced motion, comprehensible content.
- **Acceptance:** Core flows work without account and without cookies beyond strictly necessary ones.

## 7. Data model (proposed relational schema)

Use PostgreSQL with UUID primary keys, temporal validity fields, audit timestamps, and provenance foreign keys.

**Geography**: `jurisdictions(id, name, type, parent_id, fips_or_official_code, boundary_ref, valid_from, valid_to)`; `districts(id, jurisdiction_id, name, type, boundary_ref, valid_from, valid_to)`.

**Election**: `elections(id, name, election_date, jurisdiction_id, official_url, status)`; `contests(id, election_id, office_id, district_id, official_contest_key, status)`; `measures(id, election_id, jurisdiction_id, official_id, title, ballot_text, official_url)`.

**People and service**: `people(id, display_name, aliases_json, identifiers_json, verified_status)`; `offices(id, name, jurisdiction_id, branch, level)`; `candidacies(id, person_id, contest_id, status, ballot_designation, source_id)`; `tenures(id, person_id, office_id, start_date, end_date, source_id)`.

**Evidence and claim graph**: `sources(id, publisher, title, canonical_url, source_class, published_at, event_at, retrieved_at, checksum, license, original_language, status, archive_url)`; `documents(id, source_id, object_storage_uri, text, version, parser_version)`; `passages(id, document_id, locator_type, locator_value, excerpt, embedding)`; `claims(id, subject_person_id, subject_measure_id, normalized_claim, claim_kind, observed_or_attributed, start_date, end_date, status, created_at)`; `claim_support(id, claim_id, passage_id, relation, extraction_method, reviewer_id, reviewed_at)` where relation includes SUPPORTS, CONTRADICTS, CONTEXT, ATTRIBUTES and `claim_kind` includes STATEMENT, ACTION, VOTE, FUNDING, PROMISE, ANALYSIS.

**Actions**: `legislative_items(id, jurisdiction_id, title, official_number, source_id, meeting_date)`; `votes(id, person_id, legislative_item_id, vote_choice, vote_date, source_id)`; `promises(id, person_id, exact_quote, source_id, promise_date, scope)`; `promise_action_links(id, promise_id, legislative_item_id, link_type, evidence_notes, reviewer_id)`.

**Finance**: `committees(id, official_id, name, jurisdiction_id)`; `finance_filings(id, committee_id, source_id, period_start, period_end, filed_at, amended_filing_id)`; `transactions(id, filing_id, donor_name_public, recipient_committee_id, amount, transaction_date, category, legal_disclosure_ref)`.

**User and governance**: `research_sessions(id, anonymous_session_token_hash, created_at, expires_at)`; `questions(id, session_id, query, created_at, opt_in_feedback_only)`; `answers(id, question_id, version, answer_text, status, model_version, retrieval_trace_id, generated_at)`; `answer_claims(id, answer_id, claim_id, citation_locator, validation_status)`; `corrections(id, target_type, target_id, report, state, review_notes, resolved_at)`; `ingestion_runs(id, connector, started_at, finished_at, result, fetched, changed, failed)`; `coverage(id, jurisdiction_id, source_class, first_indexed_at, last_checked_at, completeness, disclaimer)`; `audit_events(id, actor, operation, object_id, timestamp, metadata)`.

Avoid copying unnecessary sensitive personal info, especially private residences, even if a filing exposes them. Define publication-safe fields separately from raw protected storage.

### Entity resolution rules

- Never merge solely on matching name.
- Resolve with jurisdiction, office, contest, dates, official IDs, and source provenance.
- Ambiguous matches require an admin review queue.
- Preserve historical roles and changed district maps.
- Add confidence only to internal matching systems, not political evaluations.

## 8. Ingestion architecture

**Connectors:** official state election pages (candidate certification and state propositions); Santa Clara county election authority; San Mateo county election authority; San Francisco Department of Elections; municipal and county meeting systems; official campaign disclosures; FEC for federal candidates. A source registry defines access method, jurisdiction, license, robots/ToS, update cadence, parser, and responsible maintainer.

**Stages:** discover -> fetch immutable source -> fingerprint/deduplicate -> parse PDF/HTML/CSV/structured feed -> extract metadata and candidate entities -> normalize -> evidence passages -> human QA for ambiguous records -> index full text and vectors -> publish with version -> monitor freshness and changes.

**Operational safeguards:** Idempotent source imports, source snapshots, retry/backoff, rate limits, polite crawling, integrity hashes, ingestion audit logs, dead-letter queue, document-removal/correction propagation, parser quality checks. Do not assume county interfaces offer APIs. Use official exports where available; otherwise implement compliant adapters.

**Initial source seeds (verify each before writing connector):**
- CA 2026 election: https://www.sos.ca.gov/elections/upcoming-elections/general-election-november-3-2026/
- CA official guide: https://voterguide.sos.ca.gov/
- CA official candidate list announcement: https://www.sos.ca.gov/administration/news-releases-and-advisories/2026-news-releases-and-advisories/california-secretary-state-shirley-n-weber-phd-certifies-candidate-list-november-3-2026-general-election
- Santa Clara ROV: https://rovservices.sccgov.org/
- San Mateo Elections: https://smcacre.gov/elections/november-3-2026-statewide-general-election
- San Francisco Elections: https://www.sf.gov/departments/department-elections
- FEC: https://www.fec.gov/

## 9. Question-answering pipeline and AI safety

**Input:** query, optional person/contest/jurisdiction/time context, anonymous session pointer.

1. Normalize dates and identify intent (lookup, compare, action, promise, finance, proposition, claim check).
2. Resolve entity/office/contest using structured data and disambiguation.
3. Determine scope and coverage; reject fabricated or unclear subjects.
4. Plan retrieval from structured SQL and approved passage index, apply source filters.
5. Generate atomic evidence claims and attach passage IDs; compute internal support status for each.
6. Compose neutral answer from supported evidence with source/date citations.
7. Run deterministic validation: each factual sentence maps to source; valid cited passages exist; links point to ingested sources; scope/dates aligned; no candidate evaluations/rankings.
8. On failure, regenerate once or withhold unsupported claims; show missing evidence and research links.
9. Persist trace and citation decisions for audit; respect user data minimization.

**Source hierarchy:** Primary authoritative election/legislative records for official facts; candidate-owned materials for their stated positions; reputable independent reporting for context with attribution; partisan/advocacy materials only when labeled as claims from that organization. Conflicting records are not silently resolved.

**Security:** Documents and external pages are untrusted inputs. Treat embedded instructions as data (prompt injection defense). Strict output schemas for tools; server-side secrets, input validation, HTML sanitization, request rate limits, malicious URL checks, outbound fetch allowlists or SSRF defenses, admin RBAC, audit logs, secret scanning, dependency scanning.

**Human review:** Required for allegation-heavy content and ambiguous identity matches before prominently publishing aggregated narratives; correction channel for all content.

**Disclosure:** Clear explanation that AI may be wrong; never imply it verified a source unless retrieved and checked; link directly to primary record whenever practical.

## 10. API contract (initial REST proposal)

Read-only public endpoints, paginated, typed and versioned `/api/v1/`:
- `GET /jurisdictions?county=...`
- `GET /people?q=...&jurisdiction_id=...`
- `GET /people/{id}`
- `GET /contests?election_id=...&district_id=...`
- `GET /measures?election_id=...`
- `GET /sources/{id}`
- `GET /coverage?jurisdiction_id=...`
- `POST /research` with `{question, context:{person_ids, contest_id, jurisdiction_id, date_range}}` (rate limited); returns `{answer, claims:[{text,source_ids,locators}], limitations, coverage, generated_at, trace_id}`.
- `POST /compare` with people IDs, topics and date window; returns symmetrical sourced fields.
- `POST /corrections` with target, report, optional contact, anti-abuse token.

**Admin endpoints:** authenticated ingestion job queue, entity resolution, review, retractions, audits. Publish OpenAPI 3.1 contract and typed frontend client. Always validate response shapes; do not leak internal admin URLs or private raw finance records.

## 11. ChatGPT / MCP distribution

Build a read-only MCP server exposing narrow tools:
- `find_public_officials`
- `get_public_person_profile`
- `search_public_evidence`
- `research_public_question`
- `compare_public_records`
- `get_jurisdiction_coverage`
- `get_source_metadata`

Every response contains source URLs, locators, date scope and known gaps. Tool descriptions must clearly state that source data is incomplete, must not score/endorse political candidates, and should cite original documents. Remote server uses HTTPS, auth/rate limits as appropriate, tracing and abuse controls. Test inside ChatGPT Work before any submission. Follow the current official plugin packaging, review and publication process; directory approval is not guaranteed. Design ordinary website pages to be crawlable (SSR pages, canonical URLs, sitemap, structured metadata, sensible robots rules). Do not expose unpublished or incomplete records to crawler indexing.

Official docs verified 2026-10-19:
- https://developers.openai.com/plugins/quickstart
- https://developers.openai.com/plugins/build/app-quickstart
- https://developers.openai.com/plugins/deploy/submission

## 12. UX design system

Trustworthy civic-research aesthetic: high-legibility typography, restrained palette, no party-coded red-vs-blue emphasis in overall framing; use neutral structural colors. Mobile-first. Search and citation controls always accessible; one-tap “open source” from every claim; differentiation between quoted source and generated synthesis; clear coverage status. Prefer simple browsing paths and progressive disclosure of original records. Plain-language explanations, side-by-side views collapse elegantly on mobile, minimal jargon. Do not overload the homepage with AI novelty features.

### Component inventory

`SiteHeader`, `CountySelector`, `CoverageNotice`, `SearchInput`, `EntityDisambiguator`, `CandidateCard`, `OfficialTimeline`, `SourceCitation`, `SourceViewer`, `EvidenceStatus`, `AskPanel`, `AnswerWithCitations`, `ClaimCard`, `ComparisonGrid`, `MeasureExplainer`, `CorrectionReport`, `MethodologyDisclosure`, `AdminReviewQueue`.

## 13. Nonfunctional requirements / SLO targets (proposed)

- Mobile-first responsive from 320 px; WCAG 2.2 AA target.
- p95 directory search under 1.5 seconds for warm queries; evidence search under 3 seconds; streamed first AI token under 6 seconds where infrastructure allows; display explicit loading state.
- 99.5% uptime goal after production stabilization; graceful read-only fallback if LLM provider fails.
- Autoscaling and fair-use quotas; caching public immutable source pages and non-personalized queries.
- Encryption in transit and at rest, regular backups with restore test, migration rollback strategy.
- Privacy-preserving analytics, documented retention, no data sale or ad profiling.
- Public correction acknowledgement target 3 business days; timing subject to staffing.
- Logs never include API keys, precise residence, or private raw query data by default.

## 14. Evaluation strategy / release gates

**Golden set:** Build at least 100 human-reviewed questions representing all three counties, election lookup, statements, votes, finance, measures, outdated sources, changed offices, ambiguous names, missing coverage, politically disputed claims, and prompt injection.

**Suggested prelaunch gates (targets, not measured results):**
- 100% factual answer claims contain accessible citation IDs.
- >= 95% independently reviewed citation-to-claim support precision on golden set.
- 0 fabricated URLs or fictitious people in verified release tests.
- 0 unsupported evaluative candidate verdicts, ratings, and endorsements.
- >= 95% correct entity resolution on benchmark, with explicit fallback for ambiguous cases.
- Correct missing-data wording in 100% of “no indexed evidence” tests.
- Source-document extraction QA pass on sampled official PDFs and HTML.
- No critical/high unresolved security findings; accessibility keyboard and screen-reader smoke tests pass.

Track: document ingestion success rate, citation precision, unsupported assertion rate, entity-resolution accuracy, source freshness, time-to-answer, research task success, correction volume/time, coverage breadth, unit economics.

## 15. Tech stack and repository structure

**Recommendation:** TypeScript monorepo managed with pnpm. Next.js App Router UI/API, PostgreSQL + pgvector, Drizzle or Prisma ORM (choose one), Python document extraction workers where PDFs/tables warrant it, Redis-backed queue if workload demands, S3-compatible object storage for immutable source snapshots, OpenTelemetry-based tracing. Deploy with environment separation (`local`, `staging`, `production`). Select concrete managed providers after costs and access are known; do not silently commit to paid vendors.

```
public-record/
  apps/web/                  # Next.js public UI, server routes
  apps/mcp/                  # remote read-only MCP server
  services/ingestion/        # Python/TS connector workers, PDF and HTML parsers
  packages/db/               # schema, migrations, seed data
  packages/contracts/        # Zod / OpenAPI schemas, shared types
  packages/evidence/         # claim representation, source validation
  packages/ui/               # reusable accessible UI
  data/fixtures/demo/        # CLEARLY LABELED synthetic demo fixtures only
  evals/                     # golden questions, harness, rubric, regression reports
  docs/                      # ADRs, methodology, coverage, runbooks, source registry
  infra/                     # environment templates, deployment manifests
  .github/workflows/         # CI: lint, types, unit, integration, security
  README.md
  AGENTS.md
```

## 16. Initial Codex tasks, in order

**Ticket PR-001 — Repository scaffolding**: Initialize pnpm monorepo, strict TS, lint/prettier, unit test runner, GitHub Actions, env example, architecture README. Acceptance: clean installation and passing lint/typecheck/tests in CI.

**PR-002 — DB and migration foundations**: Build core jurisdiction/election/contest/people/sources/passages/coverage schema; versioned migrations; constraints; seed three county jurisdiction entities and isolated synthetic demo people. Acceptance: migrations up/down tested; uniqueness and FK tests; no synthetic entities outside demo.

**PR-003 — Frontend shell**: Mobile-first home, explore, person, research, source, coverage, methodology pages using fixtures; accessible nav and search. Acceptance: keyboard-accessible core flows, no unlabeled fake political facts.

**PR-004 — Source registry and import job**: State 2026 candidate roster import, immutable fetched copy, checksum, parser, ingestion events, data QA; protected admin controls. Acceptance: import idempotency and provenance; malformed files quarantined; match official counts where available.

**PR-005 — County adapter pilot**: Pick one official source for Santa Clara and build source adapter; reusable base class. Acceptance: documented source rules, licensing/access review, at least one verified source-to-person link, clean missing-data state.

**PR-006 — Evidence UI**: Search official source passages; open original URL and locator; coverage status. Acceptance: every result shows source origin/date, broken links detectable.

**PR-007 — Grounded research endpoint**: Implement research pipeline with structured response schema, deterministic citation validation, missing-source fallback. Acceptance: golden test suite passes, no citation fabrication.

**PR-008 — Comparison and measures**: Candidate comparison, proposition page, official ballot descriptions. Acceptance: source parity, correct election/jurisdiction and date.

**PR-009 — Corrections, analytics and QA**: Report issue, admin review workflow, revision history, privacy analytics, production security checklist.

**PR-010 — Broader ingestion**: Expand San Mateo and San Francisco, add current official histories, finance connector and official public action records with ongoing freshness monitoring.

**PR-011 — MCP plugin**: Build read-only tools, integration tests, public metadata and submission bundle only after core evidence and accuracy checks pass.

## 17. Budget and ongoing operations

Design for low operational cost: use authoritative bulk exports and incremental refresh rather than continuous crawling; cache public answers by question+source version; reserve expensive models for ambiguous synthesis, not straightforward lookups; separate research ingestion from serving. Track cost per indexed source, cost per research answer, compute/storage, and manual review burden. Establish a monthly budget before production. Publish funding/operational transparency if taking grants or donations.

## 18. Risk register

| Risk | Mitigation |
|---|---|
| False claims/hallucinations | Atomic claim attribution; deterministic citation validation; golden evaluation; human reviews |
| Incomplete local coverage | Public coverage ledger and last-checked timestamps; never imply absence |
| Political bias in sources and synthesis | Source-class labeling; equal source standards; neutral wording; review and correction procedures |
| Misleading candidate identity merges | Strong entity identifiers and review queue |
| Source disappears or changes | Immutable snapshots where permitted, URL/sha256/version metadata, stale source alerts |
| Defamation/reputational harm | Avoid unsupported allegations or motive inference; factual source links; escalation for sensitive claims |
| Personal privacy violations | Minimize collection, redact unnecessary addresses, rate-limit extraction, prohibit profiling |
| Legal/licensing/robots constraints | Per-source compliance review and attribution/rights registry |
| Election-day traffic | Caching, graceful degradation, fixed-cost controls |
| ChatGPT directory not approved | Website and API remain independently valuable |

## 19. Source register and verification notes

Official CA Secretary of State 2026 statewide election page verifies candidate-list links and relevant election information: https://www.sos.ca.gov/elections/upcoming-elections/general-election-november-3-2026/ . Certified list announced August 27, 2026. Official guide: https://voterguide.sos.ca.gov/ . Use county election offices for local candidates, do not infer county/local roster completeness from state list. The precise accessibility, formats and permissions of county feeds still require adapter-level verification.

OpenAI current plugin developer guides (checked 2026-10-19) describe MCP server tools and optional UI for ChatGPT/Codex, plus directory review and publication; approval is not guaranteed. See URLs in section 11.

## 20. Decisions still open

1. Hosting and monthly budget cap.
2. Entity/data ingestion source-by-source approval and licensing review.
3. What proportion of user queries to retain, and for how long, if at all.
4. Whether to use hosted LLM(s), which ones, and what privacy terms apply.
5. Editorial reviewer(s), correction escalation, legal review availability.
6. Languages beyond English and accessibility testing resources.
7. Whether to offer an open downloadable research dataset and under which license.
8. Official source capture and snapshots permitted by each source.

**Baseline assumption:** Proceed with anonymous English-language public research and no user profile or advertising model; make open decisions configurable, not blockers for foundational work.

## 21. Copy/paste kickoff prompt for Codex

> You are the implementation engineer for The Public Record. Read `THE_PUBLIC_RECORD_SPEC.md` fully and treat it as the baseline product requirements. Start with PR-001 only. Propose the exact files and commands, create a TypeScript pnpm monorepo with Next.js public UI scaffold, strict typing, a small test framework, linting, formatting, env example, GitHub Actions CI, README and AGENTS.md. Do not connect paid services, deploy, or generate real political claims. Label demo data clearly. Run the tests and report results. Stop after PR-001 and wait for the next milestone authorization.