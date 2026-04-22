# AI Projects

## Portfolio Website Chatbot

### Overview
This chatbot is a deployed AI agent that lives on my portfolio website and answers questions as me — in my voice, with accurate information about my background, experience, and skills. Visitors can have a real conversation about my career, and the bot can email my resume on request. It started as a course exercise and evolved significantly through iteration and problem-solving.

### Tech Stack
- **Language:** Python
- **LLM:** OpenAI GPT-4o (via OpenAI API)
- **UI:** Gradio ChatInterface, deployed on Hugging Face Spaces
- **Email:** Gmail SMTP (smtplib) — sends resume to visitors, notifies me on requests
- **Logging:** Google Sheets API (gspread + google-auth) — two tabs: unknown questions and all conversations
- **PDF parsing:** pypdf — extracts text from resume PDF for LLM context
- **Environment:** python-dotenv, uv for dependency management

### How It Works
The bot is built around a system prompt that injects several context files — a personal summary, skills list, company details, portfolio website copy, and parsed resume text — so the LLM has rich, accurate context before any conversation starts.

Tool calls are the core of the agentic behavior. The bot has three tools:
- **`get_employment_history`** — returns structured employment data filtered by duration and role type (employer vs. independent). Used for any question about work history, companies, or job durations.
- **`send_resume`** — emails my resume PDF to the visitor and sends me a notification email.
- **`record_unknown_question`** — logs questions the bot couldn't answer to a Google Sheet with a timestamp.

Every conversation is also logged (question + answer + timestamp) to a separate Google Sheets tab for quality review.

### Key Features
- Answers questions in first person, in my voice and communication style
- Proactively offers to email my resume on the first message
- Accurately answers any question about employment history, job durations, or company counts — regardless of threshold (1 year, 2 years, etc.)
- Logs unanswered questions to Google Sheets for review
- Logs all conversations to Google Sheets for quality monitoring
- Closing remarks are controlled in code — offered once on the first message, never repeated

### How It Evolved
The project started as a lab from an Udemy Agentic AI course (originally built by Ed Donner). The base version used Pushover for push notifications and GPT-4o-mini.

**Changes I made:**
1. Replaced Pushover with Gmail SMTP — consolidated notifications and resume sending into one service
2. Added resume-sending capability — visitors can request my resume and receive it by email automatically
3. Replaced the LinkedIn PDF source with my resume PDF — cleaner parsing, more accurate and current data
4. Upgraded from GPT-4o-mini to GPT-4o — better factual reasoning
5. Added Google Sheets logging for unknown questions — avoids email bursts that could trigger Gmail's spam detection
6. Added full conversation logging — allows ongoing quality review
7. Added multiple context files — skills, company details, portfolio copy, in addition to the summary and resume
8. Refactored into a class-based structure (`Me` class) for cleaner organization

### Technical Challenges & Solutions

**Challenge: LLM giving inconsistent answers about employment history**
The LLM was missing roles, miscounting, and giving different answers to the same question. Even with the data in the system prompt, it would hallucinate or skip entries — particularly when roles overlapped in time (Sesame Communications and HostingCT ran simultaneously).

*Solution:* Built a `get_employment_history` tool that stores employment data as a structured Python list. The tool filters by minimum duration (in months) and role type, returning only relevant entries. The LLM calls the tool to get the data rather than trying to recall it from context.

**Challenge: LLM truncating the last item in numbered lists**
Even when the tool returned correct data, the LLM would consistently write the final list number (e.g. "5.") but leave it blank. This happened reliably regardless of list length.

*Solution:* Moved list generation entirely into code. After the tool is called, the chat method strips any numbered list the LLM generated using regex, then appends a pre-formatted list built from the tool result. The LLM provides only a prose intro sentence.

**Challenge: LLM adding closing remarks despite instructions**
The LLM consistently added phrases like "feel free to ask!" or "let me know if you need anything" at the end of every response, even when explicitly told not to.

*Solution:* Two-layer approach — a `strip_closing_remarks` function that uses regex to detect and remove common invitation patterns from the end of responses, plus a code-controlled first-message closing that appends a fixed string once and never again.

**Challenge: Inconsistent and broken list formatting**
The LLM generated lists inconsistently — mixing `*` and `-` bullet styles, nesting items as sublists with progressively smaller font sizes, and cramming multiple list items onto a single line separated by inline dashes. Formatting instructions in the system prompt had no effect.

