# Portfolio Chatbot — Project Context

## What this is
An AI chatbot deployed on Hugging Face Spaces that represents Natalie Hall on her portfolio website. Visitors ask questions about her background; the bot responds in first person, in her voice.

## Tech stack
- Python, OpenAI GPT-4o (`openai` SDK)
- Gradio (UI, deployed on Hugging Face Spaces)
- Gmail SMTP (resume delivery + self-notifications)
- Google Sheets API via `gspread` (conversation logging, unknown question logging)
- `openai-agents` is installed (`from agents import trace` imported but not actively used)

## File structure
```
app.py                  — main application
requirements.txt        — dependencies for Hugging Face deployment
me/
  summary.md            — personal background/bio
  skills.md             — skills list
  company-details.md    — employer context
  portfolio-website-copy.md — portfolio site copy
  ai-projects.md        — AI project descriptions (including this chatbot)
  *.pdf                 — resume (gitignored)
```

## How to run locally
```bash
python app.py
```
Requires `.env` with: `OPENAI_API_KEY`, `GMAIL_USER`, `GMAIL_APP_PASSWORD`, `GOOGLE_CREDENTIALS` (JSON), `GOOGLE_SHEET_ID`

## Key design decisions
- **Tools over prompting**: employment history is handled by `get_employment_history()` in code, not model reasoning — avoids inconsistent counts with overlapping roles
- **Post-processing**: `strip_closing_remarks()` and `fix_list_formatting()` run on every response to enforce consistent output
- **Conversation logging**: every Q&A pair is written to a "All Conversations" tab in Google Sheets
- **Unknown question logging**: model calls `record_unknown_question` tool when it can't answer; logged to Sheet1
- **Resume flow**: visitor provides name + email → `send_resume` emails PDF + notifies Natalie

## Tools
| Tool | Purpose |
|---|---|
| `get_employment_history` | Returns structured employment data filtered by duration/type |
| `send_resume` | Emails resume PDF to visitor, notifies Natalie |
| `record_unknown_question` | Logs unanswered questions to Google Sheets |

## Deployment
Deployed on Hugging Face Spaces. `requirements.txt` must include all runtime dependencies — `gradio` must be present or the app will fail to start.

## Known quirk
`from agents import trace` is imported but `trace` is never called. This is a leftover from course scaffolding. Safe to remove if cleaning up.
