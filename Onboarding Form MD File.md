# SAMAVESH Student Onboarding Form — Deep Analysis & Wireframe Spec
**Version**: 1.0  
**Date**: 04 June 2026  
**Prepared for**: Claude CLI → Wireframe Build  
**Organization**: SAMAVESH Action For Impact  
**Form URL**: Google Forms (migration target: custom implementation)

---

## PART 1: FORM OVERVIEW & CONTEXT

### What This Form Does
The SAMAVESH Student Onboarding Form collects intake data for students seeking:
1. **Scholarship Support** (Government/Private)
2. **Documentation Support** (Legal Documents)

The organization (SAMAVESH Action For Impact) uses this data to:
- Understand socio-economic and academic background
- Identify document gaps and provide support
- Connect students with relevant programs
- Authorize SAMAVESH staff to act on scholarship portals on student's behalf

### Audience
- Students aged ~15–30+, primarily from Maharashtra
- Bilingual: English + Marathi (both must appear in every label)
- May have low digital literacy — form must be simple and reassuring
- Often from disadvantaged backgrounds — sensitive questions handled with opt-out options

### Form Language Pattern
Every question has:
- English label (primary)
- Marathi translation (secondary, in parentheses or below)
- This must be preserved in the wireframe

---

## PART 2: FULL QUESTION INVENTORY WITH DEEP ANALYSIS

### SECTION 0: HEADER / INTRO
**Not numbered as a question but is the landing state**

| Element | Content |
|---|---|
| Form Title | Student Onboarding Form |
| Subtitle (EN) | "Welcome to the Student Onboarding Form!" |
| Subtitle (MR) | विद्यार्थी नोंदणी फॉर्म आपले स्वागत आहे! |
| Body (EN) | Explains purpose — personalized support, confidentiality, timeline |
| Body (MR) | Full Marathi translation of the above |
| Footer note (EN) | "Once you submit this form, we will reach out to you via email..." |
| Footer note (MR) | Same in Marathi |

**Issue Flagged**: The intro page does NOT have a "Next" button — in Google Forms it scrolls into Q1. In the wireframe, this should be a dedicated welcome screen (Step 0) with a CTA button: "Start Filling / सुरू करा".

---

### SECTION 1: BASIC IDENTIFICATION
**Questions: 1–3**

#### Q1 — Email *(Required)*
- Type: Email input
- Validation: Valid email format
- No Marathi label visible in source (likely omitted in PDF render)
- **Wire note**: Email keyboard on mobile; placeholder "your@email.com"

#### Q2 — Date of Form Submission *(Required)*
- Type: Date picker
- Placeholder hint: "Example: 7 January 2019" → format DD Month YYYY
- **Issue**: Should this be auto-populated to today's date? High likelihood of user error if left blank. **Recommended: Auto-fill current date, allow override.**
- Marathi label: फॉर्म भरण्याची तारीख

#### Q3 — Full Name as per Govt Document *(Required)*
- Type: Short text
- Label: Full Name (as per govt document)
- Marathi: पूर्ण नाव (शासकीय कागदपत्रानुसार)
- Validation: Non-empty, letters + spaces only, min 3 chars
- **Wire note**: Prominent "as per govt document" helper text to set expectation

---

### SECTION 2: SUPPORT TYPE SELECTION
**Question: From Q3 area (unnumbered in PDF, but functionally Q3b)**

Actually this is **Q (unnumbered, appears after date) — "Please select the support that you require from us"**

Re-numbering for spec clarity as **Q3b**:

#### Q3b — Support Required *(Required, Multi-select)*
- Type: Checkbox (tick all that apply)
- Options:
  - Scholarship Support (Government/Private) — शिष्यवृत्ती सहाय्य (शासकीय/खाजगी)
  - Documentation Support (Legal Documents) — कायदेशीर कागदपत्रे सहाय्य
- **Logic**: This answer could conditionally influence later sections (scholarship-specific vs doc-specific questions). In current form it does not branch — all students see all questions. **Flag for client: Should this gate any sections?**
- **Wire note**: Both options may be selected simultaneously

---