A secondary bug compounded the issue: the `strip_closing_remarks` function split text on sentence boundaries and rejoined with `" "` (a space), which destroyed all newlines in the response. This collapsed properly formatted multi-line lists into a single line of prose, making it appear that lists had stopped working entirely.

*Solution:* Two fixes. First, a `fix_list_formatting` function that normalizes all `*` bullets to `-`, removes list item indentation, and splits inline ` - ` separated items onto their own lines using regex. Second, rewrote `strip_closing_remarks` to split on `\n` instead of sentence boundaries, preserving list structure — it now strips closing remarks by working backwards through lines, with a fallback to sentence-level stripping only within the final line.

**Challenge: First/third person inconsistency**
When users referred to me as "she" or "Natalie," the bot would sometimes respond in third person rather than staying in first person.

*Solution:* Added an explicit system prompt instruction: "Always respond in first person (I, me, my), even if the user refers to you in third person."

**Challenge: Google Credentials in .env**
Storing a multi-line service account JSON in a .env file caused python-dotenv parse errors and a JSON decode failure at runtime.

*Solution:* The entire JSON must be minified to a single line and wrapped in single quotes in the .env file. Generated using: `python3 -c "import json; print(json.dumps(json.load(open('credentials.json'))))"`.

### What I Learned
- LLMs are unreliable for factual recall and enumeration tasks — tools are the right solution for structured data
- Instruction-based control of LLM output has limits; code-level post-processing is more reliable for deterministic behavior
- System prompt length matters — context buried deep in a long prompt gets ignored; critical instructions belong at the end
- Separating concerns across multiple context files (summary, skills, company details) makes the system easier to maintain and update
- Google Sheets is a practical, free alternative to push notification services for low-volume logging



## Newsletter Digest Deduper Agent

### What it is                                                      
A personal full-stack web tool that reads newsletters from a designated email folder, removes duplicate stories that appear across multiple sources, and surfaces a clean,          
deduplicated digest in the browser. The motivation was practical: subscribing to 10+ AI/tech newsletters means reading the same story 4–5 times a day. This tool collapses them into
  one pass.                                                                                                                                                                          
The user visits a web UI, picks a folder and date range, hits generate, and gets back a ranked list of unique stories with sources, dates, and links — with a one-click PDF export  
for saving or sharing.

---                                                                                                                                                                                 
### Technology stack
                                                                                                                                                                                    
┌─────────────────┬────────────────────────────────────────────────────────────────────────────┐
│      Layer      │                                  Choices                                   │
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ Backend         │ Python 3.11, FastAPI, Uvicorn                                              │
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ Database        │ SQLite via SQLAlchemy Core (async) + aiosqlite                             │                                                                                    
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ Email ingestion │ IMAPClient, html2text, BeautifulSoup4 + lxml                               │                                                                                    
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ AI / NLP        │ Anthropic API (claude-haiku-4-5), sentence-transformers (all-MiniLM-L6-v2) │
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤                                                                                    
│ PDF export      │ weasyprint (primary), reportlab (fallback)                                 │
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤                                                                                    
│ Frontend        │ Vanilla HTML/CSS/JS, Pico.css, marked.js — no build step                   │
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤                                                                                    
│ Config          │ pydantic-settings, python-dotenv                                           │
└─────────────────┴────────────────────────────────────────────────────────────────────────────┘                                                                                    
  
The AI stack is intentionally two-tiered: a small local embedding model for fast semantic clustering, and a hosted LLM only where judgment is needed (noise filtering, pairwise     
dedup refinement, editorial keep/drop). This keeps per-run cost low while getting the quality benefits of both.

--- 

### How it works (the pipeline)
IMAP → Parse emails → LLM noise filter → Embed + cluster → LLM pairwise refinement → Select representative → LLM editorial filter → SQLite → Browser

