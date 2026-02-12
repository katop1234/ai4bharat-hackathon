# Design: AI Hiring Agent

## Overview
AI Hiring Agent provides fully autonomous recruitment from job description to vetted candidates:
1) Parse JD and search LinkedIn for matching candidates
2) Enrich with contact info (email/phone), salary estimates, notice periods
3) Automatically call candidates for AI-powered phone screens
4) Evaluate transcripts and present only qualified candidates (7.5+ fit score)
5) Hiring manager reviews vetted candidates and schedules final interviews

## High-level architecture
- **Frontend (React)**: JD input, live reasoning traces, candidate results, call management
- **Backend (FastAPI)**: Orchestrates pipeline, manages state, handles API integrations
- **Manus AI**: Autonomous candidate sourcing and scoring from LinkedIn
- **EazyReach API**: Contact enrichment (email + phone extraction)
- **GPT-4o-mini**: Salary estimation, InMail generation, transcript evaluation
- **Vapi.ai**: Voice calling infrastructure with AI interview agent
- **Storage**: File-based JSON storage (projects, runs, candidates, transcripts)

## Data model
- **Candidate:**
  ```python
  {
    'name': str,
    'profile_url': str,              # LinkedIn URL
    'location': str,                 # City extracted from profile
    'experience_years': int,         # Years of experience
    'current_role': str,             # "Senior Engineer at Google"
    'strengths': str,                # Key skills (1-2 sentences)
    'concerns': str,                 # Red flags or "None"
    'score': float,                  # 7.0-10.0 (Manus initial score)
    
    # Enriched data (post-processing)
    'email': str,                    # From EazyReach
    'phone': str,                    # From EazyReach
    'enrichment_status': str,        # "searching"|"found"|"not_found"
    'salary_min': int,               # From GPT-4o-mini (INR)
    'salary_max': int,
    'currency': str,                 # "INR"
    'notice_period_days': int,       # Rule-based estimate
    'inmail': {                      # From GPT-4o-mini
      'subject': str,
      'body': str
    },
    
    # Phone screening data
    'call_transcript': str,          # Full conversation text
    'call_evaluation': {             # From GPT-4o-mini
      'overall_assessment': str,
      'strengths': [str],
      'red_flags': [str],
      'recommendation': str,         # "Strong Yes"|"Yes"|"Maybe"|"No"
      'fit_score': float             # 0-10 based on interview
    }
  }
  ```

- **Run:**
  ```python
  {
    'id': str,                       # run_00001, run_00002, etc.
    'status': str,                   # "running"|"completed"|"failed"
    'candidates': [Candidate],
    'action_traces': [str],          # Live reasoning from Manus
    'created_at': timestamp,
    'duration_seconds': float,
    'credit_usage': int              # Manus credits spent
  }
  ```

## End-to-end flow

### **Phase 1: Sourcing (3-5 mins)**
1. User pastes JD (e.g., "Senior ML Engineer in Chennai with 5+ years Python/TensorFlow")
2. Manus AI automatically:
   - Searches LinkedIn for matching profiles
   - Evaluates each candidate against JD requirements
   - Scores 7.0-10.0 (only returns 7.0+)
   - Streams reasoning traces to UI ("Found 8 strong matches...", "Analyzing senior engineers...")
3. Returns 15 candidates with: name, LinkedIn URL, experience, role, strengths, concerns, score

### **Phase 2: Enrichment (1-2 mins, parallel)**
Four tasks run concurrently for all 15 candidates:

**A. Contact Discovery** (EazyReach API)
- Fetches email + phone from LinkedIn profiles
- Success rate: ~60-80% depending on profile visibility
- Cost: ~$0.02 per profile

**B. Salary Estimation** (GPT-4o-mini, 15 parallel calls)
- Input: role, experience, location
- Output: salary_min, salary_max (INR default)
- Cost: ~$0.0001 per candidate = $0.0015 total
- Time: ~1-2 seconds (parallel execution)

**C. Notice Period** (Rule-based, zero cost)
- Senior roles (8+ years): ~60 days
- Mid-level (4-7 years): ~30 days
- Junior (<4 years): ~15 days

**D. InMail Generation** (GPT-4o-mini, 15 parallel calls)
- Personalized recruiting message based on JD + candidate profile
- Subject + body formatted for LinkedIn
- Cost: ~$0.0003 per candidate = $0.0045 total

**Result:** ~10 candidates have phone numbers (5 dropped), all have complete metadata

### **Phase 3: Automated Phone Screening (30-60 mins)**

**Calling Strategy:**
- Batch processing: 3 concurrent calls at a time
- Respects business hours (10 AM - 6 PM local time)
- Retry logic: 2 retries if no answer (1 hour apart)

**Per-candidate call flow:**
1. Vapi.ai initiates outbound call
2. AI agent introduces itself and company
3. Conducts 5-10 min structured interview:
   - Experience verification
   - Technical skills assessment
   - Project examples and achievements
   - Availability and salary expectations
   - Interest level and questions
4. Call ends → Full transcript saved
5. GPT-4o-mini analyzes transcript → Evaluation with fit_score

**AI Evaluation Scoring:**
- Technical skills match (40%)
- Experience relevance (30%)
- Communication clarity (15%)
- Cultural fit indicators (10%)
- Availability/interest (5%)
- Output: Overall assessment, strengths, red flags, recommendation, fit_score (0-10)

### **Phase 4: Automated Filtering**
System applies vetting criteria:
```python
QUALIFIED = {
  'recommendation': ['Strong Yes', 'Yes'],
  'fit_score': >= 7.5,
  'red_flags': <= 1
}
```

**Example outcome:**
- 10 candidates called
- 7 passed vetting (3 "Strong Yes", 4 "Yes")
- 3 filtered out (2 "Maybe", 1 "No")

### **Phase 5: Presentation to Hiring Manager**
Dashboard shows only vetted candidates (7 total) with:
- All metadata (experience, salary expectations, notice period)
- Contact info (email, phone)
- Call transcript + AI evaluation
- Pre-generated InMail ready to send
- Action buttons: "View Transcript", "Send InMail", "Schedule Interview"

## Tooling / integrations
- **Backend:** FastAPI (Python)
- **Frontend:** React.js with real-time polling
- **Sourcing:** Manus AI (agentic LinkedIn search + evaluation)
- **Enrichment:** EazyReach API (email/phone extraction)
- **LLM:** GPT-4o-mini for salary, InMails, transcript evaluation
- **Voice:** Vapi.ai (AI-powered phone calling)
- **Storage:** File-based JSON (projects, runs, candidates)
- **Deployment:** Railway (backend), Vercel (frontend)

## Safety, privacy, and compliance
- No LinkedIn credentials stored (Manus has own access)
- Session-isolated data (anon_* for unauthenticated users)
- Run data can be deleted
- Automatic daily limits (50 candidates for registered, 15 for anonymous)
- Graceful error handling (enrichment/calling failures don't block pipeline)
- Usage tracking and audit logs

## Failure modes & mitigations
- **No phone numbers found** → Fallback to InMail outreach
- **All candidates fail vetting** → Suggest relaxed criteria to user
- **API key exhausted** → Automatic rotation across multiple keys
- **Call not answered** → Retry up to 2x, mark for manual follow-up
- **High cost concerns** → Batch LLM calls, parallel execution, cache results
- **Trace streaming breaks** → Debug logging shows what Manus returns during polling
