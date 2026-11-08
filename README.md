# Taint Shield

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