1. Fetch emails from IMAP (read-only, BODY.PEEK[] so the mailbox is never modified)
2. Parse HTML emails into story records — each newsletter's HTML gets converted to structured story objects with title, body, links, sender, and date
3. LLM noise filter (pre-cluster): removes structural non-content in batches — sponsor blocks, referral prompts, navigation headers — fail-open so a bad API call never drops real stories
4. Embed + cluster: all-MiniLM-L6-v2 encodes each story body and community_detection groups semantically similar ones. Threshold tuned to be high-recall (errs toward grouping)
5. LLM pairwise refinement: for each multi-story cluster, the LLM compares all pairs and classifies them as same_story, related_but_distinct, or different. Only confirmed duplicates are merged (via union-find for transitivity)
6. Select representative: from each merged cluster, picks the story with the longest body, then has a title, then has links
7. LLM editorial filter: final keep/drop pass on deduplicated representatives. Drops anything that slipped through but isn't real content. Borderline decisions get flagged to a local file for development review
8. Store + return: result saved to SQLite, returned to the frontend as JSON

---                                                             

### How it evolved
The project was built in four phases:

Phase 1 — CLI pipeline. The entire parsing and dedup pipeline was built as a command-line tool first, with no API or UI. This made each stage independently testable. Email parsing, embedding, dedup, and LLM filtering could all be validated with real emails before any web infrastructure existed.

Phase 2 — API layer. FastAPI was added on top of the pipeline. A single POST /api/digests/generate endpoint runs the pipeline end-to-end and stores the result. The synchronous pipeline design (no background tasks) kept the API simple — the browser just waits.

Phase 3 — Frontend + PDF export. Vanilla HTML/CSS/JS with Pico.css for semantic styling. localStorage persists the last-used folder and dates. marked.js renders story bodies as markdown so inline links work. PDF export uses weasyprint, with a full reportlab fallback since weasyprint has system dependency issues on some machines.

Phase 4 — Real-world parsing. The most technically interesting phase — confronting the reality that newsletter HTML is wildly inconsistent. Most of the hard work landed here.

---

### What worked well                                                
Two-stage deduplication was the right architecture. Embeddings alone over-cluster (different stories with similar topics get merged); LLM alone at scale is expensive and slow.
Doing a coarse semantic grouping first, then using the LLM only to adjudicate borderline cases, gets high precision at low cost.
                                                                
Fail-open LLM calls turned out to be essential. Network failures, rate limits, and malformed responses happen in practice. Every LLM call is wrapped so a failure falls back to keeping all items in that batch rather than dropping them. You lose precision on that batch but never lose real stories.

Conservative parsing philosophy. The parser is designed to over-include rather than over-filter. Any ambiguous section gets passed through to the LLM filter, which is the right place for content judgment. False positives (noise that gets through) are recoverable; false negatives (real stories dropped early) are not. This philosophy was tested repeatedly and held up every time.
                                                                
Pico.css was the right UI call for a solo project. Pure semantic HTML styling means the form, cards, and layout work correctly without writing almost any CSS, and the result looks clean enough for a portfolio.

---                                                             

### What didn't work / required redesign
Dedup signal. The first instinct was to deduplicate on titles. This fails: two newsletters covering the same story often have completely different headlines. Switching to body text as the embedding signal (with URL noise stripped before encoding) was a significant improvement.

LLM for story generation. An early prototype had the LLM rewrite and summarize stories. This was scrapped entirely. It introduced hallucination risk, obscured sources, and was expensive. The final system only makes binary keep/drop decisions — all story text is the original newsletter's words.

weasyprint as the only PDF renderer. weasyprint produces beautiful output but has a libpango/libharfbuzz system dependency that fails silently or with cryptic errors depending on the OS. Adding a full reportlab fallback added significant code but made the PDF feature work reliably across machines.

---                                                      

### Real-world edge cases that weren't foreseen

AP Wire formats — articles bleeding together. The AP sends three distinct newsletter formats: Morning Wire, Afternoon Wire, and News Alert. Initially all three were parsed as undifferentiated HTML. The result: multi-story emails produced one giant "article" containing 4–5 unrelated stories merged together.

The fix required reverse-engineering the HTML structure of each format. Morning Wire uses 22px font spans for article headlines; Afternoon Wire uses 20px for headlines and 22px for section headers ("IN OTHER NEWS"). The parser now detects which format it's looking at, promotes the right spans to <h1>, and injects sentinel nodes before each headline to split the HTML into per-article blocks before any text conversion happens.

1440 Daily Digest — boxed sections absorbed into articles. 1440 uses <table style="border:6px solid #f4f4f4"> to wrap each self-contained content section (sponsored articles, topic categories, link roundups). These boxed tables appeared after the main featured articles and kept getting absorbed into the preceding story's body. The "Masters Tees Off" story ended up containing a completely unrelated sponsored article about an energy investment.

