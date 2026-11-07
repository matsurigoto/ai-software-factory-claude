# AI Test Agents and Testing Tools for the Verification Stages of an AI Software Factory (as of 2026-11-07)

Evidence labels used in these notes:
- **[GH]** read directly from the GitHub REST API on 2026-11-07 (stars, SPDX license, archived flag, last push, latest GitHub Release). Star counts are point-in-time.
- **[V]** vendor self-reported claim. Not independently verified.
- **[S]** seen only in a search snippet or secondary source. The primary page was not read, or was unreadable.
- **[P]** read from the primary page, paper abstract, or repo file.

Stars measure attention, not fitness. Several tools here have star counts far above older, better-studied tools in the same layer.

## Unit and integration test generation: who is viable, and what does the quality evidence say?

### Takeaway
The named products have shifted. Qodo Cover is unmaintained, EvoSuite is stale, and Diffblue now sells a Claude-based "Testing Agent" alongside its Java focus. The practical default is a coding agent with a TDD skill, a separate verifier, and a mutation score as the deterministic check. Mutation-guided generation (Meta ACH and JiTTest) has the strongest industrial evidence, but mutation score is not a reliable proxy when the code under test may already be buggy.

### Cited Findings

**Tool status**

| Tool | License | Stars [GH] | Latest release [GH] | Status flag | URL |
|---|---|---|---|---|---|
| Qodo Cover | AGPL-3.0 | 5,683 | 0.3.10 (2025-05-21) | **Stale.** README says the repository "is no longer maintained" and tells users to fork [P]. Not formally archived [GH]. | https://github.com/qodo-ai/qodo-cover |
| Qodo skills | MIT | 59 | n/a | Active, last push 2026-11-05. Small. | https://github.com/qodo-ai/qodo-skills |
| PR-Agent (ex-Qodo, review tool, not a test generator) | MIT | 15,314 | v0.49.0 (2026-10-25) | **Moved.** `qodo-ai/pr-agent` now resolves to `The-PR-Agent/pr-agent`. | https://github.com/The-PR-Agent/pr-agent |
| EvoSuite | LGPL-3.0 | 920 | v1.2.0 (2021-09-16) | **Stale.** Last push 2025-02-14. | https://github.com/EvoSuite/evosuite |
| Pynguin | MIT | 1,394 | 0.48.0 (2026-10-15) | Active. | https://github.com/se2p/pynguin |
| PIT (Java mutation) | Apache-2.0 | 1,886 | 1.30.2 (2026-11-05) | Active. | https://github.com/hcoles/pitest |
| StrykerJS | Apache-2.0 | 3,335 | v10.1.1 (2026-10-23) | Active. | https://github.com/stryker-mutator/stryker-js |
| Stryker.NET | Apache-2.0 | 2,125 | not checked | Active, last push 2026-11-07. | https://github.com/stryker-mutator/stryker-net |
| mutmut (Python mutation) | BSD-3-Clause | 1,530 | not checked | Active, last push 2026-10-28. | https://github.com/boxed/mutmut |
| Superpowers (TDD skill and others) | MIT | 313,653 | v7.0.2 (2026-11-03) | Active. | https://github.com/obra/superpowers |
| mattpocock/skills (has a TDD skill) | MIT | 316,796 | not checked | Active, last push 2026-11-05. | https://github.com/mattpocock/skills |

