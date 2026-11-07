# On-Premises LLM Stack for Coding and SDLC Agents (as of 7 Nov 2026)

Evidence labels used throughout:
- **[API]** read directly from the GitHub or Hugging Face API, or from a raw LICENSE/README file, on 2026-11-07.
- **[Vendor]** self-reported by the party with a stake in the result.
- **[3P]** independent or third-party operator. **[2nd]** secondary blog, reseller or aggregator; treat as directional.
- **[ESTIMATE]** my own calculation. Inputs are stated.

Disclosure: this research was produced by an Anthropic model, and Anthropic models appear in the comparisons. Claude Code cost figures come from Anthropic's own documentation. Weigh them accordingly.

Limits of verification: many web pages were read through a summarizing fetch tool, not verbatim. License quotes and model metadata marked [API] were read from the source. Search-result claims marked [2nd] were not traced to primary sources.

## Open-weight models: licenses, size, tool-calling, agentic evidence, and origin rules

### Takeaway
Three models are realistically deployable at department scale and competitive on agentic coding: GLM-5.3, DeepSeek-V4.1-Flash-1017 and, for 2TB-class memory budgets, Kimi K3.1 Pro. All three are Chinese-origin. If Taiwan government-linked rules apply, that field collapses to Mistral Large 4, NVIDIA Nemotron 3 Ultra, Poolside Laguna, Gemma 4 and gpt-oss, which carry a visibly larger capability penalty.

### Cited Findings

**Model table** (params, storage, context and license tag from the HF API; context = `max_position_embeddings` in config.json)