The fix was to detect those border-style tables with a regex and inject an <hr> before each one during preprocessing. html2text renders <hr> as * * *, which is below the minimum section length threshold and triggers a clean break in the story collection loop. Simple, surgical, no changes to the collection logic itself.

Pull quotes and attributions as separate "articles." 1440 uses decorative pull quotes with the quote on one line and the attribution (— Three-time Masters champion Gary Player) on the next. Both were appearing as separate "story records" in the output, despite having no links and no real content. The fix: all records without at least one non-boilerplate URL are dropped at the parser stage. Real newsletter articles reliably have links; structural decoration does not.

Morning Brew sub-section headings triggering story collection. Morning Brew uses ## **Sub-heading** inside articles to introduce sub-sections. The story collection loop was treating these as new article starts, which caused it to sweep all following content (including completely unrelated articles) into the same record. The fix tracks whether the most recently processed section was content or just a heading — a ## **bold** heading following content is treated as a within-article sub-heading (skipped), not a new story trigger.

Morning Brew "Tour de headlines." Some Morning Brew table cells pack 2–3 unrelated articles into a single cell, separated only by **Bold headline sentence.** body text patterns with no HTML structure to split on. These needed a dedicated splitting function that detects the **Title.** pattern and separates them into individual article dicts before any record assembly happens.

html2text trailing-space artifacts fusing sections. html2text renders table cell boundaries as   \n  \n (two spaces before each newline) rather than \n\n. This caused a sponsor label and the following article heading to be fused into a single section string, which broke the heading-detection logic. The fix was a two-line heading check: if the second non-empty line is a #-prefixed heading, the section is still treated as a story start, with the first line (the fused label) absorbed into the article body.

---

### Testing approach
The test suite (pytest, ~69 tests at project end) is structured as a pyramid:
- Unit tests on individual parser functions (_extract_title, _split_list_section, _collect_links, _is_story_heading) — fast, precise, and used to nail down edge case behavior before wiring it into the full pipeline
- Integration tests via parse_emails() with synthetic HTML — minimal MIME emails built in-test, asserting on StoryRecord output shape, content, and counts
- Real-email regression tests using saved .eml files from actual newsletters — these catch regressions when parser changes affect real-world formats

One important lesson: many tests were written with link-free HTML because they were testing structural behavior (heading extraction, body assembly), not link behavior. When the link-free filter was added late in the project, 27 tests failed simultaneously. This revealed a gap in the test data design — minimal test HTML should still be realistic enough to survive the full filter stack.

---