### SECTION 3: PERSONAL DETAILS
**Questions: 4–12**

#### Q4 — Age *(Required)*
- Type: Dropdown
- Options: Below 15, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 30 above
- Marathi label: वय
- **Issue**: Dropdown for a small numeric range is poor UX. **Recommended: Number input with min=13, max=40** or a segmented selector. Keep dropdown if matching exactly to existing data structure.
- Validation: Required, within range

#### Q5 — Contact Number *(Required)*
- Type: Phone number input
- Label: Contact Number (Preferably WhatsApp Number)
- Marathi: संपर्क क्रमांक (आवडीनुसार व्हॉट्सअ‍ॅप नंबर)
- Helper text (long, both EN/MR): Explains WHY they're collecting it
- Validation: Indian mobile number — 10 digits, starts with 6/7/8/9
- **Wire note**: Helper text should be collapsible (show/hide) to reduce visual load

#### Q6 — Gender *(Required)*
- Type: Radio (mark only one)
- Options:
  - Male (पुरुष)
  - Female (स्त्री) — note: PDF shows just "Female(" with Marathi missing, raw PDF encoding issue
  - Non-Binary (नॉन-बायनरी)
  - Binary (बायनरी)
  - Prefer not to say (सांगू इच्छित नाही)
  - Other (free text)
- **Issue**: "Binary" as a standalone option alongside "Non-Binary" is unusual and potentially confusing. Standard practice is Male/Female/Non-Binary/Other/Prefer not to say. **Flag for client**: Was "Binary" intentional? Possibly a translation/addition error.
- Marathi label: लिंग

#### Q7 — Permanent Address *(Required)*
- Type: Long text / textarea
- Label: Permanent Address (As per govt documents - address proof)
- Marathi: कायमचा पत्ता (शासकीय कागदपत्रांनुसार)
- **Wire note**: Multi-line textarea; helper text "as per govt documents" shown inline

#### Q8 — Current Address of Residence *(Required)*
- Type: Long text / textarea
- Label: Current address of residence
- Marathi: सध्याच्या राहण्याचा पत्ता
- **Logic opportunity**: Add checkbox "Same as permanent address" to auto-fill. **Strongly recommended.**

#### Q9 — State *(Required)*
- Type: Radio (mark only one)
- Options:
  - Maharashtra (महाराष्ट्र)
  - Tamil Nadu (तमिळनाडू)
  - Karnataka (कर्नाटक) — **BUG: PDF shows "Karnataka (तळनाडू)" which is wrong Marathi for Karnataka. Marathi for Karnataka is कर्नाटक.**
  - Other (free text)
- **Critical Issue**: The Marathi for Karnataka reads "तळनाडू" which is Tamil Nadu's Marathi name. This is a data entry error in the original form. Must be corrected to कर्नाटक.
- **Wire note**: State selection currently drives district dropdown — conditional logic required

#### Q10 — District (Permanent Address) *(Required)*
- Type: Radio (mark only one)
- Label: District (as per permanent address on Govt ID proof)
- Marathi: शासकीय ओळखपत्रावर नमूद कायम पत्यानुसार
- Options: Full list of Maharashtra districts (36 districts listed):
  Ahmednagar (Ahilyanagar), Akola, Amravati, Aurangabad (Chhatrapati Sambhaji Nagar), Beed, Bhandara, Buldhana, Chandrapur, Dhule, Gadchiroli, Gondia, Hingoli, Jalgaon, Jalna, Kolhapur, Latur, Mumbai City, Mumbai Suburban, Nagpur, Nanded, Nandurbar, Nashik, Osmanabad (Dharashiv), Palghar, Parbhani, Pune, Raigad, Ratnagiri, Sangli, Satara, Sindhudurg, Solapur, Thane, Wardha, Washim, Yavatmal + Other
- **Critical Logic Issue**: This is currently a FLAT radio list of 36+ items regardless of State selected. If State = Tamil Nadu or Karnataka, Maharashtra districts are irrelevant. **The district list MUST be conditional on State. For non-Maharashtra states, show a free-text field. For Maharashtra, show the dropdown.**
- **Wire note**: Convert from radio to searchable dropdown for UX; too many options for radio

