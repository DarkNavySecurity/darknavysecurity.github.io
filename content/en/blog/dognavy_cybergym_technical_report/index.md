+++
title = 'DoGNAVY CyberGym Technical Report'
date = 2026-08-05T14:57:28+08:00
draft = false
canonicalURL = 'https://deepsec.darknavy.net/blog/cybergym'
ShowCanonicalLink = true
+++

DoGNAVY is a joint AI security harness by [deepsec@DARKNAVY](https://deepsec.darknavy.net) and independent security researchers based in Shanghai. In a full CyberGym Level 1 evaluation, it passed 1,369 of 1,507 tasks (90.84%).

A broader report from the DoGNAVY team, with deepsec@DARKNAVY as a collaborator, covers vulnerability reproduction, exploitation, and repair: [DoGNAVY: Autonomous Agents for Full-Lifecycle Vulnerability Management (PDF)](https://deepsec.darknavy.net/reports/dognavy-autonomous-agents-full-lifecycle-vulnerability-management.pdf)

{{< figure src="attachments/cybergym-level-1-benchmark-comparison.png" alt="CyberGym Level 1 leaderboard comparison showing DoGNAVY with a 90.84% verified rate and 96.4% vulnerable-build crash rate, alongside MDASH at 92.0%, Atlas at 90.9%, GPT-5.5-Cyber at 85.6%, and Claude Mythos Preview at 83.1%" caption="CyberGym Level 1 leaderboard comparison" >}}

## Abstract

This report presents DoGNAVY, an agentic system for vulnerability reproduction, and documents its evaluation on the complete [CyberGym](https://www.cybergym.io/cybergym/) Level 1 task set. DoGNAVY combines reachability analysis, proof-of-concept (PoC) construction, dynamic testing, and independent review in a multi-agent workflow. The system can revise its approach as evidence accumulates, subject to explicit task boundaries and runtime controls.

To limit benchmark-specific assistance from the harness, the evaluation provided only general vulnerability-research knowledge. It included no CyberGym-specific solutions, historical PoCs, patches, or cross-task memory. Each task started in a fresh, isolated environment. We also built a secure sandbox for the agent based on [AgentDoG’s](https://github.com/AI45Lab/AgentDoG) design, constraining its behavior to prevent any impact on real-world networks.

The results in this report reflect the server-side evaluation. DoGNAVY passed differential validation on 1,369 of 1,507 tasks, for a success rate of 90.84%. In total, it produced inputs that crashed the vulnerable build in 1,453 tasks, covering 96.42% of the benchmark. Of these, 79 also crashed the patched build and therefore did not pass differential validation. DoGNAVY submitted no candidate PoC for the remaining 54 tasks. The result set includes one outcome for each of the 1,507 tasks.

For resource accounting, we used the execution designated as canonical for each task. These executions used 39.28 billion tokens and 524,049 LLM requests, with an estimated model cost of USD 22,648.43. The summed agent-trace activity span was 2,195.89 hours. Per task, the corresponding averages were 26.06 million tokens, 347.74 LLM requests, USD 15.03, and 87.43 minutes.

## System Design

### Workflow and Autonomous Decision-Making

DoGNAVY decomposes vulnerability reproduction into explicit, inspectable stages. Within those stages, agents choose analysis strategies, form hypotheses, and select tools. As new evidence becomes available, the system can shift effort among source analysis, input construction, and runtime testing, or return to an earlier assumption that is no longer supported.

Separate review agents assess whether a candidate PoC reaches the expected code path, matches the target vulnerability, reproduces reliably, and is not an assertion failure, environmental anomaly, or adjacent flaw.

### Vulnerability Reachability Analysis

DoGNAVY connects a local vulnerability description to the evaluation program’s actual entry point by reconstructing the call chain and the parsing, state, and data constraints that lead to the flaw. Code indexing narrows large codebases, while task state records confirm paths, constraints, and open questions. When static evidence is inconclusive, the system preserves uncertainty and uses dynamic validation rather than assuming the path is unreachable.

### Static and Dynamic Analysis

DoGNAVY combines static and dynamic analysis in a feedback loop. Static analysis proposes vulnerability paths and input constraints; dynamic analysis tests them using coverage, error locations, crash types, and stability. Each result updates the next analysis or input. A PoC is accepted only if it produces repeatable, target-relevant behavior under the actual entry point and runtime conditions; exits, assertion failures, unrelated crashes, and environment-specific anomalies are insufficient.

### Knowledge Base and Memory

For this evaluation, DoGNAVY had access only to general security-research knowledge, with no task-specific identifiers, historical PoCs, patches, project solutions, or dataset-target knowledge. Cross-task memory was disabled, so each task began independently, while within-task memory retained compressed paths, constraints, failed attempts, runtime feedback, and unresolved hypotheses. This sacrifices continual learning but provides a clearer test of generalization on open-source programs.

### Sandbox and Runtime Controls

Cybersecurity evaluations create two distinct risks. An agent may obtain benchmark answers from outside the evaluation, or it may act on real systems beyond the intended scope. The July 2026 [OpenAI–Hugging Face incident](https://openai.com/index/hugging-face-model-evaluation-security-incident/), in which models escaped an ExploitGym evaluation environment and accessed Hugging Face production systems, showed why both controls matter.

DoGNAVY therefore ran each task in a dedicated sandbox. The runtime guardrails draw on [AgentDoG's](https://github.com/AI45Lab/AgentDoG) approach to trajectory-level safety diagnosis and intervention. Network egress was restricted to the model service and the resources required to prepare the task environment.

## Evaluation Setup

### Scope and System Configuration

| Item | Configuration |
| :--- | :--- |
| Benchmark | CyberGym Level 1 |
| Tasks | 1,507 |
| Projects | 188 open-source projects |
| Model | GLM-5.2 |
| Per-task time limit | 4 hours (14,400 seconds) |
| Cross-task memory | Disabled |
| Access to the patched build | None |
| Web search and retrieval tools | The allowlist was limited to model APIs and build setup. |

### Separation Between Agent and Validator

The DoGNAVY execution environment and the CyberGym validation service ran in separate environments. The only connection between them was a controlled submission interface. Agents could not access the validator's address, server-side credentials, or patched build. During PoC development, runtime feedback came only from the vulnerable build. The validation service independently executed submitted inputs against both builds.

### Per-Task Runtime Environment

Each task used a separate runtime image derived from the official CyberGym container. The image preserved the target entry point, build configuration, and sanitizer settings, while adding the general-purpose static and dynamic analysis tools used by DoGNAVY. Agents inspected source code, executed candidate inputs, and collected runtime evidence in the vulnerable build's intended environment.

Each task also received a separate container, working directory, and task state. Containers did not mount data from other tasks, the experiment database, submission logic, or the Docker control interface. Conversations and task memory were isolated in the same way.

### Input Sanitization

At the start of each task, the agent received the vulnerable source tree, a vulnerability description, and the minimum metadata needed to run the target. Before execution, the environment removed reference PoCs, Git history, and other materials that could reveal the expected answer. Task identifiers and server-side submission metadata remained outside the agent workspace.

### Network Restrictions

The egress allowlist was intended only for model API calls and dependency or build preparation. No web-search or web-fetch adapters, and no MCP servers, were exposed to the agents. Agents were instructed not to retrieve known crashing inputs, public fuzzing corpora, issue-tracker material, or other vulnerability-specific artifacts. We separately audited execution traces for external access; the result of that audit is reported below.

### Time Budget

Each task had a configured time limit of 4 hours, or 14,400 seconds. This is a maximum budget, not a statement about the observed duration of every task.

## Results

### Outcome Distribution

| Outcome | Tasks | Share |
| :--- | ---: | ---: |
| Passed differential validation | 1,369 | 90.84% |
| Passed validation, but belong OOM | 5 | 0.33% |
| Both builds crashed | 79 | 5.24% |
| No candidate PoC submitted | 54 | 3.58% |
| **Total** | **1,507** | **100.00%** |

Results differed modestly by source. DoGNAVY passed 1,240 of 1,368 ARVO tasks, for a success rate of 90.64%, and 129 of 139 OSS-Fuzz tasks, for a success rate of 92.81%.

### Runtime Distribution

Runtime statistics use the agent-trace activity span for the canonical execution of each task. The start is the earliest valid timestamp across the main-agent and subagent traces, and the end is the latest valid timestamp. This measurement excludes preparation before the first trace event, including image download, image loading, and image construction. It also excludes teardown after the final trace event. Valid start and end timestamps were available for all 1,507 tasks.

| Time Definition | Number of Tasks | P25 | Median | P75 | P90 | Mean |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: |
| Agent trace activity span | 1,507 | 26.88 minutes | 57.97 minutes | 138.55 minutes | 215.90 minutes | 87.43 minutes |

The aggregate agent trace activity span is 7,905,206.00 seconds, or 2,195.89 hours; dividing it by all 1,507 tasks yields 5,245.66 seconds/task, or 87.43 minutes/task. The maximum is 14,428.00 seconds.

### Tokens, LLM Requests, and Estimated Cost

Resource accounting uses raw server traces from the canonical execution of each task. Main-agent and subagent JSONL traces share one deduplication scope. Responses are deduplicated by `message.id`, and the final cumulative usage record for each response is used so that streamed fragments are not counted more than once. Zero-token `<synthetic>` responses are excluded from the request count. Where a JSONL file was malformed, recoverable timestamps, response identifiers, and usage records were parsed line by line. Complete resource statistics were available for all 1,507 tasks.

| Metric | Total | Mean per task |
| :--- | ---: | ---: |
| Non-cached input tokens | 11,789,091,762 | 7,822,887.70 tokens/task |
| Cache-read input tokens | 27,268,463,296 | 18,094,534.37 tokens/task |
| Output tokens | 219,436,852 | 145,611.71 tokens/task |
| Total tokens | 39,276,991,910 | 26,063,033.78 tokens/task |
| LLM requests | 524,049 | 347.74 requests/task |
| Estimated cost | 22,648.43 USD | 15.03 USD/task |

| Per-Task Metric | P25 | Median | P75 | P90 | Mean | Maximum |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: |
| Total tokens | 10,041,808.50 | 14,955,870 | 27,717,142.50 | 60,598,896.20 | 26,063,033.78 | 196,487,587 |
| LLM requests | 169 | 235 | 382 | 730.8 | 347.74 | 2,352 |
| Estimated cost (USD) | 5.11 | 8.56 | 15.77 | 37.33 | 15.03 | 111.86 |

### Network-Access Audit

We audited the trajectories for unintended use of external vulnerability-specific information. The results were:

| Audit Category | Number of Cases |
| :--- | ---: |
| Clean | 1,329 |
| Dependency-related access only | 104 |
| Attempted search or access that failed or returned no real result | 73 |
| Flagged non-dependency Git access | 1 (`arvo_50683`) |
| **Total** | **1,507** |

For `arvo_50683`, external repository content was successfully retrieved in a subagent path, but that path was not used in the final accepted solution. The audit found no evidence that any successful result relied on vulnerability-specific external information.

## Conclusion

DoGNAVY uses a structured workflow that can revisit earlier hypotheses as new evidence emerges. It combines vulnerability descriptions with analysis of program entry points, code reachability, input constraints, and runtime behavior. Independent review stages screen out incidental crashes and prioritize reproducible PoCs that match the target vulnerability and hold up under server-side evaluation. The system used no CyberGym task-specific knowledge, shared no memory across tasks, and had no access to patched builds. Under these conditions, DoGNAVY passed server-side differential validation on 1,369 of 1,507 Level 1 tasks, achieving a validation rate of 90.84%.

{{< figure src="attachments/cybergym-level-1-leaderboard.png" alt="CyberGym Level 1 leaderboard showing DoGNAVY ranked third with a 90.8% success rate on August 3, 2026, behind MDASH at 92.0% and Wiz Atlas at 90.9%, among the top 10 agents and models" caption="CyberGym Level 1 leaderboard with DoGNAVY ranked third" >}}

DoGNAVY-v0.7<br>
July 29, 2026
