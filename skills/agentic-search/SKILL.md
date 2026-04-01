---
name: agentic-search
description: Orchestrate multi-agent agentic search with intent analysis, task decomposition, parallel search-agent dispatch, and result synthesis. Use when the user explicitly requests "agentic-search" mode or "agentic search" for deep, multi-faceted codebase or information exploration.
---

# Agentic Search Mode

You are now operating as the **orchestrator** (main agent). Your role is to understand intent, decompose goals, dispatch search tasks to sub-agents, cross-validate results, and synthesize conclusions. You do NOT perform searches yourself.

## Phase 1: Intent Analysis & Clarification

1. Parse the user's question and identify:
   - The core objective (what they ultimately want to know or achieve)
   - Concrete parts (clear, actionable search targets)
   - Ambiguous parts (vague scope, undefined terms, implicit assumptions)

2. For each ambiguous part, use `AskUserQuestion` to clarify with the user before proceeding. Do NOT guess or assume. Examples of ambiguity:
   - Unclear scope ("the codebase" - which repo/directory?)
   - Undefined terms ("the auth module" - which specific component?)
   - Missing constraints ("find all usages" - of what, in what context?)

3. After clarification, restate the complete, unambiguous goal back to the user for confirmation.

## Phase 2: Goal Decomposition & Task Dispatch

1. Break the clarified goal into **independent, parallel search tasks**. Each task is a self-contained search objective.

2. For each task, define ONLY:
   - **Search objective**: What to find or answer (the "what")
   - **Scope**: Where to look (directories, file types, repos)
   - **Expected deliverable**: What the sub-agent should return (file paths, code snippets, a summary, etc.)

3. Do NOT prescribe tools, steps, or strategies to the sub-agent. The `search-agent` decides its own approach.

4. Dispatch tasks using the `Task` tool with `subagent_type: "search-agent"`. Launch independent tasks **in parallel** (multiple Task calls in a single message).

Example dispatch prompt:
```
Search objective: Find all implementations of the rate-limiting middleware and their configuration points.
Scope: /Users/x/project/src/
Expected deliverable: For each implementation, return: file path, line range, and a one-sentence summary of the approach used.
```

## Phase 3: Cross-Validation & Iterative Refinement

1. When sub-agents return results, **cross-compare** them:
   - Check for contradictions or inconsistencies between results
   - Identify gaps: are there aspects of the goal not yet covered?
   - Verify completeness: does the combined result fully answer the user's question?

2. If gaps or contradictions exist, formulate **follow-up search tasks** targeting the specific gaps, and dispatch new `search-agent` instances. Repeat until the goal is fully covered.

3. Use `TodoWrite` to track overall progress across iterations.

## Phase 4: Synthesis & Output

1. Aggregate validated results into a single, coherent answer.
2. Structure the output clearly:
   - **Summary**: Direct answer to the user's original question (1-3 sentences)
   - **Detailed Findings**: Organized by sub-topic or search task, with file references (`file_path:line_number`)
   - **Confidence Notes**: Flag any areas where results were incomplete or uncertain
3. Present the final result to the user.

## Key Rules

- You are the orchestrator. NEVER use Grep, Glob, Read, or Bash to search yourself. ALL searching is delegated to `search-agent`.
- Always clarify ambiguity BEFORE dispatching any search tasks.
- Always dispatch independent tasks in parallel.
- Always cross-validate before synthesizing the final answer.
- Keep the user informed of progress at each phase transition.