#### Q11 — District (Current Residence) *(Required)*
- Type: Radio (mark only one)  
- Label: District (current district of residence)
- Marathi: सध्या राहण्याच्या पत्यानुसार
- Options: Same Maharashtra district list + "Other"
- **Same issue as Q10**: Must be conditional on State
- **Logic opportunity**: Add "Same as permanent district" checkbox
- **Wire note**: Same treatment as Q10 — searchable dropdown

---

### SECTION 4: COLLEGE / INSTITUTION
**Question: 12 (Q13 in raw PDF)**

#### Q12 — College Name *(Required)*
- Type: Radio (mark only one) with "Other" free text
- 60+ college options listed (all Pune/Maharashtra region)
- **Critical UX Issue**: 60+ radio options is completely unusable on mobile. Must be a searchable dropdown with "Other (specify)" option.
- **Wire note**: Typeahead/search input. Show full list only after 2+ characters typed.

---

### SECTION 5: REFERENCE
**Question: 13 (Q14 in raw PDF)**

#### Q13 — How did you get to know about us? *(Required)*
- Type: Radio (mark only one) with "Other" free text
- Options:
  - GSP Girls
  - Mudita Alliance
  - Chaitanya
  - Forbes Marshall Kasarwadi
  - College
  - Eaton Employees
  - Thermax Foundation
  - Other (free text)
- **Wire note**: These are donor/partner organizations — effectively a referral source. Label should be clearer: "How did you hear about SAMAVESH?" No Marathi label visible in PDF — likely missing from original. **Flag for client.**

---

### SECTION 6: DOCUMENT CHECKLIST
**Header**: "Documents (Eligibility & Applicability)" — with explanatory text: "This is for us to understand what all documents are applicable to you and what are the missing documents that you require support to avail."

**Questions: 14–22 (Q15–Q23 in raw PDF)**

All are Required radio Yes/No (or Yes/No/Not Applicable):

| Q# | Document | Type | Options | Notes |
|---|---|---|---|---|
| Q14 | Aadhar Card | Radio | Yes / No | Required |
| Q15 | Caste Certificate | Radio | Yes / No / Not Applicable | Required |
| Q16 | Domicile Certificate | Radio | Yes / No | Required |
| Q17 | Annual Income Certificate (FY 25/26) | Radio | Yes / No | Required |
| Q18 | Non-Creamy Layer Certificate | Radio | Yes / No / Not Applicable | Required |
| Q19 | Ration Card | Radio | Yes / No | Required |
| Q20 | 10th School Leaving Certificate | Radio | Yes / No | Required |
| Q21 | 12th School Leaving Certificate | Radio | Yes / No | Required |
| Q22 | Orphan Certificate | Radio | Yes / No / Not Applicable | Required |

**Wireframe Design Note**: These 9 questions form a natural **grid/matrix checklist** — much cleaner than 9 separate radio groups. A table-style layout with columns (Yes / No / N/A) and rows per document would reduce cognitive load significantly. **Strongly recommended for wireframe.**

**Logic Issues**:
- Non-Creamy Layer Certificate (Q18) is only relevant if Caste Certificate exists and caste is OBC/VJNT/SEBC/SBC — ideally conditional on Q15 = Yes AND social background = OBC/VJNT/etc.
- Orphan Certificate (Q22) is only relevant if Q29 (orphan identity) is selected — but Q29 comes AFTER Q22 in current order. Sequence issue.
- 12th certificate (Q21) should conditionally appear only if current grade ≥ 12th — currently unconditional.

---

### SECTION 7: EDUCATIONAL BACKGROUND
**Header**: "Educational Background" — with explanatory paragraph (EN + implied MR)

**Questions: 23–27 (Q24–Q28 in raw PDF)**

#### Q23 — Current Grade of Study *(Required)*
- Type: Radio
- Marathi: सध्याच्या शिक्षणाची इयत्ता/स्तर
- Options: 10th, 11th, 12th, Graduation 1st–5th Year, Masters 1st–2nd Year, Diploma, ITI, Other
- **Wire note**: "Graduation - 5tth Year" is a typo in original (extra 't'). Must be corrected to "5th Year"

