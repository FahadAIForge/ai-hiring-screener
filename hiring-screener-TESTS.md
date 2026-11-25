# AI Hiring Screener — Live Tests

Workflow: **AI Hiring Screener** (12 nodes, 4 AI steps) — JSON built, NOT yet imported
Builder: ~/workspace/linkedin/staged/2026-11-21/build-hiring-screener.py
JSON: ~/workspace/linkedin/staged/2026-11-21/hiring-screener.json (validated: 12 nodes, all connections resolve)
Editor (after import): https://chuck-jumbo-compress.ngrok-free.dev
Production webhook (after activation): https://chuck-jumbo-compress.ngrok-free.dev/webhook/screen
Active: NO — the run must import, then activate from the editor, then run these tests.

## Design (12 nodes)
1. README Note (green sticky, 420x280)
2. New Application (Webhook POST /screen: name, message)
3. Validate Application (Code — message non-empty and > 5 chars)
4. Parse Application (AI Agent + Groq — extracts skills, experience_years, support_experience, english_level, tools, availability, notes as JSON)
5. Parse Application JSON (Code — parses the agent's JSON text; normalises years/English level; support flag also from skills text)
6. Score Fit (Code — DETERMINISTIC scoring, demo role Customer Support Agent: experience >=2y +40 / 1y +25 / >0 +10; support background +25; English fluent +20 / good +15 / basic +5; tools +15; cap 100)
7. Route Application (Switch — Shortlist / More Info / Reject; fallback More Info)
8. Invite to Interview (AI Agent — interview invite, asks for 2 time slots)
9. Request More Info (AI Agent — one clear ask for the missing detail)
10. Polite Rejection (AI Agent — warm rejection, details kept for future roles)
11. Reply — Screening Response (Respond to Webhook — status, score, breakdown, name, role, reply)
12. Groq Chat Model (SHARED — one node, connected via ai_languageModel to all 4 AI Agents; same pattern as Triage fwBHaEK1bsvc1pgs and Order Taker c6BKAZAK4kUhsZJK)

No Gmail/Sheets nodes (VM egress proxy blocks gmail.googleapis.com + sheets.googleapis.com).
No email is sent — the reply lives in the webhook RESPONSE only. Demo role only, no real hiring.

## Scoring-logic unit tests (run standalone in node 2026-11-20 ~08:15 PST, using the EXACT Score Fit code extracted from hiring-screener.json — PASSED 5/5)
- A: 3y support, fluent, Zendesk → score **100**, shortlist (40+25+20+15)
- B: 1y retail, basic English, no tools → score **30**, reject (25+0+5+0)
- C: 2y support, good English, no tools → score **80**, shortlist (40+25+15+0)
- D: 0.5y support, good English, HubSpot → score **65**, more_info (10+25+15+15)
- E: no usable info ("I want a job") → score **0**, more_info (hasInfo rule, not a reject)

## Live test cases (all POST, Content-Type: application/json) — RUN MUST EXECUTE, NOT YET RUN
### Test A — Strong applicant — EXPECTED: shortlist, score 100
- Request: {"name":"Ayesha","message":"Hi, I have 3 years of customer support experience at a telecom company, fluent English, I have used Zendesk, and I can start immediately"}
- Expect: HTTP 200, status "shortlist", score 100, reply invites her to interview and asks for 2 time slots.

### Test B — Weak fit — EXPECTED: reject, score 30
- Request: {"name":"Bilal","message":"I worked 1 year in retail, my English is basic, looking for a job"}
- Expect: HTTP 200, status "reject", score 30, reply is a warm, polite rejection.

### Test C — Good fit, no tools — EXPECTED: shortlist, score 80
- Request: {"name":"Sara","message":"2 years customer support experience, good English, available from next week"}
- Expect: HTTP 200, status "shortlist", score 80, reply invites her to interview.

### Test D — Junior, partial fit — EXPECTED: more_info, score 65
- Request: {"name":"Omar","message":"I did a 6 month customer support internship, good English, I have used HubSpot"}
- Expect: HTTP 200, status "more_info", score 65, reply asks for work history / missing detail.

### Test E — No usable info — EXPECTED: more_info, score 0
- Request: {"name":"Zain","message":"I want a job, please call me"}
- Expect: HTTP 200, status "more_info", score 0, reply asks for CV details (work history, English, tools).

## Notes for the run
- Import hiring-screener.json into the VM n8n first (same route as the Order Taker import), then activate from the editor (production webhook /screen only works when active). Then run the 5 curl tests above.
- Scores must match EXACTLY (100 / 30 / 80 / 65 / 0) — if the Parse Application agent misreads experience, English level or tools, the score will be wrong; that is a real failure, fix the prompt, do not fudge the post text.
- Screenshot: wide PNG canvas capture (Zoom to Fit + one zoom-out step, like Chaser 1860x680 / Triage 2561x1080) — white/light mode, all 12 nodes + sticky note readable in one frame, 1080 height. Do NOT use print-to-PDF (crops the canvas).
- Post text: ~/workspace/linkedin/staged/2026-11-21/post-text-hiring-screener.txt is a DRAFT — replace its live-test paragraph with the ACTUAL curl results before publishing.