| Model | Origin | License (exact) | Params | HF repo storage | Context | Agentic evidence |
|---|---|---|---|---|---|---|
| [GLM-5.3](https://huggingface.co/zai-org/GLM-5.3) | CN | [GLM-5.3 License](https://huggingface.co/zai-org/GLM-5.3/raw/main/LICENSE): MIT-like. Only a MaaS operator with >US$10B group revenue must pass a Z.AI "security review" | 753B | not captured | 1M per [2nd](https://geotoolbox.ai/blog/best-open-source-llms) | DeepSWE 69% ±3 [3P] |
| [GLM-5.3-Flash](https://huggingface.co/zai-org/GLM-5.3-Flash) | CN | MIT [API] | 321B | 657GB | 1,048,576 | DeepSWE 63% ±4 [3P] at $0.24/task |
| [DeepSeek-V4.1-Flash-1017](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017) | CN | MIT [API] | 763B [API]; [2nd](https://geotoolbox.ai/blog/best-open-source-llms) says 552B (conflict) | 511GB | 1,048,576 | DeepSWE 77.0, Terminal-Bench 4.0 42.3 [Vendor](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017/raw/main/README.md) |
| [DeepSeek-V4-Pro-0813](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro-0813) | CN | MIT [API] | 1.65T | not captured | 1M per [2nd](https://geotoolbox.ai/blog/best-open-source-llms) | DeepSWE 63% ±6 [3P]; 9.0% in a [table attributed to Artificial Analysis](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md) (see capability section) |
| [Kimi K3](https://huggingface.co/moonshotai/Kimi-K3) / [K3.1 Pro](https://huggingface.co/moonshotai/Kimi-K3.1-Pro) | CN | [Kimi K3 License](https://huggingface.co/moonshotai/Kimi-K3/raw/main/LICENSE) / [K3.1-Pro License](https://huggingface.co/moonshotai/Kimi-K3.1-Pro/raw/main/LICENSE): MaaS over US$20M group revenue needs an agreement; §4 exempts internal use | 2.78T | 1,561GB | 1,048,576 | K3 DeepSWE 69% ±5 [3P]; K3.1 AA Index v5 46 [3P] |
| [Kimi K2.7-Code](https://huggingface.co/moonshotai/Kimi-K2.7-Code) | CN | "modified-mit" tag [API]; text not read | 1.03T | 595GB | 262,144 | no independent score found |
| [Qwen3.8-Max (2.4T-A95B)](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B) | CN | [Qwen3.8-Max License](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/LICENSE): MaaS or "AI Work Assistant" business over US$50M revenue needs a separate license; internal use exempt if not exposed to third parties | 2.45T | not captured | 262,144, extensible to 1.01M | DeepSWE 57% ±3 [3P] |
| [Qwen3.8-Flash-Next](https://huggingface.co/Qwen/Qwen3.8-Flash-Next) | CN | [Qwen Community 1.0](https://huggingface.co/Qwen/Qwen3.8-Flash-Next/raw/main/LICENSE): **any** MaaS or AI Work Assistant business needs a license, no revenue floor; internal use exempt | 180B | 360GB | 262,144 | DeepSWE 58.7 [unverified aggregator](https://llm-stats.com/benchmarks/deepswe-1.1) |
| [Qwen3.8-27B](https://huggingface.co/Qwen/Qwen3.8-27B) | CN | Apache-2.0 [API] | 27.8B | 56GB | 262,144 | DeepSWE 42.2 [unverified aggregator](https://llm-stats.com/benchmarks/deepswe-1.1) |
| [Qwen3-Coder-Next](https://huggingface.co/Qwen/Qwen3-Coder-Next) | CN | Apache-2.0 [API] | 80B | not captured | not captured | Feb 2026, aging |
| [MiniMax-M3](https://huggingface.co/MiniMaxAI/MiniMax-M3) | CN | [MiniMax Community](https://huggingface.co/MiniMaxAI/MiniMax-M3/raw/main/LICENSE): grant is non-commercial; commercial use needs "Built with" display, plus notice or (over US$20M) prior authorization. Definition includes APIs supporting commercial "operations" | 427B | 1,708GB (all variants) | 1,048,576 | no independent score found |
| [MiniMax-M2.7](https://huggingface.co/MiniMaxAI/MiniMax-M2.7) | CN | [Non-Commercial License](https://huggingface.co/MiniMaxAI/MiniMax-M2.7/raw/main/LICENSE): commercial use prohibited without prior written authorization | 229B | not captured | not captured | avoid on license grounds |
| [Mistral Large 4](https://huggingface.co/mistralai/Mistral-Large-4-1T-A52B) | FR | [Mistral Ecosystem Open-Weight License](https://huggingface.co/mistralai/Mistral-Large-4-1T-A52B/raw/main/LICENSE): any MaaS business needs a commercial license. Internal-use treatment beyond the text I read should be checked by counsel | ~1T per [2nd](https://www.developersdigest.tech/blog/mistral-large-4-weights-huggingface-vllm-license-2026) | 1,061GB | 1M per [2nd](https://www.datastudios.org/post/mistral-large-4-open-weights-1m-context-modified-mit-license) | FrontierCode 19.7 (best open-weight in that snapshot); DeepSWE 61.7 [unverified](https://llm-stats.com/benchmarks/deepswe-1.1) |
| [Mistral Medium 3.5](https://huggingface.co/mistralai/Mistral-Medium-3.5-128B) | FR | [Modified MIT](https://huggingface.co/mistralai/Mistral-Medium-3.5-128B/raw/main/LICENSE): **no rights at all if company monthly revenue exceeds US$20M** | 128B | 267GB | 262,144 | large enterprises are excluded |
| [Devstral 2 123B](https://huggingface.co/mistralai/Devstral-2-123B-Instruct-2512) | FR | tag "other" [API]; text not read, likely similar to Medium 3.5 (my inference) | 125B | 257GB | 262,144 | Dec 2025, aging |
| [Devstral Small 2](https://huggingface.co/mistralai/Devstral-Small-2-24B-Instruct-2512) | FR | Apache-2.0 [API] | 24B | 52GB | 393,216 | aging |
| [Nemotron 3 Ultra](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Ultra-550B-A55B-BF16) | US | [OpenMDW-1.1](https://openmdw.ai/license/1-1/): permissive, with notice retention and patent-suit termination | 561B | not captured | up to 1M | Terminal-Bench 2.1 56.4, Toolathlon 34.3 per [Poolside's table](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md) |
| [Nemotron 3.5 Super VL](https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Super-VL-120B-A12B-BF16) | US | OpenMDW-1.1 [API]. Older [Nemotron 3 Super](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Super-120B-A12B-BF16) uses the separate NVIDIA Nemotron Open Model License | 125B | 249GB | not captured | none found |
| [Laguna S 2.1](https://huggingface.co/poolside/Laguna-S-2.1) | Poolside (HQ not verified) | OpenMDW-1.1 [API] | 118B | 235GB | 1,048,576 | [Vendor](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md): TB2.1 70.2, DeepSWE 40.4, Toolathlon 49.7; publishes [trajectories](https://trajectories.poolside.ai) |
| [Laguna XS 2.1](https://huggingface.co/poolside/Laguna-XS-2.1) / [M.1](https://huggingface.co/poolside/Laguna-M.1) | Poolside | OpenMDW-1.1 / Apache-2.0 [API] | 33B / 226B | 67GB / 452GB | 262,144 | none independent |
| [Gemma 4 31B](https://huggingface.co/google/gemma-4-31B-it) | US | Apache-2.0 [API] | 31B | 176GB | 262,144 | no verified coding score found |
| [gpt-oss-120b](https://huggingface.co/openai/gpt-oss-120b) | US | Apache-2.0 [API] | 117B | not captured | 128K per [2nd](https://geotoolbox.ai/blog/best-open-source-llms) | last modified Aug 2025, stale |
| [Tencent Hy4 Preview](https://huggingface.co/tencent/Hy4-Preview) | CN | Apache-2.0 [API] | 780B | 1,560GB | 1,048,576 | AA v5 35 [3P](https://artificialanalysis.ai/leaderboards/models?open_weights=true) |
| Llama / Muse | US | Llama licenses carry a [700M MAU cap per 2nd](https://geotoolbox.ai/blog/best-open-source-llms). Muse Spark Open uses a custom license and Muse Glimmer Apache-2.0, per the same source; I could not confirm their HF repo IDs | n/a | n/a | n/a | Muse Spark Open FrontierCode 6.2; DeepSWE 60.0 [unverified](https://llm-stats.com/benchmarks/deepswe-1.1) |

**Download momentum is not quality.** The HF API shows heavy use of redistributed and "abliterated" derivatives, such as [Qwen3.5-9B-Kimi-k3-Distilled](https://huggingface.co/mradermacher/Qwen3.5-9B-Kimi-k3-Distilled-GGUF). These carry unknown provenance.

**Tool-calling reliability**
- [Toolathlon Verified](https://huggingface.co/zai-org/GLM-5.3/raw/main/README.md), official evaluation service, as reported on model cards: GLM-5.3 73.0. Via [Poolside's table](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md): Muse Spark 1.1 75.6, DeepSeek-V4-Pro 55.9, Laguna S 49.7, Inkling 45.5, Nemotron 3 Ultra 34.3.
- Engine fidelity is a separate failure mode from model quality. vLLM's own debugging of Kimi K2 found [a silently dropped chat-template argument and brittle tool-call ID parsing](https://vllm.ai/blog/Kimi-K2-Accuracy), with schema accuracy reaching about 76% after fixes. [K2 Vendor Verifier tables](https://gh.hhj520.top/MoonshotAI/K2-Vendor-Verifier) showed vLLM and SGLang varying by run and version (mirror page, stale). The successor [Kimi-Vendor-Verifier](https://github.com/MoonshotAI/Kimi-Vendor-Verifier) was active on 2026-11-05 [API].
- [vLLM docs](https://docs.vllm.ai/en/latest/features/tool_calling/) warn that unconstrained `auto` arguments may be malformed, and that a forced named call parses but is "not a high-quality one." `--tool-strict-level parameter` pins schemas.

**Taiwan rules and origin**
- Deputy Digital Minister Hou said on 20 Nov 2025 that [government agencies are "already prohibited from using Chinese-made generative AI models"](https://www.taiwannews.com.tw/en/news/6248783). The article names no legal instrument, enforcement detail or open-source treatment.
- [MODA's early-2025 DeepSeek action](https://www.thestar.com.my/aseanplus/aseanplus-news/2025/02/01/taiwan-bans-government-agencies-from-using-deepseek) covered ministries, public schools, state-owned enterprises and quasi-government bodies. Reports extend it to [critical-infrastructure personnel and government-owned foundations](https://www.thestar.com.my/aseanplus/aseanplus-news/2025/02/01/taiwan-bans-government-agencies-from-using-deepseek).
- A [2019 Executive Yuan principle](https://cna.com.tw/news/aipl/202502030237.aspx) bars public bodies from procuring or using PRC-brand ICT products, including software. Public universities and research institutes [may use DeepSeek for study if approved](https://cna.com.tw/news/aipl/202502030237.aspx); one minister suggested offline standalone machines for academic use.
- The [National Security Bureau tested five Chinese apps](https://www.taiwannews.com.tw/en/news/6245677) including Qwen in Nov 2025, reporting security and content-bias failures. The tests covered apps, not self-hosted weights. MODA says it will [keep warning the private sector](https://www.taiwannews.com.tw/en/news/6248783), but [no private-sector ban was reported](https://www.taipeitimes.com/News/taiwan/archives/2025/11/20/2003847542).

**Traditional Chinese**
- [TAIDE on HF](https://huggingface.co/taide): newest are Gemma-3-TAIDE-12b-Chat-2602 (Feb 2026) and embeddinggemma-GTAIDE-300m-2605 (June 2026). Llama-3.1-TAIDE-LX-8B is under the [taide-l community license, gated](https://huggingface.co/taide/Llama-3.1-TAIDE-LX-8B-Chat) [API].
- [MediaTek Research on HF](https://huggingface.co/MediaTek-Research): [Breeze-7B](https://huggingface.co/MediaTek-Research/Breeze-7B-Instruct-v1_0) is Apache-2.0 (7.5B, 2024); Llama-Breeze2 is under llama3.2 (2025). No 2026 general LLM. The 2026 releases are Breeze-ASR-26 and [Breeze-Guard-26](https://huggingface.co/MediaTek-Research/Breeze-Guard-26) (Apache-2.0).
- Benchmarks are old: [TMLU](https://arxiv.org/html/2403.20180v1) (2024) found Breeze beating GPT-3.5 on direct answers. The [Breeze model card](https://build.nvidia.com/mediatek/breeze-7b-instruct/modelcard) shows 0-shot TMMLU+: Qwen1.5-7B 45.65, GPT-3.5 43.56, Breeze-7B 42.67 [Vendor].
- Artificial Analysis v5 ["primarily tests English-language text tasks"](https://www.implicator.ai/artificial-analysis-index-v5-open-model-gap/), so it says nothing on zh-TW.

### Inferences
- **Default private-sector shortlist:** GLM-5.3 (MIT-like, effectively unrestricted for internal use), DeepSeek-V4.1-Flash-1017 (MIT, but verify vendor scores) and GLM-5.3-Flash (MIT, strong score per dollar).
- **Default Taiwan government-linked shortlist:** Mistral Large 4, Nemotron 3 Ultra, Laguna S 2.1, with Gemma 4 and gpt-oss as light tiers.
- **Treat "self-hosted removes the data-flow objection" as insufficient.** It addresses transmission risk, not the stated [information-manipulation](https://www.taiwannews.com.tw/en/news/6248783) rationale nor the [PRC-brand software principle](https://cna.com.tw/news/aipl/202502030237.aspx). Assume Chinese-origin weights are blocked for agencies, SOEs and likely their contractors until counsel confirms otherwise.
- **License traps for this project:**
  - Mistral Medium 3.5's revenue cap excludes most large enterprises.
  - MiniMax terms plausibly reach internal commercial operations.
  - Qwen Community and Mistral Ecosystem licenses trigger if the factory is offered to other legal entities as a service. Counsel should test whether affiliates or subsidiaries count as third parties, especially in a multi-entity group.
- Derivatives inherit provenance questions. Require a lineage record for any fine-tune (TAIDE, community zh-TW models, distillations).
- TAIDE and Breeze are not coding-agent candidates, on size and evidence. Use them, if at all, for zh-TW drafting or guardrails, with a frontier-class open model doing the coding.

### Gaps
- No 2026 independent zh-TW evaluation of GLM, Qwen, DeepSeek, Kimi or Gemma found. Run an in-house suite: zh-TW requirements to code, Simplified-character leakage rate, Taiwan terminology, and politically sensitive content behavior.
- No legal text located on whether locally run PRC-origin weights fall under the 2019 principle, and no 2026 update was found. Only reported statements. Obtain MODA or Executive Yuan text.
- Mistral Large 4 and Devstral 2 terms beyond what I read, plus TAIDE's full license (gated, returned 401), need direct legal review.
- Tool-calling reliability has no neutral cross-model benchmark on self-hosted engines. Toolathlon reflects provider-hosted runs.

## Serving stacks

### Takeaway
Use vLLM or SGLang for serving, with llm-d as the Kubernetes layer once beyond one or two nodes. Keep Ollama and llama.cpp to developer desktops. Do not start anything on TGI. Assume every engine must sit behind an authenticating gateway on an isolated network, because all of them had serious 2026 vulnerabilities.

### Cited Findings

**Status [API, 2026-11-07]**

| Project | License | Latest tag | Stars | Notes |
|---|---|---|---|---|
| [vLLM](https://github.com/vllm-project/vllm) | Apache-2.0 | v0.33.0rc3 (2 Nov), an RC tag returned as "latest" | 96k | Broadest parser list |
| [SGLang](https://github.com/sgl-project/sglang) | Apache-2.0 | v0.5.23 (2 Nov) | 38.5k | Documented [hardware matrices](https://docs.sglang.io/cookbook/autoregressive/DeepSeek/DeepSeek-V4) |
| [llama.cpp](https://github.com/ggml-org/llama.cpp) | MIT | v0.9.0 (4 Nov) | 133k | Desktop, Apple, CPU |
| [Ollama](https://github.com/ollama/ollama) | MIT | v0.40.15 (3 Nov) | 186k | Desktop |
| [TGI](https://github.com/huggingface/text-generation-inference) | Apache-2.0 | v3.3.7 (Dec 2025) | 10.9k | **Archived, maintenance-mode README; recommends vLLM and SGLang** |
| [LocalAI](https://github.com/mudler/LocalAI) | MIT | v4.12.1 (23 Oct) | 50k | [Wrapper over llama.cpp, vLLM, MLX](https://github.com/mudler/LocalAI), with Anthropic-compatible API claim |
| [llm-d](https://github.com/llm-d/llm-d) | Apache-2.0 | [v1.0.0](https://github.com/llm-d/llm-d/releases/tag/v1.0.0) (20 Oct) | 5.2k | Flow control, session affinity, disaggregated prefill/decode. Stopped building its own engine images |
| [KServe](https://github.com/kserve/kserve) | Apache-2.0 | v0.21.0 (25 Sep) | 6.2k | Adds LLMInferenceService work and DRA support |

**NVIDIA NIM:** production use requires [AI Enterprise, reported at US$4,500/GPU/year](https://pi3g.com/nvidia-ai-enterprise-subscription-cost-in-2026/) or $22,500 perpetual with 5 years of support [2nd]. Jensen Huang's ["~$2,000" remark is not a price list](https://fourweekmba.com/ai-nvidias-huang-pegs-software-at-about-2000-per-gpu-a-year/). [vLLM or SGLang avoid the fee](https://www.beri.net/article/nvidia-ai-enterprise-alternatives-2026-vllm-sglang-red-hat-ai-per-gpu-licence) [2nd].

**Throughput and concurrency (mostly secondary; no controlled 2026 four-way benchmark found)**
- A roundup citing Red Hat: [vLLM about 793 tok/s vs Ollama about 41 under load](https://pub.towardsai.net/vllm-vs-ollama-vs-llama-cpp-vs-sglang-ollama-collapses-to-41-tokens-under-load-8f7a850d1e07); [roughly 44x vLLM over llama.cpp at 64 users](https://aicybr.com/blog/ollama-vs-llama-cpp-vs-vllm-vs-sglang). Not verified against Red Hat's originals.
- [SGLang about 16,200 vs vLLM about 12,500 tok/s on H100 with Llama 3.1 8B](https://aicybr.com/blog/ollama-vs-llama-cpp-vs-vllm-vs-sglang); a [Clarifai run on gpt-oss-120b favored vLLM's TTFT](https://aicybr.com/blog/ollama-vs-llama-cpp-vs-vllm-vs-sglang). Results flip by workload.
- Prefix caching: SGLang RadixAttention is built for shared prefixes. [vLLM V1 prefix caching costs <1% at zero hit rate](https://www.morphllm.com/vllm-benchmarks) [2nd].
- For long contexts, a community test showed [vLLM at about 119 tok/s past 50k context, with llama.cpp far slower](https://reddit.sentinel-team.org/posts/1w4k3ai/snapshots/2026-09-04T21%3A20%3A12.3645Z) (anecdotal, and the snippet's attribution to that thread is uncertain).

**Tool-call parsers**
- vLLM lists [hermes, qwen3_xml, kimi_k2, deepseek_v3/v31, glm45/glm47, openai (gpt-oss), mistral, mimo, step3p5, muse, and others](https://docs.vllm.ai/en/latest/features/tool_calling/). Poolside's card uses [`poolside_v1`](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md).
- [SGLang's function_call directory](https://github.com/sgl-project/sglang/tree/main/python/sglang/srt/function_call) contains deepseekv4, deepseekv41, kimik3, glm47_moe, minimax_m3, gemma4, poolside_v1, muse_glimmer detectors [API].
- [DeepSeek V4 on SGLang](https://docs.sglang.io/cookbook/autoregressive/DeepSeek/DeepSeek-V4) uses `--tool-call-parser deepseekv4 --reasoning-parser deepseek-v4`.

**Security**
- **vLLM:** [advisories page](https://github.com/vllm-project/vllm/security/advisories) has a long run of Aug-Nov 2026 items. Highest concern:
  - [GHSA-h3rc-6mm3-gc2m](https://github.com/vllm-project/vllm/security/advisories/GHSA-h3rc-6mm3-gc2m): request-controlled `code_revision` allows RCE.
  - [CVE-2026-115915](https://github.com/vllm-project/vllm/security/advisories/GHSA-h3rc-j82p-2c3h): EPS input leads to DoS or Ghostscript RCE.
  - [CVE-2026-90553](https://github.com/vllm-project/vllm/security/advisories/GHSA-3c86-2m5g-59q7): malicious model code runs despite `trust_remote_code=False`.
  - [CVE-2026-27893](https://advisories.gitlab.com/pypi/vllm/CVE-2026-27893/): hard-coded trust_remote_code, fixed in 0.18.0.
  - [GHSA-q9v9-qpjp-fc24](https://github.com/vllm-project/vllm/security/advisories/GHSA-q9v9-qpjp-fc24): unauthenticated EngineCore crash.
  - [Permissive default CORS](https://github.com/vllm-project/vllm/security/advisories/GHSA-vvpg-rrww-wpmh).
  - Many of the rest are multimodal (video, audio, image) DoS.
- **SGLang:** [CERT/CC VU#665416](https://www.kb.cert.org/vuls/id/665416) on unauthenticated ZMQ `pickle.loads` RCE ([CVE-2026-3059](https://advisories.gitlab.com/pypi/sglang/CVE-2026-3059/), 9.8; 3060), [fixed in 0.5.10](https://advisories.gitlab.com/pypi/sglang/CVE-2026-3059/). [CVE-2026-14890](https://cve.halosecurity.com/cve-advisory/cve-2026-14890-sglang-unauthenticated-zeromq-pull-socket-rce) has a single source and is unconfirmed. SGLang's GitHub advisory API returned none.
- **Ollama:** ["Bleeding Llama" CVE-2026-7482](https://www.cyera.com/blog/bleeding-llama-a-critical-memory-leak-in-the-worlds-most-popular-local-ai-platform), unauthenticated heap read via GGUF, CVSS 9.1, [fixed in 0.17.1](https://threatprotect.qualys.com/2026/05/11/ollama-heap-out-of-bounds-read-vulnerability-leads-to-remote-process-memory-leak-cve-2026-7482/). [Windows auto-update CVE-2026-42248/42249 reported unpatched](https://labs.cloudsecurityalliance.org/research/csa-research-note-ollama-bleeding-llama-cve-2026-7482-ai-inf) at the time of that note.
- **llama.cpp:** [CVE-2026-34159](https://github.com/ggml-org/llama.cpp/security/advisories/GHSA-j8rj-fmpv-wcxw), critical unauthenticated RPC-backend RCE; [CVE-2026-33298](https://github.com/ggml-org/llama.cpp/security/advisories/GHSA-96jg-mvhq-q7q7) and [CVE-2026-27940](https://github.com/ggml-org/llama.cpp/security/advisories/GHSA-3p4r-fq3f-q74v), GGUF heap overflows.

### Inferences
- Run **text-only** endpoints. Most recent vLLM advisories sit in multimodal paths, and coding agents rarely need them. Disable or block media inputs at the gateway.
- Choose per model, not per brand: pilot both engines with your real traces, and follow the model card's recommended engine and parser. New architectures often appear in both engines quickly, but parsers lag behind weights.
- Pin engine versions and upgrade on a security cadence. The vLLM advisory rate means a monthly patch window is a realistic minimum, which in air-gapped sites requires a tested offline image process.
- Production readiness ranking for this use case (my judgment): vLLM ≈ SGLang > llm-d as orchestration (v1.0 is days old) > KServe (useful if already on it) > NIM (only if buying AI Enterprise support) > LocalAI, Ollama, llama.cpp (edge) > TGI (no).

### Gaps
- No neutral 2026 benchmark of vLLM vs SGLang on 100k+ context agent traces with multi-tenant load. InferenceX pages do not state the engine.
- Current patch status of Ollama's Windows auto-update flaws not verified.

## Embedding and reranking

### Takeaway
Qwen3-Embedding and Qwen3-Reranker (Apache-2.0) have the strongest published code-retrieval numbers, and bge-m3 (MIT) remains the workhorse. Both are Chinese-origin. Jina v5 is non-commercial. For restricted contexts, EmbeddingGemma or TAIDE's variant are candidates, but need an in-house evaluation.

### Cited Findings
- [BAAI/bge-m3](https://huggingface.co/BAAI/bge-m3): MIT [API], 30M downloads; last modified 2024. [bge-reranker-v2-m3](https://huggingface.co/BAAI/bge-reranker-v2-m3): Apache-2.0, 8,194 max positions [API].
- [Qwen3-Embedding-0.6B / 4B / 8B](https://huggingface.co/Qwen/Qwen3-Embedding-4B) and [Qwen3-Reranker](https://huggingface.co/Qwen/Qwen3-Reranker-4B): Apache-2.0, 40,960 max positions on 4B [API].
- [Qwen3 Embedding paper](https://arxiv.org/pdf/2506.05176): MTEB(Code) about 78.75 (8B) and 78.21 (4B) vs bge-m3 dense about 49.65 [Vendor; row labeling provisional per snippet]. It includes C-MTEB tables, but C-MTEB is built around Simplified Chinese (my inference), not zh-TW.
- [Jina embeddings v5](https://huggingface.co/jinaai/jina-embeddings-v5-text-small) and [reranker v3.5](https://huggingface.co/jinaai/jina-reranker-v3.5): **CC-BY-NC-4.0** [API], so unusable commercially without a separate deal.
- [taide/embeddinggemma-GTAIDE-300m-2605](https://huggingface.co/taide/embeddinggemma-GTAIDE-300m-2605): license "other", June 2026 [API]. [google/embeddinggemma-300m](https://huggingface.co/google/embeddinggemma-300m) and [embeddinggemma-2](https://huggingface.co/google/embeddinggemma-2) exist; terms and Chinese performance were not verified.
- Configuration can dominate: a vendor page reports [Chinese Recall@1 dropping from 80% to 15% from a config mistake on bge-m3](https://docs.apiyi.com/en/live/2026-08/bge-m3-relaunch) [Vendor].
- vLLM pooling had [a DoS from negative token IDs in `/v1/embeddings`](https://github.com/vllm-project/vllm/security/advisories/GHSA-25q3-v2hm-8vpf).

### Inferences
- Start with Qwen3-Embedding-4B plus Qwen3-Reranker-4B where origin is acceptable, with bge-m3 as baseline for hybrid dense and sparse. Build a labeled zh-TW plus code retrieval set before choosing. Mixed zh-TW and English identifiers are the case no public benchmark covers.
- Embedding models are small; one GPU or a partition of a pilot card suffices (my judgment). Keep them separate from the agent LLM pool so that re-indexing does not starve interactive agents.

### Gaps
- No Traditional-Chinese or Chinese code-search benchmark found. No independent 2026 embedding comparison found.

## Gateways and per-department budget without cloud

### Takeaway
LiteLLM has the richest budget model and the worst security record. Agent Router or agentgateway are stronger security-posture bets but only meter tokens as rate limits, so a dollar-style budget needs extra work. Kong's token limits are Enterprise-only. On-prem, a "budget" should be a share of GPU capacity, not money.

### Cited Findings

**LiteLLM** ([repo](https://github.com/BerriAI/litellm): [MIT outside `enterprise/`](https://github.com/BerriAI/litellm/blob/main/LICENSE); v1.108.2, 6 Nov)
- Compromise: [PyPI 1.82.7 and 1.82.8 on 24 Mar 2026](https://docs.litellm.ai/blog/security-update-march-2026), a credential stealer. 1.82.8 added a `.pth` that executes on any Python start. Suspected path: the Trivy dependency in CI/CD, then a maintainer's PyPI account. Official Docker and `ghcr.io/berriai/litellm` images were reported unaffected because dependencies were pinned. The exposure window is stated as both about 40 minutes and 10:39 to 16:00 UTC on the same page.
- Remediation: [v1.83.0 via CI/CD v2; cosign-signed GHCR images from v1.83.0-nightly](https://docs.litellm.ai/blog/security-update-march-2026), and SHA-256 audits of releases 1.78.0 to 1.82.6.
- The post-incident advisory stream is severe: [CVE-2026-42208 SQL injection in API key verification](https://github.com/BerriAI/litellm/security/advisories/GHSA-r75f-5x8p-qvmc) (critical), [CVE-2026-49468 auth bypass via Host header](https://github.com/BerriAI/litellm/security/advisories/GHSA-4xpc-pv4p-pm3w) (critical), [CVE-2026-114612 escalation to Proxy Admin](https://github.com/BerriAI/litellm/security/advisories/GHSA-7hp6-4w63-5g45) (critical), [unauthenticated file read leaking the master key on Enterprise proxy](https://github.com/BerriAI/litellm/security/advisories/GHSA-w4c7-34wj-hqm8) (critical, Oct 2026), [CVE-2026-42271 command execution via MCP stdio test endpoints](https://github.com/BerriAI/litellm/security/advisories/GHSA-v4p8-mg3p-g94g), [CVE-2026-42203 SSTI](https://github.com/BerriAI/litellm/security/advisories/GHSA-xqmj-j6mv-4862).
- Anthropic's docs note LiteLLM is ["unaffiliated with Anthropic and has not been audited for security"](https://code.claude.com/docs/en/costs).
- Budgets: [team, member, key, user and customer budgets, `budget_duration`, multiple windows, TPM/RPM/parallel limits](https://docs.litellm.ai/docs/proxy/users). **PostgreSQL required; without it the global budget fails open.** Redis handles multi-instance counters, with `fail_closed_rate_limit_enforcement` available. Per-model budgets on keys and users are Enterprise-labeled.

**agentgateway** ([repo](https://github.com/agentgateway/agentgateway): Apache-2.0, v1.7.0, 27 Oct)
- [Post-response token counting; approximate; key by CEL such as `jwt.team`; local limits per replica; global needs an external rate-limit service](https://agentgateway.dev/docs/kubernetes/latest/llm/rate-limit/). Cached tokens count toward the budget. Dollar budgets were not documented on that page.
- [Recent advisories](https://github.com/agentgateway/agentgateway/security/advisories) focus on MCP session and authorization boundaries, up to high severity.

**Agent Router** (formerly Envoy AI Gateway; [`envoyproxy/ai-gateway` now resolves to theagentrouter/agent-router](https://github.com/theagentrouter/agent-router) [API]; Apache-2.0, v1.2.1, 6 Nov)
- [Agentic AI Foundation project; input, cached, output, total and CEL cost; Global Rate Limit API; Redis required; charged after completion](https://theagentrouter.ai/docs/capabilities/traffic/usage-based-ratelimiting/). CEL allows discounted cached tokens.
- [Six advisories published 6 Nov 2026](https://github.com/theagentrouter/agent-router/security/advisories) including a high-severity MCP proxy listening on all interfaces.

**Kong** ([repo](https://github.com/Kong/kong): Apache-2.0, 3.9.4, 12 Oct)
- [ai-rate-limiting-advanced is "only available as part of our AI Gateway Enterprise offering"](https://developer.konghq.com/plugins/ai-rate-limiting-advanced/). It supports token and cost strategies.

### Inferences
- **Recommended pattern (my judgment):** a thin gateway in front of vLLM/SGLang, with OIDC-issued JWTs and a department claim. Choose Agent Router or agentgateway where the team runs Envoy/Kubernetes well; choose LiteLLM only if per-team budget UX matters more than attack surface.
- **If using LiteLLM:** run a digest-pinned, cosign-verified container (never `pip install` from PyPI at deploy time), disable MCP, prompt-test and custom-code features, keep admin endpoints off the developer network, and mirror images through a verified internal registry.
- **Budget design:**
  - Express quotas as a **weighted token unit** via CEL: input ×1, cached ×0.1, output ×4 as starting weights (my assumption; calibrate on GPU-seconds).
  - Give each department a guaranteed share plus burst. Place interactive above batch using llm-d's [priority and fairness flow control](https://github.com/llm-d/llm-d/releases/tag/v1.0.0).
  - Report chargeback from gateway metrics to Prometheus and OTel. [Claude Code OTel](https://code.claude.com/docs/en/costs) can supply per-user data if that client is used.
- Post-hoc charging in agentgateway and Agent Router means runaway streams overshoot by one response. Add max-token and max-concurrency caps per principal.

### Gaps
- No independent security audit or head-to-head load test of these gateways found. Kong OSS's current packaging status beyond the repo tag was not confirmed. Vendor comparisons ([API7](https://api7.ai/kong-ai-gateway-vs-litellm), [Maxim](https://www.getmaxim.ai/articles/top-5-litellm-alternatives-in-2026/)) conflict on what LiteLLM gates behind Enterprise; the LiteLLM docs above are the better source.

## Hardware sizing

### Takeaway
Serious open models need 8-GPU servers, not workstations: about 0.5 to 2.3TB of GPU memory. A workable planning figure is **40 to 100 developers per 8-GPU H200-class server on GLM-5-class models** [ESTIMATE], and it must be validated by load test, because no public benchmark reproduces long-context multi-tenant agents. AMD MI355X shows notable throughput and memory advantages on third-party dashboards, with more software risk. Workstations (Mac Studio, DGX Spark) are single-developer tools.

### Cited Findings

**Memory fit**
- [DeepSeek V4 on SGLang](https://docs.sglang.io/cookbook/autoregressive/DeepSeek/DeepSeek-V4):
  - V4-Flash (284B): RTX PRO 6000 TP=2, or B200/H200 TP=4, MI355X single node.
  - V4-Flash 0731 (304B): 8×B200 or 4×GB300 or 4×H200 verified; DGX Spark ×2 with a special image.
  - V4-Pro (1.6T): 8×B200, H200 FP4 TP=8, or 2-node H200 FP8 TP=16; MI355X single node. MI300X unsupported for Pro.
- [Kimi K3 at FP4](https://fourweekmba.com/ai-nvidia-b200-costs-up-to-2-3x-amd-mi355x-per-kimi-k3-token/): SemiAnalysis, as relayed, says MI355X offers 2,304GB HBM per server, while B200 has <1.5TB and must cross scale-out networking. K3.1 Pro weights are [about 1.56TB](https://huggingface.co/moonshotai/Kimi-K3.1-Pro) [API].
- [Laguna S 2.1 needs about 236GB BF16, "needs multiple GPUs"](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md). Qwen3.8-27B, 56GB BF16 [API], runs on one 96GB card.
- [ESTIMATE] GLM-5.3 at FP8 about 750GB plus KV cache: fits 8×H200 (about 1.13TB, assuming 141GB per H200, spec recalled and not re-fetched) tightly, and 8×B200 more comfortably. DeepSeek-V4.1-Flash-1017 at 511GB: same class. Mistral Large 4 at 1.06TB: 8×B200 or 8×MI355X.

**Throughput evidence** (InferenceX by SemiAnalysis. [3P], but "interpolated", and no engine stated. Concurrency and "total token" definitions differ across pages, so do not compare between pages.)

| Model | Workload | Chip | tok/s/chip @ tok/s/user | $/M tok (hyperscaler TCO) |
|---|---|---|---|---|
| [GLM 5/5.1](https://inferencex.semianalysis.com/compare/glm-5-1-h200-vs-mi355x) | 8k/1k fp8 | H200 | 740 @34; 563 @51; 434 @68 | $0.458; $0.602; $0.781 |
| same | | MI355X | 1,276 @34; 957 @51; 747 @68 | $0.327; $0.436; $0.558 |
| [gpt-oss-120b](https://inferencex.semianalysis.com/compare/gptoss-120b-h200-vs-mi355x) | 8k/1k fp4 | H200 | 6,714 @92; 2,812 @201 | $0.050; $0.121 |
| same | | MI355X | 26,412 @92; 10,455 @201 | $0.016; $0.040 |
| [GLM-5.3](https://inferencex.semianalysis.com/compare/glm-5-3-h200-vs-mi355x) | agentic traces fp4 | MI355X | 17,311 @71; 9,604 @140 | $0.024; $0.043 (H200 not measured) |
| [Kimi K3](https://fourweekmba.com/ai-nvidia-b200-costs-up-to-2-3x-amd-mi355x-per-kimi-k3-token/) | agentic | B200 / MI355X | 2,678 / 5,436 @100 | $0.179 / $0.077 ([2nd] reading of dashboard) |

- Chip-hour TCO assumptions used by InferenceX are hyperscaler-volume, not enterprise: [B200 $1.73, MI355X $1.50, B300 $2.26](https://fourweekmba.com/ai-nvidia-b200-costs-up-to-2-3x-amd-mi355x-per-kimi-k3-token/).
- [MLPerf Inference v6.0](https://digitalterminal.in/device/amd-instinct-mi355x-achieves-over-1m-tokenssec-in-mlperf-60-ai-benchmark) (Apr 2026): MI355X about 100,282 tok/s on Llama 2 70B Server; AMD claims 97% of B200 on that test [Vendor framing]. MLPerf measures old models and no agent traffic, so it has limited sizing value.
- Workstation evidence:
  - [DGX Spark about 49.7 tok/s on gpt-oss-20b, RTX PRO 6000 about four times that; RTX PRO 6000 163.1 tok/s on gpt-oss-120b per StorageReview](https://www.spheron.network/blog/dgx-spark-vs-rtx-pro-6000-blackwell-which-local-ai-box-wins/) [2nd].
  - Community RTX PRO 6000: [about 80 tok/s single-stream and about 400 tok/s aggregate with sub-agents on a 27B model](https://reddit.sentinel-team.org/posts/1w1gvqu/snapshots/2026-09-04T21%3A20%3A12.3645Z). Dual-card Qwen3.5-27B: [2,332 tok/s at rate=16](https://bittide.aicompass.dev/article/49db488a-d566-4b94-864c-b6ee39e2b076?locale=en). Anecdotal; the snippet attributions to specific threads are uncertain.

**Price, availability, power (all approximate; no official list prices found; expect quotes to differ)**

| Option | Memory | Price | Availability | Power |
|---|---|---|---|---|
| [HGX H200 8-GPU](https://intuitionlabs.ai/articles/nvidia-ai-gpu-pricing-guide) | ~1.13TB | $315-420K [2nd] | [Reseller reports in stock](https://viperatech.com/news); [Jan 2026: 4-8 weeks](https://cyfuture.ai/blog/buy-nvidia-h200-gpu-pricing-availability-lead-time-guide) | about 11kW implied [ESTIMATE from InferenceX tok/s/MW: about 1.37kW/chip all-in] |
| [HGX B200 8-GPU](https://www.besthub.dev/articles/nvidia-server-gpu-2026-buying-guide-specs-prices-compliance-channels-9c6195bac2c4) | ~1.5TB | $400-500K [2nd] | [8-20 weeks golden SKU, 26+ custom](https://viperatech.com/news) | [10-15kW per node](https://intuitionlabs.ai/articles/nvidia-hgx-data-center-requirements); [14kW, air 10U or liquid 4U](https://www.amcompute.com/compare/b200-vs-h100) |
| MI355X 8-GPU | [288GB each](https://www.tomshardware.com/pc-components/gpus/amd-announces-mi350x-and-mi355x-ai-gpus-claims-up-to-4x-generational-gain-up-to-35x-faster-inference-performance); 2.3TB | no verified purchase price; [rental $2.59/GPU-hr, one provider](https://www.gpuperhour.com/rent/mi355x) | Not verified | [1,400W per GPU, mainly liquid](https://www.tomshardware.com/pc-components/gpus/amds-instinct-mi355x-accelerator-will-consume-1-400-watts); about 16.7kW implied [ESTIMATE, 2.09kW/chip all-in] |
| [MI350P PCIe](https://www.techzine.eu/news/infrastructure/141152/amd-introduces-instinct-mi350p-for-drop-in-enterprise-ai-hardware/) | 144GB | [$30-40K unconfirmed](https://tech-insider.org/amd-mi350p-vs-nvidia-h200-2026) | [OEM from Jul 2026; shipments unconfirmed](https://tech-insider.org/amd-mi350p-vs-nvidia-h200-2026) | [600W, down to 450W, air](https://tweaktown.com/news/111491/amd-launches-the-instinct-mi350p-gpu-with-144gb-of-hbm3e-and-a-600w-tbp/index.html) |
| [RTX PRO 6000 Blackwell](https://www.spheron.network/blog/dgx-spark-vs-rtx-pro-6000-blackwell-which-local-ai-box-wins/) | 96GB | near $16K retail Oct 2026; launch $8,565 [2nd]. [Three ~30% Nvidia hikes in 2026, supply strained](https://viperatech.com/product/nvidia-rtx-pro-6000-blackwell-Series) | Supply strained | card only; no NVLink |
| [DGX Station GB300](https://www.tomshardware.com/desktops/nvidias-gb300-powered-dgx-station-desktop-tower-listed-for-nearly-usd100-000-online-enterprise-ai-powerhouse-now-available-to-buy-for-mere-mortals-with-lots-of-cash) | [748GB, including 496GB LPDDR5X](https://wccftech.com/nvidia-dgx-station-upgraded-gb300-blackwell-ultra-desktop-superchip/) ([HP says 784GB](https://techradar.com/pro/hp-announces-the-most-powerful-windows-ai-pc-ever-built-nvidia-gb300-workstation-can-handle-one-trillion-parameters-thanks-to-its-784gb-unified-memory-but-it-wont-be-cheap)) | $85K to $125K | [4-12 weeks, July; 10-20% rise forecast](https://pi3g.com/?p=31780) | desktop tower |
| [DGX Spark](https://www.spheron.network/blog/dgx-spark-vs-rtx-pro-6000-blackwell-which-local-ai-box-wins/) | 128GB | about $6,950 [2nd]; launch $3,999 | Available | [273GB/s bandwidth limits speed](https://www.spheron.network/blog/dgx-spark-vs-rtx-pro-6000-blackwell-which-local-ai-box-wins/) |
| [Mac Studio M5 Ultra](https://techradar.com/pro/apple-will-now-sell-you-a-mac-studio-with-512gb-of-ram-but-itll-cost-you-as-much-as-a-good-sized-family-car) | up to 512GB | $22,299 to $29,799 at 512GB | [Shipping, some lead time](https://pasqualepillitteri.it/news/27929/mac-studio-m5-ultra-512gb-4tb-consegna-23-novembre) | no LLM throughput evidence found |

Cooling: [air stops being workable around 40kW/rack per one source](https://intuitionlabs.ai/articles/nvidia-hgx-data-center-requirements); [Gigabyte's liquid HGX B200 rack reaches 103kW with 8 servers](https://www.gigabyte.com/Enterprise/GIGAPOD-Rack-Scale/AI-DLC-Rack_NVIDIA-HGX-B200).

Demand evidence: [Claude Code enterprise average about $13/developer/active day, $150-250/month, under $30/active day for 90%](https://code.claude.com/docs/en/costs) [Vendor]. The same page's [recommended rate limits](https://code.claude.com/docs/en/costs) fall from 100-150k TPM per user (5-20 users) to 15-20k (100-500 users) as concurrency thins. Agent teams use [about 7x more tokens](https://code.claude.com/docs/en/costs) in plan mode.

### Inferences

**[ESTIMATE] Demand model. All my calculation.**
1. The Claude Code cost page's illustrative session is 1.2k input, 5.3k output, 940k cache read, 50k cache write for $0.55. Scaling to $13/day (×23.6) gives about 125k output, 1.2M cache writes and 22M cache reads per developer-day. This is one example session, not a measured distribution.
2. Over 8 hours: about 4.3 output tok/s and about 41 prefill tok/s per developer. The prefill-to-output ratio (about 9.5:1) resembles InferenceX's 8k/1k shape, so that benchmark is a reasonable shape proxy, except for context length.
3. Multiply ×3 for peak, ×3 for open-model verbosity, retries and background agents (assumption): about **40 output tok/s per developer at peak**.
4. Supply: 8×H200 on GLM 5/5.1 at 34 tok/s/user = 5,920 tok/s; derate 50% for long context, KV eviction and multi-tenancy (assumption) = about 2,960 tok/s, or about **74 developers per server, range 40-100**. An SDLC factory running autonomous agents can consume this several times faster; treat one always-on agent as 3-5 developers (assumption).

**[ESTIMATE] Three tiers** (capex is 2026 street estimate; excludes facility fit-out and any NVIDIA AI Enterprise fees of about $4,500/GPU/yr)

| | Pilot | Department | Multi-department |
|---|---|---|---|
| Developers (plus always-on agents) | 10-25 (few) | 50-100 (up to ~10) | 300-500 (up to ~50) |
| Hardware | 4× RTX PRO 6000 in one host, or 1× DGX Station | 2× 8-GPU (H200 or B200; MI355X as benchmarked alternative) | About 8× 8-GPU including N+1, 400G fabric, shared storage |
| Primary model | Qwen3.8-27B or Laguna S 2.1 (fits with quantization); V4-Flash on 2 cards | GLM-5.3 or DeepSeek-V4.1-Flash-1017; or Mistral Large 4 for restricted orgs | Same, plus GLM-5.3-Flash as a fast tier; K3.1 Pro on MI355X only if memory economics hold |
| Expected quality | Weak on long-horizon (DeepSWE about 40%) | Near-frontier on some benchmarks; well below on merge quality | Same |
| Capex | about $90K | about $1.05M ($900K + $150K network, storage) | about $4.2M ($3.6M + $0.6M) |
| IT power / facility (×1.4 PUE) | ~3.5 / 4.9kW | ~28 / 39kW | ~112 / 157kW |
| Cooling | Office-adjacent or server room air | Existing data hall with high-density air, or liquid | Liquid or rear-door preferred; 3-4 racks at under 40kW each |
| Staff | 0.5 FTE | 1.5 FTE | 4 FTE |

Workstation guidance: Mac Studio and DGX Spark suit a single developer or offline experiment. Do not treat their large unified memory as serving capacity, since memory bandwidth is what limits speed. DGX Station holds large models, but only its HBM portion is fast (about 252GB if the 748GB and 496GB figures are both right [ESTIMATE]).

AMD: published numbers favor MI355X by about 1.7-3.9× in tok/s/chip on these workloads, but credit [AMD kernel work and vLLM engineering](https://fourweekmba.com/ai-nvidia-b200-costs-up-to-2-3x-amd-mi355x-per-kimi-k3-token/), and parity is workload-specific. Require a proof of concept on your chosen model and engine before buying, especially for air-gapped sites that cannot pull fixes quickly.

### Gaps
- No credible public load test for N developers on any open model with 100k+ contexts; do a trace-replay test on rented hardware before purchase.
- MI355X purchase price and Taiwan-specific price, lead time, and import or support terms not found. Global prices are volatile (DRAM shortage).
- KV-cache memory per session at long context not derivable from sources.

## TCO: on-prem vs cloud API

### Takeaway
On pure cost, on-prem usually loses to frontier API seats at current average developer usage, and loses more to hosted open-weight APIs, unless utilization is sustained. My calculation puts department-tier cost near $570/developer/month against $150-250 for frontier seats. The justification for on-prem here is sovereignty and control. Cost only wins with heavy background agent utilization.

### Cited Findings
- Published break-even claims vary so widely that they are not usable as evidence: [break-even in 4-6 months for a coding assistant at 50M tokens/month on an RTX 4090](https://www.sitepoint.com/local-llms-vs-cloud-api-cost-analysis-2026/) vs [roughly 500M tokens/day](https://tkmxai.it.com/on-prem-vs-api-inference-in-2026-2), and [Spheron (a cloud seller) finding cloud beats on-prem at near-full utilization](https://www.spheron.network/blog/llm-inference-on-premise-vs-cloud/), [VRLA (an on-prem seller) finding on-prem amortizes rapidly](https://vrlatech.com/llm-inference-on-premise-vs-cloud-cost-2026/). All have an interest or weak methodology.
- API prices (third-party, not official pages; verify): [Opus 5.5 $4/$20, cache read $0.20](https://www.gradually.ai/en/llm-comparison/claude-opus-5.5-vs-deepseek-v4.1-flash/); [GLM-5.3 $1.085/$3.41 on OpenRouter](https://openrouter.ai/compare/deepseek/deepseek-v4.1-flash/z-ai/glm-5.3); [GLM-5.3-Flash $0.15/$0.50](https://anotherwrapper.com/tools/llm-pricing/glm-5.3-flash/deepseek-v4-flash-0423); [DeepSeek V4.1 Flash $0.30/$1.20](https://anotherwrapper.com/llm-pricing/deepseek-v4.1-flash), with [other listings far lower](https://pricepertoken.com/pricing-page/model/deepseek-deepseek-v4.1-flash).
- [DeepSWE average cost per task](https://deepswe.datacurve.ai/) (Sep 2026): Opus 5 $11.84, GPT-6 Astra $4.43, GLM-5.3 $3.99, Kimi K3 $4.65, GLM-5.3-Flash **$0.24**, GPT-5.6 Luna $0.61.
- Depreciation: [Microsoft books GPUs over six years, Meta 5.5](https://fourweekmba.com/ai-nvidia-gpu-depreciation-durability-debate-huang-2026/); critics such as [Burry argue that understates](https://fourweekmba.com/ai-nvidia-gpu-depreciation-durability-debate-huang-2026/). American Compute puts [H100 at 11-19% of list in 2027, 5-7% in 2028 (conservative band)](https://www.amcompute.com/appraisals/h100) [seller of appraisal views]. Rental markets are not one-way: [H100 about $2.50-3.50/hr after a 64-70% decline](https://introl.com/blog/gpu-cloud-price-collapse-h100-64-percent-drop-2025), then [Nebius raised rates up to 21% in 2026](https://www.digitalcitizen.life/nvidia-a100-h100-and-b200-rental-prices-rise-as-ai-demand-stays-strong/). Evidence on useful life [remains unresolved](https://www.datagravity.dev/p/how-long-does-a-gpu-last).
- Electricity: last official-looking Taiwan figure found is [industrial average NT$4.29/kWh, Oct 2024](https://www.bnext.com.tw/article/74485/taipower-electric-valance-increase), with a [0.71% average change in Sep 2025](https://uanalyze.com.tw/articles/4985133668). A 2026 rise is claimed only by a [personal-finance blog](https://roo.cash/blog/electrity-bill) and could not be confirmed.
- Model churn: GLM-5.2 scored [44% and GLM-5.3 69% on DeepSWE](https://deepswe.datacurve.ai/), although [Z.ai's card says both share a base model](https://huggingface.co/zai-org/GLM-5.3/raw/main/README.md). The HF API shows DeepSeek-V4-Flash, then 0731, V4.1 and 1017 created within roughly four months.

### Inferences

**[ESTIMATE] Annual cost, my calculation.** Assumptions: 3-year straight-line; support 10% of capex; PUE 1.4; US$0.134/kWh (NT$4.29 ÷ 32 FX, stale); US$120K per FTE; no facility build-out, no AI Enterprise, no residual value.

| Tier | Depreciation | Support | Power | Staff | **Total/yr** | Per developer/month |
|---|---|---|---|---|---|---|
| Pilot, 20 devs | $30K | $9K | $6K | $60K | **$105K** | **~$437** |
| Department, 100 devs | $350K | $105K | $46K | $180K | **$681K** | **~$568** |
| Multi-dept, 400 devs | $1.4M | $420K | $184K | $480K | **$2.48M** | **~$518** |

- Frontier API at the [$150-250/month enterprise average](https://code.claude.com/docs/en/costs): department tier about $180-300K/yr vs $681K. Break-even **headcount** at 100-dev stack cost: about 284 developers at $200/month (beyond this stack's estimated capacity), or about **86 developers at $30/active-day and 22 days** (heavy users).
- **Utilization math, department tier on H200:** 16 chips × 740 tok/s = about 373B output tokens/yr at 100% utilization, so $1.82/M output at 100%, $9.12 at 20%, $18.20 at 10%.
- Compare at an 8:1 input-to-output ratio, per 1M output tokens:
  - GLM-5.3 hosted uncached: $12.09, for on-prem break-even at about **15% utilization**.
  - With 90% caching at an assumed cached price of 20% of input ($0.217): $5.84, break-even about **31%**.
  - Opus 5.5 with 90% caching: $24.64, break-even about **7%**, but not capability-equal.
- A 9-to-6 developer pattern typically averages maybe 10-25% utilization (assumption). So fill off-hours with batch agents (test generation, code review, migration), or the cost case does not hold.
- **Risks:** a 3-year life is deliberately more conservative than hyperscaler accounting because model requirements are growing (open flagship weights rose from hundreds of GB to 1.5TB+ during 2026). Buy memory headroom. Software churn is roughly quarterly: budget for requalifying models and parsers each cycle. Hardware price inflation (DRAM, RTX PRO hikes) can improve residuals but raises expansion cost.

### Gaps
- No independent, peer-reviewed enterprise TCO study for agent workloads found. Cached-input API pricing for open models is poorly documented. Official 2026 Taipower rate not confirmed. Taiwan labor and colocation costs not sourced; my FTE figure is a placeholder.

## Capability gap and hybrid escape hatch

### Takeaway
The gap is benchmark-dependent, and honest reporting has to show both: about 5 points on DeepSWE, but more than 2× on FrontierCode's merge-quality test, and 26 points on Artificial Analysis v5 for practically deployable models. Expect local models to complete many scoped tasks, with more review burden on long-horizon and production-quality work. Keep a routed escape hatch designed in from day one.

### Cited Findings
- **[DeepSWE v1.1](https://deepswe.datacurve.ai/)** (Datacurve, 22 Sep 2026; all models on mini-swe-agent, the only fixed-harness comparison here; [113 original tasks, 91 repos](https://arxiv.org/abs/2607.07946)): gpt-6-astra, gemini-3.8-flash, claude-opus-5 **74%**; GLM-5.3 and Kimi K3 **69%**; GLM-5.3-Flash and DeepSeek-V4-Pro 63%; Qwen3.8-Max 57%; DeepSeek-V4-Flash 53%; GLM-5.2 44%. Newer frontier models and DeepSeek-V4.1 are not on the official page as read.
- **Vendor scores on the same benchmark are not interchangeable.** [DeepSeek claims 77.0 for V4.1-Flash-1017 (Mini-SWE-V2)](https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash-1017/raw/main/README.md), above its own figure of 74.0 for Opus 5. [Moonshot reports K3.1 Pro 75.5 and Opus 5.5 74.2 using Claude Code](https://huggingface.co/moonshotai/Kimi-K3.1-Pro/raw/main/README.md). [LLM Stats](https://llm-stats.com/benchmarks/deepswe-1.1) lists **0 verified** of 53 results. Meanwhile [Poolside's table shows DeepSeek-V4-Pro at 9.0% on DeepSWE via Artificial Analysis](https://huggingface.co/poolside/Laguna-S-2.1/raw/main/README.md) vs Datacurve's 63%: a roughly sevenfold discrepancy I could not resolve. It may reflect harness, version or serving differences.
- **[FrontierCode 2.0](https://artificialanalysis.ai/evaluations/frontiercode-2-0)** (Cognition's benchmark, run by Artificial Analysis; 132 private tasks, graded on mergeability): Opus 5.5 46.4%, GPT-6 Astra 42.4%. [BenchLM's 6 Nov snapshot](https://benchlm.ai/md/benchmarks/frontiercode-2.md) places open-weight at Mistral Large 4 19.7, K3.1 Pro 19.6, DeepSeek-V4.1-Flash-1017 19.3, Kimi K3 17.7, Muse Spark Open 6.2. Caveats: Cognition sells a coding agent. BenchLM lists different harnesses per model (claude-code, codex, devin-cli, vibe), while AA's page describes its Stirrup harness. This conflict is unresolved. Older frontier Opus 4.8 (14.7) scores below these open models, so open-weight models are perhaps a generation or so behind on this measure (my inference).
- **[Artificial Analysis Index v5](https://www.implicator.ai/artificial-analysis-index-v5-open-model-gap/)** (27 Oct 2026, 25% coding, about 60% private tests): Opus 5.5 at 58 at launch; DeepSeek-V4.1-Flash-1017 and GLM-5.3 at 32. [K3.1 Pro at 46](https://letsdatascience.com/news/artificial-analysis-index-v5-drops-kimi-k31-pro-from-fourth-563911c6), gap 12, vs 3 under v4.3. The [live leaderboard](https://artificialanalysis.ai/leaderboards/models?open_weights=true) now shows Gemini 4 Carbon 60, Opus 5.5 59, K3.1 Pro 46, Hy4 35, Qwen3.8 Max 34, DeepSeek 33, GLM-5.3 32. Weights for K3.1 Pro are now [public on HF, created 2 Nov](https://huggingface.co/moonshotai/Kimi-K3.1-Pro) [API]. Scores across index versions are not comparable.
- **Long-horizon weakness shows even on vendor tables:** [Kimi's card](https://huggingface.co/moonshotai/Kimi-K3.1-Pro/raw/main/README.md): ProgramBench K3.1 31.5, DeepSeek 29.1, GLM-5.3 19.0 vs Opus 5.5 65.0; Terminal-Bench 4.0 K3.1 65.7 vs Opus 5.5 66.4, but GLM-5.3 41.8 and DeepSeek 42.3.
- **Harness effects evidence conflicts:**
  - [Arjmandi 2026](https://arxiv.org/abs/2609.11987): native vs neutral harness differences of about ±1.25pp, not significant on 80 paired tasks; task-type interaction post hoc.
  - [AgenticVBench](https://arxiv.org/pdf/2605.27705): 20pp shift for GPT-5.5 across harnesses, on a non-coding video task.
  - [The Scaffold Effect](https://arxiv.org/pdf/2607.22585) argues harness is a hidden variable (read only via search summary).
  - Blog claims of very large multipliers ([e.g.](https://www.mindstudio.ai/blog/better-model-vs-better-harness-agent-benchmark-score)) lack traceable primary sources.
  - Vendors ran different harnesses on the same benchmark: [GLM used Claude Code for Terminal-Bench and mini-swe-agent for DeepSWE](https://huggingface.co/zai-org/GLM-5.3-Flash/raw/main/README.md); [Qwen3.8-27B used Claude Code for DeepSWE](https://huggingface.co/Qwen/Qwen3.8-27B/raw/main/README.md).
- **Hosted model scores may overstate self-hosted results.** Qwen says its [official service adds a 1M default context and non-thinking mode](https://huggingface.co/Qwen/Qwen3.8-2.4T-A95B/raw/main/README.md) over the released weights. Moonshot's K3.1 scores are [internal evaluations on its own setup](https://huggingface.co/moonshotai/Kimi-K3.1-Pro/raw/main/README.md), and engine fidelity differs (see serving section). Whether a benchmark score holds at the quantization you deploy is untested in the sources I found.

**Hybrid options**
- [Claude via Bedrock, Google Agent Platform or Foundry](https://code.claude.com/docs/en/costs) bills to your cloud account; no metrics returned to Anthropic per the docs. The first-party API carries a [1.1× data-residency rate](https://code.claude.com/docs/en/costs).
- Azure's [Taiwan North is rolling out in stages for selected clients](https://focustaiwan.tw/business/202609010030) as of Sep 2026. Taiwan-local frontier inference is **unconfirmed**. One blog claims [only 3 of 10 OpenAI regional domains infer locally](https://ecorpit.com/openai-per-request-regional-processing-india-storage-only-2026/). Residency in storage does not imply inference locality.
- Private rented GPUs, such as [MI355X at $2.59/GPU-hr](https://www.gpuperhour.com/rent/mi355x), can run the same open weights in a VPC, useful for bursts.

### Inferences
- **Honest capability statement for stakeholders:** local models can credibly run a large part of the workload (scoped fixes, tests, docs, CI triage, code review pre-screening). Evidence does not support claiming parity on unattended long-horizon engineering or first-pass merge quality. Budget more review and retries. Requirements like "the whole agent workload can run locally" are feasible as a **continuity capability**, with lower quality, not as a quality-equal substitute.
- **Run your own bake-off before hardware purchase**: 50-100 internal tasks on your repositories (Azure DevOps project code), fixed harness, self-hosted serving at target quantization, tracking first-pass merge rate, human minutes and rework. Rent the GPUs for this.
- **Sensitivity routing (my design):**
  - S3 restricted: on-prem only; no fallback.
  - S2 confidential: on-prem default; approved private VPC open-weight burst.
  - S1 internal: on-prem default, frontier via private cloud endpoint on escalation (for example after two failed attempts or planning-class tasks).
  - S0 public: any.
  - Enforce at the gateway by repo classification and principal, with failure defaulting to deny. Keep prompts and tool schemas portable, and test the fallback regularly.
- Prefer model-agnostic harness and gateway APIs so moving a task class between tiers is configuration, not rewrite.
- Note that frontier-based escalation conflicts with "avoid cloud," and for Taiwan public-sector-linked data may need separate approval. Treat it as an explicit exception route, not default.

### Gaps
- Official AA FrontierCode table for all open models was truncated in my read; open-model values rely on BenchLM's snapshot. No benchmark tests self-hosted quantized open models under real enterprise harnesses.
- Bedrock Taipei and Vertex Taiwan frontier-model availability not confirmed. Check each provider's model-region matrix.

## Air-gapped operations and model supply chain

### Takeaway
Accept only safetensors, pin by commit and hash, sign internally, and treat scanners as a secondary control: pickle scanners have been repeatedly bypassed, and `trust_remote_code=False` has not been a complete guarantee. Treat chat templates and engine images as supply-chain artifacts too.

### Cited Findings
- Picklescan had repeated critical blocklist bypasses: [GHSA-7wx9-6375-f5wh, GHSA-vvpj-8cmc-gx39, GHSA-g38g-8gr9-h9xp (Mar 2026)](https://github.com/mmaitre314/picklescan/security/advisories) [API]. Also [CVE-2026-53875](https://www.sentinelone.com/vulnerability-database/cve-2026-53875/) and [CVE-2026-53873, fix version conflicting across sources](https://vuln.today/cve/CVE-2026-53873). Advice from these sources: re-scan previously cleared models and move to safetensors.
- [ProtectAI modelscan](https://github.com/protectai/modelscan): Apache-2.0, v0.8.8 from Feb 2026 [API], so a slow release cadence.
- [safetensors](https://github.com/huggingface/safetensors): Apache-2.0, v0.8.0 (June 2026) [API].
- [sigstore/model-transparency](https://github.com/sigstore/model-transparency): Apache-2.0, repo active Oct 2026, but the latest release is v1.1.1 from Oct 2025 [API]; small community (249 stars).
- vLLM model-loading risks: [CVE-2026-90553](https://github.com/vllm-project/vllm/security/advisories/GHSA-3c86-2m5g-59q7) (malicious model code despite `trust_remote_code=False`), [CVE-2026-27893](https://advisories.gitlab.com/pypi/vllm/CVE-2026-27893/). GGUF parser flaws: [Ollama](https://www.cyera.com/blog/bleeding-llama-a-critical-memory-leak-in-the-worlds-most-popular-local-ai-platform), [llama.cpp](https://github.com/ggml-org/llama.cpp/security/advisories/GHSA-96jg-mvhq-q7q7). Safetensors alone does not cover config-driven code paths or GGUF parsers.
- Chat templates matter functionally ([Kimi K2 tool-call failures](https://vllm.ai/blog/Kimi-K2-Accuracy)) and as an attack surface ([unbounded Jinja2 DoS](https://github.com/vllm-project/vllm/security/advisories/GHSA-542m-9234-8gvf)).
- Software pipelines fail too: [LiteLLM's compromise traced to a scanner dependency, Trivy](https://docs.litellm.ai/blog/security-update-march-2026), a warning that security tools themselves are supply-chain risks.

### Inferences (recommended process, my design)
1. **Intake (connected DMZ):** download by exact commit (not branch), record the HF commit SHA and per-file SHA-256, and take a license snapshot. Reject `.bin/.pt/.pkl` and any repo requiring remote code unless the code is vendored and reviewed.
2. **Inspect on an isolated, no-credential host:** run modelscan plus a second scanner in fail-closed mode. Diff config, tokenizer and chat template against previous versions, and review any Python files.
3. **Behavior test:** tool-call conformance (Kimi-Vendor-Verifier style), internal coding regression suite, zh-TW suite, and backdoor-oriented canary prompts (secret exfiltration, insecure code patterns). Scanning cannot detect poisoned weights; only behavioral tests partially can.
4. **Sign and transfer:** sign with an internal key (model-transparency or cosign), move via one-way transfer or scanned media, and re-verify inside.
5. **Serve:** read-only mounts, container digests pinned, no egress, HF offline mode, non-root, engine ports reachable only by the gateway.
6. **Updates:** quarterly model refresh and monthly engine security bundles (given the advisory rate). Keep the prior model plus parser version for rollback. Record model and engine version on every agent-generated commit for audit.
7. **Sustain:** mirror PyPI and container dependencies by hash. Keep the NVIDIA driver, CUDA and license server (if used) in the offline bundle, and confirm NIM offline licensing with NVIDIA.

### Gaps
- No authoritative, independent evaluation of model-backdoor detection for open coding models found. Air-gapped NVIDIA AI Enterprise licensing terms were not confirmed. Hugging Face's own scanning and OpenSSF signing adoption were not established by sources I could read.