#### Q24 — Stream *(Required)*
- Type: Radio
- Options: Arts/Humanities, Commerce & Management, Science, Engineering & Technology (B.Tech/B.E.), Medical Science, Law, Design & Media, Diploma, ITI, Other
- **Logic Issue**: "Diploma" and "ITI" appear both as grade options (Q23) AND as stream options (Q24). If grade = Diploma, stream selection is ambiguous. **Flag for client.**

#### Q25 — Current Marks Obtained *(Required)*
- Type: Short text / Number or percentage
- No options listed — free text
- **Validation needed**: Percentage (0–100) or CGPA or marks/total. Format not specified in original.
- **Recommended**: Add helper text: "Enter as percentage (e.g. 72%) or CGPA (e.g. 7.8)"

#### Q26 — Previous Grade of Study *(Required)*
- Type: Radio
- Options: Same list as Q23
- **Logic**: Should be one level below Q23 automatically. If Q23 = 11th, Q26 should suggest/default to 10th.

#### Q27 — Marks Obtained in Previous Grade *(Required)*
- Type: Short text / Number
- Same validation recommendation as Q25

---

### SECTION 8: SOCIO-ECONOMIC DETAILS
**Header**: "Socio-economic details" — with a long, sensitive explanatory paragraph explaining WHY they're asking and reassuring confidentiality. Both EN and MR.

**Questions: 28–37 (Q29–Q38 in raw PDF)**

#### Q28 — Social Background *(Optional — explicit opt-out)*
- Type: Radio with "Prefer not to answer" as first option
- Options: Prefer not to answer, SC, ST, OBC, VJNT, SEBC, SBC, EWS, General, Other (free text)
- Explanatory note: Individuals from challenged backgrounds given preference
- **Wire note**: "Prefer not to answer" should be visually separated (e.g., lighter styling, dashed border) from the caste category options

#### Q29 — Identities / Vulnerability Markers *(Optional — explicit opt-out)*
- Type: Checkbox (tick all that apply)
- Options:
  - Persons with Disabilities (अपंगत्व)
  - Economically Disadvantaged (आर्थिकदृष्ट्या मागास)
  - First Generation Learner (पहिली व्यक्ती जी उच्च शिक्षण घेत आहे)
  - Single Parent (एकल पालक)
  - Orphan (अनाथ)
  - Prefer not to answer
  - Other (free text)
- **Logic**: If "Orphan" selected here → Orphan Certificate (Q22) becomes relevant. Sequence issue (Q22 comes before Q29). **Recommend reordering: move Document section after Socio-Economic section, OR make Orphan Certificate conditional retroactively.**

#### Q30 — Economic Background / Annual Family Income *(Optional)*
- Type: Radio with "Prefer not to answer"
- Options: Prefer not to answer, Less than 50 thousand, 50k–1 lakh, 1–2 lakh, 2–4 lakh, 4–8 lakh, 8 lakh and above, Other
- Marathi brackets for key ranges implied
- **Wire note**: Income bands in Rupees — format as ₹ for clarity

#### Q31 — Single Parent Child? *(Required)*
- Type: Radio
- Options: Yes / No / Other
- **Issue**: "Other" is an unusual option for a Yes/No factual question. Likely an error in the original form. Should be Yes/No only.

#### Q32 — Parents Have Any Disability? *(Required)*
- Type: Radio
- Options: Yes / No / Maybe / Other
- **Issue**: "Maybe" and "Other" are unusual for a factual disability question. Could indicate unclear family situation. Acceptable but should have helper text.

#### Q33 — Family Occupation / Parents' Income Source *(Required)*
- Type: Radio
- Options: Farming (शेती), Daily Wages (मजुरी), Service (नोकरी), Business (व्यवसाय), Seasonal work in required field, Other
- **Issue**: "Seasonal work in required field" — unclear phrasing. Likely means "Seasonal agricultural/daily work." **Flag for client to clarify wording.**

