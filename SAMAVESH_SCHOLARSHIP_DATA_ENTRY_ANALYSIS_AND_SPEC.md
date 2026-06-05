# SAMAVESH Scholarship Data Entry 25-26 — Deep Analysis & Wireframe Spec
**Version**: 1.0
**Date**: 04 June 2026
**Prepared for**: Claude CLI → Wireframe Build
**Organization**: SAMAVESH Action For Impact
**Form title (exact)**: SAMAVESH Scholarship Data Entry 25-26
**Academic Year**: 2025–26

---

## PART 1: FORM IDENTITY & CRITICAL CONTEXT

### This Is NOT a Student-Facing Form
This is fundamentally different from the Student Onboarding Form analyzed previously.

| Attribute | Student Onboarding Form | Scholarship Data Entry Form |
|---|---|---|
| Filled by | Student themselves | SAMAVESH Fellow / Staff |
| Language | Bilingual (EN + Marathi) | English only (internal) |
| Purpose | Intake / registration | Operational case management |
| Sensitivity | High (student data collection) | Very High (credentials + financial records) |
| Audience literacy | Variable (student) | High (trained fellow) |
| Frequency | Once per student | Multiple times (status updates) |

### What This Form Does
Fellows (SAMAVESH program staff) use this form to:
1. Log that they have assisted a student with **scholarship application** (on a govt/portal)
2. Log that they have assisted a student with **technical support** (KYC services)
3. Record the student's **portal login credentials** (username + password — entered by fellow)
4. Record the **scholarship scheme applied to**
5. Track the **application status** and **disbursement status**
6. Upload **proof documents** (submitted PDF, benefits proof)
7. Record **exact financial amounts** (installment-wise, student vs institute)

### MAJOR SECURITY FLAG
**Q27 (Applicant Password) and Q45 (Login Password) collect PLAINTEXT PASSWORDS in a Google Form.**
This is a severe data security issue regardless of the operational justification (fellows acting on behalf of students). This must be flagged prominently for the client. In the wireframe, this field should be handled with:
- Password-type masking (show/hide toggle)
- Explicit data handling notice
- Ideally replaced by a secure credential vault in any production implementation

---

## PART 2: FULL QUESTION INVENTORY — DEEP ANALYSIS

### SECTION 0: HEADER

| Element | Content |
|---|---|
| Form Title | SAMAVESH Scholarship Data Entry 25-26 |
| Description | "This form is to be filled by all those who fill out the scholarship application" |
| Footer | Standard Google Forms footer |

**Wire note**: Header description is slightly ambiguous — "all those who fill out the scholarship application" could be read as the student themselves. Recommend clarifying to: "This form is to be filled by **SAMAVESH Fellows** who assist students with scholarship applications."

---

### SECTION 1: FELLOW IDENTIFICATION
**Questions: 1–2**

#### Q1 — Email *(Required)*
- Type: Email input
- Standard Google Forms account email — likely auto-captured from login
- **Wire note**: In production, this should be auto-filled from the logged-in fellow's account. Show as read-only pre-filled field with fellow's name displayed.

#### Q2 — Fellow Name *(Required)*
- Type: Dropdown
- Label: "Fellow Name"
- Options (19 names — appears to be a closed staff list):
  - Akshay K
  - Anil K
  - Sandip K
  - Dhanashree O
  - Vaibhav D
  - Vishwas W
  - Adesh G
  - Tanuja K
  - Shalmon S
  - Balaji C
  - Vijay J
  - Sandhya S
  - Sanchita
  - Vaibhavi
  - Anosh W
  - Rushikesh K
  - Kiran D
  - Samiksha T
  - Shivani T
- **Issue**: Last name is abbreviated to initial only — no disambiguation if two fellows share first name and initial (e.g., two people named "Sandip K" could theoretically exist). Low risk currently but worth flagging.
- **Issue**: Fellow names in dropdown = role-based access not enforced. Any fellow could submit under another fellow's name. In production, this should be auto-populated from authenticated session.
- **Wire note**: Dropdown with 19 names — acceptable as-is. In wireframe, show as searchable dropdown or pre-filled based on email.

---

### SECTION 2: SUBMISSION METADATA
**Question: 3**

#### Q3 — Application Submission Date *(Required)*
- Type: Date picker
- Placeholder hint: "Example: 7 January 2019"
- **Issue**: Date format example (7 January 2019) is a past year — confusing for 25-26 data entry. Should show current year example.
- **Issue**: "Application Submission Date" is ambiguous — does this mean (a) the date the fellow is filling this Google Form, or (b) the date the scholarship application was submitted on the govt portal? These could differ by days/weeks.
- **Recommendation**: Split into two fields: "Date of Portal Submission" and "Date of This Data Entry" (auto-filled). OR rename clearly to one meaning.
- **Wire note**: Auto-fill today's date with ability to override for retrospective entries.

---

### SECTION 3: STUDENT IDENTIFICATION
**Questions: 4–6**

#### Q4 — Students Name *(Required)*
- Type: Short text
- **Issue**: Apostrophe missing — should be "Student's Name"
- **Issue**: No student ID reference. If the same student has multiple entries (different schemes, multiple follow-ups), there's no way to link records without exact name matching — which is fragile (spelling variations, nicknames).
- **Recommendation**: Add a "Student ID" field cross-referencing the onboarding form entry. Or a phone number cross-reference field.
- **Wire note**: Text input, max ~60 chars; helper text: "Enter full name as per Govt ID"

#### Q5 — Gender *(Required)*
- Type: Dropdown
- Options: Male / Female
- **Issue**: Only Binary options — inconsistent with the Student Onboarding Form which includes Non-Binary, Other, Prefer not to say. If a student identifies otherwise on the onboarding form, the fellow has no matching option here.
- **Recommendation**: Add Non-Binary / Prefer not to say to match onboarding form options.
- **Wire note**: Dropdown with 2 options — could be radio buttons for speed.

#### Q6 — Age *(Required)*
- Type: Dropdown
- Options: Below 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35 and above
- **Issue**: "Below 15" jumps directly to 16 — the value 15 is missing. Range goes 15→16 with a gap.
- **Issue**: Range extends to 35 and above — consistent with onboarding form's "35 and above" but the onboarding form went to "30 above." Inconsistency between the two forms. Which is canonical?
- **Wire note**: Dropdown is acceptable for fellow-facing form (faster than typing). Fix the missing 15.

---

### SECTION 4: LOCATION DETAILS
**Questions: 7–12**

