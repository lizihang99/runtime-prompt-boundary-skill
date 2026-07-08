---
name: agent-written-prompt-boundary
description: Use when an AI coding agent is implementing or editing a software feature that calls an LLM and must write or revise that product prompt without mixing development discussion, implementation detail, sample data, or unsupported assumptions into the runtime prompt.
---

# Agent-Written Prompt Boundary

## Overview

When a coding agent implements a software feature that calls an LLM, it may also write the product prompt for that call. This skill keeps that agent-written prompt bounded: development context may shape the design, but each fact must land in the cheapest artifact that preserves its runtime effect.

## When to Use

- adding or editing a product LLM call while implementing a software feature
- turning mixed product, UX, architecture, and prompt discussion into runtime artifacts
- shrinking a prompt that has absorbed too much project backstory
- aligning prompt text with downstream schemas, parsers, tools, or UI consumers

## When Not to Use

- choosing a model, provider, price tier, or latency budget without prompt design work
- wiring SDK calls, auth, retries, logging, or transport code when prompt content is not in scope
- generic "should we use AI here?" discussion without a concrete call site
- prompt-engineering chat detached from a real feature, prompt file, or runtime surface

## Non-Negotiable Safety Gate

Before removing, distilling, or relocating content, check whether it expresses safety, compliance, privacy, abuse-prevention, uncertainty disclosure, escalation, identity, or authorization constraints.

- preserve the behavioral effect first
- compress wording only if the safety behavior stays explicit
- never remove a constraint just because it is rare, verbose, or absent from happy-path examples

When safety and brevity conflict, safety wins.

If a rule protects the user, the downstream system, or the external world from harm, default it to `derived-runtime-rule` or `output-contract`, not `design-context`.

Do not invent new domain policy while preserving safety. If you add a guardrail not present in the user's brief, repository, schema, or cited policy, mark it as a proposed assumption instead of silently folding it into the runtime contract.

## Core Questions

For each contested piece of information, ask:

1. Who consumes it?
2. Is it available at design time, runtime, or only in implementation?
3. Does it change behavior as instruction, input, example, contract, or test?
4. What is the cheapest artifact that preserves that effect?

## Role Classification

Use these roles:

| Role | Use |
| --- | --- |
| `design-context` | Shapes prompt design, but should not appear in the runtime prompt. |
| `derived-runtime-rule` | Stable instruction that changes model behavior at runtime. |
| `runtime-variable` | Dynamic user, content, or context data supplied at runtime. |
| `few-shot-example` | Example kept only because it materially improves runtime behavior. |
| `eval-case` | Test or review artifact kept out of the prompt body. |
| `proposed-assumption` | Helpful default, enum, schema, threshold, policy extension, or downstream expectation that is not evidenced yet. Keep outside the runtime contract until accepted by the user or proven by code. |
| `output-contract` | Schema, JSON shape, tool contract, parser requirement, or type expectation. |
| `implementation-detail` | Code, storage, model settings, logging, transport, or wiring detail. |
| `agent-only` | Helps the coding agent do the task; keep out of product runtime prompts. |
| `exclude` | Ignore for this LLM call. |

Safety, refusal, escalation, and uncertainty rules usually belong in `derived-runtime-rule` unless the downstream contract requires them structurally.

Only use `output-contract` for requirements supported by the user brief, existing code, schema, parser, tool, or downstream consumer. If the output shape is unknown, write `not specified` and state any suggested default as an assumption outside the runtime contract.

When a downstream contract names exact fields, enum values, or output type, preserve that contract exactly. Do not add required fields, enum values, wrappers, or parser obligations unless the code or user request supports them. Put helpful additions under `Proposed assumptions`, not inside the runtime contract.

Proposed assumptions must not appear inside the runtime prompt's formal output format as if they were required. If an enum, field, or wrapper is only a proposal, keep it outside the contract and call it out separately.

Examples inside a runtime prompt count as policy too. Do not add illustrative triggers, categories, or thresholds that broaden the user's stated policy unless they are marked as non-contractual assumptions outside the prompt.

Common conventions are not evidence. A field named `risk_level` does not prove an enum such as `low | medium | high`; a field named `confidence` does not prove a numeric range. If only a field name is evidenced, type it generically, such as `string`, and list conventional values only under assumptions.

## Conflict Resolution

When one fact seems to fit multiple roles, resolve it in this order:

1. If it is irrelevant to this call, use `exclude`.
2. If it only helps the coding agent, use `agent-only`.
3. If it only affects wiring or product code, use `implementation-detail`.
4. If it exists only to test or review behavior, use `eval-case`.
5. If it is dynamic runtime data, use `runtime-variable`.
6. If it is useful but not evidenced by the brief, code, schema, or downstream consumer, use `proposed-assumption`.
7. If downstream code or tooling requires it structurally, use `output-contract`.
8. If it imposes stable behavior or safety policy, use `derived-runtime-rule`.
9. If an example materially improves runtime behavior, use `few-shot-example`.
10. Otherwise keep it as `design-context`.

Rules:

- assign exactly one primary role to each contested fact
- use a secondary note only to explain the choice, not to duplicate the fact
- do not copy the same requirement into instructions, examples, and evals unless the duplication is intentional and justified

For tie-break examples, see `references/priority-rules.md`.

## Workflow

1. Establish the runtime job:
   - LLM call or feature
   - user-facing purpose
   - upstream inputs
   - downstream consumer
   - failure mode that matters most
   - which output shape is evidenced, if any
2. Write an `Intent Check` that states what the runtime prompt is for, what it uses, what it outputs, what it must not do, and the main risk if misunderstood.
3. If the `Intent Check` exposes material ambiguity in the goal, inputs, output, downstream consumer, or safety boundary, ask for confirmation before finalizing the prompt.
4. Choose the path:
   - existing prompt: audit current lines before rewriting
   - new prompt: start from a boundary ledger
5. Build a compact ledger for contested or high-risk information.
6. Write or edit the prompt with separate sections for:
   - instructions
   - runtime inputs
   - output contract
   - safety, refusal, uncertainty, or escalation rules
7. If one prompt is doing multiple incompatible jobs, split the call or stage the workflow before adding more instructions.
8. Return the required artifacts.

For existing prompts, preserve the current behavior boundary first. Prefer minimal diffs unless the structure itself is causing leakage or overload.

## Required Output

Unless the user explicitly asks for raw prompt text only, return:

- `Intent Check`
- `Boundary Summary`
- `Boundary Ledger`
- `Runtime Prompt`
- `Runtime Variables`
- `Output Contract`
- `Change Note` for existing prompts

Use the exact headings from `references/output-template.md`.

Start with `Intent Check`; do not add process narration before the required artifact headings.

If the user asks for raw prompt text only, still do the classification work internally before drafting.

## Pressure Testing

Before deploying edits to this skill, or after materially changing the workflow, run at least three scenarios from `cases/`.

The set must include:

- one safety-preservation case
- one dynamic-data extraction case
- one overloaded-prompt split or stage case

A pass means the resulting artifacts:

- preserve safety and uncertainty constraints
- keep development-only context out of the runtime prompt
- assign contested facts consistently

## Final Checks

- Can the runtime model succeed without seeing the development conversation?
- Does the `Intent Check` make the prompt's purpose, inputs, output, non-goals, and misunderstanding risk obvious to the user?
- If the `Intent Check` is uncertain in a way that changes the prompt boundary, did you ask before finalizing?
- Does every included sentence have a clear runtime owner?
- Are safety, refusal, escalation, and uncertainty constraints explicit?
- Are added safety or domain rules grounded in evidence, or clearly marked as assumptions?
- Did examples avoid expanding policy beyond the brief?
- Are dynamic facts variables instead of hard-coded examples?
- Is output structure defined only where downstream code or user requirements justify it?
- If exact fields or enum values are named, did you avoid expanding them?
- Did any proposed assumption accidentally enter the formal runtime output format?
- Did you avoid treating common conventions as downstream evidence?
- Do instructions, variable notes, and output contract agree with each other?
- Does every ledger `Final handling` claim match the actual returned prompt, variables, contract, examples, or assumptions?
- Did you avoid process narration outside the requested artifact?
- Would deleting any remaining sentence change runtime behavior?
- Can a future maintainer explain why each rule exists?

## Common Failure Modes

- pasting product history instead of distilling it into runtime behavior
- hiding safety rules inside vague tone language
- hard-coding user or example data into stable instructions
- duplicating the same requirement across prompt text, examples, and evals without reason
- inventing a schema, parser, or policy rule when the brief only supports a recommendation
- telling the model to use a variable while also forbidding its required appearance
- keeping one prompt for multiple incompatible jobs when a split would be cleaner