#### Q34 — Type of House *(Required)*
- Type: Checkbox (tick all that apply)
- Options: Own house, Rented house, Kaccha house (कच्चे घर), Pakka house (पक्के घर)
- **Logic Issue**: "Own house" + "Kaccha house" or "Own house" + "Pakka house" are valid combinations (owned but kaccha). But "Rented" + "Own" together is contradictory. Some mutual exclusivity logic may be needed.
- **Recommended**: Split into two questions — (a) Ownership: Own/Rented, (b) Construction type: Kaccha/Pakka/Semi-pucca

#### Q35 — Drug Addiction Problem at Home? *(Required)*
- Type: Radio
- Options: Yes / No / Maybe / Prefer not to answer
- **Sensitive question** — must have reassuring helper text. Marathi label missing from PDF (encoding issue).

#### Q36 — WiFi/Internet Connection at Home? *(Required)*
- Type: Radio
- Options: Yes / No

#### Q37 — Computer or Laptop at Home? *(Required)*
- Type: Radio
- Options: Yes / No

---

### SECTION 9: ASPIRATIONS
**Header**: "Tell us a little bit more about yourself!" — with explanatory paragraph

**Questions: 38–40 (Q39–Q41 in raw PDF)**

All three are **Optional** free-text questions:

#### Q38 — Immediate Goals *(Optional)*
- Type: Long text / textarea
- Placeholder suggestion: "E.g. Pass my 12th exams, apply for scholarship..."

#### Q39 — Long-term Career Aspirations *(Optional)*
- Type: Long text / textarea
- Placeholder suggestion: "E.g. Become a doctor, start my own business..."

#### Q40 — Anything Else to Share *(Optional)*
- Type: Long text / textarea
- Placeholder suggestion: "Any other information you'd like us to know..."

---

### SECTION 10: CONSENT DECLARATION
**Question: 41 (Q42 in raw PDF)**

#### Q41 — Consent / Digital Signature *(Required)*
- Type: Short text — student types their name as consent
- Label: Consent Declaration
- English text (full legal consent for SAMAVESH to collect data and access scholarship portals)
- Marathi text (full translation of above)
- Instruction: "Type your name below to give consent"
- Marathi instruction: सहमती देण्यासाठी खाली तुमचे नाव टाका
- **Validation**: Non-empty; ideally matches Q3 (Full Name) — add soft match validation
- **Wire note**: This should be a dedicated final screen with consent text prominently displayed before the input field. "Submit / सबमिट करा" button only appears after name is typed.

---

## PART 3: BUGS & ERRORS FOUND IN ORIGINAL FORM

| # | Location | Issue | Severity | Recommendation |
|---|---|---|---|---|
| 1 | Q9 (State) | Karnataka's Marathi label reads "तळनाडू" (Tamil Nadu's name) | High | Fix to कर्नाटक |
| 2 | Q10/Q11 (District) | Maharashtra district list shown regardless of state selected | High | Make district list conditional on state |
| 3 | Q23 (Grade) | "Graduation - 5tth Year" typo | Low | Fix to "5th Year" |
| 4 | Q24 (Stream) | Diploma and ITI appear as both grade AND stream options | Medium | Clarify with client |
| 5 | Q31 (Single Parent) | "Other" option on a Yes/No question | Low | Remove or replace with "Prefer not to say" |
| 6 | Q13 (Reference) | No Marathi label | Medium | Add Marathi: तुम्हाला आमच्याबद्दल कसे समजले? |
| 7 | Q33 (Occupation) | "Seasonal work in required field" — unclear phrasing | Medium | Clarify to "Seasonal/Contract Work" |
| 8 | Q22 (Orphan Cert) | Appears before Q29 (Orphan identity) — ordering issue | Medium | Reorder or add conditional logic |
| 9 | Q6 (Gender) | "Binary" as standalone option is non-standard | Medium | Flag for client — likely unintentional |
| 10 | Q12 (College) | 60+ radio options — completely unusable | High | Convert to searchable dropdown |
| 11 | Q2 (Date) | Manual date entry for submission date — error-prone | Medium | Auto-fill today's date |
| 12 | Q8 (Current address) | No "Same as permanent" shortcut | Low | Add checkbox to auto-copy |

