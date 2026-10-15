# AgentTrace

AgentTrace turns an AI coding agent's session log into an evidence-linked report of what the agent touched and whether it was allowed to.

## What this first working slice does

The page opens at the top with instructions, a synthetic sample already loaded, and a clearly labeled **Sample report** below the inputs. A security engineer can try the complete flow:

1. Load an agent session log (pick one of three synthetic samples or paste a log, one JSON action per line).
2. Enter a plain-English scope policy (a default policy is pre-filled).
3. Click **Analyze session** and see a parsed action timeline with secrets redacted, a list of flagged actions (severity, rationale, confidence, and clickable log-line citations), and a draft summary.
4. Confirm or dismiss each flag, edit the summary, check the review box, and download the report as Markdown.

Export stays locked until every finding is confirmed or dismissed, the summary is nonempty, and the summary review box is checked. Review progress shows how many findings have been reviewed, not how many agent actions are safe.

Editing the log or policy hides the previous report, clears its review decisions and summary approval, and asks you to analyze again. Editing the summary also resets its approval. Downloading the report completes the flow.

## Open the prototype

- [Open the live prototype (Replit)](https://genai-practice-karitonze.replit.app/)
- Source code: https://github.com/stardess/genai-practice
- Run it yourself: open `index.html` in a browser, or run `python3 -m http.server 5000 --bind 0.0.0.0` from the repository directory and open the server's preview. No package installation, API key, or backend is required.

Use synthetic logs only. Download remains available if the browser blocks clipboard access.

## AI role: what's real and what's simulated

**Intended AI system.** An LLM will (a) normalize messy shell and cloud CLI commands into structured meaning (resource, operation, read/write/destructive), (b) judge each action against the natural-language policy and detect multi-step patterns such as "access denied → assume role → retry delete," and (c) draft an audience-appropriate summary in which every sentence cites log lines. A validator will drop any claim without a valid citation.

| Part of the flow | Status |
| --- | --- |
| Log parsing into a numbered action timeline | **Working** (deterministic, in the browser) |
| Secret redaction (AWS keys, session tokens, password/token/secret values) | **Working** (regex) |
| Rule checks: destructive commands, IAM/credential use, production cloud changes, writes outside the folders listed after "May edit" | **Working** (regex and path rules) |
| Flag review (confirm/dismiss), review gate, Markdown export | **Working** |
| AI policy judgments, multi-step pattern detection, rationales, confidence | **Simulated.** Prewritten by hand for the three sample logs and labeled "AI review (simulated)" in the UI |
| AI summary draft | **Simulated** for samples. Custom logs get a fill-in template labeled "AI not connected" |
| Prompt-injection resistance | **Simulated** in sample 3 (the prewritten result shows the embedded instruction being ignored); no model is called, so there is nothing to inject into yet |

The app detects a sample by exact text match. If you edit a sample log, it is treated as a custom log and only the real rule checks run.

## Test results

Run on 2026-10-15 in a real Chromium browser against the Replit prototype. Downloaded Markdown files were opened and inspected; these are observed results, not expected outcomes.

| Test | Input or action | What happened | Pass, partial, or fail |
| --- | --- | --- | --- |
| Typical case | Select "Deleted files + data rewrite", click Analyze, review the findings and summary, then download | Parsed 7 actions and showed 2 high and 1 medium simulated findings, with rule tags on L3/L5. Clicking an L5 citation highlighted the evidence. Confirm, Dismiss, and Undo updated progress correctly. Export stayed blocked until review was complete; editing the summary reset approval. The downloaded file contained the reviewed summary, scope policy, confirmed/dismissed decisions, and all 7 timeline rows; the completion message appeared. | Pass |
| Challenge case | Select "Role workaround + injection", analyze, inspect evidence, review all findings, then download | Parsed 7 actions and displayed 4 simulated findings: 3 high and 1 info, including the permission workaround and embedded-instruction notice. The fake keys/token on L4–L5 were redacted in both the visible timeline and the actual downloaded file. Read-only L1 had no rule tag. The prepared response treated the embedded instruction as data; this does not demonstrate live model injection resistance. | Pass |
| Invalid or empty case | Clear → Analyze; then try `hello`, `{"type":"fly"}`, and an empty policy | An empty log prompted the user to add a session log. `hello` produced "No actions could be read" and "Line 1: not valid JSON"; the unsupported type and missing policy also received specific errors. No previous report remained visible, and both export controls stayed disabled. | Pass |

Additional browser checks passed: the Clean session produced no findings and unlocked export after summary review; changing either input immediately invalidated the old report; editing a sample switched it to custom rules-only output. At 402 × 874, input and report panels stacked with no horizontal page overflow. The initial page stayed at the top, and no JavaScript exceptions were observed.

## Known limitations

1. **AI judgment is simulated.** Sample findings, rationales, and summaries are prewritten: editing the policy changes deterministic rule tags, not those AI judgments. A future version would use a model with structured output, citation validation, and evaluation against representative logs.
2. **Custom logs get rules only.** Regex and path checks can miss multi-step workarounds or unusual command phrasings, and redaction only covers recognized patterns. Future work would improve classification and redaction; only synthetic logs should be used now.
3. **Limited input format and session-only review.** Real agent transcripts are not imported, and review decisions are not saved or signed; keep the downloaded report before leaving the page. A future adapter could accept real transcript formats, while durable review history remains outside this first slice.

## Repository safety

The app requires no API keys or credentials. The credential strings in sample 3 are explicitly fake examples used to test redaction; never put real logs, passwords, API keys, or access tokens in this public repository.

## Submission note

The three required cases are in **Test results** above, and the three limitations are in **Known limitations**. Submit the public repository link and the working prototype link; confirm both are accessible before submitting.