#### Q7 — Permanent Address *(Required)*
- Type: Long text / textarea
- Label: "Permanent Address (As per govt documents - address proof)"
- Same as onboarding form — consistent, good.
- **Wire note**: Multi-line textarea, placeholder: "Full address as on Aadhar/ID proof"

#### Q8 — District (permanent address) *(Required)*
- Type: Dropdown
- Full Maharashtra district list (36 districts) + Other
- **Issue**: Uses updated district names (Ahilyanagar, Chhatrapati Sambhaji Nagar) in Q11 (current district) but Q8 uses older names (Ahmednagar, Aurangabad). Inconsistency within the same form:
  - Q8 says: "Ahmednagar (Ahilyanagar)" → acknowledges both
  - Q11 says: "Ahilyanagar" only → drops the legacy name
  - These should be standardized to one format across both fields
- **Wire note**: Searchable dropdown; use consistent naming

#### Q9 — Taluka as per district *(Required)*
- Type: Short text (free text)
- **NEW field not in onboarding form**
- This is an important addition — taluka-level granularity for govt scholarship eligibility
- **Issue**: No dropdown or validation — taluka spelling will vary wildly across entries ("Haveli" vs "Mulshi" vs "Maval" etc.)
- **Recommendation**: Make this a conditional dropdown based on District selection (each district has a fixed taluka list). This is complex to implement but critical for data quality in a scholarship management context.
- **Wire note**: For wireframe, show as text input with helper "Enter taluka/tehsil name" and flag as needing dropdown in production

#### Q10 — Current Address of Residence *(Required)*
- Type: Long text / textarea
- Same as onboarding — consistent.
- **Wire note**: Add "Same as permanent address" checkbox

#### Q11 — District (current district of residence) *(Required)*
- Type: Dropdown
- Same 36 Maharashtra districts + Other
- **Issue**: Name inconsistency with Q8 (as noted above — Ahilyanagar vs Ahmednagar)
- **Wire note**: Same searchable dropdown treatment; "Same as permanent district" checkbox

#### Q12 — State *(Required)*
- Type: Radio
- Options: Maharashtra / Other (free text)
- **Critical observation**: This form is Maharashtra-only in scope (only Maharashtra districts offered) but allows "Other" state. If Other is selected, the district dropdowns in Q8/Q11 are still Maharashtra-only — no conditional logic.
- **Issue**: Compared to the onboarding form (which had Maharashtra, Tamil Nadu, Karnataka, Other), this form simplifies to just Maharashtra/Other — appropriate for the scholarship context (state govt schemes) but should be consistent.
- **Wire note**: Radio with 2 options + text field for Other

---

### SECTION 5: ACADEMIC DETAILS
**Questions: 13–18**

#### Q13 — Marks Obtained in the Previous Year (in %) *(Required)*
- Type: Radio (predefined bands)
- Options:
  - 90% and above
  - 80% – 89%
  - 70% – 79%
  - 60% – 69%
  - 50% – 59%
  - 40% – 49%
  - Below 40%
- **Design Decision**: Unlike the onboarding form (which asked for actual marks/percentage), this form uses percentage BANDS — better for data standardization and scholarship eligibility filtering.
- **Issue**: No option for students who haven't received results yet ("Result Awaited") — relevant for first-year students or those whose results are pending.
- **Wire note**: Radio group; clean 7-option layout. Consider adding "Result Awaited / N/A" for edge cases.

#### Q14 — Grade Currently Enrolled In *(Required)*
- Type: Radio
- Options: 10th, 11th, 12th, Graduation-1st year, Graduation-2nd year, Graduation-3rd year, Graduation-4th year, Graduation-5th year, Masters-1st year, Masters-2nd year, Other (free text)
- **Note**: "5tth Year" typo from onboarding form does NOT appear here — this form correctly says "5th year." Good.
- **Issue**: No Diploma/ITI options (unlike onboarding form). Fellows assisting Diploma/ITI students would have no matching option.
- **Recommendation**: Add Diploma (1st/2nd/3rd year) and ITI to match onboarding form.

#### Q15 — Stream *(Required)*
- Type: Radio
- Options: Engineering & Technology (B.Tech/B.E.), Medical Science, Commerce & Management, Science, Arts/Humanities, Law, Design & Media, Diploma, ITI, Other (free text)
- **Same issue as onboarding form**: Diploma and ITI appear as Stream options even though they should be Grade options. If Grade = Diploma (not available), Stream = Diploma is the only way to capture it — that's the workaround in use.
- **Wire note**: Display as vertical radio list; "Other" with inline text field

#### Q16 — Course Details *(Required)*
- Type: Short text (free text)
- **NEW — not in onboarding form**
- Captures specific course name (e.g., "B.E. Computer Engineering", "B.Com Banking & Finance")
- **Validation needed**: Min 3 chars; max ~100 chars
- **Wire note**: Placeholder: "e.g. B.E. Computer Engineering / B.Com / BA Economics"

#### Q17 — Course Fees *(Required)*
- Type: Short text (free text — numeric)
- **NEW — not in onboarding form**
- Scholarship-critical field — fee amount determines scholarship eligibility and amount
- **Critical Validation**: Must be numeric; must be positive; likely in Indian Rupees
- **Issue**: No currency format guidance — fellows may enter "50000" or "50,000" or "₹50,000" — inconsistent data.
- **Recommendation**: Number input with ₹ prefix; validation: integers only, no commas, no symbols; range: likely ₹0–₹5,00,000
- **Wire note**: ₹ prefix label, numeric input, helper: "Enter annual fee in rupees (numbers only, e.g. 45000)"

#### Q18 — College Name *(Required)*
- Type: Radio with "Other" free text
- Long list of 100+ colleges (Pune/PCMC region primarily)
- Same colleges as onboarding form + additional entries
- **Same critical UX issue as onboarding form**: 100+ radio options is completely unusable. Must be searchable dropdown.
- **Wire note**: Typeahead/searchable dropdown. "Other" with free-text field.

#### Q19 — College Geographical Areas *(Required)*
- Type: Radio
- **NEW — not in onboarding form**
- Options:
  - Pune
  - PCMC (Pimpri-Chinchwad Municipal Corporation)
  - Khed / Rajgurunagar / Chakan
  - Baramati
  - Other (free text)
- **Purpose**: Allows SAMAVESH to segment scholarship data by geographical cluster — operationally useful
- **Issue**: "Khed / Rajgurunagar / Chakan" groups three distinct localities — minor but could cause ambiguity in reporting
- **Wire note**: Radio with 4+Other options; clean and simple