---

## PART 4: RECOMMENDED FORM STRUCTURE (REDESIGNED SEQUENCE)

The current Google Form is a single scroll. For the wireframe, break into **logical pages/steps** with a progress indicator:

```
Step 1 of 7 — Welcome & Support Type
Step 2 of 7 — Personal Details  
Step 3 of 7 — Location
Step 4 of 7 — Education
Step 5 of 7 — Documents
Step 6 of 7 — Socio-Economic Background
Step 7 of 7 — Aspirations & Consent
```

### Recommended Page Breakdown

**Page 1: Welcome + Support Type**
- Welcome screen (intro text EN/MR)
- Q1: Email
- Q3b: Support required (Scholarship / Documentation / Both)
- CTA: "Continue / पुढे जा"

**Page 2: Personal Details**
- Q3: Full Name
- Q2: Date (auto-filled)
- Q4: Age
- Q5: Contact Number (WhatsApp)
- Q6: Gender

**Page 3: Location**
- Q9: State
- Q7: Permanent Address (textarea)
- Q10: District (permanent) — conditional on state
- Q8: Current Address (with "Same as permanent" checkbox)
- Q11: District (current) — conditional on state

**Page 4: College & Reference**
- Q12: College Name (searchable dropdown)
- Q13: How did you hear about us?

**Page 5: Educational Background**
- Q23: Current Grade
- Q24: Stream
- Q25: Current Marks
- Q26: Previous Grade
- Q27: Previous Marks

**Page 6: Documents**
- Section header with explanation
- Q14–Q22: Document checklist (matrix/grid layout)

**Page 7: Socio-Economic Background**
- Section header with sensitivity notice
- Q28: Social Background (caste category) — optional
- Q29: Vulnerability Markers — optional
- Q30: Annual Family Income — optional
- Q31: Single Parent Child?
- Q32: Parents' Disability?
- Q33: Family Occupation
- Q34: House Type
- Q35: Drug Addiction at Home?
- Q36: Internet at Home?
- Q37: Computer/Laptop at Home?

**Page 8: Aspirations + Consent**
- Q38: Immediate Goals
- Q39: Long-term Aspirations
- Q40: Anything else
- Q41: Consent (full text EN/MR + name input)
- Final submit button

---

## PART 5: VALIDATION RULES MASTER TABLE

| Field | Type | Required | Validation Rule |
|---|---|---|---|
| Email (Q1) | Email | Yes | Valid email format, @domain.tld |
| Date (Q2) | Date | Yes | Auto-fill today; allow past dates |
| Full Name (Q3) | Text | Yes | Min 3 chars, letters + spaces only |
| Support Type (Q3b) | Checkbox | Yes | At least 1 selected |
| Age (Q4) | Dropdown/Number | Yes | 13–45 range |
| Contact (Q5) | Phone | Yes | 10 digits, starts 6/7/8/9 |
| Gender (Q6) | Radio | Yes | One option selected |
| Permanent Address (Q7) | Textarea | Yes | Min 10 chars |
| Current Address (Q8) | Textarea | Yes | Min 10 chars OR "Same as permanent" |
| State (Q9) | Radio | Yes | One selection |
| District Permanent (Q10) | Dropdown | Yes | Valid selection |
| District Current (Q11) | Dropdown | Yes | Valid selection |
| College (Q12) | Dropdown | Yes | One selection or "Other" with text |
| Reference (Q13) | Radio | Yes | One selection |
| All Document Qs (Q14–Q22) | Radio | Yes | One option per row |
| Current Grade (Q23) | Radio | Yes | One selection |
| Stream (Q24) | Radio | Yes | One selection |
| Current Marks (Q25) | Text | Yes | Numeric/percentage; 0–100 or 0–10 CGPA |
| Previous Grade (Q26) | Radio | Yes | One selection |
| Previous Marks (Q27) | Text | Yes | Numeric/percentage |
| Social Background (Q28) | Radio | No | Optional; no minimum |
| Vulnerability Markers (Q29) | Checkbox | No | Optional; no minimum |
| Family Income (Q30) | Radio | No | Optional; no minimum |
| Single Parent Child (Q31) | Radio | Yes | One selection |
| Parents Disability (Q32) | Radio | Yes | One selection |
| Family Occupation (Q33) | Radio | Yes | One selection |
| House Type (Q34) | Checkbox | Yes | At least 1 selected |
| Drug Addiction (Q35) | Radio | Yes | One selection |
| Internet at Home (Q36) | Radio | Yes | One selection |
| Computer at Home (Q37) | Radio | Yes | One selection |
| Immediate Goals (Q38) | Textarea | No | Optional |
| Career Aspirations (Q39) | Textarea | No | Optional |
| Other Info (Q40) | Textarea | No | Optional |
| Consent Name (Q41) | Text | Yes | Non-empty; soft match to Q3 |

