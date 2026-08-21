# AI Evaluation Metrics

This document defines a practical evaluation framework for AI systems.

It is designed to work across both:
- Non-agentic AI systems (for example, single-turn or multi-turn LLM applications without tool orchestration)
- Agentic AI systems (for example, LLM plus RAG plus tool-calling workflows)

## How to Use This Framework

Not every metric applies to every architecture. Use metrics by applicability:

- Universal metrics: apply to all AI systems
- RAG-specific metrics: apply when retrieval is part of the architecture
- Agentic-specific metrics: apply when the system performs tool orchestration and multi-step actions

## Quick Term Guide (Plain English)

- Rubric: The rulebook for judging quality (what counts as success vs failure).
- Representative sample: A sample that reflects real user behavior, not only easy or ideal cases.
- Independent review: Two reviewers label separately without seeing each other's decisions.
- Production proxy signal: An indirect real-world indicator (for example, retries, escalations, complaints).
- Drift: Performance changing over time, often after model, prompt, or traffic changes.
- Groundedness: How much of the answer is supported by the provided source context.
- Hallucination: When the model states things that are fabricated or not supported by evidence.
- Top-k retrieval: The top number of chunks returned by search (for example, top-5).
- p50 latency: The median response time (what a typical user experiences).
- p95 latency: The slower end of response time (the slowest 5 percent of requests).
- Throughput (TPS): How fast the model generates tokens per second.
- Benchmark set: A fixed test set used repeatedly to compare versions fairly.

## Group 1: Generation Quality and Accuracy

These metrics evaluate whether model outputs are accurate, useful, and aligned to user goals.

### 1. Task Success Rate (Goal Accuracy)

- What it means: The percentage of interactions where the AI reaches the intended user goal without human correction.
- Why it matters: This tells us most clearly whether the AI is actually useful in real work.
- Applies to: Universal
- How to create it: Label each evaluated interaction as success or failure using a clear rubric.
- Formula: successful outcomes / total evaluated outcomes
- Data needed: interaction ID, expected goal, evaluator outcome label
- Example:
	1. Reviewed interactions: 200
	2. Fully resolved by AI without human help: 170
	3. Task Success Rate = 170/200 = 85%
- Manual evaluation workflow:
	1. Define the success rubric (what counts as success) for task completion.
	2. Sample 100 to 300 representative interactions.
	3. Have 2 human reviewers label each interaction independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly score and trend.
	6. Compare with production proxy signals (real-world indirect indicators, such as more user retries, more escalations, more complaints, or lower completion rate) to catch drift (performance change over time).
- Suggested target: 85 percent or higher for mature production use cases.

### 2. Groundedness (Faithfulness)

- What it means: For retrieval-based systems, the percentage of claims in a response that can be directly verified in retrieved context.
- Why it matters: Keeps outputs anchored to trusted evidence instead of unsupported model priors.
- Applies to: RAG-specific
- How to create it: Break each response into atomic claims, then verify each claim against retrieved passages.
- Formula: verifiable claims / total claims in response
- Data needed: final response, claim annotations, retrieved context IDs and text
- Example:
  1. Retrieved context says: "Refunds are allowed within 30 days with receipt."
  2. AI response says:
	  - "Refund window is 30 days." -> verifiable
	  - "Store credit is available after 30 days." -> not in retrieved context, not verifiable
  3. So groundedness would be:
	  - verifiable claims = 1
	  - total claims = 2
	  - groundedness = 1/2 = 50%
- Manual evaluation workflow:
	1. Define the claim-verification rubric (what counts as a supported claim) for groundedness.
	2. Sample 100 to 300 representative retrieval interactions.
	3. Have 2 human reviewers verify claims independently.
	4. Resolve disagreements and finalize claim labels.
	5. Track weekly groundedness score and trend.
	6. Compare with production proxy signals (for example, user trust or citation complaints) to catch drift (performance change over time).

### 3. Hallucination Rate

- What it means: The rate of outputs containing false, fabricated, or unverified claims.
- Why it matters: High hallucination risk reduces trust and increases compliance exposure.
- Applies to: Universal (especially critical in RAG and high-risk domains)
- How to create it: Use evaluator checks or automated fact checks to label outputs with hallucinated claims.
- Formula: hallucinated responses / total evaluated responses
- Data needed: response text, fact-check labels, reference sources
- Example:
	1. Evaluated responses: 300
	2. Responses with fabricated or unsupported facts: 9
	3. Hallucination Rate = 9/300 = 3%
- Manual evaluation workflow:
	1. Define the hallucination rubric (what counts as fabricated or unverifiable).
	2. Sample 100 to 300 representative interactions.
	3. Have 2 human reviewers label hallucinations independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly hallucination score and trend.
	6. Compare with production proxy signals (for example, correction loops) to catch drift (performance change over time).
- Suggested target: Under 2 to 5 percent depending on domain risk.

