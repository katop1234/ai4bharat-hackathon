# Project: AI Hiring Agent

## One-line summary
AI agents automate candidate sourcing, screening, and phone interviews for recruiters.

## Problem
- Recruiters spend hours manually searching LinkedIn and screening candidates
- 80% of sourcing time is repetitive (search → evaluate → reach out)
- Scaling hiring requires expensive recruiting teams

## Goals
- G1: Automate candidate shortlisting from job description to scored candidates
- G2: Conduct AI-powered first-round phone screens
- G3: Reduce recruiter time per hire by 70%+

## Non-goals
- NG1: Final interview decisions (human stays in the loop)
- NG2: ATS replacement (integrates with existing tools)

## Users
- Primary: Recruiters and hiring managers at startups/SMBs
- Secondary: HR teams at mid-market companies

## Key user stories
1. As a recruiter, I want to paste a job description and get 20 scored candidates in 5 minutes, so I can focus on top matches.
2. As a hiring manager, I want AI to conduct first phone screens, so I only talk to pre-qualified candidates.
3. As a recruiter, I want personalized outreach messages generated, so I can reach out faster.

## Functional requirements
- FR1: Accept job description text, extract requirements automatically
- FR2: Search LinkedIn and source 20+ matching candidates
- FR3: Score candidates 0-10 with detailed evaluation
- FR4: Generate personalized InMail messages for each candidate
- FR5: Conduct AI voice phone screens with transcript and evaluation

## Non-functional requirements
- NFR1: Latency: <5 min for candidate sourcing, <10 min for phone screen
- NFR2: Cost: <$5 per screening run (60-70 Manus credits + GPT-4)
- NFR3: Privacy: No LinkedIn credentials stored, session-isolated data
- NFR4: Reliability: Auto-retry with API key rotation, orphan run cleanup
- NFR5: Accessibility: English interface, supports Indian phone numbers

## Success metrics
- SM1: Time to get 20 screened candidates: <5 minutes (vs 2-3 hours manual)
- SM2: Phone screen completion rate: >60% (candidate picks up and completes call)
- SM3: Recruiter satisfaction: 4+ stars on candidate quality

## Assumptions & constraints
- A1: Candidates have public LinkedIn profiles or recruiter has access
- C1: Requires Manus API credits (~60-70 per run) and OpenAI API access
