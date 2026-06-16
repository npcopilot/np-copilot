# NP Co-Pilot — Project Context for Claude Code
**Founder:** Rachel Ho, NP | rachel@npcopilot.ai  
**Website:** npcopilot.ai  
**Product:** AI Clinical OS for nurse practitioners  
**Stage:** Beta / founding pilot (50 NPs)
---
## What NP Co-Pilot Is
A single-file HTML clinical operating system built for nurse practitioners. Runs entirely in the browser — no backend, no accounts, no installation. Everything stays on the device. Covers the full visit workflow: demographics, consent, 14-system physical exam, HPI, assessment & plan, E/M coding, orders (lab, imaging, Rx, referral), SOAP note output, and AI Chief Resident.
**Core positioning:**
- Vertical AI — NP-specific clinical logic (billing rules, USPSTF protocols, HCC coding) is the moat
- Agentic AI Chief Resident — runs 6 steps autonomously before every visit: chart summary, missing labs, USPSTF gaps, differential prompts, coding risk flags, medication review
- Structured-data architecture — zero-hallucination by design; every field is clinician-confirmed, nothing is inferred from ambient audio
---
## Technical Architecture
### HTML App Rules (CRITICAL — must be followed exactly)
- **Single-file HTML app** — all HTML, CSS, and JS in one file; exactly **1 `<script>` tag**
- JS must pass `node --check` (no syntax errors)
- `var` for module-level state; `let`/`const` inside functions only
- `localStorage` with `npcp_` prefix for all persistence
- No external dependencies, no CDN, no backend
### Key State Variables
- `var DEMO_FIELD_IDS` — array of all field IDs cleared by `clearDemographics()`
- `var demoFieldsWrapper` — div controlling lock/unlock/opacity for demographics
### Key Functions
| Function | Purpose |
|----------|---------|
| `lockDemographics()` | Locks demo fields, sets pointer-events:none |
| `unlockDemographics()` | Unlocks demo fields |
| `clearDemographics()` | Clears all DEMO_FIELD_IDS including EC fields |
| `savePatientDemo()` | Saves all DEMO_FIELD_IDS to `localStorage('npcp_demographics')` |
| `loadSavedDemographics()` | Restores saved demographics on page load |
| `openPrescriberSettings()` | Opens prescriber info modal |
| `savePrescriberSettings()` | Saves prescriber info to localStorage |
| `printRx(idx)` | Print drug order |
| `printLabOrd(idx)` | Print lab order |
| `printImgOrd(idx)` | Print imaging order |
| `printRefOrd(idx)` | Print referral order |
| `updatePatientHeader()` | Updates topbar with patient name |
### DEMO_FIELD_IDS (complete list)
```
ptFirst, ptMiddle, ptLast, ptDOB, ptAge, ptSex, ptMRN, ptPhone, ptEmail,
ptStreet, ptCity, ptState, ptZip, ptInsurance, ptMemberID, ptGroupNum,
ecName, ecRelationship, ecPhone, ecPhoneAlt, ecEmail, ecAddress, ecBestTime, ecNotes,
ec2Name, ec2Relationship, ec2Phone, ec2Email,
naConsent, consent-notes, hipaaConsent, hipaaDate, hipaaMethod
```
### Consent Tab Structure (correct order)
1. Patient ref code
2. Demographics (inside `demoFieldsWrapper`) — includes EC fields
3. Save Patient Demographics button (OUTSIDE demoFieldsWrapper — always clickable)
4. Telehealth consent section
5. HIPAA Privacy Notice & Acknowledgment (hipaaConsent, hipaaDate, hipaaMethod, hipaaNotes)
6. Advanced Directive
### savePatientDemo() — current implementation
```javascript
function savePatientDemo() {
  const first = (document.getElementById('ptFirst')?.value || '').trim();
  const last  = (document.getElementById('ptLast')?.value || '').trim();
  if (!first || !last) {
    alert("Please enter the patient's first and last name before saving.");
    document.getElementById('ptFirst')?.focus();
    return;
  }
  const saved = {};
  DEMO_FIELD_IDS.forEach(function(id) {
    const el = document.getElementById(id);
    if (!el) return;
    if (el.type === 'checkbox') saved[id] = el.checked;
    else saved[id] = el.value;
  });
  localStorage.setItem('npcp_demographics', JSON.stringify(saved));
  // badge, header update, topbar flash follow
}
```
### loadSavedDemographics() — current implementation
```javascript
function loadSavedDemographics() {
  try {
    const raw = localStorage.getItem('npcp_demographics');
    if (!raw) return;
    const saved = JSON.parse(raw);
    Object.keys(saved).forEach(function(id) {
      const el = document.getElementById(id);
      if (!el) return;
      if (el.type === 'checkbox') el.checked = !!saved[id];
      else el.value = saved[id];
    });
    updatePatientHeader();
  } catch(e) {}
}
```
Called on page load after `loadPrescriber()`.
---
## Files in /Users/rachelho/Downloads/
| File | Description |
|------|-------------|
| `NP_CoPilot_Blank_Form (37).html` | Production blank form — current working version |
| `NP_CoPilot_Demo (15).html` | Pre-filled demo form — current working version |
| `NP_CoPilot_FAQ.md` | 20 Q&As across Privacy/HIPAA, How It Works, Clinical/Billing, Getting Started, Orders, Reliability |
| `NP_CoPilot_AANP_Outreach_Email.md` | 3 outreach emails to AANP conference AI speakers (Lisa Anderson DNP; TBD speaker; GW professor/Center for Health Policy Executive Director) |
| `NP_CoPilot_USF_Meeting_Prep.md` | Talking points, Q&A, and 8 objection responses for USF Dean meeting |
| `NP_CoPilot_USF_Elevator_Pitch.md` | 2-minute pitch for USF Dean (Dr. Eileen K. Fry-Bowers, efrybowers@usfca.edu) |
| `NP_CoPilot_USF_Pilot_Surveys.md` | Student survey (23 Qs) and Faculty survey (23 Qs) for USF pilot |
| `NP_CoPilot_Attorney_Inquiry_Email.md` | Full attorney email re: Optum employment protection (6 issues, cites CA BPC §16600 and Labor Code §2870) |
| `NP_CoPilot_Attorney_Inquiry_Email_Short.md` | Same as above, under 2000 characters |
| `NP_CoPilot_Website_Copy_Airo_Instructions.md` | Copy additions for Airo (web designer) — 3 new sections for npcopilot.ai |
| `CLAUDE.md` | This file |
---
## Known Bugs Fixed (do not re-introduce)
| Bug | Fix Applied |
|-----|-------------|
| Duplicate EC block appended after `</body></html>` | Truncated file at first `</body>\n</html>` |
| Consent tab div balance was -2 in Demo | Removed 2 orphaned `</div>` tags and stray comment |
| Telehealth consent missing from Blank Form | Extracted from Demo, inserted after demoFieldsWrapper |
| HIPAA Privacy Notice missing from Blank Form | Extracted from Demo, inserted before Advanced Directive |
| Save button locked (was inside demoFieldsWrapper) | Moved Save button outside demoFieldsWrapper |
| savePatientDemo() didn't actually save | Rewrote to use localStorage with all DEMO_FIELD_IDS |
| hipaaConsentBanner orphaned inside demoFieldsWrapper | Removed; replaced with proper HIPAA section |
| Prescriber Info button hard to find | Updated to prominent teal-filled style in both files |
---
## Business Context
### Legal / Employment
- Rachel works at Optum Housecalls (UnitedHealth Group subsidiary), California
- Building NP Co-Pilot entirely on personal time with personal resources
- Disclosure letter drafted to Optum General Counsel (file: `NP_CoPilot_Optum_Disclosure_Letter_v2_4.docx`)
- Attorney email drafted for employment protection review
- Key statutes: CA BPC §16600, CA Labor Code §2870
### HIPAA / AI
- No BAA with Anthropic — AI features require removing patient identifiers before use
- Local HTML file = no data transmitted = no BAA required from Rachel's side
- Path to HIPAA-eligible AI: AWS Bedrock (HIPAA-eligible, runs Claude)
- On HIPAA-compliant device, real PHI can be entered in the non-AI form fields
### Website (npcopilot.ai)
- Hosted on Netlify (demo gated behind request-access form)
- Airo = web designer handling site updates
- Three positioning gaps identified (see `NP_CoPilot_Website_Copy_Airo_Instructions.md`):
  1. Vertical AI moat — needs explicit articulation
  2. Agentic Chief Resident — add word "agentic" to existing section
  3. Structured-data vs. ambient audio — new section needed