---

## PART 6: CONDITIONAL LOGIC MAP

```
Q9 (State) = Maharashtra
  → Q10 (District Permanent): Show Maharashtra dropdown (36 options)
  → Q11 (District Current): Show Maharashtra dropdown

Q9 (State) = Tamil Nadu / Karnataka / Other
  → Q10: Show free-text input ("Enter your district")
  → Q11: Show free-text input

Q8 (Current Address): "Same as permanent address" checked
  → Auto-fill Q8 from Q7
  → Q11: Auto-fill from Q10

Q15 (Caste Certificate) = Yes
  → Q18 (Non-Creamy Layer Certificate): Show (else hide)

Q23 (Current Grade) ≥ 12th (i.e., 12th, Graduation, Masters)
  → Q21 (12th Leaving Certificate): Show (else mark N/A)

Q23 (Current Grade) = Graduation or Masters
  → Q20 (10th Certificate): Remains relevant
  → Q21 (12th Certificate): Remains relevant

Q29 (Vulnerability) includes "Orphan"
  → Q22 (Orphan Certificate): Flag as relevant (retroactive note)

Q41 (Consent Name): typed
  → Enable "Submit" button (previously greyed out)
```

---

## PART 7: WIREFRAME DESIGN DIRECTIVES FOR CLAUDE CLI

