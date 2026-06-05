# GOAL
Build and deploy the Heebee HR System — a full Firebase + GitHub Pages PWA for Heebee Coffee Pvt. Ltd. Phase 1 is complete (index.html delivered). Claude Code must: create the GitHub repo, push the file, enable GitHub Pages, and scaffold Phase 2+ features as Gavish requests them.

# KEY DECISIONS

- **Stack:** Single `index.html` — no npm, no build tools, no Firebase Hosting. Firebase Realtime DB for data only. GitHub Pages for hosting. CDN-only dependencies.
- **Firebase project:** `heebee-hr` (already created by Gavish at console.firebase.google.com)
- **AI resume parsing:** Anthropic API (`claude-opus-4-5`) called directly from browser. API key stored in `localStorage` via Settings panel inside the app — never hardcoded.
- **Auth:** Email + PIN hardcoded in JS. Users: `hr@heebeecoffee.com/1234`, `gavish@heebeecoffee.com/0000`, `nitigya@heebeecoffee.com/1111`
- **Employee ID format:** `HB-{OUTLET}-{###}` e.g. `HB-GHB-001`, `HB-SHB-001`, `HB-JLD-001`. Sequential per outlet, never reused. Counter stored at Firebase `config/last_id_{OUTLET}`.
- **Outlets:** GHB (Sarabha Nagar HQ), SHB (Ghumar Mandi), JLD (Jalandhar Model Town)
- **Payroll uploads:** Two Excel sheets per month — (A) Bank Transfer: Employee Name / Account No. / IFSC / Bank Name / Amount; (B) Govt Salary Sheet: standard Punjab labour format with Basic / HRA / DA / Special Allowance / Gross / Deductions / Net Pay. Both matched to employees by name normalisation.
- **Salary slip PDF:** Generated via jsPDF CDN. Dark header with gold Heebee branding. One tap from employee remuneration history.
- **Design system:** Heebee Luxury UI — Nunito Sans 200/300/400/600 + Space Mono, glass morphism, gold accents (`#b8860b` light / `#f0c040` dark), ambient orbs, dark `#0a0a0f` / light `#f5f5f7`. Light/dark toggle. Non-negotiable.
- **No Firebase Hosting, no npm, no Node** — Gavish deploys via iPhone Safari / GitHub web UI.

# ARTIFACTS (verbatim)

## Firebase config (from screenshot — Gavish to confirm databaseURL)
```json
{
  "apiKey": "AIzaSyBy0H_QWFINvhsU4Io-o0SLauhdVaGvm70",
  "authDomain": "heebee-hr.firebaseapp.com",
  "projectId": "heebee-hr",
  "storageBucket": "heebee-hr.firebasestorage.app",
  "messagingSenderId": "657802787219",
  "appId": "1:657802787219:web:a88262dc8adb6bee039066"
}
```
⚠ `databaseURL` not visible in screenshot — Gavish must add it. Format: `"databaseURL": "https://heebee-hr-default-rtdb.firebaseio.com"`

## Firebase Realtime DB structure
```
/candidates/{pushId}
  name, phone, email, address, current_role, experience_years,
  skills[], education, past_employers[], languages[],
  screening{ hire_recommendation, reasons[], suggested_role, red_flags[] },
  status (Applied|Shortlisted|Hired|Rejected),
  outlet (GHB|SHB|JLD|""),
  date_added (ISO string)

/employees/{HB-OUTLET-###_normalized}
  emp_id, name, phone, email, address,
  outlet, position, joining_date, last_working_day,
  status (Active|Resigned|Terminated),
  ctc, basic, hra,
  aadhaar, pan, account_no, ifsc, bank_name,
  skills[], experience_years, education, past_employers[],
  notes[{ text, ts }],
  remuneration_log/{YYYY-MM}{
    amount, basic, hra, da, special_allowance,
    gross, deductions, net_pay,
    account_no, ifsc, bank_name,
    bank_transfer_uploaded (ISO), govt_sheet_uploaded (ISO)
  },
  created_at (ISO string)

/config/last_id_GHB (integer)
/config/last_id_SHB (integer)
/config/last_id_JLD (integer)
```

