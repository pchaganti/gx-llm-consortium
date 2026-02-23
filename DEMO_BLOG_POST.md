# AI-Assisted Technical Debt Recovery: Rescuing the LLM Consortium

*How we transformed a sprawling, broken codebase into a clean, 100% tested dual-branch architecture in one session.*

If you build tools in the generative AI space, you know how fast things move. APIs deprecate, architectures pivot from async to sync overnight, and prompt engineering evolves by the hour. Before you know it, your experimental staging ground turns into a graveyard of abandoned git branches and failing test suites.

That was the state of the `llm-consortium`—a multi-agent LLM orchestration tool. We had 17 stale branches, two disconnected worktrees, and a test suite flashing red with 24 failing tests due to massive API drift. 

Here is how we used AI pair programming to completely repair and consolidate the project in record time.

### 1. The Great Branch Pruning (Without Losing History)
Our first roadblock was Git entropy. We had branches like `wip/2025-08-11` colliding with `tracing-auto-evals`. 

Instead of letting them sit forever, we enforced a strict **dual-branch architecture**: `main` for stability and `experimental` for R&D. We cherry-picked the valuable commits into `experimental`, but we didn't want to just `rm -rf` the rest. 

The elegant solution? We converted all 15 stale branches into lightweight `archive/*` tags and pushed them to GitHub. This preserved every line of legacy code for future reference, allowing us to confidently run `git push origin --delete <branch_names>` and clear out the clutter. The repository topology was instantly pristine.

### 2. Taming the API Drift with Red-Green TDD
The core orchestrator of the consortium had recently undergone a massive architectural shift:
- The execution loop shifted from `asyncio` event loops to standard synchronous execution. Because the underlying `llm` models rely on blocking network calls, `asyncio` was creating unnecessary overhead. By moving to synchronous methods backed by Python's `ThreadPoolExecutor`, we achieved true parallel execution for LLM prompts while drastically simplifying the API for developers.
- Initialization moved from sprawling kwargs to a strict Pydantic `ConsortiumConfig` model.
- XML parsing logic for the LLM Arbiter was heavily modified.

This left 24 unit tests broken. Using a systematic Red-Green approach, we triaged the failures file by file. We updated the `setUp` blocks to inject the new `ConsortiumConfig`, rewrote the async tests to properly execute the sync pipeline, and adjusted the voting strategy assertions to match character-level similarity grouping. Within an hour, we went from 24 failures to **60 passing tests and 0 failures**.

### 3. Hardening the LLM Arbiter Against Hallucinations
Finally, what good is a consortium of AI models if the judge is easily fooled? We noticed the Arbiter model was occasionally ranking hallucinated, confident answers higher than brief, accurate ones.

We rewrote the XML prompt templates (`rank_prompt.xml`), injecting strict new criteria:
> **Goal:** Give higher rank to answers that are *accurate, truthful, and grounded in the information actually provided*, even if they are brief. Down‑rank answers that invent data or “hallucinate” details that were not supplied.

We explicitly instructed the arbiter to prioritize models that dynamically acknowledge missing information over those that fabricate data. 

### Conclusion
By treating the AI assistant not just as a code generator, but as a holistic DevOps and repository management partner, we cleared out months of technical debt. The `llm-consortium` is now passing all CI checks, properly testing its parsing logic, and utilizing a highly grounded Arbiter judge—all staged perfectly across a clean `main` and `experimental` branch. 

When your AI tools build up technical debt, it turns out AI is pretty good at paying it off.
