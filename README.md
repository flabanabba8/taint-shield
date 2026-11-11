# Taint Shield

Spot prompt injections hidden in webpages, documents, emails, transcripts or file contents before they redirect an assistant’s work. Taint Shield combines suspicious-instruction checks with source tracking and guards around sensitive actions.

**At a glance**

- **Install:** `hermes plugins install flabanabba8/taint-shield`, then `hermes plugins enable taint-shield`.
- **Needs:** Hermes 0.22 or later. No keys or extra tools.
- **Internet:** Nothing: it watches tool calls in memory and makes no network calls.
- **Limits:** Linux is the only platform checked. It is a heuristic guard that blocks or asks before risky actions that follow untrusted content; it is not a complete defence against prompt injection and does not bypass any Hermes approval.
- **Credits:** No third-party code or data is bundled. Licence: GPL-3.0-or-later.
- **AI assistance:** Written with AI coding assistants (Anthropic Claude and OpenAI Codex agents) under the maintainer's direction; what was actually tested is listed in VERIFICATION.md.

A prompt-injection firewall for Hermes. Text the agent reads from the web, a browser, mail or similar sources is untrusted, and attackers hide instructions in it (text telling the AI to discard its earlier rules and send a private key to a server). Taint Shield remembers what came from untrusted content and stops the agent acting on it.

How it works (Hermes hooks: `pre_llm_call`, `post_tool_call`, `transform_tool_result`, `pre_tool_call`):

1. Anything you write is recorded. URLs, domains, addresses and bare domains in your messages are trusted for that session.
2. Output from tools whose names look like web, browser, fetch, mail, search, scrape, download and similar is untrusted. Its URLs, domains, emails and IPs become *tainted*, and so do command lines it contains (`curl`, `wget`, `bash`, `crontab`, `ssh`, ...). Output that contains instruction-like phrases to an AI gets a one-line warning appended, so the model is reminded it is data.
3. Before a sensitive tool runs (terminal, execute_code, send/post/email tools, write_file, patch, browser navigation/typing), its arguments are checked:
   - **Block**: reading local secrets (`~/.ssh`, `.env`, cloud credentials, `printenv`) and sending them out; editing login or startup files (`authorized_keys`, shell rc, crontab, git hooks, systemd) after untrusted content was read; running a command line copied from untrusted output; using a tainted destination after an injection-like phrase was seen.
   - **Ask a person (approval)**: using a destination that appeared only in untrusted output and not in anything you wrote.
   - Everything else, including normal coding and test commands, is untouched.
4. `shield_trust` adds a permanent trusted destination, and always requires a person's approval, so injected text cannot trust itself.

Tools: `shield_report` (sources, flags, decisions per session), `shield_scan` (analyze any text), `shield_trust`.

Limits: this is heuristic provenance, not proof. It matches literal values, so an injection that makes the model rephrase a destination, encode it, or spell it out can slip past; commands the user also wrote are trusted; it only watches tools it can recognize by name; and it cannot see what happens inside a command after it starts. It reduces risk, it does not make an agent safe to run on hostile content. It adds vetoes and approval requests only and bypasses no approval. Block rules are tuned on a small hand-written corpus (below), not a benchmark.


Change in 0.2.2: the skill tells the agent to show the shield's own message word for word and carry on with the rest of the request, so the user sees the shield's own words in the chat.

Change in 0.2.1: command lines inside a document read with Hermes' file reader (a JSON result with escaped newlines and line-number prefixes) are now recognized as copied from untrusted content; a local copy of a secret-named file after an injection-like phrase is blocked. Found in live runs on 2026-11-10.
