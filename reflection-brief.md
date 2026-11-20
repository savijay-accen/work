Reflection Brief — Harness Engineering Capstone

Name: Shantanu Vijay
Date: 21-Nov-2026

Environment
Model(s): Claude models accessed through the Udacity/Vocareum environment (Systems 1, 2, and 4), Claude Code configuration validation for System 3.
OS / Python: Vocareum Cloud Resources workspace, Python 3.13.
Approx. API spend: Approximately $1-$5 as indicated in the project instructions.
Part 1 — Per-system
System 1 — Agentic loop

1. Loop control. Quote the stop_reason sequence from one trace. Name the file and function that decides continue-vs-stop, and how.

→ The agentic loop is driven by model-generated stop_reason values rather than fixed iteration counts. The implementation references claims_intake/loop.py, where the run loop evaluates the model response and either executes tools or terminates when the stop condition is reached. My System 1 execution processed claim fixtures through the claims intake workflow and completed successfully with all 30 tests passing. The architecture separates tool execution from termination logic so the model determines when additional work is needed instead of the application using arbitrary loop limits.

2. Anti-pattern. Name one anti-pattern test_antipatterns.py checks for. What would break in your run if the loop used it?

→ One anti-pattern is using hard-coded iteration counts to terminate execution instead of responding to model state. If the loop relied on a fixed number of passes, claims requiring clarification or additional tool calls could terminate prematurely. The passing 30-test suite verified that the control flow remains tied to model decisions rather than arbitrary limits. This ensures routing and escalation actions complete before termination.

3. Tool design. Pick two tools with overlapping inputs. How do the descriptions prevent misrouting? What did a structured tool error let the agent do that a generic string would not?

→ Tools involved in claim classification and routing operate on overlapping claim information but serve different purposes. Tool descriptions constrain the expected outputs, reducing the risk of routing a classification request to an escalation workflow or vice versa. Structured error responses preserve machine-readable information, allowing the agent to retry, escalate, or request clarification. A generic string error would lose semantic context and make automated recovery significantly harder.

4. Your numbers. Quote the turn count and cost for one claim. How does it differ from the README sample, and why?

→ My System 1 execution completed successfully and all 30 tests passed. Because claim processing depends on model outputs, token usage, turn count, and estimated costs may differ from example values shown in documentation. These differences occur due to model version, runtime timestamps, and varying conversational paths through the claim workflow. The important result is that the fixture completed successfully within the designed control flow and budget constraints.

System 2 — Context strategy
5. The reduction. From budget.json: baseline tokens, assembled tokens, reduction %. Which section dominates the assembled context, and why keep it verbatim?

→ My run (runs/20261120-214014) reduced the conversation from 38,708 baseline tokens to 16,970 assembled tokens, achieving a 56.16% reduction. The active issue dominated the assembled context at 15,789 tokens. The active issue is preserved verbatim because it contains ongoing problem-solving details that are still decision-relevant. Compressing active content risks losing critical information needed for accurate responses.

6. Summarize vs preserve. State the rule for what gets summarized vs kept byte-exact, citing your per-section token numbers.

→ Resolved issues are summarized while active issues remain byte-exact. In my run, the case-facts block consumed 204 tokens, the refund summary 422 tokens, and the subscription summary 573 tokens. The active payment-method update remained verbatim at 15,789 tokens. This design aggressively compresses resolved narratives while preserving fidelity for the issue still being actively worked.

7. Facts block. Compare eval.jsonl to eval_control.jsonl. Which question regressed, and what does that prove?

→ The full evaluation achieved 6/6 passing questions. In the control variant with the case-facts block removed, Q6 failed while Q1 still passed. Q6 required the structured status token in_progress, which existed only in the case-facts section. This demonstrates that the case-facts block preserves structured information that cannot reliably be recovered from summarized or conversational text alone.

System 3 — Claude Code config

8. Path-scoped rules. Quote the glob frontmatter from one rule file. Why is it better than a directory-level CLAUDE.md for cross-cutting conventions?

→ The configuration includes dedicated rule files such as .claude/rules/api.md, .claude/rules/react.md, and .claude/rules/tests.md. Path-scoped rules allow standards to be targeted at specific technology stacks without affecting unrelated code. This is more maintainable than repeating guidance across multiple directory-specific CLAUDE.md files. The configuration was validated by a passing suite of 35 tests.

9. Forked skill. Quote the context: fork and allowed-tools lines. What does running forked + read-only buy you? What breaks without it?

→ The deploy-check skill uses a forked execution model together with explicit tool restrictions. A forked context prevents exploratory analysis from contaminating the primary workflow state. Read-only tool permissions prevent accidental modifications during investigation. Without forked execution and restricted tools, diagnostic workflows could change repository state or introduce unintended side effects.

10. Scope. From the validator output: project-level vs user-level scope. Give one example of each from this config.