### Takeaways
- HTML email parsing is genuinely hard. There is no standard. Every newsletter vendor (beehiiv, Mailchimp, custom, AP CMS) produces different structure, and the same sender can use multiple formats depending on story type. Building a parser that handles real-world email requires iterating against real emails, not synthetic test cases.
- Two-stage AI pipelines beat single-stage ones when the problem has both a recall dimension (find all duplicates) and a precision dimension (confirm they're actually the same).
The fast local model handles recall; the expensive remote model handles precision.
- Design for observability from the start. The pipeline logging was added late, and it immediately revealed behaviors (exact drop counts per filter per newsletter) that were invisible before. These would have helped debug the bleeding issues faster.
- The parser is the hardest part. Not the AI, not the API, not the frontend. The parsing of real-world HTML newsletters into structured, clean story records is where the majority of engineering time went, and where the most interesting problems lived.

---

### One-sentence version (for resume bullets)                       
"Built a full-stack Python/FastAPI newsletter digest agent that fetches emails over IMAP, parses multi-format HTML newsletters into story records, deduplicates across sources using a two-stage sentence-embedding + LLM pipeline, and surfaces a clean digest via a browser UI with PDF export."



## Deterministic Agent Development Workflow

### Overview
I designed and refined a structured workflow for building AI agents that removes ambiguity from both implementation and validation.
The goal was to turn AI-assisted development from something unreliable and guess-based into a repeatable, deterministic system.

⸻

### What the workflow does
This workflow enables an AI agent to:
* break down a product into discrete deliverables
* generate structured implementation plans
* execute those plans in a controlled environment
* validate results using exact, verifiable outputs
* iterate reliably until completion
Instead of relying on prompts and intuition, the system enforces:
Select → Plan → Execute → Validate → Commit → Repeat

⸻

### Core components of the workflow
1. PRD-driven development
  Everything starts with a Product Requirements Document (PRD) that:
  * defines system scope and architecture
  * breaks work into phases (MVP → future)
  * outlines deliverables in sequence
  From this, a prime summary is generated, which acts as the source of truth for what to build next.

2. Deliverable-based execution loop
  Work is done one small unit at a time (files, modules, components).
  Each deliverable follows a strict loop:
  Select → Plan → Execute → Validate → Commit → Repeat

3. Plan generation (/plan-feature)
  Each deliverable is converted into a structured plan that includes:
  * step-by-step implementation tasks
  * validation commands
  * manual verification checklist
  This removes ambiguity before execution even begins.

4. Agent execution (/execute)
  The agent:
  * writes code
  * creates/modifies files
  * installs dependencies
  * runs validation commands
  All within a controlled environment.

5. Deterministic validation system (core innovation)
  This is the most important part of the workflow.
  Instead of vague validation like:
  “make sure it works”
  The system enforces:
  👉 Every requirement must map to visible output
  Each check becomes:
  * a command
  * an exact expected output
  Example:
  Command:
  .venv/bin/python -c "import module; print('OK')"

  Expected:
  OK
  No interpretation is allowed.

6. Validation Output Reference layer
  To eliminate confusion, I introduced a system that:
  * aggregates all required checks
  * maps them to exact outputs
  * ensures nothing is skipped
  It also enforces:
  * no duplicate checks
  * no “confirmed by” shortcuts
  * no hidden/internal assertions
  * no summarized outputs
  Only what is visibly provable counts.

7. Strict agent constraints
  To make the workflow reliable, I designed rules that control agent behavior:
  * no summarizing validation output
  * no duplicate validation entries
  * no indirect verification
  * no placeholder outputs
  * no missing checklist items
  This transforms the agent from:
  “best guess generator”
  into:
  controlled execution system

8. Built-in environment and dependency handling
  The workflow includes:
  * Python virtual environments (.venv)
  * dependency installation (pip install, npm install)
  * consistent execution paths (.venv/bin/python)
  This ensures reproducibility across runs.

9. Integrated version control
  Each loop ends with:
  * staging changes
  * committing per deliverable
  * pushing to GitHub
  This creates:
  * clean history
  * traceable progress
  * isolated changes

⸻

### How the workflow evolved
Early stage
  * loose prompts
  * inconsistent validation
  * frequent ambiguity and rework

Mid stage (breakthrough)
  * introduced explicit validation commands
  * began mapping checks to outputs

Late stage (refinement)
  * enforced strict validation rules
  * eliminated duplication and noise
  * added output reference system
  * stabilized agent behavior

Final state
  * deterministic
  * repeatable
  * scalable to complex systems

⸻

### What didn’t work
* relying on the agent to “understand” correctness
* vague validation instructions
* mixing human-readable checks with machine validation
* allowing summarized or indirect verification
* overloading prompts without structural constraints

⸻

### Real-world edge cases encountered
  1. Environment inconsistencies
  * virtual environment confusion
  * command failures due to incorrect context
  Fix: enforced explicit .venv execution rules

2. Tool vs shell confusion
  * /execute vs terminal commands behaving differently
  Fix: clarified execution context separation

3. Validation ambiguity
  * outputs that looked correct but weren’t explicitly verified
  Fix: required exact output matching

4. Over-constrained prompts
  * too many rules caused bloated or redundant outputs
  Fix: refined constraints to focus only on visible output

5. Duplicate validation entries
  * same check appearing in multiple forms
  Fix: enforced one-to-one validation mapping

⸻

### Why this workflow is powerful
This system turns AI development from:
Prompt → Output → Debug → Repeat
into:
Plan → Execute → Prove → Commit → Repeat

Key advantages
  * eliminates ambiguity in validation
  * prevents silent failures
  * enforces correctness at every step
  * scales across multi-component systems
  * makes AI behavior predictable and controllable

⸻

### One-line summary
Designed a deterministic, PRD-driven agent development workflow that enforces correctness through structured planning and exact output validation, enabling reliable and repeatable AI-assisted software development.