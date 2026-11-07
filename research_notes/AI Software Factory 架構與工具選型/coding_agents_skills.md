# Coding Agents, Skills, and Methodologies per SDLC Stage for a Multi-Department AI Software Factory (as of 2026-11-07)

Reading notes for the report writer:
- All GitHub stars, licenses, last-push dates, and latest releases were read directly from the GitHub REST API on 2026-11-07 unless marked otherwise.
- Evidence labels used below: **[primary]** official docs, repo, filing, or the study itself. **[vendor]** a vendor's claim about itself or a rival. **[secondary]** blog or aggregator, not checked against the original. **[unverified]** single or unconfirmed source.
- Conflict-of-interest note: these notes were written by Claude, an Anthropic model, and Claude Code docs were the most accessible governance source. Treat the depth of Claude Code coverage as a reflection of what was readable, not as a ranking.

## Which coding agents exist, what are their properties, and which have been shut down, acquired, or renamed?

### Takeaway
Tool churn is now a primary selection risk: within 12 months Roo Code shut down, Cursor was acquired by SpaceX, Windsurf was renamed Devin Desktop, Gemini CLI lost individual accounts, and Amazon Q Developer's IDE plugins got an end-of-support date. Only some agents are both open source and active with model freedom (OpenCode, Goose, Cline, OpenHands, Qwen Code, Codex CLI, with caveats on each).

### Cited Findings

**Status changes (flags)**

