# Verification

- Eleven local tests pass, including a hand-written corpus of 8 injected pages with the actions they try to cause (secret exfiltration, copied command lines, persistence writes, tainted messages and navigation) that must all be stopped, and 6 ordinary agent flows that must be left alone. The corpus is small and written by the plugin author; it is not an independent benchmark.
- 2026-11-07 through Hermes's real tool path in a throwaway home: a page with an HTML-comment injection was recorded through the real `post_tool_call` hook; the real `model_tools.handle_function_call("terminal", ...)` for the exfiltration command returned the Taint Shield block and the command never ran, while `echo` ran normally; `transform_tool_result` appended the warning; `shield_report` showed the source, flag and decision.
- A test found and fixed a real bug: trusting a domain did not cover full URLs on that domain.
- Not yet verified: a live agent session with a real model reading a real hostile page, rephrased or encoded injections, false-positive rate on real long sessions, Windows.

## 2026-11-10 maintained scope revision

Thirteen tests pass after adding local-document/transcription/research origins, role-spoofing and approval-bypass phrases, bounded-scan disclosure and read-file attack/exfiltration coverage. Heuristics are not a universal injection defense or a safety certificate. Hermes admission and retirement preflight pass; detailed local receipts are retained in the contest maintenance evidence. Old evidence above keeps its original scope.

## 2026-11-10 gap found while preparing the first live demo

A hand check showed that after an injected document, a command that copies a secret-named
file somewhere else (`cp .env.production /tmp/x/`) was not stopped: the secret rule needed a
network-sending command, and `cp` was not a recognized command line. Added a staging rule
(terminal or execute_code, secret-looking path plus copy/move/archive/redirect, after an
injection-like phrase or a copied command line) and `cp`/`mv`/`rsync` to the recognized
command lines. Corpus now has 9 attacks and 7 ordinary flows (one is a copy of a non-secret
after a hostile page, which must stay allowed). All tests pass; heuristic limits unchanged.

## 2026-11-10 second live-run finding

In a real Hermes session the file reader returns a JSON string with escaped newlines and
"N|" line-number prefixes, so command lines inside a read document were never recognized
and `python3 --version` copied from the page ran unchallenged (the injection warning was
still appended). `observe` now unwraps that JSON and strips the prefixes; a test uses that
exact shape. Not yet re-run live.

## 2026-11-10 version 0.2.1

Version bumped for the two fixes above. Tests (13 across 3 files), workflow preflight and Hermes admission (`validate_plugin_dir`, all 15 checks pass, no warnings) were re-run on 0.2.1; receipts (test output, preflight and admission results) are kept in the maintainer's private evidence folder, which is not part of this repository.

## 2026-11-10 version 0.2.2

The skill now tells the agent to show the shield's own message word for word and continue with the rest of the request (an instruction inside the message itself was tried and removed because it showed up in the quoted text). Text-only change; the 14 existing tests pass. Whether the model relays it is checked by the demo run only.

The skill also tells the agent to run each requested check as its own command: in a live run two checks joined with && were stopped together because one part came from the untrusted file, so the ordinary check never ran. Text-only; checked by the demo run.

The skill also says to run a check the user asked for instead of refusing in advance: in a live run the agent declined on its own, so the shield's ask box never appeared. The shield, not the agent's own caution, is the guard. Text-only; checked by the demo run.