## Group 2: Retrieval and Tool Execution Efficiency

These metrics evaluate whether the system can fetch the right information and execute external actions reliably.

### 4. Retrieval Hit Rate (Context Recall and Precision)

- What it means: How often retrieval returns the right context to answer the query.
- Why it matters: If retrieval fails, answer quality drops even with a strong model.
- Applies to: RAG-specific
- How to create it: Define relevance criteria and score whether top-k retrieved chunks contain answer-critical evidence.
- Formula: queries with relevant retrieved context / total retrieval queries
- Data needed: query, top-k retrieved chunks, relevance labels
- Example:
	1. Retrieval queries checked: 120
	2. Queries where top-k chunks included answer-critical evidence: 102
	3. Retrieval Hit Rate = 102/120 = 85%
- Manual evaluation workflow:
	1. Define the relevance rubric (what counts as relevant evidence) for retrieved context.
	2. Sample 100 to 300 representative retrieval queries.
	3. Have 2 human reviewers label relevance independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly retrieval hit score and trend.
	6. Compare with production proxy signals (for example, downstream answer failures) to catch drift (performance change over time).

### 5. Tool Success Rate (Function-Calling Accuracy)

- What it means: The percentage of successful tool invocations, including correct tool choice, argument structure, and execution order.
- Why it matters: Agentic systems depend on reliable action execution, not just text generation.
- Applies to: Agentic-specific
- How to create it: Log each tool attempt and mark it successful only if selection, parameters, and outcome are all correct.
- Formula: successful tool executions / total tool execution attempts
- Data needed: tool name, input arguments, validation status, execution result
- Example:
	1. Total tool calls: 90
	2. Calls with correct tool, correct parameters, and successful execution: 81
	3. Tool Success Rate = 81/90 = 90%
- Manual evaluation workflow:
	1. Define the tool success rubric (what counts as a correct tool action: tool, parameters, and order).
	2. Sample 100 to 300 representative tool-invocation traces.
	3. Have 2 human reviewers label success independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly tool success score and trend.
	6. Compare with production proxy signals (for example, failed task completion) to catch drift (performance change over time).

## Group 3: User Experience and Handoff

These metrics capture real user friction and operational effectiveness.

### 6. Escalation Rate (Human-in-the-Loop Frequency)

- What it means: The percentage of sessions that require transfer to a human.
- Why it matters: A direct indicator of automation quality and support cost.
- Applies to: Universal (more visible in support and operations workflows)
- How to create it: Mark sessions with an escalation event and track the rate over total sessions.
- Formula: sessions escalated to humans / total sessions
- Data needed: session ID, escalation flag, escalation reason
- Example:
	1. Total sessions this week: 1,000
	2. Sessions escalated to human support: 60
	3. Escalation Rate = 60/1,000 = 6%
- Manual evaluation workflow:
	1. Define the escalation rubric (what counts as valid escalation vs avoidable escalation).
	2. Sample 100 to 300 representative sessions.
	3. Have 2 human reviewers label escalation quality independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly escalation score and trend.
	6. Compare with production proxy signals (for example, support workload changes) to catch drift (performance change over time).

### 7. Correction or Re-Prompt Rate

- What it means: How often users must immediately rephrase or retry due to poor initial responses.
- Why it matters: A strong early signal of frustration and poor prompt or system alignment.
- Applies to: Universal
- How to create it: Detect retry intent in immediate follow-up turns within a defined time window.
- Formula: sessions with immediate correction or retry / total sessions
- Data needed: conversation turns, timestamps, retry-intent labels
- Example:
	1. Total sessions: 800
	2. Sessions with immediate retry/correction messages: 140
	3. Correction or Re-Prompt Rate = 140/800 = 17.5%
- Manual evaluation workflow:
	1. Define the correction rubric (what counts as a retry due to poor quality).
	2. Sample 100 to 300 representative sessions.
	3. Have 2 human reviewers label correction intent independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly correction score and trend.
	6. Compare with production proxy signals (for example, abandonment rate) to catch drift (performance change over time).

## Group 4: Performance and Cost

These metrics ensure the system stays responsive and financially sustainable.

### 8. Latency (p50 and p95)

- What it means: End-to-end response time. p50 captures typical experience, p95 captures slow-tail behavior.
- Why it matters: Users feel p95 delays as instability, even if median speed looks good.
- Applies to: Universal
- How to create it: Capture end-to-end duration per request and compute percentile distributions.
- Formula: p50 and p95 percentiles of request duration
- Data needed: request ID, request start time, response end time
- Example:
	1. Measured request durations for one day
	2. p50 latency = 1.8 seconds
	3. p95 latency = 6.5 seconds
	4. Interpretation: typical response is fast, but slow-tail responses still affect a subset of users
