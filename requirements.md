# Project: AI Hiring Agent

## One-line summary
Autonomous AI agent that finds, contacts, interviews, and vets hundreds of candidates in minutes.

## Problem
- Recruiters spend 10-15 hours per hire manually searching and screening candidates
- 90% of sourcing work is repetitive: search LinkedIn → evaluate fit → find contact info → schedule calls → conduct first screens
- Scaling hiring requires expensive recruiting teams ($80K-150K per recruiter)
- High-quality candidates get snatched up quickly (speed = competitive advantage)

## Goals
- G1: Fully automate candidate discovery to vetted shortlist (JD → qualified candidates in 45-60 mins)
- G2: Eliminate manual sourcing and first-round screening work
- G3: Reduce cost per hire by 95% ($5 per fully-vetted candidate vs $500+ recruiter fees)
- G4: Enable hiring managers to scale to hundreds of roles without expanding recruiting team

## Non-goals
- NG1: Final hiring decisions (human reviews vetted candidates and makes offers)
- NG2: Replace ATS (integrates as a candidate pipeline source)
- NG3: Cultural fit assessment (AI focuses on skills/experience, humans assess culture)

## Users
- **Primary:** Hiring managers at startups/SMBs who want to hire fast without recruiters
- **Secondary:** Recruiting teams at growth companies scaling from 50 to 500 people
- **Tertiary:** HR leaders evaluating AI-powered recruiting tools

## Key user stories
1. As a hiring manager, I want to paste a JD and get 7 phone-vetted candidates in under an hour, so I can schedule final interviews same day.
2. As a recruiter, I want AI to handle sourcing/screening so I can focus on closing candidates and negotiating offers.
3. As a startup founder, I want to hire 10 engineers in a month without paying $50K to recruiting agencies.

## Functional requirements

### Core Pipeline
- **FR1:** Accept free-form job description text (no structured input required)
- **FR2:** Automatically search LinkedIn and source 15 qualified candidates (7.0+ score)
- **FR3:** Extract contact info (email + phone) from LinkedIn profiles
- **FR4:** Estimate salary ranges in INR based on role/experience/location
- **FR5:** Generate personalized InMail messages for each candidate
- **FR6:** Automatically call candidates with phone numbers
- **FR7:** Conduct 5-10 min AI voice interviews covering technical skills, experience, and availability
- **FR8:** Analyze transcripts and score candidates (0-10 fit score)
- **FR9:** Present only qualified candidates (7.5+ fit score, ≤1 red flag) to hiring manager

### User Experience
- **FR10:** Stream live reasoning traces showing AI's search/evaluation process
- **FR11:** Display real-time status for enrichment ("Searching...", "Found", "Not found")
- **FR12:** Show call status ("Calling...", "In Progress", "Completed", "Failed")
- **FR13:** Allow overriding active runs (always-clickable input field)
- **FR14:** Persist state across page refreshes (resume polling for active runs)

### Data Management
- **FR15:** Session-based isolation (anon users can't see each other's runs)
- **FR16:** Run history with full candidate details, transcripts, and reasoning traces
- **FR17:** Export candidates to CSV (for ATS import)

## Non-functional requirements
- **NFR1: Speed**
  - Sourcing: 3-5 mins for 15 candidates
  - Enrichment: 1-2 mins (parallel execution)
  - Phone screens: 5-10 mins per call, 3 concurrent = ~30-60 mins for 10 candidates
  - **Total: 45-75 mins from JD to vetted candidates**

- **NFR2: Cost (per run with 15 candidates)**
  - Manus sourcing: ~50 credits (~$3-5 depending on complexity)
  - EazyReach enrichment: $0.30 (15 × $0.02)
  - Salary estimation: $0.0015 (15 × $0.0001)
  - InMail generation: $0.0045 (15 × $0.0003)
  - Phone calls: $0.50-1.50 (10 calls × $0.05-0.15 per call)
  - Transcript evaluation: $0.003 (10 × $0.0003)
  - **Total: ~$4-7 per run** (vs $500+ recruiter/agency fees)

- **NFR3: Reliability**
  - Auto-retry with API key rotation (up to 3 Manus keys)
  - Graceful degradation (enrichment failures don't block pipeline)
  - Corrupted run detection and cleanup
  - Atomic file writes to prevent data corruption

- **NFR4: Privacy & Compliance**
  - No LinkedIn credentials stored (Manus has own access)
  - Session-isolated data (each user only sees their runs)
  - Daily usage limits (50 candidates for registered, 15 for anonymous)
  - Usage tracking for audit logs

- **NFR5: Scalability**
  - Parallel LLM calls (10 workers for salary/InMail generation)
  - Concurrent phone calls (3 at a time)
  - Stateless architecture (can scale horizontally)

- **NFR6: Accessibility**
  - Supports Indian phone numbers (Twilio + Vapi integration)
  - INR currency default for salary ranges
  - Works in Indian business hours

## Success metrics
- **SM1:** Time from JD to vetted candidates: **<60 mins** (vs 2-3 days manual)
- **SM2:** Phone screen success rate: **>60%** (candidates pick up and complete interview)
- **SM3:** Vetting accuracy: **>80%** of AI "Yes" recommendations pass human final interview
- **SM4:** Cost per vetted candidate: **<$1** (vs $500+ recruiter fees)
- **SM5:** User satisfaction: **4+ stars** on candidate quality and system reliability

## Assumptions & constraints
- **A1:** Candidates have public LinkedIn profiles OR Manus can access via recruiter accounts
- **A2:** Candidates answer phone calls from unknown numbers (60-80% pickup rate)
- **A3:** EazyReach has phone data for 60-80% of Indian professionals
- **A4:** GPT-4o-mini can accurately evaluate technical skills from 10-min conversation
- **C1:** Requires Manus API credits (~50-70 per run)
- **C2:** Requires OpenAI API credits (negligible cost <$0.01 per run)
- **C3:** Requires EazyReach credits (~$0.30 per run)
- **C4:** Requires Vapi.ai credits (~$1-2 per run for 10 calls)
- **C5:** Backend must be deployed with all environment variables set (MANUS_API_KEY, OPENAI_API_KEY, EAZYREACH_TOKEN, VAPI_API_KEY)