| Tool | Event | Source and confidence |
|---|---|---|
| Roo Code | **Shut down.** README says the extension "was shut down on May 15th"; repo archived 2026-05-15. Recommends community fork Zoo Code, or Cline | [RooCodeInc/Roo-Code](https://github.com/RooCodeInc/Roo-Code) [primary] |
| Cursor | **Acquired.** SpaceX all-stock deal at about $60B, signed 2026-06-16, closed 2026-08-14; team joins SpaceXAI to work on Grok and Cursor | [Cursor on X](https://x.com/cursor_ai/status/2088249881718919393?lang=en) [primary]; [SpaceX 10-Q](https://www.sec.gov/Archives/edgar/data/0001181412/000162828026052535/spcx-20260630.htm) [primary]; closing date from [Wikipedia](https://en.wikipedia.org/wiki/Cursor_(company)) and [TNW](https://thenextweb.com/news/spacex-cursor-acquisition-completed-gpu-fleet) [secondary] |
| Windsurf | **Acquired, then renamed.** Cognition signed to acquire in July 2025; rebranded Devin Desktop in June 2026; Cascade replaced by Devin Local around 2026-07-01. One review gives a conflicting December 2025 / ~$250M purchase detail | [Wikipedia: Cognition](https://en.wikipedia.org/wiki/Cognition_AI); [The AI Agent Index](https://theaiagentindex.com/agents/windsurf); [Hackceleration](https://hackceleration.com/labs/review/windsurf) [secondary] |
| Gemini CLI | **Being replaced by Antigravity CLI (`agy`).** Announced 2026-05-19; stopped serving Google AI Pro/Ultra and free individual accounts 2026-06-18. Paid API keys and Gemini Code Assist Standard/Enterprise reportedly keep access. Repo still ships (v0.67.0, 2026-11-04) | [Google Developers Blog](https://developers.googleblog.com/en/an-important-update-transitioning-gemini-cli-to-antigravity-cli/); [GitHub discussion #28017](https://github.com/google-gemini/gemini-cli/discussions/28017); [repo README](https://github.com/google-gemini/gemini-cli) confirms the 2026-06-18 move [primary] |
| Amazon Q Developer | **IDE plugins end of support 2027-04-30**, new signups stopped 2026-05-15; AWS points users to Kiro. Console and chat-app Q continue | [AWS DevOps Blog](https://aws.amazon.com/blogs/devops/amazon-q-developer-end-of-support-announcement/); [AWS docs](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/q-developer-ide-end-of-support.md) (dates read via search summary) |
| Aider | **Stale, not dead.** Last release v0.86.0 on 2025-08-09; last push 2026-05-22; MCP support issue still open | [Aider-AI/aider](https://github.com/Aider-AI/aider); [issue #3314](https://github.com/Aider-AI/aider/issues/3314) [primary] |
| Goose | **Moved** from Block to the Agentic AI Foundation at the Linux Foundation (`block/goose` resolves to `aaif-goose/goose`) | [aaif-goose/goose](https://github.com/aaif-goose/goose); [LF press release](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation?hsLang=en) [primary] |
| OpenCode | **Repo moved** (`sst/opencode` resolves to `anomalyco/opencode`) | [anomalyco/opencode](https://github.com/anomalyco/opencode) [primary] |
| OpenHands | **Org renamed** (`All-Hands-AI` to `OpenHands`). Now positioned as "Agent Canvas", a self-hosted control center that can also run Claude Code, Codex, Gemini CLI, Pi, and OpenCode through ACP | [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) [primary] |

**Open-source and public-repo agents (GitHub API, 2026-11-07)**

| Agent | License | Stars | Latest release | Headless / CI | MCP / Skills | Models |
|---|---|---|---|---|---|---|
| [OpenCode](https://github.com/anomalyco/opencode) | MIT | 225,081 | v1.19.5 (2026-10-28) | [`opencode run`](https://opencode.ai/docs/cli) | [MCP](https://opencode.ai/docs/mcp-servers), [SKILL.md](https://opencode.ai/docs/skills) | Many providers, [local Ollama](https://opencode.ai/docs/providers) |
| [Claude Code](https://github.com/anthropics/claude-code) | **Proprietary** (LICENSE: © Anthropic, Commercial Terms) despite public repo | 154,810 | v2.1.319 (2026-11-06) | [`claude -p`, seeded CI plugins](https://code.claude.com/docs/en/plugins/org.md) | MCP, Skills, plugins | Claude only, through Anthropic, Bedrock, Google Agent Platform, Foundry, or an [LLM gateway](https://code.claude.com/docs/en/costs) |
| [Codex CLI](https://github.com/openai/codex) | Apache-2.0 | 133,493 | rust-v0.167.0 (2026-11-06) | [`codex exec`, SDK](https://learn.chatgpt.com/docs/pricing) | Skills catalog at [openai/skills](https://github.com/openai/skills); MCP not re-verified this session | OpenAI by default; API keys; other providers not re-verified |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | Apache-2.0 | 108,756 | v0.67.0 (2026-11-04) | Not re-verified | [Skills, hooks, subagents, extensions carried to Antigravity](https://developers.googleblog.com/en/an-important-update-transitioning-gemini-cli-to-antigravity-cli/) | Gemini; see flag above |
| [OpenHands](https://github.com/OpenHands/OpenHands) | MIT per GitHub (enterprise-directory terms not verified) | 92,839 | v1.37.0 (2026-11-06) | Agent Server REST API, SDK, automation | ACP agents; in [Agent Skills logos](https://github.com/agentskills/agentskills/tree/main/docs/images/logos) | Multi-provider, self-hosted backends |
| [Cline](https://github.com/cline/cline) | Apache-2.0 | 71,410 | v4.1.26 (2026-10-26) | Headless CLI, `@cline/sdk` | MCP, Rules and Skills | Managed in CLI; breadth not re-verified |
| [Goose](https://github.com/aaif-goose/goose) | Apache-2.0 | 56,314 | v1.57.0 (2026-10-28) | [`goose run`](https://goose-docs.ai/docs/guides/running-tasks) | MCP, [Agent Skills](https://goose-docs.ai/docs/guides/context-engineering/using-skills) | Multi-provider (not re-verified this session) |
| [Aider](https://github.com/Aider-AI/aider) | Apache-2.0 | 49,992 | v0.86.0 (2025-08-09) | [`--message`](https://aider.chat/docs/scripting.html) | No native MCP ([open issue](https://github.com/Aider-AI/aider/issues/3314)) | [Cloud and local](https://aider.chat/docs/llms.html) |
| [Qwen Code](https://github.com/QwenLM/qwen-code) | Apache-2.0 | 29,057 | v0.26.9 (2026-11-06) | `qwen -p` | MCP, Skills, hooks | OpenAI, Anthropic, Gemini, Qwen APIs, Ollama, vLLM |
| [Kilo Code](https://github.com/Kilo-Org/kilocode) | MIT | 27,883 | v7.8.27 (2026-11-05) | Not verified | Not verified | Not verified |
| [Roo Code](https://github.com/RooCodeInc/Roo-Code) | Apache-2.0 | 24,241 | v3.54.0, **archived** | n/a | n/a | n/a |
| [Zoo Code](https://github.com/Zoo-Code-Org/Zoo-Code) (Roo fork) | Apache-2.0 | 2,097 | v3.96.0 (2026-11-07) | Not verified | Not verified | Not verified |
| [Kiro repo](https://github.com/kirodotdev/Kiro) | No license detected; repo holds docs and scripts, no releases | 4,451 | none | See below | See below | See below |

**Commercial agents: pricing and controls**

| Agent | Pricing model | Enterprise controls and deployment | Headless / model notes |
|---|---|---|---|
| Claude Code | Token-based; seat allowances on Team/Enterprise or per-token on Console and clouds. Vendor states average about $13 per developer per active day, $150-250 per month, under $30 per active day for 90% of users | Managed settings (server, MDM, file), OTel, spend limits, Analytics API, self-hosted gateway | [Costs doc](https://code.claude.com/docs/en/costs) [vendor, self-reported averages] |
| Codex | ChatGPT plans: Plus $20, Pro from $100, Business $20/user annual ($25 monthly), Enterprise custom; credits priced per token | Enterprise: SCIM, EKM, Compliance API audit logs, retention and residency | API key described as suited to CI, but excludes cloud features such as GitHub code review. [Pricing](https://learn.chatgpt.com/docs/pricing) [primary] |
| GitHub Copilot | **Moved to usage-based AI Credits 2026-06-01**, token-based at published model rates. Code review also consumes Actions minutes | Org and enterprise policies, audit log for partner agents | [GitHub blog](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/) [primary]; Claude and Codex available as [third-party agents in public preview](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents), billed as credits plus Actions minutes [primary] |
| Cursor | Free; Plus $8 intro (**Grok models only**); Premium $30; Super $100; Ultra $200; Teams $30+/user; Enterprise custom. Third-party models billed at API list price | SSO, audit logs, SCIM, repo/model/MCP access controls (Enterprise) | [Pricing](https://cursor.com/pricing) [primary]; [headless print mode](https://cursor.com/docs/cli/headless), [SKILL.md](https://cursor.com/docs/skills) [primary] |
| Devin / Devin Desktop | Free; Pro $20; Max $200; Teams $80 + $40/dev seat; Enterprise custom. Daily and weekly allowance, then API pricing. **Page no longer mentions ACUs** | VPC, dedicated deployment, SSO, teamspace isolation | [Pricing](https://devin.ai/pricing) [primary] |
| Factory Droids | Tier details conflict across trackers; Teams and Enterprise by sales; BYOK has an allowance | [Air-gapped mode](https://docs.factory.ai/enterprise.md) with models and OTel inside the customer network | [`droid exec`](https://docs.factory.com/cli/droid-exec/overview) with read-only default and `--auto low/medium/high` tiers, custom models [primary]. Tier conflict: [Activepieces](https://www.activepieces.com/blog/factory-ai-pricing-in-2026-and-what-each-plan-costs.md) vs [official](https://docs.factory.ai/pricing.md) |
| Amazon Kiro | Free 50 credits; Pro $20 (1,000); Pro+ $40; Pro Max $100; Power $200; add-on $0.04/credit; team pooling $20/seat. GovCloud about 20% higher | SSO, per-user/group limits, GovCloud | [Pricing](https://kiro.dev/pricing/) [primary]. Models include GPT-5.6, Claude, DeepSeek, MiniMax, GLM, Qwen. Official doc pages exist for [headless mode](https://kiro.dev/docs/cli/headless) and [Agent Skills](https://kiro.dev/docs/skills) (titles confirmed, body not rendered); details from [secondary](https://rywalker.com/research/kiro) |

- Conflict: OpenSpec's README says Kiro is ["limited to Claude models"](https://github.com/Fission-AI/OpenSpec); Kiro's [pricing page](https://kiro.dev/pricing/) lists OpenAI and open-weight models. The README claim appears stale.
- Agent Skills client assets name, among others, Claude Code, Cursor, Factory, Gemini CLI, GitHub, VS Code, Goose, Junie, Kiro, OpenCode, OpenHands, Qodo, Roo Code, Tabnine, and Trae — [repo logo directory](https://github.com/agentskills/agentskills/tree/main/docs/images/logos). This is a proxy read from repo assets because the [showcase page](https://agentskills.io/clients.md) renders client-side; it still lists shut-down Roo Code, so it is not a live support matrix.

### Inferences
- Stars are a weak maturity signal: the Claude Code repo's stars attach to a proprietary product, and Roo Code still has 24k stars after shutdown.
- For vendor-neutral or sovereign departments, the shortlist is OpenCode, Goose (neutral foundation governance), Qwen Code, Cline, and OpenHands as an orchestration layer. Aider should not be a new standard.
- Cursor's cheapest plan being Grok-only and third-party models billing separately is a post-acquisition lock-in signal for a neutral-model strategy.
- Pricing has converged on usage or token billing (Copilot, Cursor, Codex, Devin, Kiro), so budgets need per-department metering rather than seat counts.

### Gaps
- Antigravity CLI's license, repo, and enterprise terms were not verified. Whether Gemini CLI remains a long-term enterprise option is unconfirmed.
- Codex MCP and non-OpenAI providers, Goose providers, Cline model breadth, Kilo and Zoo features, and OpenHands enterprise licensing were not confirmed this session; doc fetches did not return usable text.
- Per-tool enterprise pricing is mostly sales-gated.

## What do the skill and methodology packs contain, how mature are they, and how portable are they?

### Takeaway
SKILL.md is the de facto portable packaging unit, and most methodology packs now ship through it across many agents. Evidence for their benefit is thin: the best independent benchmarks show small average gains for software engineering, and no study directly tests SDD tool adoption.

### Cited Findings

| Pack | License | Stars | Latest | Contents | Portability and maturity |
|---|---|---|---|---|---|
| [Agent Skills spec](https://agentskills.io/home) ([repo](https://github.com/agentskills/agentskills)) | Apache-2.0 | 27,024 | No releases | Folder with `SKILL.md` (name, description, instructions) plus optional scripts, references, assets; three-stage progressive disclosure | Originated at Anthropic, released as an open standard; no tagged releases, so no versioned spec to pin |
| [anthropics/skills](https://github.com/anthropics/skills) | **None at repo root per GitHub**; check per skill | 188,202 | No releases | 18 skills: docx, pdf, pptx, xlsx, skill-creator, mcp-builder, webapp-testing, frontend-design, and others | Mostly document and creative work, little SDLC method; vendor-maintained |
| [obra/superpowers](https://github.com/obra/superpowers) | MIT | 313,653 | v7.0.2 (2026-11-03) | 15 skills: brainstorming, writing-plans, executing-plans, TDD, systematic-debugging, subagent-driven-development, worktrees, code review, verification | Strong on implementation discipline; star count is anomalously high |
| [mattpocock/skills](https://github.com/mattpocock/skills) | MIT | 316,793 | v1.3.2 (2026-10-29) | Engineering set: tdd, code-review, domain-modeling, codebase-design, diagnosing-bugs, to-spec, to-tickets, triage, wizard, others | Individually authored, opinionated, small composable skills |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | MIT with **trademark notice** (GitHub shows NOASSERTION) | 55,085 | v6.12.4 (2026-10-14) | Whole-lifecycle agile method: clarify, plan, build and verify, learn | Stable v6.12 frozen except fixes; **v7 preview alpha with breaking changes**; installs via npm, `npx skills`, Claude and Codex plugins |
| [GitHub Spec Kit](https://github.com/github/spec-kit) | MIT | 148,212 | v1.3.2 (2026-11-06) | `/speckit-constitution`, specify, plan, tasks, implement, converge | Many agents; community extension catalog; backed by GitHub |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | MIT | 74,176 | v1.16.1 (2026-11-07) | Change-proposal workflow (`/opsx:propose`), schema bundles | Claims 30+ tools [vendor]; very active |
| [Agent OS](https://github.com/buildermethods/agent-os) | MIT | 5,587 | v3.0.0 (2026-01-20) | Injects codebase standards, shapes specs | Slower cadence; last push 2026-10-07 |
| [get-shit-done](https://github.com/gsd-build/get-shit-done) | MIT | 64,155 | v1.42.3, **archived** | Moved to [open-gsd/gsd-core](https://github.com/open-gsd/gsd-core) (MIT, 11,305 stars, v1.20.1, 2026-11-05) | Successor supports Claude Code, OpenCode, Antigravity, Kimi, Kilo, Codex, Copilot, Cursor, Windsurf. **Star counts did not transfer** |
| [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | None detected | 78,954 | n/a | Curated list | Index, not a vetted source |
| [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) | None detected | 15,728 | n/a | Curated list; last push 2026-04-28 | Stale-leaning |
| [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) | MIT | 36,934 | n/a | 1000+ skills listed | Index |
| [sickn33/agentic-awesome-skills](https://github.com/sickn33/agentic-awesome-skills) (renamed from antigravity-awesome-skills) | MIT | 49,289 | v20.6.0 | Claims 2,926+ skills | Bulk collection; treat as high review burden |
| [github/awesome-copilot](https://github.com/github/awesome-copilot) | MIT | 40,882 | n/a | Instructions, agents, skills | GitHub-hosted community content |
| [openai/skills](https://github.com/openai/skills) | None detected | 28,557 | n/a | Skills catalog for Codex | Vendor-maintained |
| [vercel-labs/skills](https://github.com/vercel-labs/skills) | MIT | 38,399 | v1.7.6 (2026-11-03) | `npx skills` cross-agent installer | The most visible neutral installation path; BMAD documents it |

**Evidence on benefit**
- SkillsBench (84 tasks, deterministic verifiers): curated skills raised pass rate by an average 16.2pp, but only +4.5pp in Software Engineering versus +51.9pp in Healthcare; self-generated skills averaged -1.3pp — [arXiv 2602.12670](https://arxiv.org/html/2602.12670v1). The [NeurIPS listing](https://neurips.cc/virtual/2026/poster/139353) gives 16.4pp and 18 of 84 tasks regressing. Harness: [benchflow-ai/skillsbench](https://github.com/benchflow-ai/skillsbench) (Apache-2.0).
- SWE-Skills-Bench: 39 of 49 software skills had zero pass-rate effect, averaging +1.2% — [arXiv 2603.15401](https://arxiv.org/html/2603.15401) (read via search summary).
- Largest SDD-related test: 88,052 PRs in 119 repos, specified versus unspecified PRs show no defect difference (-0.6pp, p=0.133). **Critical caveat: artifacts were mostly issue and ticket references, and "SDD tool adoption itself remains untested"**. Single independent author — [ESEM 2026](https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ESEM.2026.84).
- Practitioner critique: Böckeler reports Kiro expanding a small bug into four user stories and sixteen acceptance criteria and Spec Kit feeling heavy for a mid-sized feature — [martinfowler.com](https://www.martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) (details read through search summary, original not fetched).
- A narrower positive result: agents documenting code contracts improved bug detection by 9.8pp (p=0.0352) in test generation — [arXiv 2608.17177](https://arxiv.org/pdf/2608.17177) (search summary). This concerns contracts in test generation, not spec-first workflows.
- A registered report on spec-driven code generation has a design but no results yet — [SANER 2026](https://conf.researchr.org/details/saner-2026/saner-2026-registered-report-track-/5/Understanding-Specification-Driven-Code-Generation-with-LLMs-An-Empirical-Study-Desi).

### Inferences
- Portability is highest for plain `SKILL.md` and lower for anything using slash-command syntax, hooks, or subagents, since Spec Kit and OpenSpec need per-tool invocation mappings.
- Enterprises should expect a modest default benefit from third-party SWE skills and measure each one. The strongest case for skills is encoding organization-specific knowledge, which the public benchmarks do not measure.
- Scale SDD ceremony to change size. Use light change proposals by default and full specs for high-risk or cross-team work.
- The archived GSD and frozen BMAD v6 lines show why packs must be vendored and pinned, not tracked from `main`.

### Gaps
- No controlled study of Spec Kit, BMAD, OpenSpec, Superpowers, or Kiro specs versus a baseline was found.
- Why mattpocock and superpowers exceed 300k stars could not be established; organic versus inflated growth is unverified.
- Per-skill licenses inside anthropics/skills and license status of unlicensed lists were not audited. Unlicensed content should be treated as not cleared for redistribution until legal review.

## How should an enterprise distribute and govern skills across departments, including security scanning?

### Takeaway
Treat skills as executable supply-chain dependencies: internal registry, pinned commits, scanner plus human review, behavior evals, telemetry. Public marketplaces show material malicious content, and scanners catch only part of it by their own measurements.

### Cited Findings

**Supply-chain findings**
- Snyk ToxicSkills (2026-02-05): 3,984 skills; 36.82% (1,467) with any flaw; 13.4% (534) critical; 76 confirmed malicious payloads; 8 still live at publication. Snyk's headline conflates payloads with flaws, and it attributes the sample inconsistently to ClawHub and skills.sh — [Snyk](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/) [vendor sells scanning].
- Koi Security audited 2,857 ClawHub skills, finding 341 malicious, 335 tied to one campaign (ClawHavoc) — reported in [SkillSieve, arXiv 2604.06550](https://arxiv.org/html/2604.06550v1) [secondary for Koi]. Not comparable with Snyk's 76 because scope and criteria differ.
- Academic crawl: 42,447 skills crawled, 31,132 analyzed; 26.1% with at least one vulnerability; 5.2% with high-severity patterns suggesting malicious intent; skills with scripts 2.12x more likely to be vulnerable — [arXiv 2601.10338](https://arxiv.org/html/2601.10338) (via search summary).
- **Scanner limits (vendor's own measurement):** on a held-out set of 839 malicious and 545 harmless skills, Cisco's rules alone caught 7.7% at HIGH (4.0% false positives). With an LLM judge, 66.7% reached MEDIUM or higher (15.4% FP) and 33.7% were blocked at HIGH (6.4% FP). The README says no findings is not proof of safety and human review is essential — [cisco-ai-defense/skill-scanner](https://github.com/cisco-ai-defense/skill-scanner). Stars 2,698; LICENSE file is Apache-2.0 (GitHub shows NOASSERTION); v2.3.2.
- SkillSieve reports 0.800 F1 on a 400-skill benchmark — [arXiv 2604.06550](https://arxiv.org/html/2604.06550v1).
- Clinejection: an AI triage workflow built on claude-code-action put issue titles into the prompt; injection plus cache poisoning led to publication of malicious `cline@2.3.0` on 2026-02-17, which installed OpenClaw. About 4,000 downloads in about eight hours; "4,000 compromised machines" overstates it — [The Hacker News](https://thehackernews.com/2026/02/cline-cli-230-supply-chain-attack.html?m=1); [AI Incident Database](https://incidentdatabase.ai/cite/1680/); [grith.ai](https://grith.ai/blog/clinejection-when-your-ai-tool-installs-another).
- CSA research note on SKILL.md context poisoning — [CSA Labs](https://labs.cloudsecurityalliance.org/research/csa-research-note-skill-md-agent-context-poisoning-20260506/). Scanners also available: [snyk/agent-scan](https://github.com/snyk/agent-scan) (Apache-2.0, 3,219 stars, v0.6.11); NVIDIA SkillSpector per [Classmethod](https://dev.classmethod.jp/articles/nvidia-skillspector-cisco-skill-scanner-agent-skills-vulnerability-scan/) [secondary].

**Distribution and control mechanisms**
- Claude Code private marketplaces: `marketplace.json` in a git repo; sources include relative path, GitHub, git-subdir, URL, archive, npm, command; git sources can be pinned to `ref` or `sha` — [docs](https://code.claude.com/docs/en/plugin-marketplaces).
- Claude Code admin keys: `extraKnownMarketplaces`, `enabledPlugins` (force or block), `strictKnownMarketplaces`, `blockedMarketplaces`, `disableSideloadFlags`, `strictPluginOnlyCustomization`, `disableCommandPluginSources`, `allowManagedHooksOnly`; read-only seed directories for CI and air-gapped fleets; stable and early-access release channels through two marketplaces; OTel install and load events — [docs](https://code.claude.com/docs/en/plugins/org.md) [primary].
- **Documented gaps in those controls:** server-managed settings give one configuration per organization, so per-department targeting needs endpoint policies or gateway policies. The allowlist does not cover plain SKILL.md folders or `--plugin-dir`; those need `strictPluginOnlyCustomization` and `disableSideloadFlags` — [docs](https://code.claude.com/docs/en/plugins/org.md).
- AGENTS.md (MIT, [repo](https://github.com/agentsmd/agents.md), 25,373 stars) and MCP sit under the Agentic AI Foundation with Goose, which gives neutral governance — [Linux Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation?hsLang=en).
- Copilot partner agents need enablement via enterprise or org AI controls, but those policies do not apply to local agents in VS Code — [GitHub docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents).
- Registry-level vetting partnerships exist, such as [Snyk and Tessl](https://snyk.io/blog/snyk-tessl-partnership/) [vendor].

### Inferences
Recommended governance model (synthesis, not an evidence-tested standard):
1. **Layered instructions.** Keep a thin org `AGENTS.md` for cross-agent rules, and repo-level files that add detail. Move long workflows into skills. Claude Code's guidance of keeping the base file short and moving workflows to skills supports this ([costs doc](https://code.claude.com/docs/en/costs)).
2. **Monorepo as canonical registry.** Author plain `SKILL.md` in a private repo with `CODEOWNERS` per department, and generate per-agent packaging (Claude plugin marketplace, `npx skills`, Codex plugin) in CI. The registry should be git, because no neutral cross-vendor registry standard was found.
3. **Tiers.** Org core (platform team owns), department (department owns), and project (repo owners). Instruction-only skills get a lighter path than script-bearing ones, given the 2.12x vulnerability finding.
4. **Intake for third-party skills.** Vendor by commit SHA, never track `main` or auto-update; record license; run two scanners (Cisco plus Snyk); require human review for scripts, hooks, MCP configs, network calls; run in a sandbox with egress limits. Scanner pass cannot be sufficient, given 33.7-66.7% detection in the vendor's own test.
5. **Versioning.** Use SemVer tags on the registry, stable and early-access channels, and a deprecation path; gate promotion on evals.
6. **Evals as CI.** For each skill, keep paired with-skill and without-skill runs with deterministic verifiers (SkillsBench pattern), track trigger accuracy, cost, and regressions. Retire skills with no measurable benefit; the SkillsBench 18-of-84 regression figure shows skills can harm.
7. **Untrusted-input rule for agents in CI.** Issue and PR text must not reach agents holding publish tokens; separate triage permissions from release secrets (Clinejection lesson).
8. **Telemetry.** Export OTel per department for use, spend, and incidents.

### Gaps
- Equivalent enterprise skill-allowlist docs for Codex, Cursor, Copilot, Kiro, and Factory were not reviewed in depth, so a cross-vendor control matrix remains open.
- Whether the Agent Skills spec defines signing, provenance, or dependency metadata was not verified. None was evident from the [overview](https://agentskills.io/home); check the [specification](https://agentskills.io/specification.md).
- Scanner comparisons on a shared neutral benchmark were not found. Cisco's figures are self-reported.

## Which code review agents fit, and how do they compare?

### Takeaway
No independent benchmark produces a stable winner: rankings change with metric, date, and who ran the test. Choose through a bake-off on your own repositories and optimize for comment acceptance and missed-defect rate.

### Cited Findings

| Tool | Pricing | Self-host | Notes |
|---|---|---|---|
| [CodeRabbit](https://www.coderabbit.ai/pricing) | Seat: Essentials $24/$30, Team $48/$60, Advanced $72/$90 (annual/monthly); $0.25 per file over allowance | Enterprise only | Charged only for developers who create PRs |
| [Greptile](https://www.greptile.com/pricing) | $1 per credit, no seat fee; reviews cost 1, 3, or 10 credits | Enterprise only | Usage-based |
| [Qodo](https://www.qodo.ai/pricing/) | Pro Team $30 base, $0.015 per credit; larger packs about 114 reviews from 16,000 credits | On-prem and air-gapped on Enterprise; BYOK | Credits expire monthly |
| [Claude Code Review](https://code.claude.com/docs/en/code-review) | Team and Enterprise; billed by usage with spend limit | **No self-hosting; not available with ZDR, HIPAA configuration, or CMEK; GitHub only; no model choice** | Multi-agent find, verify, rank, post. Never approves or blocks. Alternative: Claude in your own CI |
| [Copilot code review](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/) | AI Credits plus Actions minutes | Not evaluated | Native GitHub integration |
| [PR-Agent](https://github.com/The-PR-Agent/pr-agent) | Free, bring your own model | Yes | MIT, 15,314 stars, v0.49.0 (2026-10-25). Moved to the `The-PR-Agent` org; description states it "is not the Qodo free tier" |
| [Factory droid-action](https://docs.factory.com/cli/droid-exec/overview) | Plan-based | Via CI | Packaged PR review flow |

**Benchmarks (all need caution)**
- Martian Code Review Bench is the closest to neutral: open methodology ([withmartian/code-review-benchmark](https://github.com/withmartian/code-review-benchmark), MIT, 300 stars), with an offline set (reported 50 PRs, 173 golden comments) and an online signal of which comments developers act on. Reported snapshots disagree: Cubic first at 64.9% F1 and Copilot fourth at 60.9% in one; CodeRabbit topping at 51.2% F1 and Qodo first on precision at 63.4% in others — [Kilo blog](https://blog.kilo.ai/p/martians-independent-benchmark-tested); [The New Stack](https://thenewstack.io/github-reviewbench-code-review/); [Baz](https://baz.co/resources/baz-1-in-precision-in-independent-code-review-bench); [Wikipedia: Qodo](https://en.wikipedia.org/wiki/Qodo) [all secondary or vendor]. The [live board](https://codereview.withmartian.com/) did not render through the fetch tool, so **no current score is verified**. Acting on comments is only a proxy for precision.
- Vendor-run: Greptile's July 2025 test (50 PRs) gave Greptile 82%, Copilot 54%, CodeRabbit 44% bug catch; Qodo's February 2026 test (100 PRs) gave Qodo 60.1% F1. Figures read through comparison pages such as [Particula](https://particula.tech/blog/greptile-vs-coderabbit-vs-qodo-ai-code-review-2026) and [Belreos](https://belreos.com/blog/best-ai-code-review-tools-2026) [vendor, stale for Greptile].
- Competitor-run: Macroscope's 118-bug test had Macroscope 48.3%, CodeRabbit 45.8%, Cursor Bugbot 42.4%, Greptile 23.6% (on 72 bugs), Graphite 18.3% — [Macroscope](https://macroscope.com/content/best-ai-code-review-tools-github-2026) [competitor]. Note Greptile falls from 82% in its own test to 23.6% here.

### Inferences
- The Greptile spread shows benchmark ownership and dataset dominate results. Do not quote any single rank in the final report as settled.
- Regulated departments are effectively limited to self-hostable options (PR-Agent, Qodo Enterprise, CodeRabbit Enterprise, Greptile Enterprise, or an agent in own CI). Claude's hosted review is excluded for ZDR, HIPAA, and CMEK organizations.
- Bake-off design: replay 50-100 historical PRs with known escaped defects, and measure recall, comment acceptance, noise per PR, and cost per PR.

### Gaps
- No live, primary Martian scores; no neutral study of whether AI review reduces escaped production defects.
- Copilot code review's own docs and self-hosting posture were not reviewed in depth.

## What is the evidence on productivity and quality, and its limits?

### Takeaway
Task-level and individual output gains are real but smaller and less certain than marketing suggests, while organization-level delivery gains are unproven and telemetry shows quality and review costs rising. Rigorous current causal estimates for frontier agents are not available.

### Cited Findings
- **METR early-2025 RCT:** 16 experienced developers were 19% slower with AI (CI +2% to +39%), while believing they were faster — [METR](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/). METR now says this no longer reflects current effects.
- **METR late-2025 follow-up:** 57 developers, 143 repos, 800+ tasks. Returning developers -18% time (CI -38% to +9%); new developers -4% (CI -15% to +9%). Both CIs include zero. METR calls the data "an unreliable signal": 30-50% of developers withheld tasks they would not do without AI, and some refused to participate. METR thinks the true speedup could be higher and is redesigning the study — [METR](https://metr.org/blog/2026-02-24-uplift-update/) [primary].
- **DORA 2025** found AI adoption associated with higher throughput and higher instability — [DORA](https://dora.dev/insights/balancing-ai-tensions/). **DORA ROI report (2026.01)** treats AI as an amplifier and models a J-curve; its 500-engineer example (about 39% first-year ROI, eight-month payback) is explicitly a "high-uncertainty estimate" and an illustrative model, not a measured result — [InfoQ](https://www.infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report/); [primary report](https://dora.dev/ai/roi/report/). Co-produced by Google Cloud [vendor interest].
- **CMU difference-in-differences (MSR '26):** 806 Cursor-adopting repos against matched controls. Lines added rose 3-5x in the first month but gains faded after two months; static analysis warnings +30% and complexity +41% persisted and predict later slowdown — [He et al.](https://arxiv.org/html/2511.04427v2). Limits: open source, adoption mainly August 2024 to March 2025 (pre-current agents), and controls may use other AI tools.
- **Faros AI telemetry** [vendor sells engineering analytics]: 2025, more than 10,000 developers: 21% more tasks, 98% more merged PRs, review time +91%, no significant company-level correlation — [Faros](https://www.faros.ai/blog/key-takeaways-from-the-dora-report-2025). 2026, about 22,000 developers and 4,000+ teams: incidents per PR +242.7%, median review time +441%, bugs per developer +54% — [Faros](https://faros.ai/blog/ai-acceleration-whiplash-takeaways); the bug figure comes via [secondary coverage](https://dev.to/mnemehq/the-acceleration-whiplash-and-the-governance-gap-b5f). Not peer reviewed, and it compares low- and high-adoption periods, so causality is unproven.
- **Benchmarks:** OpenAI stopped reporting SWE-bench Verified on 2026-02-23, citing flawed tests and training contamination, and recommended SWE-bench Pro — [OpenAI](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/). Models above 70% on Verified reportedly score about 23% on Pro's public split — [Decrypt](https://decrypt.co/359012/...) [secondary, URL truncated in the results]. A critic claims 30% of Pro tasks are broken — [LessWrong](https://www.lesswrong.com/posts/nAMhbz5sfpcynjPP5/swe-bench-pro-is-even-worse) [unverified single audit].
- **Harness effect:** reported Terminal-Bench 2.0 scores for one model range from 58.0% to 76.4% by harness, and 14 of 20 models varied by at least 10% — [Future AGI](https://futureagi.com/blog/coding-agent-harness-benchmark/) and [Codex-focused blog](https://codex.danielvaughan.com/2026/06/11/terminal-bench-2-1-june-2026-benchmark-landscape-codex-cli-harness-engineering-model-scores/) [secondary, attribution among result pages approximate]. A paper argues scores are not valid without disclosing the harness — [arXiv 2605.23950](https://www.alphaxiv.org/abs/2605.23950); an audit found about ten-point differences for the same model on SWE-bench Verified — [arXiv 2605.21404](https://arxiv.org/html/2605.21404).
- **Cost self-report:** Anthropic's about $150-250 per developer per month is a vendor average — [docs](https://code.claude.com/docs/en/costs). Spend per head therefore needs local measurement.

### Inferences
- Plan for capacity at the bottleneck: review, testing, and release controls should scale before generation throughput. DORA, CMU, and Faros converge on this despite differing methods.
- Measure with paired metrics: lead time and throughput with change failure rate, incident rate per PR, review time, and maintainability. Self-reported speedup is unreliable, per METR's perception gap.
- Benchmarks should inform shortlist only. Final choice needs internal evals on company repositories, with the harness version recorded.

### Gaps
- No RCT of 2026-generation agents at enterprise scale. Stanford's 120,000-developer work and GitClear 2026 were searched for but no primary text was retrieved, so they are excluded apart from DORA's citation of Stanford's 35-40% greenfield versus about 10% or less on complex legacy ([InfoQ](https://www.infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report/)).
- Faros 2026 figures should be checked against the [full report](https://faros.ai/research/ai-acceleration-whiplash) before headline use.
- Evidence per SDLC stage (migration, documentation, DevOps) is essentially absent from neutral sources.

## Which pattern of mapping agents to stages does the evidence support: one primary agent or best-of-breed per stage?

### Takeaway
No study directly compares the two. The available evidence best supports a hybrid: one governed primary harness per department with portable artifacts, an independently evaluated review layer, and narrow exceptions for specialist stages. This is an inference, not a tested result.

### Cited Findings
- Developers already mix tools: a reported 900+ developer Pragmatic Engineer survey found 70% use 2-4 tools and 15% use 5 or more — [DEV summary](https://dev.to/chrisbuildsonline/were-all-juggling-4-ai-tools-now-and-pretending-thats-normal-25pp) [secondary; original not reached]. JetBrains (January 2026, 10,000+ developers): 90% use at least one AI tool at work, and Copilot leads at work at 29% — [JetBrains](https://blog.jetbrains.com/research/2026/04/which-ai-coding-tools-do-developers-actually-use-at-work/).
- Platforms are becoming multi-agent: GitHub hosts Claude and Codex agents in preview — [GitHub docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents); OpenHands runs third-party agents through ACP — [OpenHands](https://github.com/OpenHands/OpenHands).
- Harness variance means model scores do not transfer between agents — [arXiv 2605.23950](https://www.alphaxiv.org/abs/2605.23950).
- Vendor-continuity risk is demonstrated by the status-change table above, with shared portable layers in [Agent Skills](https://agentskills.io/home), [AGENTS.md and MCP](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation?hsLang=en).
- Governance controls are per-vendor. For example, Claude Code's plugin keys "govern Claude Code" only — [docs](https://code.claude.com/docs/en/plugins/org.md). Each added agent therefore adds a policy surface.

### Inferences

**Recommended pattern**
- **Default:** one primary agent per department, from an approved list of at most two or three org-wide harnesses, each with a headless mode, OTel or equivalent, and managed policy.
- **Neutral layer:** keep skills, AGENTS.md, MCP servers, evals, and telemetry outside any one vendor, with an LLM gateway where permitted. Run a quarterly portability test by executing core skills on the secondary harness.
- **Independent review:** run a review tool separate from the author agent, or at least a fresh context. This is reasoned from separation of duties; it has no direct evidence.
- **Add best-of-breed only when** a bake-off shows a material gain on internal evals exceeding the added governance cost, or when a data-residency constraint forces it.

**Stage map (judgment based on verified capabilities; no stage-level comparative evidence exists)**

| Stage | Agent fit | Skills and method | Control |
|---|---|---|---|
| Design and architecture | Primary interactive agent; Kiro for spec-first teams | OpenSpec (light), Spec Kit or BMAD (large initiatives), mattpocock `domain-modeling`, `codebase-design` | Human-approved ADR; scale ceremony to risk |
| Implementation | Primary agent; cloud agents for backlog tasks | Superpowers or mattpocock `tdd`, `diagnosing-bugs`; org conventions | Tests and sandbox required |
| Code review | Dedicated reviewer chosen by bake-off; PR-Agent or Qodo Enterprise when self-hosting is needed | Org `REVIEW.md`-style rules | Advisory first; humans merge |
| Refactor | Primary agent in worktrees; `codebase-design` skills | Characterization tests before change | Track complexity, given the CMU finding |
| Documentation | Low-risk headless runs (for example, `droid exec --auto low`, `claude -p`, `codex exec`) | Doc skills; doc templates | Doc tests, link checks, reviewer sign-off |
| Migration | Batch headless with parallel worktrees and per-module verification | Org migration playbooks as skills | Staged rollout, rollback, no remote writes by the agent |
| DevOps | Read-only or plan-only agents by default | IaC plan-review skills | Keep push and deploy in pipeline steps, as Factory recommends: commit and push "in your own script steps" rather than granting `--auto high` ([docs](https://docs.factory.com/cli/droid-exec/overview)) |

**Model vendor and lock-in options**
- Highest lock-in: Claude Code (Claude models only, multi-cloud access), Claude Code Review (hosted, no model choice), Cursor (now SpaceX-owned, with plan economics favoring Grok), Devin, Copilot's first-party platform.
- Medium: Codex (open-source client with a commercial OpenAI core), Kiro (multi-model but AWS-hosted IDE), Factory (BYOK and air-gapped, proprietary client).
- Lowest: OpenCode, Goose, Qwen Code, Cline, OpenHands, and PR-Agent. Local model routes verified for OpenCode (Ollama), Qwen Code (Ollama, vLLM), and Aider; Factory supports models hosted inside an air-gapped environment.

### Gaps
- No controlled or longitudinal enterprise comparison of single-agent versus multi-agent strategies, and no neutral total-cost-of-ownership data, were found.
- Enterprise standardization surveys were not found; the multi-tool data describes individuals, not policy outcomes.
- Local-model coding quality relative to frontier models for each stage was out of reach in this pass.