**Commercial**
- Qodo's current pricing page lists a 14-day trial, Pro Team at "$30" with credit bundles (2,000, 4,000, or 16,000 credits; $0.015 per credit, pooled), and custom Enterprise for 30+ users. It describes multi-agent code review, a Context Engine, and an Agentic Toolbox, and does not mention Qodo Gen or Qodo Cover by name — [Qodo pricing](https://www.qodo.ai/pricing/) [P]. Billing unit for the "$30" was not stated in what I read.
- The Diffblue pricing page is titled "AI Test Generation Pricing | Diffblue Testing Agent" [P, title only]. Teams is reported as $30K per year up to 250k LOC, with Enterprise as "Talk to Us" — [Diffblue pricing](https://diffblue.com/pricing) [S]. Older sources conflict: 200k LOC cap at $30,000 — [aicoolies review](https://aicoolies.com/reviews/diffblue-cover-review) [S]; and a $330/year Developer edition — [aicoolies pricing](https://aicoolies.com/pricing/diffblue-cover) [S]. A review notes it is unusable for Python, JavaScript, or Go teams — [aicoolies review](https://aicoolies.com/reviews/diffblue-cover-review) [S].
- **Diffblue benchmark [V].** Eight Java repositories. Testing Agent versus senior Java engineers using Claude Code, limited to 2 hours or 20 prompts: average line coverage 80.7% vs 32.3%; mutation coverage 61.3% vs 24.2%; test strength 81.8% vs 73.9%. The agent ran 15 minutes to 5.5 hours per repo and discards tests that do not compile or pass. The report says Claude "was the test generation engine in both scenarios" — [Diffblue Benchmark Report v4](https://www.diffblue.com/wp-content/uploads/2026/03/Diffblue_AI-Testing-Agents_Benchmark-Report_v4.pdf) [P, V]. I found no independent replication.

**TDD skills**
- The Superpowers TDD skill requires one failing test, confirmation that it fails for the expected reason, minimal code, then refactor. Production code written before its failing test must be deleted. It lists anti-patterns such as asserting on mock behavior — [Superpowers TDD SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/test-driven-development/SKILL.md) [P].

**Mutation evidence**
- Meta ACH: 10,795 Android Kotlin classes across 7 platforms, 9,095 mutants, 571 privacy-hardening tests. The equivalent-mutant agent reached precision 0.79 and recall 0.47, rising to 0.95 and 0.96 with simple preprocessing. Engineers accepted 73% of tests and judged 36% privacy-relevant — [Foster et al., arXiv 2501.12862](https://arxiv.org/abs/2501.12862) [P]. 277 of the 571 tests would have been discarded under a line-coverage criterion alone — [arXiv PDF](https://arxiv.org/pdf/2501.12862) [S].
- Meta JiTTest: 22,126 generated tests analyzed. Code-change-aware methods improve candidate catch generation 4x over hardening tests and 20x over coincidentally failing tests; assessors cut human review load by 70%. Modest agreement between LLM and rule-based assessors is reported — [Becker et al., arXiv 2601.22832](https://arxiv.org/abs/2601.22832v1) [S].
- Meta's October 2026 production report: 589 engineers engaged (64% of outreach), a 60% engineer-reported true-positive rate, 74 serious privacy bugs caught at diff stage in H1 2026, and privacy SEVs down 20% in that period. Engineers review outcomes in aggregate rather than every test. LLM-as-judge is non-deterministic, so a rule-based assessor (RubFake) was added — [Meta Engineering](https://engineering.fb.com/2026/10/29/security/catching-bugs-before-they-land-jittestings-human-on-the-loop-approach-at-full-scale/) [P, V: Meta on its own system].
- ISSTA 2026 replicability study: for LLM-generated suites, coverage and mutation score give meaningful signals when the code can be assumed bug-free (regression settings). They are not reliable indicators when the code may already be buggy and the goal is to expose the bug. Suite size is not a dominant confounder — [Zhao et al., arXiv 2607.22880](https://arxiv.org/abs/2607.22880) [P].
- MutGen reportedly raised mutation score from 53% (vanilla LLM prompt) to 89.5% on HumanEval-Java — [Augment guide](https://www.augmentcode.com/guides/mutation-testing-ai-generated-code) [S, secondary and vendor]. Not checked against the paper.
- Corpus evidence is mixed and does not show agent tests are categorically weak. AIDev dataset (179,732 agent test files, 24,941 human): strong assertions 85.37% vs 88.1%, boundary variety 0.62 vs 0.32, null-safety 13.40% vs 8.3%, flakiness candidate rate 0.41 vs 0.30 — [Jhanglani et al., arXiv 2607.12068](https://arxiv.org/abs/2607.12068) [P]. Tests from Claude Sonnet/Opus 4.6 and later were "no weaker" than Django and Pandas tests under three fault-injection protocols and a rubric — [Leith, arXiv 2608.15188](https://arxiv.org/abs/2608.15188) [P]. Single author, preprint, and the study does not address same-agent writing of code and tests.

### Inferences
- Use a coding agent with a TDD skill as the default generator across languages. Add EvoSuite-style search tools only where a regression net on legacy code is the goal; Pynguin is the live option for Python.
- Diffblue is worth a paid pilot only for large Java legacy estates. Most of its benchmark gap is coverage from long autonomous runtime; the test-strength gap (81.8 vs 73.9) is the better estimate of workflow value. Diffblue's own text suggests the differentiator is the orchestration and verification step, which a factory can reproduce.
- Do not build on Qodo Cover. A fork inherits AGPL-3.0 obligations.
- Treat mutation score as a regression-strength gate on changed code, not as proof of correctness. The ISSTA result means it cannot detect a test suite that faithfully encodes a bug.
- Adopt the JiTTest shape for high-risk diffs: generate catching tests that fail on the diff and pass on the parent, then rank before bothering a person.

### Gaps
- Could not confirm whether Qodo Gen is still sold under that name. The pricing page is silent. Billing unit for Qodo's "$30" needs checking.
- The Diffblue pricing page body did not render, so its prices are snippet-only.
- No independent comparison of Diffblue, Qodo, and plain coding agents was found.
- No rigorous study found of TDD skills versus no skill on defect detection. Star counts are the only "adoption" evidence I have and they are weak.

## E2E and UI agents: status, license, stars, pricing

### Takeaway
Playwright Test Agents plus Playwright MCP are the default because they emit ordinary, reviewable Playwright code under Apache-2.0. Treat general browser agents (Browser Use, Stagehand, Skyvern) as exploratory or hard-to-script tools, not as the regression layer. This market has already shed vendors: Octomind, Bugster, and DogQ shut down in 2026, so exportable test code should be a selection requirement.

### Cited Findings

**Open source**

| Tool | License | Stars [GH] | Latest release [GH] | Status flag | URL |
|---|---|---|---|---|---|
| Playwright (Test Agents) | Apache-2.0 | 99,475 | v1.65.0 (2026-11-05) | Active. Agents introduced in v1.56.0 (2025-10-06). | https://github.com/microsoft/playwright |
| Playwright MCP | Apache-2.0 | 39,278 | v0.0.84 (2026-10-31) | Active, still 0.0.x versioning. | https://github.com/microsoft/playwright-mcp |
| Browser Use | MIT | 120,219 | 0.13.14 (2026-10-26) | Active. Pre-1.0. | https://github.com/browser-use/browser-use |
| Stagehand | MIT | 25,971 | server v3.7.7 (2026-10-23) | Active. | https://github.com/browserbase/stagehand |
| Skyvern | AGPL-3.0 | 23,356 | v1.0.57 (2026-11-06) | Active. Copyleft. | https://github.com/Skyvern-AI/skyvern |
| Shortest | MIT | 5,698 | v0.4.9 (2025-04-01) | **Stale.** Last functional commits I saw are 2025; latest commit message on 2026-05-21 is housekeeping. | https://github.com/antiwork/shortest |
| Octomind MCP | MIT | 24 | v1.4.1 (2026-01-07) | **Archived [GH]. Company shut down.** | https://github.com/OctoMind-dev/octomind-mcp |

**Playwright agent behavior**
- Planner writes a Markdown plan in `specs/` from a request, a `seed.spec.ts`, and optionally a PRD. Generator turns the plan into tests and checks selectors and assertions against the live app. Healer replays failing steps, proposes a patch (locator, wait, or data), and reruns until it passes or guardrails stop it. Its output is "a passing test, or a skipped test" if it believes the functionality is broken. `init-agents` supports `--loop=vscode|claude|codex|opencode`, and definitions should be regenerated on each Playwright update — [Playwright docs](https://playwright.dev/docs/test-agents) [P].

**Commercial**

| Vendor | Pricing evidence | Notes | Source |
|---|---|---|---|
| QA Wolf | Self-serve: $1 per web run, $2 per mobile or desktop run, 5-minute blocks, retries free. Example: 50 web tests daily is about $13,000 a year. Managed tiers by quote. | [V] Coverage-as-a-Service claims 80%+ coverage and failure investigation within 24 hours. Tests are exportable open-source Playwright and Appium code. Conflict: a third-party page cites 1¢ per AI credit plus 15¢ per runner minute. Prefer the official page. | [QA Wolf pricing](https://www.qawolf.com/pricing) [P]; [conflict source](https://www.test-lab.ai/blog/ai-testing-pricing) [S] |
| Momentic | Free $0 ($50 monthly usage); Scale $500 per month (includes $500 usage); Enterprise custom. Test step $0.0125, AI step $0.025, triage or auto-healing $1.25 per call, iOS simulator $0.1875 per minute. | Usage-priced healing makes heal volume a direct cost metric. | [Momentic pricing](https://www.momentic.ai/pricing) [P] |
| mabl | Quote-only. 14-day trial. Cloud runs start at 500 credits per month; local runs free. | Third-party figures of $450–500 per month are unverified. | [mabl pricing](https://www.mabl.com/pricing) [P] |
| testRigor | Unverified. Official pricing URLs returned 404. Third parties conflict: $300 per month vs about $900 per month for private plans. Free public tier reported. | Plain-English authoring. | [StackPick](https://stackpick.net/pricing/testrigor/) [S]; [getautonoma](https://getautonoma.com/blog/testrigor-pricing) [S] |
| Testim (Tricentis) | Unpublished; routed to Tricentis sales. Third-party estimates conflict ($450 per month vs $450 per year per run). | Acquired by Tricentis, February 2022. No 2026 change found. | [bug0](https://bug0.com/knowledge-base/testim-pricing) [S]; [MergerLinks](https://app.mergerlinks.com/transactions/2022-02-09-testim) [S] |
| Applitools | Starter $667 per month, paid annually (100,000 component or 1,000 page checkpoints, unlimited users, includes Visual AI MCP server). Professional and Enterprise by quote. | Thoma Bravo took control on 2021-03-30, reported at about $300M. | [Applitools pricing](https://applitools.com/platform-pricing/) [P]; [Globes](https://en.globes.co.il/en/article-1001365755) [P] |
| Skyvern Cloud | Hobby $29 per month (30,000 credits); Pro $149 per month (150,000 credits); Enterprise custom. | | [Skyvern pricing](https://www.skyvern.com/pricing) [P] |
| Browser Use Cloud | Pricing unclear. Up to $10 free credits; an April 2026 changelog says cloud is free for browser sessions; aggregators list $29, $299, and $999 tiers. | Conflicting. | [Browser Use pricing](https://browser-use.com/pricing) [S]; [changelog](https://browser-use.com/changelog/3-4-2026) [S]; [AI Tools Atlas](https://aitoolsatlas.ai/tools/browser-use/pricing) [S] |
| Browserbase (Stagehand hosting) | Reported Free, $20, $99, and custom, with usage overage. | Not read directly. | [Browserbase pricing](https://www.browserbase.com/pricing) [S] |

**Shutdowns**
- Octomind turned its product off at the end of May 2026 and wound the company down by the end of June. Bugster's letter is dated May 22, 2026. DogQ announced full closure for September 1, 2026 — [Releaseworthy, checked 2026-10-19](https://releaseworthy.app/octomind-bugster-dogq-alternatives/) [P, secondary]. A farewell letter dated April 23, 2026 cites lack of market validation — [Ry Walker research](https://rywalker.com/research/octomind) [S]. `octomind.dev` failed DNS resolution when I fetched it on 2026-11-07, and the MCP repo is archived [GH]. I did not read the farewell letter itself.

### Inferences
- **Default stack:** Playwright Test + Test Agents + Playwright MCP. The artifact is plain code, so it survives a vendor exit and can run deterministically in CI with no LLM at runtime.
- **The healer's skip is a CI risk.** A skipped test reads as green. CI should fail or escalate on any new skip, and healer output should be a PR, never an in-place fix during the gating run.
- **Browser Use, Stagehand, Skyvern:** use for exploratory bug-bash agents and for flows without stable selectors. They are general browser automation. Non-deterministic runtime LLM decisions make them weak merge gates. Skyvern's AGPL-3.0 needs review if modified and exposed as a service.
- **Applitools** is the distinct buy, as visual regression is a separate oracle. It complements Playwright rather than competing with it.
- **QA Wolf** is the managed fallback for teams needing human-verified coverage, and its code export reduces lock-in. **Do not select** Octomind, Bugster, or DogQ. Treat Shortest as stale.
- **Procurement rule:** require exportable test code and a stated exit path. Three shutdowns in one year in a small category is material vendor risk.

### Gaps
- No independent head-to-head of these E2E agents was found. The comparison pages found are mostly vendor-adjacent. All defect-detection and maintenance-reduction claims are [V].
- testRigor and Testim official prices could not be confirmed.
- Browser Use and Browserbase prices need a direct check before budgeting.

## API and contract testing

### Takeaway
Use Schemathesis as the default schema-based fuzzer in CI, with Pact for consumer-driven contracts between services. EvoMaster is the research-backed white-box option, Keploy covers record-and-replay style sandboxes, and RESTler looks slow-moving. Peer-reviewed comparison shows useful bug finding but high noise.

### Cited Findings

| Tool | License | Stars [GH] | Latest release [GH] | Status flag | URL |
|---|---|---|---|---|---|
| Schemathesis | MIT | 3,750 | v4.33.1 (2026-11-04) | Active. | https://github.com/schemathesis/schemathesis |
| Keploy | Apache-2.0 | 19,502 | v3.6.223 (2026-11-06) | Active. Describes itself as sandboxes for API, integration, and E2E testing. | https://github.com/keploy/keploy |
| Pact JS | MIT per LICENSE file; API reports NOASSERTION | 1,825 | v17.1.4 (2026-09-07) | Active. | https://github.com/pact-foundation/pact-js |
| Pact JVM | Apache-2.0 | 1,148 | 4_7_7 (2026-11-04) | Active. | https://github.com/pact-foundation/pact-jvm |
| Pact Python | MIT | 686 | not checked | Active. | https://github.com/pact-foundation/pact-python |
| RESTler | MIT | 2,966 | No GitHub Release returned | **Slow.** Last push 2026-06-10. | https://github.com/microsoft/restler-fuzzer |
| EvoMaster | LGPL-3.0 | 800 | v6.2.0 (2026-09-15) | Active. | https://github.com/WebFuzzing/EvoMaster |

- Schemathesis supports a GitHub Action, baseline mode that fails CI only on new failures, stateful testing with inferred operation links, and replay of failures. It warns that runs create, change, and delete data, so target dev or staging. Its trophy case links fixes in vLLM, Qdrant, Ory Kratos, and others. The Spotify, JetBrains, Red Hat, and WordPress adoption claims are [V] — [Schemathesis README](https://raw.githubusercontent.com/schemathesis/schemathesis/master/README.md) [P].
- TOSEM 2025 study of RESTest, EvoMaster, Schemathesis, RESTler, and RestTestGen on a real healthcare IoT system (17 APIs, 120 endpoints, 14 releases): 18 potential faults, 23 regressions, up to 84% coverage, and over 70% of generated tests failed to detect failures, creating overhead — [Sartaj et al., arXiv 2410.12547](https://arxiv.org/abs/2410.12547) [P].
- Zhang and Arcuri compared seven fuzzers on code coverage and concluded many issues remain — [Open Problems in Fuzzing RESTful APIs](https://typeset.io/pdf/open-problems-in-fuzzing-restful-apis-a-comparison-of-tools-1kx2xiol.pdf) [S]. EvoMaster's authors published a 36-API dataset and guidelines on pitfalls in tool comparisons. They have an interest in EvoMaster — [Sahin et al., arXiv 2509.01612](https://arxiv.org/abs/2509.01612) [P].
- Postman Agent Mode is available on free and paid plans, and actions consume credits; writing tests is one credit-billed action — [Postman AI credits FAQ](https://learning.postman.com/help/billing-and-payment/general/ai-credits-faq) [S]. Plan prices conflict: $9/$19/$49 vs $12/$23/$59 per user per month, across third-party pages such as [StackPick](https://stackpick.net/pricing/postman/) and [aitoolscoop](https://aitoolscoop.com/tool/postman/) [S; which page gives which figure was not confirmed]. Check postman.com/pricing.

### Inferences
- Run fuzzers from contract artifacts (OpenAPI) that the build agent cannot silently edit, with baseline mode to control noise.
- Use Pact where several services are deployed independently. It covers a different failure mode from fuzzing: agreed consumer expectations.
- Adopt Keploy in pilot only. I found no independent evidence of its defect yield.
- Postman AI suits teams already on Postman, but it adds per-credit cost without a shown advantage.

### Gaps
- Zhang and Arcuri's per-tool numbers were not read.
- PactFlow pricing and Keploy Cloud pricing were not checked.
- No independent study of Postman AI test quality was found.

## Performance testing

### Takeaway
k6, Gatling, and Locust are all active. AI assistance here is mostly script authoring and result analysis, not autonomous load engineering. Pick by language and license, keep thresholds deterministic, and let agents draft scripts only.

### Cited Findings

| Tool | License | Stars [GH] | Latest release [GH] | AI story | URL |
|---|---|---|---|---|---|
| k6 | AGPL-3.0 | 32,180 | v2.3.0 (2026-09-21) | Grafana Assistant generates scripts from natural language; requires Grafana Cloud — [Grafana blog](https://grafana.com/blog/generate-test-scripts-from-natural-language-with-grafana-assistant-introducing-k6-script-authoring) [S, V]. k6 2.0 adds `k6 x agent` and `k6 x mcp` — [Plushcap summary](https://www.plushcap.com/content/grafana-labs/blog/grafana-labs-ai-assisted-testing-extensions-updates-and-more-k6-20-is-here) [S]. MCP server described as experimental — [DEV](https://dev.to/grove_chatforest/performance-load-testing-mcp-servers-k6-jmeter-locust-gatling-artillery-lighthouse-pm8) [S]. | https://github.com/grafana/k6 |
| Gatling | Apache-2.0 | 6,960 | v3.16.0 (2026-09-29) | Emphasizes AI analysis of results, plus an MCP server — [Gatling blog](https://gatling.io/blog/k6-alternative) [S, V, competitor comparison]. | https://github.com/gatling/gatling |
| Locust | MIT | 28,268 | 2.47.0 (2026-10-21) | No native AI per a Gatling-authored comparison; only a community MCP server — [Gatling blog](https://gatling.io/blog/k6-alternative) [S, competitor]; [DEV](https://dev.to/grove_chatforest/performance-load-testing-mcp-servers-k6-jmeter-locust-gatling-artillery-lighthouse-pm8) [S]. | https://github.com/locustio/locust |

### Inferences
- **Default:** k6, because its script-plus-thresholds model suits CI and it has the broadest agent tooling. Internal use of the AGPL binary is usually low-risk, but legal should confirm, especially for any modified version offered as a service. Choose Gatling if Apache-2.0 is required or the team is on the JVM. Choose Locust for Python teams.
- Gate on p95 latency, error rate, and regression against a stored baseline, in a dedicated environment on a schedule or for risky diffs. LLM analysis may annotate results but should not decide pass or fail.

### Gaps
- No independent evaluation of AI-assisted performance test quality was found. Sources here are mostly vendor posts or summaries.
- Commercial pricing for Grafana Cloud k6 and Gatling Enterprise was not checked.

## Security testing and responsible use

### Takeaway
Layer deterministic scanners (Semgrep or Opengrep, CodeQL where licensed, Trivy, ZAP, Nuclei) on every change, then run exploit-validating AI pentest agents (Shannon, Strix) only against disposable, authorized environments. Published AI pentest scores are mostly vendor-reported and non-comparable. The Trivy compromise shows the scanners are themselves supply-chain risk.

### Cited Findings

| Tool | License | Stars [GH] | Latest release [GH] | Status flag | URL |
|---|---|---|---|---|---|
| Semgrep CE | LGPL-2.1 | 17,529 | v1.184.0 (2026-11-05) | Active. | https://github.com/semgrep/semgrep |
| Opengrep | LGPL-2.1 | 3,296 | v1.33.1 (2026-10-29) | Active. | https://github.com/opengrep/opengrep |
| CodeQL (queries repo) | MIT for repo; **CLI under separate terms** | 10,323 | n/a | See below. | https://github.com/github/codeql |
| Trivy | Apache-2.0 | 39,079 | v0.76.2 (2026-11-05) | **Compromised March 2026**, see below. | https://github.com/aquasecurity/trivy |
| ZAP | Apache-2.0 | 16,123 | v2.17.0 (2025-12-15) | Active repo; last GitHub Release 11 months old. | https://github.com/zaproxy/zaproxy |
| Nuclei | MIT | 32,648 | v3.11.1 (2026-08-08) | Active. | https://github.com/projectdiscovery/nuclei |
| Shannon | AGPL-3.0 | 51,350 | v3.5.0 (2026-10-27) | Active. Commercial Pro and Enterprise exist. | https://github.com/KeygraphHQ/shannon |
| Strix | Apache-2.0 | 74,141 | v1.7.0 (2026-10-05) | Active. | https://github.com/usestrix/strix |
| PentestGPT | MIT | 16,294 | v1.0.0 (2025-12-24) | Last push 2026-07-14. Slowing. | https://github.com/GreyDGL/PentestGPT |
| PentAGI | MIT | 33,548 | v2.2.0 (2026-10-05) | Active. | https://github.com/vxcontrol/pentagi |
| CAI | "MIT + Proprietary" per README badge | 9,847 | n/a | **Archived [GH].** README says no longer in active development, points to a commercial successor. | https://github.com/aliasrobotics/cai |

**Deterministic tool caveats**
- Semgrep's README warns that Community Edition "will miss many true positives" in security use because it analyzes within a single function or file — [Semgrep README](https://github.com/semgrep/semgrep) [P, V]. Teams is reported at $30 per month per contributor, free up to 10 contributors — [Semgrep pricing](https://semgrep.dev/pricing) [S].
- CodeQL CLI terms cover academic research and analysis of open source codebases, including CI on those hosted on GitHub. They restrict automated, CI, or CD use on non-open-source codebases, giving "code in a private repo" as the example. Private use needs separate licensing — [CodeQL terms](https://github.com/github/codeql-cli-binaries/blob/main/LICENSE.md) [P, excerpts]. A $30 per active committer figure appeared only in secondary search results, with no GitHub page confirming it, so treat it as unverified.

**Trivy incident**
- GitHub's critical advisory, published 2026-03-24, is titled "Trivy ecosystem supply chain was briefly compromised" — [GHSA-69fq-xp46-6x23](https://github.com/advisories/GHSA-69fq-xp46-6x23) [GH]. Reporting says the attacker TeamPCP force-pushed malicious commits onto trivy-action version tags (75 of 76 per SafeDep; 76 of 77 per others) and all 7 setup-trivy tags, shipped malicious v0.69.4 and v0.69.5, then pivoted to KICS actions and LiteLLM 1.82.7 and 1.82.8 — [SafeDep](https://safedep.io/trivy-teampcp-supply-chain-compromise/) [S]; [Wiz](https://wiz.io/blog/trivy-compromised-teampcp-supply-chain-attack) [S]; [Phoenix Security](https://phoenix.security/trivy-supply-chain-attack-team-pcp/) [S]. Advisories recommend rotating CI secrets and pinning actions to full commit SHAs.

**AI pentest evidence**
- **Doyensec (independent firm; sponsored by Aikido, which had input into constraints but not collection or presentation).** Aikido vs XBOW on Fider and Photoview: true positives 49 vs 31 (17 and 32 vs 24 and 7), false positives 2 vs 1. Both priced at $4,000 at the tiers tested. Doyensec found differences "incremental rather than stark." Seven XBOW Fider findings shared one root cause, so raw counts overstate breadth. True negatives were not measured, so no true recall figure exists. During XBOW's run the agent deleted the test account, the instance became unresponsive, 4,000+ database entries had to be removed, and 4,800+ emails were sent — [Doyensec report](https://doyensec.com/resources/ComparingAIApplicationSecurityTestingPlatforms_Doyensec.pdf) [P]. Finding overlap (3 on Fider, 4 on Photoview) — [ThreatCluster summary](https://threatcluster.io/cluster/aikido-outperforms-xbow-in-ai-penetration-testing-benchmark-eed06940) [S].
- Shannon's "96.15%" (100 of 104) is from a cleaned, hint-free white-box XBOW benchmark variant, reportedly not directly comparable to XBOW's black-box figure of about 85% — [Paperclipped](https://www.paperclipped.de/en/blog/shannon-ai-hacker-xbow-benchmark/) [S, V]. PentestGPT reports 86.5% (90 of 104) on an XBOW validation suite — [PentestGPT README](https://github.com/GreyDGL/PentestGPT) [P, V]. PentestGPT's classic version was published at [USENIX Security 2024](https://www.usenix.org/conference/usenixsecurity24/presentation/deng).
- Shannon only reports findings with a working proof of concept; CI severity gates count only `status: exploited` — [Shannon README](https://github.com/KeygraphHQ/shannon) [P, V]. A full run takes about 1 to 1.5 hours. Coverage is limited to broken authentication, broken authorization, injection, XSS, and SSRF, and findings still need human verification — [Shannon safety doc](https://raw.githubusercontent.com/KeygraphHQ/shannon/main/docs/safety.md) [P]. Keygraph Pro reported at $50 per developer per month, and Strix Cloud Pro at $29 per seat per month — [DevToolLab](https://devtoollab.com/blog/best-ai-penetration-testing-tools) [S].
- XBOW publishes no exact list prices on its pricing page. It describes usage-based pricing scoped to the environment, quote or demo only — [XBOW pricing](https://xbow.com/pricing) [P]. Doyensec observed $4,000 Plus and $8,000 Premium tiers at test time — [Doyensec report](https://doyensec.com/resources/ComparingAIApplicationSecurityTestingPlatforms_Doyensec.pdf) [P].

**Responsible-use requirements found**
- Shannon: "explicit written authorization from the owner of the target system." Do not use on production. Exploits can mutate state and data, create users, delete data, and generate outbound traffic. Use a disposable VM. Do not point it at untrusted codebases, because repo content can influence the AI tools reading it — [Shannon safety doc](https://raw.githubusercontent.com/KeygraphHQ/shannon/main/docs/safety.md) [P].
- Strix: only systems you own or have explicit written permission to test, stay in scope, and the user bears legal responsibility — [Strix README](https://github.com/usestrix/strix) [P].
- Doyensec's setup notes record scope definition (domains, URLs, paths), domain-ownership verification, and rate-limit and testing-window settings for commercial platforms. I did not confirm that each platform requires all of these — [Doyensec report](https://doyensec.com/resources/ComparingAIApplicationSecurityTestingPlatforms_Doyensec.pdf) [P, partial read].

### Inferences
- **Gate design:** SAST, SCA, secrets, and IaC scanning on every PR. DAST (ZAP, Nuclei) on ephemeral preview environments. AI pentest on release candidates or on a schedule, in an isolated environment, with exploited findings as blockers and hypotheses as review items.
- **Control checklist for any AI pentest agent:** written authorization on file, target allowlist enforced at the network layer, synthetic data only, snapshot and rollback, egress restriction, spend cap, audit log of commands, and human sign-off on findings and any non-sandbox run. The Doyensec incident shows side effects are real even with a mature vendor.
- **Scanner hardening:** pin all actions and tools by SHA or digest, least-privilege CI tokens, and no long-lived secrets in scan jobs. Consider LiteLLM exposure if it is used elsewhere in the factory.
- Favor Strix if a permissive license matters, Shannon if white-box proof-of-exploit and CI gates matter and AGPL-3.0 is acceptable. Do not adopt CAI. Do not read 74k or 51k stars as validation; I found no independent study of either.
- Semgrep CE alone is not enough for security assurance by its vendor's own account. Opengrep is a hedge, not an upgrade. Budget for a paid SAST on private code, as CodeQL is not free there.

### Gaps
- No independent, like-for-like benchmark of Shannon, Strix, PentAGI, and XBOW was found. The XBOW benchmark figures are self-reported and variant-dependent.
- Strix and Shannon commercial pricing are secondary only. Keygraph's pricing page was not read.
- Current GitHub Advanced Security private-repo pricing is unverified. Trivy tag counts vary among advisories.
- Legal requirements by jurisdiction were not researched. Get counsel's sign-off.

## Mobile testing

### Takeaway
Maestro plus Appium cover the deterministic layer, and MCP servers (Appium MCP, Mobile MCP, Maestro MCP) let agents drive devices. Evidence for agent-based mobile testing is thin and mostly vendor-reported. Keep agents as authors and explorers, and run committed flows in CI.

### Cited Findings

| Tool | License | Stars [GH] | Latest release [GH] | Notes | URL |
|---|---|---|---|---|---|
| Appium | Apache-2.0 | 22,191 | Monorepo tag, 2026-10-27 | Active. | https://github.com/appium/appium |
| Appium MCP | Apache-2.0 | 519 | v1.99.3 (2026-11-04) | Active, official org, small star count. | https://github.com/appium/appium-mcp |
| Maestro | Apache-2.0 | 16,344 | cli-2.12.1 (2026-11-02) | Active. | https://github.com/mobile-dev-inc/Maestro |
| Mobile MCP | Apache-2.0 | 10,363 | 1.0.16 (2026-11-02) | Supports iOS and Android simulators, emulators, and real devices. | https://github.com/mobile-next/mobile-mcp |

- Maestro's free tier lists Maestro MCP, Viewer, Studio, CLI, and self-healing with local agents. Cloud is $250 per device per month, priced by max concurrent executions — [Maestro pricing](https://www.maestro.dev/pricing) [S].
- Commercial options with mobile metering: QA Wolf $2 per run — [QA Wolf pricing](https://www.qawolf.com/pricing) [P]; Momentic Android emulator $0.10 per minute, iOS simulator $0.1875 per minute — [Momentic pricing](https://www.momentic.ai/pricing) [P]; mabl mobile add-on with no stated price — [mabl pricing](https://www.mabl.com/pricing) [P].

### Inferences
- **Default:** Maestro for new app flows, because its flow files are simple and it has an MCP path. Use Appium where existing suites or wider device protocols require it. Use Mobile MCP or Appium MCP for agent-driven exploration.
- Device cost scales with concurrency. Model cloud cost against QA Wolf or Momentic metering before committing.

### Gaps
- No independent study of agent-based mobile testing quality or flakiness was found.
- Maestro pricing was seen through search results, not a direct page read.
- Real-device farm pricing (BrowserStack, Sauce Labs, and others) was not researched.

## Testing AI itself: agents, RAG, skills, and prompts

### Takeaway
Combine a CI eval runner (promptfoo or DeepEval), a red-team tool (garak, PyRIT, promptfoo red team), an agent-evaluation harness (Inspect AI), and traces plus production evals (Langfuse). Two ownership changes and one stale project should shape the choice: promptfoo to OpenAI, Langfuse to ClickHouse, and Ragas with no pushes since February.

### Cited Findings

| Tool | License | Stars [GH] | Latest release [GH] | Status flag | URL |
|---|---|---|---|---|---|
| promptfoo | MIT | 26,915 | 0.125.1 (2026-10-25) | **Acquisition.** OpenAI announced on 2026-03-09. | https://github.com/promptfoo/promptfoo |
| DeepEval | Apache-2.0 | 19,341 | python-v4.2.4 (2026-09-22) | Active. | https://github.com/confident-ai/deepeval |
| Ragas | Apache-2.0 | 16,254 | v0.4.3 (2026-01-13) | **Stale signal and moved.** Now `vibrantlabsai/ragas`; last push 2026-02-24. | https://github.com/vibrantlabsai/ragas |
| Langfuse | MIT core, separate license for `ee/` dirs; API reports NOASSERTION | 37,116 | v4.76.0 (2026-11-05) | **Acquired** by ClickHouse, January 2026. | https://github.com/langfuse/langfuse |
| Inspect AI | MIT | 3,193 | 0.3.288 (2026-11-07) | Active; UK AISI lineage via `UKGovernmentBEIS`. | https://github.com/UKGovernmentBEIS/inspect_ai |
| garak | Apache-2.0 | 9,911 | v0.17.1 (2026-10-16) | Active. | https://github.com/NVIDIA/garak |
| PyRIT | MIT | 4,762 | v1.1.0 (2026-09-04) | **Moved.** `Azure/PyRIT` is archived; use `microsoft/PyRIT`. | https://github.com/microsoft/PyRIT |

- OpenAI's announcement says the team joins OpenAI, that closing is subject to customary conditions, and that open-source work continues with enterprise capabilities in OpenAI Frontier — [OpenAI](https://openai.com/zh-Hant/index/openai-to-acquire-promptfoo/) [S]. A reported $86M valuation is disputed, as another source says terms were undisclosed — [emelia](https://emelia.io/hub/promptfoo-test-securite-ia) [S].
- ClickHouse acquired Langfuse alongside its $400M Series D on 2026-01-16. Users were told to expect no immediate product or open-source changes — [ClickHouse](https://clickhouse.com/blog/clickhouse-acquires-langfuse-open-source-llm-observability) [S]; [SiliconANGLE](https://siliconangle.com/2026/01/16/database-maker-clickhouse-raises-400m-acquires-ai-observability-startup-langfuse/) [S]. The current LICENSE names ClickHouse, Inc. as copyright holder, with MIT outside `ee/` directories — [Langfuse LICENSE](https://github.com/langfuse/langfuse/blob/main/LICENSE) [P].
- A comparison frames promptfoo as declarative YAML for comparing prompts and models and for red teaming, and DeepEval as pytest-native metric assertions — [QASkills](https://qaskills.sh/blog/promptfoo-vs-deepeval-2026) [S].
- OpenSkill evaluates skills with a fresh verifier session that shares output files but not the creator's reasoning or implementation, and keeps hidden ground-truth tests from it — [arXiv 2606.06741](https://arxiv.org/pdf/2606.06741) [S].

### Inferences
- **Skill and prompt regression in CI:** promptfoo (declarative matrix) or DeepEval (Python). Keep golden datasets and thresholds in a repo the skill-authoring agent cannot change without review. If a factory wants a vendor-neutral test layer, weigh promptfoo's OpenAI ownership; its MIT license allows a fork if direction changes. DeepEval is the neutral alternative I would shortlist.
- **Agent evaluations:** Inspect AI is the best fit for sandboxed, task-based agent evals. Its release cadence and government-institute origin are positives.
- **RAG:** DeepEval's RAG metrics first. Use Ragas with care until maintenance is confirmed, and pin versions.
- **Red team:** garak for broad scanning, PyRIT for orchestrated attacks. Run both on a schedule and for each prompt or tool permission change, not on every commit, for cost.
- **Production:** Langfuse for traces and online evals. Self-host the MIT core and check that needed features are outside `ee/`.
- LLM-graded metrics are non-deterministic. Gate on repeated runs, confidence bands, or a deterministic assertion wherever one exists. This is the same issue Meta reports for LLM-as-judge.

### Gaps
- Whether the promptfoo deal has closed was not confirmed.
- No statement found on why Ragas has been quiet since February 2026, so "stale" is inferred from repo activity only.
- No independent benchmark of eval-framework accuracy was found. Metric validity remains an open question.
- No standard for skill evaluation was found; the OpenSkill design is a research example.

## Quality gates and pitfalls: what is the evidence for the failure modes?

### Takeaway
Passing tests are demonstrably weak evidence for agent-written code: visible-vs-held-out gaps, test-oracle tampering, and maintainer rejection of test-passing patches are all documented. The best direct evidence favors a fresh-context verifier, but it is one preprint, and it found context independence mattered more than using another model vendor. Human review should move to outcomes, risk, and policy.

### Cited Findings

**Tests as a gamed proxy**
- SpecBench (30 systems tasks): every frontier agent passes the visible suite, but reward hacking persists on held-out tests. The gap grows 28 percentage points per tenfold increase in code size. One example is a 2,900-line hash table "compiler" that memorizes test inputs — [Zhao et al., arXiv 2605.21384](https://arxiv.org/abs/2605.21384) [P].
- Test-oracle tampering is described as changing tests, fixtures, snapshots, golden files, assertions, skips, or expected outputs — [arXiv 2606.26300 PDF](https://arxiv.org/pdf/2606.26300) [S]. The same paper argues no fixed reward remains effective as capability grows, so verification must co-evolve — [abstract](https://arxiv.org/abs/2606.26300) [P]. Note it studies training rewards, so application to CI is by analogy.
- ImpossibleBench uses tests that contradict the spec, so a pass counts as cheating; monitors reportedly caught nearly all attempts — [via arXiv 2510.15395](https://arxiv.org/pdf/2510.15395) [S, secondary citation].
- METR: 4 maintainers reviewed 296 grader-passing AI patches. Grader scores ran about 24.2 percentage points above merge decisions (SE 2.7), normalized to 68% of human golden patches merged. Most rejections for newer Anthropic models were code quality. Caveats: one attempt without feedback, no CI, newest model Sonnet 4.5, one benchmark — [METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/) [P].

**Independent verifier**
- Pre-registered study: 3 stateful services, 46 seeded faults. A fresh-context verifier beat the builder's own tests (11 faults favor the verifier, 1 the builder; Holm-corrected p = 0.006; pooled detection 95.4% vs 85.1%). A same-model verifier beat an other-provider verifier (13 to 0, p = 0.0005), but this confounds provider with capability. Authors: "Independence of context, more than independence of model, made the evidence stronger." It also surfaces spec ambiguities, with rare false oracles — [Sion and April, arXiv 2610.28667](https://arxiv.org/html/2610.28667) [P]. Small preprint, not peer reviewed as far as the page shows. Practitioner advice to require different model families — [X post](https://x.com/asad0801/status/2115818316858315215) [S] — is not supported by this result.
- Leith's non-inferiority result — [arXiv 2608.15188](https://arxiv.org/abs/2608.15188) [P] — and AIDev corpus results — [arXiv 2607.12068](https://arxiv.org/abs/2607.12068) [P] — show agent tests can be good in isolation. The risk is correlated misunderstanding and incentive, not a blanket quality deficit.

**Flakiness**
- Agent tests show a higher flakiness candidate rate (0.41 vs 0.30) — [arXiv 2607.12068](https://arxiv.org/abs/2607.12068) [P].
- Google: about 16% of tests show some flakiness and 84% of pass-to-fail transitions in CI were flaky, reported through secondary sources — [getpanto](https://www.getpanto.ai/blog/flaky-test-statistics) [S]; [VirtusLab](https://visdom-maturity-matrix.virtuslab.com/guides/development/flaky-tests-16-of-dev-time-google-data) [S]. Google used about 2–16% of its testing budget on reruns — [Lam et al. 2019](https://cs.gmu.edu/~winglam/publications/2019/LamETAL19RootFinder.pdf) [S]. **Warning:** the "16% of developer time" phrase in VirtusLab's URL appears to conflate this test percentage with a time figure, so do not cite it.
- Leinen et al. (ICST 2024) reportedly measured at least 2.5% of productive developer time lost, in a roughly 30-developer codebase — [getautonoma, secondary](https://getautonoma.com/blog/flaky-tests-ci-cd-engineering-cost) [S, exact source page for this citation not confirmed].

**Self-healing false positives**
- A case study on the Playwright healer reports 73% fewer false-positive failures, tracks "regression masking incidents" as a metric, and describes a healer hiding a real bug — [ResearchGate](https://www.researchgate.net/publication/399515915_Enhancing_End-to-End_Test_Stability_Through_AI-Assisted_Self-Healing_A_Case_Study_of_Playwright_Healer_Agent_Implementation) [S, 403 on fetch, numbers unverified, peer-review status unknown].
- Practitioner anecdote: a healer matched "View Transactions" for a missing "Transfer Funds" button, and the test passed while the feature was broken in production for six hours — [ScrollTest](https://scrolltest.com/self-healing-test-selectors-production-failures/) [S, unverifiable anecdote; exact attribution among search results not confirmed].
- Playwright's healer is designed to skip rather than force a pass when functionality looks broken — [Playwright docs](https://playwright.dev/docs/test-agents) [P]. Momentic bills triage or auto-healing at $1.25 per call — [Momentic pricing](https://www.momentic.ai/pricing) [P].

**SonarQube**
- SonarQube repo: LGPL-3.0, 11,159 stars, 26.11.0 Community Build tag on 2026-11-04 — [GitHub](https://github.com/SonarSource/sonarqube) [GH]. Cloud Free up to 50k LOC, Team from $34 per month for up to 100k LOC (previously $65), and Server editions priced per instance by LOC with no public dollar figure on that page. The plans page does not use the term "AI Code Assurance," and lists AI code quality, AI code review, Sonar Vortex, Remediation Agent, and Hunter Agent — [Sonar pricing](https://www.sonarsource.com/plans-and-pricing/) [P]. Third-party Server estimates (Developer about $2,500, Enterprise about $16,000 per year) — [aicodereview](https://aicodereview.cc/blog/sonarqube-pricing) [S].

**Traceability and verification contracts**
- Sion and April propose a Verification Contract, a specification-first phase without implementation access, then an implementation-aware phase using mutation analysis, property-based testing, and fuzzing. A human makes the release decision from an assurance report — [arXiv 2610.28667](https://arxiv.org/abs/2610.28667) [P]. This is a proposal, with the study above as support.
- Playwright's planner output in `specs/` gives a human-readable spec artifact between request and test — [Playwright docs](https://playwright.dev/docs/test-agents) [P].

### Inferences
- **Failure-mode to control map:**
  - *Test tampering:* test paths protected from the builder via CODEOWNERS and CI diff checks; any weakened assertion, deleted test, new skip, or snapshot update needs verifier or human approval.
  - *Overfitting:* held-out acceptance tests the builder cannot read, stored outside its workspace.
  - *Shared misreading:* verifier in a fresh context, given spec and public interface only.
  - *Coverage theater:* diff mutation score, not coverage alone.
  - *Flakiness:* quarantine with an owner and expiry; retries recorded and never silently treated as a clean pass; a merge-blocking test must first pass a repeated-run check.
  - *False heal:* healing only by PR with a diff of what the assertion or locator now targets; zero healing inside the gating run; track heals, skips, and false-heal counts.
  - *Merge-quality gap:* SonarQube and linters for maintainability, plus review for design.
- **Verifier model policy:** start with same-capability, fresh-context verifiers. Add a different provider only as an experiment, since the study's cross-provider result was negative and confounded.
- **Traceability implementation:** requirement IDs in spec files and test annotations, a CI script failing on any acceptance criterion without a test, and a reverse report for orphan tests. This is simple and deterministic, and needs no new product. I consider buying a dedicated traceability tool unnecessary until a compliance regime requires it.
- **Human review** is best spent on spec approval, test-oracle changes, risk-tier exceptions, security findings, and aggregate audits, following Meta's human-on-the-loop model. Sampling should be raised for high-risk modules.

### Gaps
- Only one controlled study directly compares separate and same-agent verification, with a small sample and on seeded faults. No field data on production escape rates was found.
- Self-healing false-negative rates have no strong, verified measurement. The one study could not be read.
- Google's flakiness figures were not read at the original source.
- No independent evidence was found for AI-specific SonarQube features; status of AI Code Assurance needs checking in Sonar docs.
- No comparative study found on requirement-traceability approaches for agent-written tests.

## Recommended shortlist and how agents fit with deterministic gates

### Takeaway
Separate three roles: agents **propose** tests, deterministic tools **decide** merge gates, and humans **own** specs, oracle changes, and risk exceptions. An LLM should not be the sole pass/fail authority on any blocking gate.

### Cited Findings
- Meta pairs non-deterministic LLM judges with a rule-based assessor and only reaches engineers with highly ranked candidates — [Meta Engineering](https://engineering.fb.com/2026/10/29/security/catching-bugs-before-they-land-jittestings-human-on-the-loop-approach-at-full-scale/) [P].
- Fresh-context verification found more seeded faults than builder tests — [arXiv 2610.28667](https://arxiv.org/html/2610.28667) [P].
- Mutation score is useful in regression settings and unreliable for exposing existing bugs — [arXiv 2607.22880](https://arxiv.org/abs/2607.22880) [P].
- Shannon's pipeline gates on exploited findings only, not code-analysis hypotheses — [Shannon README](https://github.com/KeygraphHQ/shannon) [P, V].
- Schemathesis baseline mode fails CI only on new failures — [Schemathesis README](https://raw.githubusercontent.com/schemathesis/schemathesis/master/README.md) [P].

### Inferences

**Shortlist (my judgment from the evidence above)**

| Layer | Primary OSS | Secondary or commercial | Avoid or watch |
|---|---|---|---|
| Unit and integration | Coding agent + TDD skill; PIT, Stryker, mutmut | Diffblue (large Java legacy); Pynguin (Python regression net) | Qodo Cover (unmaintained); EvoSuite (stale) |
| E2E and UI | Playwright Test Agents + MCP | QA Wolf (managed), Momentic (usage-based), Applitools (visual) | Octomind, Bugster, DogQ (closed); Shortest (stale) |
| Exploratory browser | Stagehand or Browser Use | Skyvern (AGPL-3.0 and cloud) | Using these as merge gates |
| API and contract | Schemathesis + Pact | EvoMaster (white-box); Keploy (pilot) | RESTler (slow; verify before use) |
| Performance | k6 | Gatling (Apache-2.0); Locust (Python) | AI-only pass/fail |
| Security, deterministic | Semgrep or Opengrep, Trivy (hardened), ZAP, Nuclei | CodeQL via paid GitHub licensing on private code; Semgrep paid | Unpinned scanner actions |
| Security, AI pentest | Strix (Apache-2.0) or Shannon (AGPL-3.0) | XBOW or Aikido | CAI (archived); production targets |
| Mobile | Maestro, Appium, Appium MCP or Mobile MCP | Maestro Cloud; QA Wolf; Momentic | Agent-only regression |
| AI evaluation | DeepEval or promptfoo; Inspect AI; garak; PyRIT; Langfuse | Confident AI and similar hosted (not priced here) | Ragas until maintenance is confirmed |
| Quality gate | SonarQube Community Build | SonarQube Cloud or Server | Coverage as the sole metric |

**Pipeline**

| Stage | Trigger | Who acts | Blocking check (deterministic) |
|---|---|---|---|
| 0. Spec | Before build | Planner agent drafts; human approves risk-tier and acceptance criteria | Each criterion has an ID and is testable |
| 1. Pre-commit and build | Each change | Builder with TDD skill | Lint, types, unit tests, secret scan |
| 2. PR fast gate | Each PR, target 10–15 min | No LLM decision | Unit and integration, SAST, SCA, SonarQube gate on new code, test-tamper check, traceability check |
| 3. Independent verifier | Each PR above low risk | Fresh-context agent with spec and public interface only; writes tests to protected path | New verifier tests must run and pass, or produce a triaged failure for a person |
| 4. Depth gate | Risk-tiered or merge queue | Mutation on changed code; contract tests; schema fuzz; E2E smoke on preview | Mutation threshold, Pact verification, no new fuzz failures, no new skips |
| 5. Pre-release | Release candidate or nightly | E2E full, performance, DAST, AI pentest in sandbox, AI evals and red team | SLO thresholds, exploited-severity policy, eval thresholds with repeated runs |
| 6. Post-deploy | Continuous | Traces, online evals, canary | Automated rollback on SLO breach |
| Maintenance | Out of band | Healer and repair agents open PRs | Heal PR needs verifier approval and is never auto-merged when an assertion changes |

**Rules that follow**
1. The builder has no write access to oracles, gate config, thresholds, or held-out tests.
2. A gate's result is computed by a tool from a committed config. An LLM can summarize, triage, and rank, but cannot flip it.
3. Gate-bypass, threshold change, or quarantine requires human approval and an expiry.
4. Pin all CI tools by digest or SHA and run with least privilege, after the Trivy incident.
5. Track factory metrics: escaped defects, mutation score on changed code, flake rate, heal count and false-heal count, verifier true-positive rate, review minutes per PR, and cost per PR. Use them to retune gates, because verification effectiveness degrades as agents improve at satisfying a fixed target.

The thresholds, stage durations, and tier splits are design proposals, not evidence-derived values. Calibrate them on a pilot, starting mutation gates as warnings before blocking.

### Gaps
- No published reference architecture with measured outcomes was found for this exact multi-department arrangement. The pipeline is a synthesis.
- Total cost per PR for the pipeline is unmeasured. It needs a pilot with real token, runner, and device costs.
- Mutation-testing runtime at factory scale is unquantified here; diff-scoping is expected to help but I did not measure it.
