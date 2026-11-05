# Submission Guide & Form Checklist

**Candidate:** Prince Kumar  
**Target Deadline:** 5th November 2026 (11:59 pm)  
**Submission Form:** https://docs.google.com/forms/d/e/1FAIpQLSeBHYcRhWEuffCSoJJC5NiuToutR_ElQE4jvPDjueafRVG8gg/viewform  

---

## 1. AI Usage Disclosure (Required by Form)

*Use this clean, honest statement in the AI disclosure field:*

> "I used Google Gemini as a pair-programming assistant to research Swytchcode's architecture and troubleshoot Windows-specific CLI behavior (specifically diagnosing the Node.js `readAllStdin` hang on non-TTY stdio). I directly tested and executed all commands in PowerShell, validated the Wrekenfile schemas, verified the demo executions, and wrote the project documentation and evaluation notes."

---

## 2. Google Form Field Mapping

| Form Question | Content to Paste / Reference |
| :--- | :--- |
| **GitHub Repository Link** | `https://github.com/Prince-creator-developer/swytchcode-ppi-submission` |
| **Demo Video Link** | Paste your unlisted YouTube, Loom, or Google Drive link (recorded using `MARKETING_AND_DEMO_SCRIPT.md`). |
| **Swytchcode in your own words (150–250 words)** | Copy text from `EXPLANATION.md`. |
| **Friction Points & Onboarding Feedback** | Copy the 5 points from `FRICTION_DIARY.md` (specifically mentioning the Windows CLI hang, PostHog timeout, and `init` flags). |
| **Marketing Asset** | `https://www.reddit.com/user/No-Cranberry9977/comments/1xojlt2/why_i_stopped_giving_ai_agents_raw_api_keys/` (and text from `MARKETING_AND_DEMO_SCRIPT.md`) |
| **AI Disclosure** | Copy the disclosure statement in Section 1 above. |

---

## 3. Git Push Verification

To make sure your latest files are on GitHub:
```powershell
cd "c:\Users\Prince kumar\OneDrive\Desktop\dsih\swytchcode-ppi-submission"
git add .
git commit -m "Update project documentation, friction diary, and demo assets"
git push origin main
```