## AI Resume Parsing Prompt (embedded in index.html)
```
You are an HR screening AI for Heebee Coffee Pvt. Ltd., a specialty café chain in Punjab, India.
Analyze this resume and respond ONLY with a valid JSON object — no markdown, no preamble.
{
  "name": "Full name",
  "phone": "Mobile number with country code if present, else just digits",
  "email": "Email address",
  "address": "City/area if mentioned",
  "current_role": "Current or last job title",
  "experience_years": "Total years of work experience as a number",
  "skills": ["skill1","skill2","skill3"],
  "education": "Highest qualification",
  "past_employers": ["Company 1","Company 2"],
  "languages": ["Hindi","English","Punjabi"],
  "screening": {
    "hire_recommendation": "Should Hire / Maybe / Pass",
    "reasons": ["Reason 1","Reason 2","Reason 3"],
    "suggested_role": "Best role at a café (Barista, Floor Manager, Cashier, etc.)",
    "red_flags": ["Any concern or empty string if none"]
  }
}
```

## index.html
The full Phase 1 app (1100 lines) was delivered in this session. It is saved as `index.html` in the outputs. Claude Code should read it from disk. Key sections:
- `<style>` — full Heebee design system CSS with light/dark tokens
- `<script type="module">` — Firebase SDK v10.12.0 ESM import, exposes `window.dbSet / dbGet / dbPush / dbUpdate`
- `<script>` — all app logic: auth, panel nav, resume parsing, candidate popup, hire modal, payroll upload, salary slip PDF, employee detail view
- CDN deps: SheetJS `xlsx.full.min.js` v0.18.5, jsPDF `jspdf.umd.min.js` v2.5.1, Firebase ESM from gstatic

## GitHub Pages deployment steps
```
1. Create repo: heebee-hr-system (public)
2. Upload index.html to root
3. Settings → Pages → Branch: main / root → Save
4. Live at: https://heebeecoffee-{hash}.github.io/heebee-hr-system
   (or custom domain if Gavish sets one)
5. Open Settings panel in app → paste Anthropic API key + Firebase config JSON → Save & Connect
```

# OPEN QUESTIONS

- `databaseURL` missing from Firebase config screenshot — must be added before Firebase connects. Gavish to confirm from Firebase console (Realtime Database tab → copy URL).
- Anthropic API key was exposed in chat and must be rotated before use. New key goes in app Settings panel only.
- GitHub repo name — assumed `heebee-hr-system`. Confirm with Gavish.
- Photo upload for employee profiles — not yet built. WhatsApp request button exists but actual photo storage (Firebase Storage) is Phase 2.
- Document uploads (Aadhaar copy, PAN copy, offer letter) — slots shown in UI but upload functionality is Phase 2.

# NEXT STEPS

1. **Claude Code: read `index.html`** from outputs directory — do not regenerate it.
2. **Add `databaseURL`** to Firebase config in the Settings section of the HTML before deploying: `"databaseURL": "https://heebee-hr-default-rtdb.firebaseio.com"` (verify exact URL with Gavish).
3. **Create GitHub repo** `heebee-hr-system`, push `index.html`, enable GitHub Pages.
4. **Gavish: rotate Anthropic API key** at console.anthropic.com → paste new key in app Settings.
5. **Gavish: paste Firebase config** in app Settings → Save & Connect → verify Firebase connects (status shows "◈ Firebase connected").
6. **Test flow:** Upload a sample resume PDF → verify AI extracts details → Save to Directory → Mark as Hired → check Firebase for employee record.
7. **Phase 2 scope (when Gavish is ready):**
   - Firebase Storage for document/photo uploads
   - Attendance & leave tracking panel
   - Reports panel (headcount, payroll summary, attrition)
   - Offer letter PDF generator
   - Multi-user roles (HR read-only vs Owner full access)
   - Dark Bean / performance flag integration with Bean System