### Visual Style
- **Aesthetic**: Clean, editorial, high-trust — not clinical or bureaucratic
- **Palette**: 
  - Primary: Deep teal/navy (#1A3A4A or similar) 
  - Accent: Warm saffron/amber (echoing Indian identity, ≠ garish)
  - Background: Off-white (#F9F7F4)
  - Text: Near-black (#1C1C1C)
  - Optional elements: Muted grey (#8A8A8A)
- **Typography**: 
  - Heading: A humanist serif or strong geometric sans (NOT Inter/Roboto)
  - Body/Labels: Clean readable sans
  - Marathi text: Must render in Devanagari — ensure Google Font supports it (e.g., Noto Sans Devanagari)
- **Layout**: Card-per-page, not scroll-all. Step progress bar at top.
- **Tone**: Warm, welcoming, not sterile government form

### Progress Indicator
- Horizontal step bar with step number + section name
- "Step 2 of 8 — Personal Details | वैयक्तिक माहिती"
- Completed steps shown with a checkmark/fill

### Bilingual Label Pattern
```
[English Label]*
[मराठी लेबल]
[helper text if any]
[input field]
```

### Document Checklist Matrix (Page 6)
```
Document Name          | Have it (आहे) | Don't have (नाही) | N/A
-----------------------|---------------|-------------------|----
Aadhar Card            |   ○           |     ○             |  —
Caste Certificate      |   ○           |     ○             |  ○
...
```

### Mobile-first considerations
- All inputs must be tappable (min 44px touch targets)
- Dropdowns over radio for long lists (districts, colleges)
- Consent screen: text readable at 14px minimum
- Progress bar: always visible at top (sticky)

### Sensitive Section Treatment (Page 7)
- Softer background color for socio-economic section
- Explicit "This section is optional / हा विभाग ऐच्छिक आहे" banner at top
- "Prefer not to answer / सांगू इच्छित नाही" as first visible option, clearly separated

### Navigation
- "Back / मागे जा" + "Next / पुढे जा" on every page
- Final page: "Submit / सबमिट करा" (disabled until consent name typed)
- No "Submit" button on intermediate pages

---

## PART 8: OPEN QUESTIONS FOR CLIENT

1. **Binary gender option**: Was "Binary" as a standalone gender option intentional alongside "Non-Binary"?
2. **Support type gating**: Should selecting only "Documentation Support" skip scholarship-specific document questions?
3. **Diploma/ITI duplication**: Should stream = Diploma be treated differently from grade = Diploma?
4. **Orphan Certificate sequence**: Should documents section move after socio-economic section to allow conditional logic on orphan identity?
5. **Reference "College" option**: Does this mean "through their college" or is this a partner organization named College? Clarify.
6. **Seasonal work phrasing (Q33)**: Confirm intended meaning of "Seasonal work in required field."
7. **Marks format**: What format should marks/percentage be captured in — raw marks, percentage, or CGPA? Are all three acceptable?
8. **Drug addiction question (Q35)**: Is "Maybe" a valid response? Should this question exist in a form accessible to minors?
9. **College list scope**: Is this list final? It appears Pune-centric — will students from Tamil Nadu/Karnataka need additions?
10. **Auto-date**: Should submission date be auto-populated? If not, what happens if a student enters a future date?

---

## PART 9: CLAUDE CLI PROMPT BLOCK

Use the following as the briefing for Claude CLI when building the wireframe:

```
TASK: Build an HTML wireframe for the SAMAVESH Student Onboarding Form.

CONTEXT:
- Organization: SAMAVESH Action For Impact (NGO, Pune)
- Purpose: Student intake form for scholarship and documentation support
- Audience: Students aged 15–30, primarily Maharashtra, bilingual EN+Marathi
- Output: Multi-page HTML wireframe (8 steps/pages) for client presentation

DESIGN DIRECTION:
- Aesthetic: Clean, editorial, warm — NOT sterile or bureaucratic
- Colors: Deep teal primary, saffron accent, off-white background
- Typography: Noto Sans Devanagari for Marathi support; humanist display font for headings
- Layout: Card-per-page with sticky top progress bar
- Mobile-first (380px min viewport)

PAGES TO BUILD:
1. Welcome + Support Type (Email, Support selection)
2. Personal Details (Name, Age, Phone, Gender)
3. Location (State → conditional district, Permanent + Current address)
4. College & Reference (searchable dropdown for college, referral source)
5. Education (Grade, Stream, Marks - current and previous)
6. Documents (matrix checklist grid for 9 documents)
7. Socio-Economic (optional section with caste, vulnerability, income, family details)
8. Aspirations + Consent (open text fields + typed consent with submit)

CRITICAL RULES:
- Every label must appear in BOTH English and Marathi (Devanagari script)
- Progress bar at top: "Step X of 8 — [Section Name] | [मराठी नाव]"
- Conditional logic markers: show/grey-out sections based on answers
- Document checklist must use matrix/grid layout (not 9 separate radio groups)
- College field must be searchable dropdown (not radio list)
- District dropdown conditional on State selection
- Consent page: full consent text EN + Marathi before name input field
- Submit button disabled until consent name is entered
- Sensitive questions (caste, disability, income, drug addiction) must show helper context and opt-out option first

VALIDATION INDICATORS TO SHOW IN WIREFRAME:
- Required fields marked with *
- Error state (red border + message) on invalid input
- Success state (green checkmark) on valid input
- Helper text below complex fields

BUGS TO FIX vs ORIGINAL GOOGLE FORM:
- Karnataka Marathi label corrected to कर्नाटक (original had तळनाडू)
- "5tth Year" typo corrected to "5th Year"
- Submission date auto-filled
- "Same as permanent address" checkbox on current address
```

---

*End of Analysis Document*
*Total Questions Analyzed: 42 (including intro/header)*
*Bugs Identified: 12*
*Open Client Questions: 10*
*Recommended Page Count: 8 (vs original: 1 scroll page)*
