# Verification

- Eleven local tests pass, including a hand-written corpus of 8 injected pages with the actions they try to cause (secret exfiltration, copied command lines, persistence writes, tainted messages and navigation) that must all be stopped, and 6 ordinary agent flows that must be left alone. The corpus is small and written by the plugin author; it is not an independent benchmark.
- 2026-11-07 through Hermes's real tool path in a throwaway home: a page with an HTML-comment injection was recorded through the real `post_tool_call` hook; the real `model_tools.handle_function_call("terminal", ...)` for the exfiltration command returned the Taint Shield block and the command never ran, while `echo` ran normally; `transform_tool_result` appended the warning; `shield_report` showed the source, flag and decision.
- A test found and fixed a real bug: trusting a domain did not cover full URLs on that domain.
- Not yet verified: a live agent session with a real model reading a real hostile page, rephrased or encoded injections, false-positive rate on real long sessions, Windows.