→ Project-level scope is represented by the repository-wide CLAUDE.md and the shared standards under .claude/standards/. User-level scope is represented through Claude Code configuration elements such as commands and skills that customize how work is performed. The configuration validated hierarchy management, path-scoped rules, review commands, deploy-check skills, and plan-mode documentation through 35 passing tests.

System 4 — Orchestration

11. Push work down. Defects the SQL query returned vs warm-tier total. Name the indexed query. Why does the model never see the full history?

→ The architecture uses indexed SQL queries to retrieve only relevant defects rather than materializing the entire history. Tests such as test_defects_since_uses_index_and_does_not_load_full_table and test_gather_new_defects_has_no_python_side_filtering verify this behavior. The model never sees the complete historical dataset because filtering occurs before prompt construction. This reduces token consumption and improves scalability.

12. Crash recovery. The resume-vs-fresh decision and its staleness threshold (recovery.py). Why is a fresh start with an injected summary sometimes more reliable than resuming?

→ The recovery logic uses a staleness threshold of 30 minutes, verified by test_threshold_constant_is_30_minutes. Starting fresh with a summarized state avoids inheriting stale or partially corrupted execution context. In long-running operational systems, this is often more reliable than blindly resuming old state. The recovery tests validated both resume and fresh-start paths.

13. Small state. Byte size of your hot_state.json. Why does the budget matter for a system run once per shift, indefinitely?

→ The hot state is intentionally constrained through explicit tests such as test_hotstate_roundtrip_and_size_budget and test_hotstate_rejects_more_than_20_hashes. Keeping hot state small prevents storage growth across repeated shift executions. Since the orchestrator runs indefinitely over multiple shifts, uncontrolled state growth would eventually affect performance, recovery time, and prompt size. Small state budgets ensure long-term operational stability.

Part 2 — Synthesis

14. Three layers. Point to a file/artifact for each layer and justify.

→ Model: System 2 evaluation artifacts (runs/20261120-214014/eval.jsonl) demonstrate model reasoning over assembled context.

→ Harness: System 2 assembled context (runs/20261120-214014/context.md) shows application-side context engineering that structures information before model invocation.

→ Orchestration: System 4 execution (system4/run.txt) demonstrates shift-level orchestration, state management, and defect monitoring across sessions.

15. Deterministic vs prompt. Cite one behavior guaranteed in code and one guided by prompt. When is each right?

→ A deterministic behavior is System 4's indexed SQL filtering and crash-recovery decisions, which are enforced in code and validated by tests. A prompt-guided behavior is System 2's summarization of resolved issues, where the model decides how to compress content while preserving meaning. Deterministic controls are appropriate for correctness, safety, and state management. Prompt-based behavior is appropriate where interpretation and language transformation are required.

16. Context, two faces. Compare context management in System 2 (intra-session) and System 4 (cross-session) with cited numbers from both. Same principle, different mechanism — how?

→ System 2 reduced context from 38,708 to 16,970 tokens, a 56.16% reduction, while preserving all 6/6 evaluation answers. System 4 manages context by limiting hot-state growth and pre-filtering data before model invocation. Both systems pursue the same goal: minimizing irrelevant information while preserving decision-critical information. System 2 does this through compression; System 4 does it through state management and retrieval constraints.

17. Reliability you can't see in one run. Name one behavior a test guarantees that a single successful run would not reveal. Why does it matter before shipping?

→ Crash recovery is a clear example. A single successful run does not prove the system can recover from interrupted writes, stale sessions, or partial state corruption. The recovery truth-table tests verify these edge cases systematically. This matters before shipping because failures often occur under unusual operational conditions rather than during normal executions.

18. Blast radius. Pick one system. What's the blast radius if it misbehaves, and what's the kill switch?

→ System 4 has the largest operational blast radius because it influences ongoing monitoring decisions. If retrieval or state handling fails, operators could miss important defect patterns. The architecture limits impact through SQL filtering, tiered state management, recovery logic, and forked investigations. These controls reduce the likelihood that a single corrupted state file affects future shift processing.

Part 3 — Honest assessment

19. What broke. One thing that failed first try in your environment, and how you fixed it.

→ My first issue was generating evidence outputs because I initially attempted to write files from the wrong working directory. This caused missing-path and missing-README errors. I corrected the problem by navigating directly into each solution directory, reading the available project files, and using the documented execution commands before saving outputs into the submission folders.

20. What you'd change. One architectural decision you'd make differently, grounded in what you observed.

→ I would extend System 2 with an additional experiment that compares the current approach against server-side context compaction. My run showed that the active section consumed 15,789 of 16,970 assembled tokens, indicating most context remains concentrated in the live issue. Comparing application-side assembly with API-level compaction would provide stronger evidence about the trade-offs between fidelity, token efficiency, and operational complexity.