---

### SECTION 6: REFERENCE & CONTACT
**Questions: 20–21**

#### Q20 — Reference (if any) *(NOT required — implicit from label)*
- Type: Radio with "Other" free text
- Options:
  - Astitva
  - Ajeevika
  - Ashta No Kai
  - Panaah
  - WFC (Women's Foundation/Centre?)
  - Mudita Alliance
  - GSP Girls
  - Nari Samata Manch
  - Doorstep Foundation
  - Swaroopwardhini
  - Other (free text)
- **Key difference from onboarding form**: Different referral organization list. Onboarding form had: GSP Girls, Mudita Alliance, Chaitanya, Forbes Marshall Kasarwadi, College, Eaton Employees, Thermax Foundation. This form has a different partner network list.
- **Issue**: These two lists should be reconciled — ideally maintained from a single master list. The data entry form's partner list (10 orgs) appears more comprehensive and current.
- **Wire note**: Radio with "Other" — mark as Optional since label says "if any"; not marked * in original

#### Q21 — Contact No *(Required)*
- Type: Short text
- **Validation**: Indian mobile number — 10 digits, starts 6/7/8/9
- **Wire note**: Phone number input; WhatsApp note from onboarding form not present here — likely intentional for internal form

---

### SECTION 7: SOCIO-ECONOMIC DETAILS
**Questions: 22–24**

#### Q22 — Category *(Required)*
- Type: Radio with "Other" free text
- Options:
  - ST (अनुसूचीत जमाती)
  - SC (अनुसूचीत जाती)
  - VJNT (मुक्त जाती आभट जमाती)
  - OBC (इतर मागासवर्गीय)
  - Minority (अल्पसंख्यांक)
  - SBC (शेष मागासवर्गीय)
  - SEBC (सामाजिक आर्थिक मागासवर्गीय)
  - EWS (Economically Weaker Section)
  - General
  - Other (free text)
- **Key difference from onboarding form**: This form includes "Minority" as a category — not present in the onboarding form. This matters because Minority scholarship schemes are separate from caste-based schemes.
- **Issue**: This field is marked as Required (*) — unlike the onboarding form where caste was Optional with "Prefer not to answer." Appropriate for data entry context (fellow needs this for scheme matching) but confirms this is a staff-operated form, not student-facing.
- **Issue**: Marathi text in original PDF has encoding artifacts (e.g., "मुक्त जाती आ भट जमाती" with extra space). Clean Marathi labels should be verified.
- **Wire note**: Radio with 10 options — display as clean vertical list; "Other" at bottom with text field

#### Q23 — Annual Income *(Required)*
- Type: Radio
- Options:
  - Less than 50 thousand
  - 50K to 1 lakh
  - 1 lakh to 2 lakh
  - 2 lakh to 4 lakh
  - 4 lakh to 8 lakh
  - 8 lakh and above
- **Scholarship relevance**: Income bands directly determine scholarship eligibility. Most government post-matric scholarships require family income < ₹2.5 lakh or < ₹8 lakh (varies by scheme).
- **Issue**: No "Prefer not to answer" option — required field. Again confirms staff-operated context (student has shared this on onboarding form; fellow transcribes here).
- **Wire note**: Radio — show as banded selector, format values with ₹ prefix: "Less than ₹50,000", "₹50,000 – ₹1 lakh" etc.

#### Q24 — Identities / Vulnerability Markers *(Required)*
- Type: Checkbox (tick all that apply)
- Options:
  - Persons with Disabilities (अपंगत्व)
  - Economically Disadvantaged (आर्थिकदृष्ट्या मागास)
  - First Generation Learner (पहिली व्यक्ती जी उच्च शिक्षण घेत आहे)
  - Single Parent (एकल पालक)
  - Orphan (अनाथ)
- **Issue**: No "None of the above" or "N/A" option. If a student has none of these markers, the fellow cannot submit this question without selecting something — or would leave it blank.
- **Issue**: Marathi text in original has encoding artifacts — needs clean verification.
- **Recommendation**: Add "None of the above" as a checkbox option.
- **Wire note**: Checkbox group; "None of the above" added at bottom

---

### SECTION 8: SERVICE TYPE SELECTION — BRANCHING POINT
**Question: 25**

#### Q25 — Type of Service *(Required)*
- Type: Dropdown (Radio)
- Options:
  - **Scholarship Support** → Skip to Q26 (Scholarship section)
  - **Technical Support** → Skip to Q44 (Technical Support section)
- **This is the CRITICAL BRANCH POINT of the entire form**
- The form splits into two completely separate workflows from here:
  - Path A (Q26–Q43): Scholarship application details, credentials, scheme, status, financials
  - Path B (Q44–Q46): Technical support (KYC services only)
- **Wire design implication**: This must be visually clear. The two paths are very different in complexity (Path A = 18 fields; Path B = 3 fields). The branching must be explicit in the wireframe.
- **Wire note**: After selecting service type, the form should visually transition/animate to show only the relevant section. In a paginated design, this drives which next page(s) appear.

---

## PATH A: SCHOLARSHIP SUPPORT (Q26–Q43)

### SECTION 9A: SCHOLARSHIP APPLICATION — STUDENT CREDENTIALS
**Questions: 26–27**

#### Q26 — Applicant User Name *(Required)*
- Type: Short text
- Context: This is the student's username on the govt scholarship portal (MahaDBT or scheme-specific portal)
- **SECURITY CONCERN**: Portal credentials being stored in Google Forms is a serious issue. Fellows enter the student's portal login here.
- **Validation**: Likely alphanumeric; portal-specific format
- **Wire note**: Monospace font styling to distinguish as a credential field; show as read-only suggestion "Enter username exactly as created on the portal"

#### Q27 — Applicant Password *(Required)*
- Type: Short text
- **CRITICAL SECURITY ISSUE**: Plaintext password stored in Google Form response sheet. This represents:
  - (a) Access to student's govt scholarship portal account
  - (b) Potential for unauthorized access/manipulation of scholarship records
  - (c) DPDP Act 2023 compliance concern (India's data protection law)
- **In wireframe**: Show as password-type input (masked, with show/hide toggle)
- **In spec notes for client**: Strongly recommend replacing with a secure credential management approach. At minimum, the Google Sheet storing responses should be access-controlled.
- **Wire note**: Password field with eye icon toggle; explicit security notice below field

---

### SECTION 9B: SCHOLARSHIP SCHEME DETAILS
**Question: 28**

#### Q28 — Scheme Name *(Required)*
- Type: Radio with "Other" free text
- Full list of Maharashtra scholarship schemes (26 schemes):
  1. Rajarshi Chhatrapati Shahu Maharaj Shikshan Shulkh Shishyavrutti Scheme
  2. Government of India Post-Matric Scholarship
  3. Post Matric Scholarship to OBC Students
  4. Post Matric Scholarship to VJNT Students
  5. Post-Matric Tuition Fee and Examination Fee (Freeship)
  6. State Minority Scholarship Part II (DHE)
  7. Dr Babasaheb Ambedkar Swadhar Yojana
  8. PCMC Scholarship
  9. Rajarshri Chhatrapati Shahu Maharaj Merit Scholarship
  10. Rajarshi Chhatrapati Shahu Maharaj Merit Scholarship (11th & 12th, VJNT & SBC)
  11. Post Matric Scholarship to SBC Students
  12. Post Matric Scholarship Scheme (Government Of India)
  13. Rajarshri Chhatrapati Shahu Maharaj Fee Reimbursement Scheme
  14. Rajarshi Chhatrapati Shahu Maharaj Shikshan Shulkh Shishyavrutti Yojna (EBC)
  15. Dr Panjabrao Deshmukh Vastigruh Nirvah Bhatta Yojna (DTE)
  16. Post-Matric Scholarship for persons with disability
  17. Tuition Fees and Examination Fees to OBC Students
  18. BOCW (Building & Other Construction Workers)
  19. Dr. Punjabrao Deshmukh Vasatigruh Nirvah Bhatta Yojna (DHE)
  20. Tuition Fee and Exam Fee For Tribal Students (Freeship)
  21. Post Matric Scholarship to Girls Belonging to OBC taking admission in Professional Courses
  22. Payment of Tuition Fees and Examination Fees to OBC Girls Pursuing Professional Courses
  23. Tuition Fees and Examination Fees to VJNT Students
  24. Other (free text)

- **Issue 1**: Schemes 1 and 9 appear very similar in name — "Rajarshi Chhatrapati Shahu Maharaj Shikshan Shulkh Shishyavrutti Scheme" (Q1) vs "Rajarshri Chhatrapati Shahu Maharaj Merit Scholarship" (Q9) — different spelling of Rajarshi/Rajarshri. Inconsistency.
- **Issue 2**: Schemes 15 and 19 both reference "Dr Panjabrao/Punjabrao Deshmukh Vastigruh/Vasatigruh Nirvah Bhatta Yojna" with different spellings. These may be the same scheme for two different departments (DTE vs DHE) — but the naming inconsistency is confusing. Need client to confirm if these are genuinely distinct schemes.
- **Issue 3**: 26 schemes as radio options — acceptable as radio since fellows need to scan the full list carefully (scheme selection is high-stakes). However, a searchable dropdown would reduce error risk.
- **Issue 4**: Scheme eligibility is determined by category (Q22), income (Q23), grade (Q14), and gender (Q5) — but the form shows ALL schemes regardless of student's profile. A filtered/conditional list would prevent errors (e.g., a General category student shouldn't be selecting an SC/ST scheme).
- **Recommendation for production**: Filter scheme list based on: Category (Q22) + Gender (Q5) + Grade (Q14). For wireframe, show full list but highlight recommended schemes based on profile.
- **Wire note**: Radio list with all 26 schemes; group by category (SC/ST, OBC, VJNT, SBC, EWS, Minority, Disability, General/Merit) with sub-headers for navigation

---

### SECTION 9C: APPLICATION TRACKING
**Questions: 29–34**

#### Q29 — Application ID *(Required)*
- Type: Short text
- The application ID assigned by the scholarship portal
- **Validation**: Format will be scheme/portal-specific (e.g., MahaDBT format)
- **Issue**: No format validation specified — alphanumeric IDs will vary by scheme
- **Wire note**: Monospace font; placeholder "Enter Application ID from portal"

#### Q30 — Total Benefits *(Required)*
- Type: Short text (numeric)
- Label: "Total Benefits"
- Contextual description: "Total Benefits for Student / Total Benefits for Institute"
- **Issue**: One field with a combined label suggesting it may need to capture two values. Should be split into:
  - Total Benefits for Student (in ₹)
  - Total Benefits for Institute (in ₹)
- **Issue**: "Benefits" is ambiguous — does it mean sanctioned amount, disbursed amount, or applied-for amount?
- **Wire note**: Split into two separate numeric fields with ₹ prefix

#### Q31 — Total Benefits for Student *(Required)*
- Type: Short text (numeric)
- Already captures this — but Q30 and Q31 overlap in intent. The form structure here is:
  - Q30: "Total Benefits" (combined label for both Q31 and Q32 context)
  - Q31: "Total Benefits for Student" (actual field)
  - Q32: "Total Benefits for Institute" (actual field)
  - This means Q30 is actually a SECTION HEADER that was implemented as a question field — a Google Forms design error.
- **Wire note**: Remove Q30 as a separate field; use it as a section header/label above Q31 and Q32

#### Q32 — Total Benefits for Institute *(Required)*
- Type: Short text (numeric)
- Separate from student benefits — some schemes pay directly to institute (tuition fee) and some to student (maintenance allowance)
- **Wire note**: Numeric input with ₹ prefix; place Q31 and Q32 side-by-side as a two-column layout

#### Q33 — Application Status *(Required)*
- Type: Radio
- Options:
  - Under Scrutiny
  - Application Approved
  - **Benefits Received → Skip to Q35** (conditional logic!)
  - Re-apply
- **Key logic**: If status = "Benefits Received", skip to Q35 (Proof of Benefits Received). This makes sense — if benefits are already received, Q34 (Upload Submitted PDF) is less relevant OR it's implied to be already on file.
- **Issue**: "Re-apply" as a status option is unusual — it implies the previous application was rejected. Should there be a "Rejected" status before "Re-apply"? The status flow seems to be: Under Scrutiny → Approved → Benefits Received / Rejected → Re-apply
- **Recommendation**: Add "Rejected/Returned" as a status between Approved and Re-apply.
- **Wire note**: Radio group with 4 options; conditional logic: Benefits Received → skip upload field

#### Q34 — Upload Submitted PDF *(Required)*
- Type: File upload
- Accepts: Files submitted (PDF presumably)
- **Context**: The PDF of the scholarship application submitted on the portal
- **Issue**: Skipped if Q33 = Benefits Received (per the Google Forms skip logic)
- **Issue**: File upload in Google Forms stores to Google Drive — access control must be verified
- **Wire note**: File upload button; accepted types: PDF; size limit should be specified (suggest max 10MB)

---

### SECTION 9D: BENEFITS RECEIVED (conditional on Q33 = Benefits Received)
**Question: 35**

#### Q35 — Proof of Benefits Received *(Required)*
- Type: File upload
- **Context**: Screenshot or document showing money received in student's/institute's account
- **Wire note**: File upload; accepted: PDF/JPG/PNG; max 10MB

---

### SECTION 9E: INSTALLMENT TRACKING
**Questions: 36–43**

This section tracks two installments, each split between Student and Institute components.
Total fields: 8 (2 installments × 2 recipients × 2 attributes [amount + status])

#### Q36 — First Installment (Student) (in numbers) *(Required)*
- Type: Short text (numeric — amount in ₹)
- **Wire note**: Numeric input with ₹ prefix

#### Q37 — First Installment (Student) Received *(Required)*
- Type: Radio
- Options: Funds Disbursed / Fund Disbursement in Process / Awaiting Approval / Other
- **Issue**: "Other" as a disbursement status is vague — what other state would exist? Consider adding "Rejected/Returned" explicitly.
- **Wire note**: Radio group

#### Q38 — First Installment (Institute) (in numbers) *(Required)*
- Type: Short text (numeric — amount in ₹)

#### Q39 — First Installment (Institute) (Received) *(Required)*
- Type: Radio
- Same options as Q37

#### Q40 — Second Installment (Student) (in numbers) *(Required)*
- Type: Short text (numeric — amount in ₹)

#### Q41 — Second Installment (Student) (Received) *(Required)*
- Type: Radio
- Same options as Q37

#### Q42 — Second Installment (Institute) (in numbers) *(Required)*
- Type: Short text (numeric — amount in ₹)

#### Q43 — Second Installment (Institute) (Received) *(Required)*
- Type: Radio
- Same options as Q37

**Structural Design Issue for Q36–Q43**:
These 8 fields are presented sequentially but belong to a 2×2×2 matrix:

```
              | First Installment | Second Installment |
Student       | Amount + Status   | Amount + Status    |
Institute     | Amount + Status   | Amount + Status    |
```

In the wireframe, this should be presented as a **table/grid**, not 8 separate sequential fields. This reduces cognitive load and makes it far easier for fellows to spot missing or inconsistent entries.

**Additional Issue**: All installment fields are marked Required (*), but what if a scholarship only has one installment? Or if the second installment hasn't been processed yet? Fellows would be forced to enter 0 or leave fields in unclear states. Consider making second installment fields conditional (appear only if first installment status = "Funds Disbursed").

---

## PATH B: TECHNICAL SUPPORT (Q44–Q46)

### SECTION 10B: TECHNICAL SUPPORT DETAILS
**Questions: 44–46**

Section header: "Technical Support — This includes services such as linkages to various KYC services"

#### Q44 — Login ID *(Required)*
- Type: Short text
- Context: Student's login ID for the KYC service being assisted with (Aadhar, UIDAI, bank portal, etc.)
- **SECURITY CONCERN**: Same as Q26 — storing portal credentials in Google Forms
- **Wire note**: Same treatment as Q26 — monospace font, credential field styling

#### Q45 — Login Password *(Required)*
- Type: Short text
- **CRITICAL SECURITY ISSUE**: Same as Q27. Plaintext password for KYC services.
- **Wire note**: Password input with show/hide toggle; same security notice

#### Q46 — Select the below mentioned technical support provided *(Required)*
- Type: Checkbox (tick all that apply)
- Options:
  - Aadhar Seeding
  - Bank Account Linkage
  - N/A
  - Bank Account Update
  - Other (free text)
- **Issue**: "N/A" as a checkbox option alongside service options is contradictory — if N/A, why is the fellow filling out this section at all? N/A here likely means the service was discussed but not completed (initiated but pending). The label should clarify this.
- **Issue**: The service list is very short (3 actual services) — is this the complete scope of technical support, or is it an incomplete list? Other potential services: DigiLocker setup, MahaDBT registration, Scholarship portal registration, eKYC update, mobile number seeding.
- **Wire note**: Checkbox group; "N/A" visually separated; "Other" at bottom with text input

---

## PART 3: BUGS & ERRORS FOUND IN ORIGINAL FORM

| # | Location | Issue | Severity | Recommendation |
|---|---|---|---|---|
| 1 | Q3 (Date) | "Submission Date" ambiguous — form-filling date or portal submission date? | High | Split into two fields or rename clearly |
| 2 | Q4 (Student Name) | No student ID cross-reference to onboarding form | High | Add phone/ID cross-reference field |
| 3 | Q5 (Gender) | Only Male/Female — inconsistent with onboarding form | Medium | Add Non-Binary/Prefer not to say |
| 4 | Q6 (Age) | Age 15 missing — jumps from "Below 15" to 16 | Medium | Add 15 as option |
| 5 | Q8/Q11 (District) | Inconsistent naming convention (Ahmednagar vs Ahilyanagar) within same form | High | Standardize to one format |
| 6 | Q9 (Taluka) | Free text — will produce inconsistent data | High | Add district-conditional dropdown in production |
| 7 | Q17 (Course Fees) | No currency format guidance — entries will be inconsistent | High | Number input with ₹ prefix + format guidance |
| 8 | Q24 (Identities) | No "None of the above" option for checkbox | Medium | Add "None of the above" |
| 9 | Q27/Q45 (Passwords) | Plaintext passwords stored in Google Form | Critical | Security alert; mask field; advise client |
| 10 | Q28 (Scheme) | "Rajarshi" vs "Rajarshri" spelling inconsistency across schemes | Medium | Standardize spelling |
| 11 | Q28 (Scheme) | Two Punjabrao Deshmukh schemes (DTE vs DHE) — unclear if same or different | High | Confirm with client |
| 12 | Q30 (Total Benefits) | Used as section header implemented as a question field | Low | Convert to section label |
| 13 | Q33 (App Status) | Missing "Rejected/Returned" status | Medium | Add before "Re-apply" |
| 14 | Q36–Q43 (Installments) | All 8 installment fields required — no way to record N/A for single-installment schemes | Medium | Make second installment conditional |
| 15 | Q46 (Tech Support) | "N/A" checkbox alongside service options — contradictory | Medium | Rename to "Not yet completed" or remove |
| 16 | Header (Description) | Ambiguous "all those who fill the scholarship application" — student or fellow? | Low | Clarify to "SAMAVESH Fellows" |
| 17 | Q2 (Fellow Name) | Fellow can select any other fellow's name — no auth enforcement | Medium | Note for production: auto-fill from session |

---

## PART 4: FORM STRUCTURE ANALYSIS — TWO-PATH ARCHITECTURE

```
┌─────────────────────────────────────────────────────────┐
│                    COMMON SECTION                        │
│  Q1: Email (fellow)                                      │
│  Q2: Fellow Name                                         │
│  Q3: Application Submission Date                         │
│  Q4: Student Name                                        │
│  Q5: Gender                                              │
│  Q6: Age                                                 │
│  Q7: Permanent Address                                   │
│  Q8: District (permanent)                                │
│  Q9: Taluka                                              │
│  Q10: Current Address                                    │
│  Q11: District (current)                                 │
│  Q12: State                                              │
│  Q13: Marks (previous year %)                            │
│  Q14: Grade Currently Enrolled                           │
│  Q15: Stream                                             │
│  Q16: Course Details                                     │
│  Q17: Course Fees                                        │
│  Q18: College Name                                       │
│  Q19: College Geographical Area                          │
│  Q20: Reference                                          │
│  Q21: Contact No                                         │
│  Q22: Category (caste)                                   │
│  Q23: Annual Income                                      │
│  Q24: Vulnerability Identities                           │
│  Q25: ← TYPE OF SERVICE BRANCH POINT →                  │
└────────────────┬──────────────────┬─────────────────────┘
                 │                  │
    ┌────────────▼──────┐  ┌────────▼────────────┐
    │  PATH A:          │  │  PATH B:             │
    │  SCHOLARSHIP      │  │  TECHNICAL SUPPORT   │
    │  SUPPORT          │  │                      │
    │                   │  │  Q44: Login ID        │
    │  Q26: Username    │  │  Q45: Login Password  │
    │  Q27: Password    │  │  Q46: Services        │
    │  Q28: Scheme      │  │                      │
    │  Q29: App ID      │  │  [END]               │
    │  Q30: Benefits    │  └──────────────────────┘
    │  Q31: Stu Amount  │
    │  Q32: Inst Amount │
    │  Q33: App Status  │
    │  Q34: Upload PDF  │
    │  Q35: Proof PDF   │
    │  Q36–43: Install. │
    │                   │
    │  [END]            │
    └───────────────────┘
```

---

## PART 5: RECOMMENDED FORM STRUCTURE (PAGINATED WIREFRAME)

### Path A: Scholarship Support — 6 pages

```
Page 1: Fellow & Submission Info
  - Q1: Fellow Email (auto-filled)
  - Q2: Fellow Name (dropdown)
  - Q3: Dates — Portal submission date + Data entry date (auto)
  - [Continue]

Page 2: Student Profile
  - Q4: Student Name
  - Q5: Gender
  - Q6: Age
  - Q21: Contact No
  - Q22: Category
  - Q23: Annual Income
  - Q24: Vulnerability Identities

Page 3: Location
  - Q7: Permanent Address
  - Q8: District (permanent, searchable dropdown)
  - Q9: Taluka
  - Q10: Current Address [+ Same as permanent checkbox]
  - Q11: District (current)
  - Q12: State
  - Q20: Reference

Page 4: Academic & College
  - Q13: Previous Year Marks %
  - Q14: Grade Currently Enrolled
  - Q15: Stream
  - Q16: Course Details
  - Q17: Course Fees (₹)
  - Q18: College Name (searchable dropdown)
  - Q19: College Geographical Area

Page 5: Service Type → Branch
  - Q25: Type of Service (Scholarship / Technical)
  - [Selecting drives which page appears next]

Page 6A: Scholarship Details (if Path A)
  - Q26: Portal Username
  - Q27: Portal Password (masked)
  - Q28: Scheme Name
  - Q29: Application ID
  - Q30–Q32: Total Benefits (Student ₹ + Institute ₹)
  - Q33: Application Status
  - Q34: Upload Submitted PDF

Page 7A: Financial Tracking (if status ≠ Under Scrutiny)
  - Benefits received proof upload (Q35) [if status = Benefits Received]
  - Installment table:
    | | First Installment | Second Installment |
    | Student | Amount + Status | Amount + Status |
    | Institute | Amount + Status | Amount + Status |

Page 6B: Technical Support Details (if Path B)
  - Q44: Login ID
  - Q45: Login Password (masked)
  - Q46: Services provided (checkboxes)
  - [Submit]
```

---

## PART 6: VALIDATION RULES MASTER TABLE

| Field | Type | Required | Validation Rule |
|---|---|---|---|
| Email (Q1) | Email | Yes | Valid email; auto-filled from session |
| Fellow Name (Q2) | Dropdown | Yes | Must select from list of 19 |
| Submission Date (Q3) | Date | Yes | Auto-fill today; allow past dates up to 6 months |
| Student Name (Q4) | Text | Yes | Min 3 chars, letters + spaces only |
| Gender (Q5) | Dropdown | Yes | One selection |
| Age (Q6) | Dropdown | Yes | Select from 15–35+ range |
| Permanent Address (Q7) | Textarea | Yes | Min 10 chars |
| District Permanent (Q8) | Dropdown | Yes | Valid Maharashtra district or Other |
| Taluka (Q9) | Text | Yes | Min 3 chars |
| Current Address (Q10) | Textarea | Yes | Min 10 chars |
| District Current (Q11) | Dropdown | Yes | Valid selection |
| State (Q12) | Radio | Yes | One selection |
| Marks % (Q13) | Radio | Yes | One band selected |
| Grade (Q14) | Radio | Yes | One selection |
| Stream (Q15) | Radio | Yes | One selection |
| Course Details (Q16) | Text | Yes | Min 3, max 100 chars |
| Course Fees (Q17) | Number | Yes | Positive integer; ₹ prefix; 0–500000 |
| College Name (Q18) | Dropdown | Yes | One selection or Other with text |
| College Area (Q19) | Radio | Yes | One selection |
| Reference (Q20) | Radio | No | Optional |
| Contact No (Q21) | Phone | Yes | 10 digits, starts 6/7/8/9 |
| Category (Q22) | Radio | Yes | One selection |
| Annual Income (Q23) | Radio | Yes | One band selected |
| Identities (Q24) | Checkbox | Yes | At least 1 OR "None of the above" |
| Service Type (Q25) | Dropdown | Yes | Scholarship or Technical |
| Portal Username (Q26) | Text | Yes (Path A) | Non-empty; min 3 chars |
| Portal Password (Q27) | Password | Yes (Path A) | Non-empty; masked display |
| Scheme Name (Q28) | Radio | Yes (Path A) | One selection |
| Application ID (Q29) | Text | Yes (Path A) | Non-empty; alphanumeric |
| Benefits Student (Q31) | Number | Yes (Path A) | Positive integer or 0 |
| Benefits Institute (Q32) | Number | Yes (Path A) | Positive integer or 0 |
| App Status (Q33) | Radio | Yes (Path A) | One selection |
| Upload PDF (Q34) | File | Conditional | Required unless Q33 = Benefits Received |
| Benefits Proof (Q35) | File | Conditional | Required if Q33 = Benefits Received |
| Installment amounts (Q36,38,40,42) | Number | Yes (Path A) | Positive integer or 0; ₹ prefix |
| Installment status (Q37,39,41,43) | Radio | Yes (Path A) | One selection per field |
| Login ID (Q44) | Text | Yes (Path B) | Non-empty |
| Login Password (Q45) | Password | Yes (Path B) | Non-empty; masked |
| Tech Services (Q46) | Checkbox | Yes (Path B) | At least 1 selected |

---

## PART 7: CONDITIONAL LOGIC MAP

```
Q25 (Service Type) = Scholarship Support
  → Show Q26–Q43 (Scholarship path)
  → Hide Q44–Q46 (Technical path)

Q25 (Service Type) = Technical Support
  → Show Q44–Q46 (Technical path)
  → Hide Q26–Q43 (Scholarship path)

Q33 (Application Status) = Benefits Received
  → Skip Q34 (Upload Submitted PDF)
  → Show Q35 (Proof of Benefits Received)

Q33 (Application Status) ≠ Benefits Received
  → Show Q34 (Upload Submitted PDF)
  → Hide Q35 (Proof of Benefits Received)

Q36 (First Installment - Student Amount) > 0 AND Q37 (Status) = Funds Disbursed
  → Enable Q40/Q41 (Second Installment - Student)
  [Recommended conditional - not in original form]

Q38 (First Installment - Institute Amount) > 0 AND Q39 (Status) = Funds Disbursed
  → Enable Q42/Q43 (Second Installment - Institute)
  [Recommended conditional - not in original form]

Q22 (Category) → Recommended scheme filter on Q28
  SC/ST → highlight SC/ST schemes
  OBC → highlight OBC schemes
  VJNT → highlight VJNT schemes
  etc.
  [Recommended feature - not in original form]
```

---

## PART 8: WIREFRAME DESIGN DIRECTIVES FOR CLAUDE CLI

### Visual Style
- **Audience**: Internal staff (SAMAVESH Fellows) — not students. Design can assume higher digital literacy.
- **Aesthetic**: Professional, efficient, data-dense — utility-first with clear information hierarchy. Think internal ops tool, not public-facing portal.
- **Palette**:
  - Primary: SAMAVESH brand color (assume deep teal/navy consistent with onboarding form)
  - Status indicators: Green (Disbursed), Amber (In Process), Red (Awaiting/Rejected), Blue (Submitted)
  - Password fields: Distinct amber/warning treatment to signal sensitivity
  - Section headers: Clean dividers, not heavy boxes
- **Typography**: Readable sans-serif; tabular data in monospace; credential fields in monospace
- **Layout**: Multi-page paginated form; progress indicator at top; denser layout than student form (fellows are trained users)

### Key Design Components

**1. Path Selector (Q25)**
```
[ SCHOLARSHIP SUPPORT ]  [ TECHNICAL SUPPORT ]
   Large tab-style toggle, not a dropdown
   Selected state clearly highlighted
   Switching clears the relevant section
```

**2. Credential Fields (Q26/Q27, Q44/Q45)**
```
⚠ Security Notice: This data is stored securely
[ Username _____________________ ]
[ Password ████████████████ 👁 ]   ← masked with show/hide toggle
```

**3. Installment Table (Q36–Q43)**
```
┌─────────────────────────────────────────────────────────────┐
│  INSTALLMENT TRACKING                                         │
├──────────────┬────────────────────────┬─────────────────────┤
│              │  FIRST INSTALLMENT      │  SECOND INSTALLMENT │
├──────────────┼────────────────────────┼─────────────────────┤
│  STUDENT     │  ₹ [_______]  [status▼]│  ₹ [_____] [status▼]│
├──────────────┼────────────────────────┼─────────────────────┤
│  INSTITUTE   │  ₹ [_______]  [status▼]│  ₹ [_____] [status▼]│
└──────────────┴────────────────────────┴─────────────────────┘
Status options: ● Disbursed  ◐ In Process  ○ Awaiting  - Other
```

**4. Scheme Selection (Q28)**
```
Show 26 schemes grouped by category with sub-headers:
▸ SC/ST Schemes (3)
▸ OBC Schemes (4)
▸ VJNT Schemes (2)
▸ SBC Schemes (1)
▸ EWS Schemes (1)
▸ Disability Schemes (1)
▸ Merit/General Schemes (4)
▸ Minority Schemes (1)
▸ Other (1)
```

**5. Status Indicators**
- Application Status pill: Under Scrutiny (grey) → Approved (blue) → Benefits Received (green) → Re-apply (amber)
- Disbursement Status: Funds Disbursed (green) / In Process (amber) / Awaiting (orange) / Other (grey)

### Progress Bar
```
Step X of 6 — [Section Name]
Fellow Info → Student Profile → Location → Academic → Service Type → [Path A/B Details]
```

### Mobile Considerations
- Installment table: stack to vertical on mobile (Student first, then Institute)
- Password fields: full-width with toggle prominent
- Scheme list: searchable dropdown on mobile even if desktop shows radio

---

## PART 9: CROSS-FORM COMPARISON — KEY DIFFERENCES

| Field | Student Onboarding Form | Scholarship Data Entry Form |
|---|---|---|
| Filled by | Student | SAMAVESH Fellow |
| Language | Bilingual EN + Marathi | English only |
| Age range top | 30 and above | 35 and above |
| Age range - 15 | Present | Missing (bug) |
| Gender options | 5 options | 2 options (inconsistency) |
| Caste field | Optional | Required |
| Income field | Optional | Required |
| District list naming | Ahmednagar (Ahilyanagar) | Inconsistent (Q8 vs Q11) |
| College question | Radio (100+ options) | Radio (100+ options) — same bug |
| Reference/Partner orgs | Different list | Different list — needs reconciliation |
| Marks format | Free text | Percentage bands (better) |
| Taluka | Not captured | Captured (good addition) |
| Course name | Not captured | Captured (good addition) |
| Course fees | Not captured | Captured (scholarship-critical) |
| Passwords | Not present | Present (security concern) |
| File uploads | Not present | Present (PDFs, proofs) |
| Financial tracking | Not present | Present (installments) |

---

## PART 10: OPEN QUESTIONS FOR CLIENT

1. **Security**: Are fellows authorized to record student portal passwords? What access controls are in place for the Google Sheet storing form responses? Recommend replacing with a credential vault.
2. **Date ambiguity (Q3)**: Does "Application Submission Date" mean the date the portal application was submitted, or the date this data entry form is being filled?
3. **Student cross-reference**: How do data entry records link back to student onboarding records? Is there a student ID or is name+phone the key?
4. **Scheme duplicates**: Are "Rajarshi Chhatrapati Shahu Maharaj Shikshan Shulkh Shishyavrutti Scheme" (Q28 item 1) and "Rajarshi Chhatrapati Shahu Maharaj Shikshan Shulkh Shishyavrutti Yojna (EBC)" (Q28 item 14) the same scheme or different? Same question for the two Punjabrao Deshmukh Vasatigruh Nirvah Bhatta entries.
5. **Single-installment schemes**: If a scholarship scheme pays in a single installment (not two), how should fellows handle the second installment fields? Should they be conditionally hidden?
6. **Q30 (Total Benefits)**: Is this the sanctioned amount, the disbursed amount, or the applied-for amount? Clarify the definition.
7. **Fellow authentication**: Is there a plan to tie form submission to fellow login, or will name selection remain on the honor system?
8. **Technical support scope (Q46)**: Is the list (Aadhar Seeding, Bank Account Linkage, Bank Account Update) complete? Are DigiLocker setup, MahaDBT registration, or eKYC updates also in scope?
9. **College list reconciliation**: The college list in Q18 appears to be the same as the student onboarding form. Is this list maintained centrally? Who updates it when new colleges are added?
10. **Partner organization reconciliation (Q20)**: The referral organizations in this form differ from those in the onboarding form. Which list is canonical? Should they be the same?

---

## PART 11: CLAUDE CLI PROMPT BLOCK

```
TASK: Build an HTML wireframe for the SAMAVESH Scholarship Data Entry Form 25-26.

CONTEXT:
- Organization: SAMAVESH Action For Impact (NGO, Pune)
- Form type: INTERNAL STAFF/FELLOW-FACING data entry form
- Filled by: SAMAVESH Fellows (trained staff), NOT students
- Language: English only (no Marathi required for this form)
- Purpose: Scholarship case management — logging portal actions, tracking status, recording financials
- Academic Year: 2025-26
- Output: Multi-page HTML wireframe for client presentation

CRITICAL ARCHITECTURE:
- The form has ONE BRANCH POINT at Q25 (Type of Service)
- PATH A (Scholarship Support): 18 additional fields including credentials, scheme, financials
- PATH B (Technical Support): 3 additional fields only
- Wire must show BOTH paths clearly — either as tabbed view or as explicit branch after Q25

PAGES TO BUILD (PATH A):
1. Fellow & Submission Info (Fellow email [auto], Fellow name dropdown, Submission date)
2. Student Profile (Name, Gender, Age, Contact, Category/Caste, Annual Income, Vulnerability Identities)
3. Location (Permanent address + district, Taluka, Current address + district [+same-as checkbox], State, Reference organization)
4. Academic & College (Previous marks %, Current grade, Stream, Course details, Course fees, College name [searchable], College geographical area)
5. Service Type Branch (large tab toggle: Scholarship / Technical Support)
6A. Scholarship Details (Portal username + password [masked], Scheme selection [grouped by category], Application ID, Total benefits student ₹ + institute ₹, Application status, PDF upload)
7A. Financial Tracking (Benefits proof upload [conditional], Installment GRID TABLE: 2×2 with amount + status per cell)

PAGES TO BUILD (PATH B):
6B. Technical Support (Login ID, Login Password [masked], Services provided checkboxes)

DESIGN DIRECTION:
- Aesthetic: Professional, utility-first, internal ops tool — not public-facing
- Palette: Deep teal/navy primary; status color coding (green=disbursed, amber=in process, red=rejected)
- Layout: Denser than student form — fellows are trained users; can handle more per page
- Password fields: Amber/warning styling; eye icon toggle; security notice
- Installment section: MUST be a 2×2 table/grid, NOT 8 sequential fields
- Scheme list: Grouped by category (SC/ST, OBC, VJNT, SBC, EWS, Disability, Merit, Minority) with sub-headers
- Status indicators: Colored pill/badge system for application and disbursement status

CRITICAL RULES:
- Password fields (Q27, Q45): Masked input with show/hide toggle; explicit "⚠ Security Notice" banner
- College field (Q18): Searchable dropdown with typeahead — NOT radio list
- Q25 branch must be visually unambiguous — tab-style toggle or full-page decision card
- Installment table must show Student/Institute rows × First/Second columns with Amount + Status per cell
- Required fields marked with *
- Progress bar: "Step X of 6 — [Section Name]"
- File upload fields: drag-drop zone + file type/size guidance
- Application status must show as workflow progression (pill-style stages)

BUGS TO FIX vs ORIGINAL GOOGLE FORM:
- Age 15 missing from dropdown (fix: add 15)
- District naming inconsistency Q8 (Ahmednagar) vs Q11 (Ahilyanagar) — standardize to official current name (Ahilyanagar) in Q8
- Q30 "Total Benefits" is a section label not a question — render as header above Q31/Q32
- Q24 (Identities) — add "None of the above" checkbox option
- Scheme names: standardize "Rajarshi" vs "Rajarshri" spelling
- Q33 Application Status — add "Rejected/Returned" status before "Re-apply"
- Q46 — rename N/A to "Not yet completed / Pending"
- All installment amount fields: add ₹ prefix and numeric-only validation
```

---

*End of Analysis Document*
*Total Questions Analyzed: 46 (Q1–Q46, including branch paths)*
*Bugs Identified: 17*
*Critical Security Issues: 2 (Q27 and Q45 — plaintext passwords)*
*Open Client Questions: 10*
*Recommended Page Count: 6–7 pages (vs original: 1 scroll, branched)*
*Form Architecture: Linear common section → single branch point → two divergent paths*