- Manual evaluation workflow:
	1. Define the latency rubric by experience tier (what counts as acceptable, degraded, or unacceptable speed).
	2. Sample 100 to 300 representative requests across workloads.
	3. Have 2 human reviewers validate trace quality and outlier classification independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly p50 and p95 trend.
	6. Compare with production proxy signals (for example, drop-off rate) to catch drift (performance change over time).

### 9. Cost per Query

- What it means: Total per-request cost, including model tokens, retrieval, tool calls, and infrastructure overhead.
- Why it matters: Connects technical performance to unit economics.
- Applies to: Universal
- How to create it: Attribute token, retrieval, tooling, and infrastructure costs per request and aggregate.
- Formula: total request cost / total queries
- Data needed: token usage, model pricing table, infra allocation, tool and retrieval costs
- Example:
	1. Total weekly operating cost (model + retrieval + tools + infra): $2,400
	2. Total weekly queries: 30,000
	3. Cost per Query = 2,400/30,000 = $0.08
- Manual evaluation workflow:
	1. Define the costing rubric (which cost components are included).
	2. Sample 100 to 300 representative queries across routes.
	3. Have 2 human reviewers validate cost attribution independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly cost per query trend.
	6. Compare with production proxy signals (for example, margin per task) to catch drift (performance change over time).

### 10. Tokens per Second (TPS)

- What it means: Model generation throughput.
- Why it matters: Impacts perceived responsiveness, especially in streaming interfaces.
- Applies to: Universal for LLM-based systems
- How to create it: Track output tokens and generation duration per request.
- Formula: output tokens / generation time in seconds
- Data needed: output token count, generation start and end times
- Example:
	1. Output tokens generated: 300
	2. Generation time: 6 seconds
	3. TPS = 300/6 = 50 tokens/second
- Manual evaluation workflow:
	1. Define the TPS rubric (what counts as normal speed vs degraded speed).
	2. Sample 100 to 300 representative generation requests.
	3. Have 2 human reviewers validate token and timing trace quality independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly TPS trend.
	6. Compare with production proxy signals (for example, perceived streaming smoothness) to catch drift (performance change over time).

## Group 5: Safety, Compliance, and Stability

These metrics reduce organizational risk and protect quality during change.

### 11. Toxicity and Guardrail Violation Rate

- What it means: Rate of unsafe outputs, policy violations, or successful safety bypasses.
- Why it matters: Protects users, brand reputation, and legal posture.
- Applies to: Universal
- How to create it: Run outputs through policy classifiers and include red-team jailbreak test outcomes.
- Formula: violating outputs / total outputs
- Data needed: output text, policy labels, safety test outcomes
- Example:
	1. Total outputs reviewed: 5,000
	2. Outputs violating safety policy: 25
	3. Toxicity or Guardrail Violation Rate = 25/5,000 = 0.5%
- Manual evaluation workflow:
	1. Define the policy violation rubric (what counts as a violation, with severity and category).
	2. Sample 100 to 300 representative outputs, including adversarial prompts.
	3. Have 2 human reviewers label violations independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly violation score and trend.
	6. Compare with production proxy signals (for example, abuse reports) to catch drift (performance change over time).

### 12. Regression Rate

- What it means: Frequency of quality drops after prompt updates, model swaps, retrieval tuning, or code changes.
- Why it matters: AI systems are probabilistic; small changes can create large downstream behavior shifts.
- Applies to: Universal
- How to create it: Maintain a fixed benchmark set and compare new run results to last known passing baseline.
- Formula: failed previously passing test cases / total previously passing test cases
- Data needed: benchmark dataset version, baseline pass/fail state, current pass/fail state
- Example:
	1. Previously passing benchmark test cases: 200
	2. Those same cases now failing after update: 14
	3. Regression Rate = 14/200 = 7%
- Manual evaluation workflow:
	1. Define the regression rubric (what counts as a functional break).
	2. Sample 100 to 300 representative benchmark interactions.
	3. Have 2 human reviewers validate regressions independently.
	4. Resolve disagreements and finalize labels.
	5. Track weekly regression score and trend.
	6. Compare with production proxy signals (for example, incident volume) to catch drift (performance change over time).

## Recommended Metric Baseline by Architecture

### Non-Agentic LLM System

Prioritize:
- Task Success Rate
- Hallucination Rate
- Correction or Re-Prompt Rate
- Latency (p50 and p95)
- Cost per Query
- Toxicity and Guardrail Violation Rate
- Regression Rate

Optional (if retrieval is used):
- Groundedness
- Retrieval Hit Rate

### Agentic RAG System

Prioritize everything above, plus:
- Groundedness
- Retrieval Hit Rate
- Tool Success Rate
- Escalation Rate

## Implementation Notes

- Start with a small, representative evaluation dataset before scaling measurement.
- Track weekly trends, not only point-in-time snapshots.
- Use benchmarks as starting ranges; final targets should be tuned by domain risk and business needs.
