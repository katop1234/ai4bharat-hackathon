# Design: AI Hiring Agent

## Overview
AI Hiring Agent automates sourcing, screening, and first-round phone interviews:
1) Parse a job description into structured requirements
2) Source candidates (LinkedIn / web / internal lists)
3) Score and shortlist candidates
4) Generate outreach messages
5) Run AI phone screens (voice), produce transcript + evaluation

## High-level architecture
- UI (web form / CLI): JD input, run start, results view
- Orchestrator (agent runner): coordinates tools + steps
- Sourcing module: candidate discovery
- Scoring module: structured evaluation + ranking
- Outreach module: personalized message generation
- Voice screening module: call, transcript, rubric scoring
- Storage: run logs + candidate snapshots + transcripts

## Data model (minimal)
- JobSpec:
  - title, location, must_haves, nice_to_haves, years_exp, skills, keywords
- Candidate:
  - name, profile_url, headline, location, experience, skills, notes, score, evidence
- ScreenResult:
  - candidate_id, call_status, transcript, rubric_scores, summary, recommendation

## End-to-end flow
1. Recruiter pastes JD + constraints (location, budget, seniority)
2. Orchestrator extracts JobSpec (LLM -> structured JSON)
3. Sourcing module retrieves candidate pool (20+)
4. Scoring module:
   - compares Candidate vs JobSpec
   - outputs score 0-10 + evidence bullets (why/why not)
5. Outreach module generates personalized message variants
6. Voice screening module calls selected candidates:
   - asks standard questions
   - produces transcript + rubric scoring
7. UI shows ranked candidates + transcripts + recommendations

## Scoring rubric (example)
- Role fit (0-4)
- Skills match (0-3)
- Communication (0-2)
- Dealbreakers (0-1)

## Tooling / integrations (examples)
- Candidate sourcing: LinkedIn search (manual export / recruiter access) or web search
- LLM: OpenAI model for extraction, scoring, outreach text
- Voice: telephony provider + speech-to-text + LLM evaluator
- Optional: ATS integration via CSV/Webhooks

## Safety, privacy, and compliance
- Do not store LinkedIn credentials
- Store only necessary candidate data per run
- Allow deletion of run data
- Include opt-out language in outreach
- Log decisions + evidence for auditability

## Failure modes & mitigations
- Low-quality sourcing -> broaden keywords, relax constraints, rerank
- Hallucinated candidate facts -> only score using retrieved text; cite evidence fields
- Call failures -> retry scheduling + fallback to SMS/email screening
- High cost -> limit candidate pool, cache extraction/scoring, batch LLM calls