### Partnerships / Outreach
- USF School of Nursing (alma mater) — pilot proposal to Dean Fry-Bowers
- AANP Conference June 23–27 — outreach to 3 AI-track speakers for beta users
- Competitor landscape: Suki, Nabla, Abridge, Freed, DeepScribe (ambient audio AI); DAX Copilot (Microsoft/Nuance); Epic/Abridge integration; Athenahealth/Commure
---
## Coding Conventions for HTML Files
When editing `NP_CoPilot_Blank_Form (37).html` or `NP_CoPilot_Demo (15).html`:
1. Always run `node --check filename.html` after edits to verify JS syntax
2. Count consent tab `<div` opens vs `</div>` closes — must balance to 0
3. Never add a second `<script>` tag
4. Never use `let`/`const` at module level — use `var`
5. All localStorage keys must use `npcp_` prefix
6. The Save Demographics button must remain OUTSIDE demoFieldsWrapper
7. `ecName` field must appear exactly once (inside demoFieldsWrapper, around char position 62816 in Blank Form)
8. When using Python to patch files, avoid backslashes inside f-string expressions — assign to variables first
---
## Product Roadmap (stated publicly)
- Direct EHR integration via FHIR
- HIPAA-eligible AI (AWS Bedrock path)
- Electronic order transmission
- Mobile/tablet support
- Institutional licensing for nursing schools
---
