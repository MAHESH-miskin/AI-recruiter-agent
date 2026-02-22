# Lovable AI — Recruit-AI Frontend Prompt

## PASTE THIS ENTIRE PROMPT INTO LOVABLE:

---

Build a modern, professional AI Resume Screening web application called **"Recruit-AI"**. This is a single-page React app built with TypeScript, Tailwind CSS, and shadcn/ui components.

---

## DESIGN & BRAND

- Color palette: Deep navy `#1F4E79` as primary, sky blue `#2E75B6` as accent, white background, light gray `#F8FAFC` for card backgrounds
- Font: Inter (Google Fonts)
- Style: Clean SaaS product — think Linear or Notion. No clutter, generous whitespace, smooth transitions
- Fully responsive (mobile + desktop)

---

## PAGE STRUCTURE

### 1. NAVBAR
- Left: Logo — a small briefcase icon (lucide-react `Briefcase`) + bold text "Recruit**AI**" where "AI" is in the accent blue color
- Right: A subtle badge pill that says "Powered by Gemini AI" with a small sparkle icon
- Sticky top, white background, subtle bottom border shadow

---

### 2. HERO SECTION
- Centered layout
- Small label badge above heading: `✦ AI-Powered Hiring` in accent blue with light blue background pill
- Large heading (3xl bold): "Screen Resumes in **Seconds**, Not Days"
- Subheading (gray, md): "Upload a resume and job description. Our AI agent scores the candidate, evaluates fit, and triggers interview scheduling — automatically."
- Three feature pills below in a row: `⚡ Instant ATS Scoring` · `📊 Detailed Evaluation` · `📅 Auto Interview Scheduling`

---

### 3. MAIN FORM CARD
A white card with rounded-2xl, subtle shadow, max-width 720px, centered on the page.

Card header:
- Title: "Submit a Candidate Application"
- Subtitle: "Fill in the candidate details and upload their resume PDF"

**Form fields (in order):**

**Field 1 — Full Name**
- Label: "Candidate Full Name"
- Input: text, placeholder "e.g. Priya Sharma", required
- Name attribute: `fullName`

**Field 2 — Email Address**
- Label: "Candidate Email"
- Input: email, placeholder "e.g. priya@example.com", required
- Name attribute: `email`

**Field 3 — Job Description**
- Label: "Job Description"
- Textarea: 5 rows, placeholder: "Paste the full job description here. Include required skills, experience level, and responsibilities...", required
- Name attribute: `jobDescription`
- Character count shown bottom-right in gray (live counter)

**Field 4 — Resume Upload**
- Label: "Upload Resume (PDF only)"
- Custom drag-and-drop upload zone:
  - Dashed border, rounded-xl, light blue tint background on hover
  - Center icon: `FileText` from lucide-react
  - Text: "Drag & drop resume PDF here, or click to browse"
  - Sub-text: "Only PDF files accepted · Max 10MB"
  - When a file is selected: show a green success state with filename, file size, and an × remove button
  - Accept only `.pdf` files
  - Name attribute: `data` (CRITICAL — the backend reads binary from field named "data")

**Submit Button:**
- Full width, navy background, white text
- Text: "Analyze Resume with AI →"
- Loading state: show a spinning loader icon + "Analyzing... This may take 20–30 seconds"
- Disabled while loading
- Icon: `Sparkles` from lucide-react when not loading

---

### 4. RESULTS SECTION

Only shown after a successful API response. Animate in with a smooth fade-up transition.

**If `atsScore >= 70` (Accepted — `success: true` and message contains "accepted"):**

Show a card with green left border accent:

- Top banner: Green pill badge "✓ Candidate Accepted" 
- Large centered ATS score display:
  - Text "ATS Match Score" above
  - Giant bold number showing the score (e.g. "82") in green
  - "/100" in smaller gray text beside it
  - Animated circular progress ring around the score (use a CSS conic-gradient circle, green color)
- Score interpretation bar: a horizontal progress bar filled to the score percentage, color green
- Section: "AI Evaluation" — show `evaluation` text from API in a light gray blockquote-style box
- Section: "What Happens Next" — show an info box:
  "🎉 Great news! A calendar invite for an interview has been automatically sent to the candidate's email. Check your Google Calendar for the scheduled meeting."

**If `atsScore < 70` (Rejected):**

Show a card with orange/red left border accent:

- Top banner: Orange pill badge "✗ Not Shortlisted"
- Large centered ATS score display same as above but in orange color
- Score interpretation bar in orange
- Section: "AI Evaluation" — show `evaluation` text
- Section: "What Happens Next" — show:
  "A polite rejection email has been automatically sent to the candidate thanking them for their application."

**Always show at the bottom of results:**
- A gray "Screen Another Candidate →" button that resets the form and scrolls back up

---

### 5. HOW IT WORKS SECTION (below the form, always visible)

Three-step horizontal cards (stacked on mobile):

**Step 1** — Icon: `Upload` — Title: "Upload Resume" — Text: "Submit candidate details and their PDF resume through the form above."

**Step 2** — Icon: `Brain` — Title: "AI Analyzes" — Text: "Our Gemini AI agent parses the resume, compares it against the job description, and calculates an ATS match score."

**Step 3** — Icon: `CalendarCheck` — Title: "Auto Action" — Text: "Candidates scoring 70+ receive an interview invite. Others get a thoughtful rejection email. All automatically."

---

### 6. FOOTER
- Centered, gray text
- "© 2026 Recruit-AI · Built with n8n + Gemini AI · Made for smart hiring teams"

---

## API INTEGRATION — CRITICAL IMPLEMENTATION DETAILS

The form must submit to this exact URL:
```
https://maheshjoicy.app.n8n.cloud/webhook/ai-recruiter
```

**Method:** `POST`  
**Content-Type:** `multipart/form-data` — use `FormData`, DO NOT set Content-Type header manually (let the browser set it with the boundary automatically)

**EXACT FormData fields the backend requires:**
```
fullName      → string  (candidate's full name)
email         → string  (candidate's email)
jobDescription → string (full job description text)
data          → File    (the PDF resume binary — field name MUST be "data")
```

**Implementation:**
```typescript
const handleSubmit = async () => {
  setLoading(true);
  setResult(null);
  setError(null);

  const formData = new FormData();
  formData.append("fullName", fullName);
  formData.append("email", email);
  formData.append("jobDescription", jobDescription);
  formData.append("data", resumeFile); // field name "data" is required by backend

  try {
    const response = await fetch(
      "https://maheshjoicy.app.n8n.cloud/webhook/ai-recruiter",
      {
        method: "POST",
        body: formData,
        // DO NOT set Content-Type header — browser sets it automatically with boundary
      }
    );

    if (!response.ok) {
      throw new Error(`Server error: ${response.status}`);
    }

    const data = await response.json();
    setResult(data);
  } catch (err: any) {
    setError(err.message || "Something went wrong. Please try again.");
  } finally {
    setLoading(false);
  }
};
```

**Expected API response shape:**
```typescript
interface APIResponse {
  success: boolean;
  message: string;       // e.g. "Application accepted! Interview scheduled."
  atsScore: number;      // e.g. 82 (0–100)
  evaluation: string;    // e.g. "Strong match on React and Node.js skills..."
}
```

---

## STATE MANAGEMENT

Use React `useState` hooks for:
- `fullName: string`
- `email: string`
- `jobDescription: string`
- `resumeFile: File | null`
- `loading: boolean`
- `result: APIResponse | null`
- `error: string | null`

---

## ERROR HANDLING

Show a red error toast/alert if:
- API call fails (network error, timeout, non-200 status)
- File is not PDF (validate `file.type === 'application/pdf'` on selection)
- Any required field is empty on submit

Error alert style: red background, white text, dismissable × button

---

## VALIDATION

Before submitting, validate:
1. `fullName` is not empty
2. `email` matches basic email format
3. `jobDescription` is at least 50 characters
4. `resumeFile` is not null and is a PDF

Show inline field-level error messages in red below each field if validation fails.

---

## ANIMATIONS & UX

- Form fields: smooth focus ring transition in accent blue
- Submit button: hover scale-105 transition
- Results card: animate in with `opacity-0 translate-y-4` → `opacity-100 translate-y-0` over 400ms
- ATS score number: count up animation from 0 to the actual score over 1.5 seconds using `setInterval`
- Progress bar: animate width from 0 to score% over 1 second
- Loading state: pulsing skeleton placeholder where the results card will appear

---

## IMPORTANT NOTES FOR LOVABLE

- Use `lucide-react` for all icons
- Use `shadcn/ui` components: Button, Input, Textarea, Card, Badge, Progress, Alert
- Do NOT use any backend or server-side code — this is a pure frontend that calls the external n8n webhook
- Single file or minimal component structure is fine
- The app must work when deployed on Lovable's preview URL without any CORS issues (the n8n webhook accepts cross-origin requests)
