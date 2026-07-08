# Agent-Written Prompt Boundary

When you use Codex or another AI coding agent to build software, the agent may not only write code; it may also write the prompt for a product LLM call.

This skill prevents a common failure: the agent tries to make that prompt look "complete" by stuffing it with development discussion, business background, implementation plans, sample data, or unsupported fields, enums, and safety rules.

A product runtime prompt is the text your software actually sends to the LLM. It is not the same thing as the requirements, notes, code plan, examples, or temporary guesses the coding agent used while building the feature.

This skill makes the agent route information first:

- rules the product model must follow every time go into the prompt
- data that changes on each call becomes runtime variables
- structures downstream code must parse become output contracts
- examples used only to check behavior become tests or eval cases
- useful ideas without evidence go to `proposed-assumption`
- context that does not help the model run is kept out of the prompt

It is not a generic prompt-polishing tool. It is a boundary checker for product prompts written by coding agents.

## 中文简介

当你用 Codex 这类 AI coding agent 开发软件时，它可能不只是写代码，还会顺手为产品里的大模型调用写 prompt。

这个 skill 主要防止一种常见跑偏：**agent 为了让 prompt 看起来“更完整”，把开发讨论、业务背景、实现方案、样例数据，甚至没有证据支持的字段、枚举和安全规则，一股脑塞进产品运行时 prompt。**

这里的“产品运行时 prompt”，指的是软件真正调用大模型时发出去的那段提示词。它和开发过程中给 agent 看的需求说明、背景解释、代码方案不是一回事。

这个 skill 会让 agent 先把信息分清楚：

- 模型每次都必须遵守的规则，才写进 prompt
- 每次调用都会变化的数据，变成运行时变量
- 下游代码必须解析的结构，写成输出契约
- 只是用来验证行为的例子，放到测试或 eval case
- 还没有被需求、代码或 schema 证明的想法，放到 `proposed-assumption`
- 对模型执行没帮助、甚至会污染判断的内容，直接移出 prompt

所以它不是普通的“prompt 润色工具”，而是一个给 agent 用的提示词边界检查器。

## Why This Exists

Codex and other coding agents often see more context than the product LLM should ever see. When the code includes an LLM call, the agent may blur three different audiences:

- the human developer explaining the feature
- the coding agent implementing the feature
- the product LLM that will receive the runtime prompt

Those audiences need different information. A fact that helps the coding agent may become noise, leakage, or policy drift for the product model.

This skill forces the agent to route information instead of pasting everything into the prompt.

## What It Produces

Unless the user asks for raw prompt text only, the skill asks the agent to return:

- Intent Check
- Boundary Summary
- Boundary Ledger
- Runtime Prompt
- Runtime Variables
- Output Contract
- Change Note for existing prompts

The first section, `Intent Check`, exists so the user can quickly confirm that the agent understood what the product LLM call is supposed to do before trusting the prompt draft.

## What It Prevents

- development background leaking into a product prompt
- implementation details being shown to the runtime model
- sample user data becoming hard-coded prompt text
- dynamic inputs being treated as stable instructions
- output schemas drifting beyond what downstream code expects
- helpful but unsupported guesses becoming runtime rules
- safety or uncertainty rules being deleted during prompt cleanup
- one overloaded prompt trying to classify, decide, summarize, and generate all at once

## Repository Layout

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
├── cases/
│   ├── 01-distill-product-backstory.md
│   ├── 02-preserve-safety-escalation.md
│   ├── 03-extract-dynamic-runtime-data.md
│   ├── 04-output-contract-vs-friendly-prose.md
│   ├── 05-split-overloaded-prompt.md
│   └── README.md
└── references/
    ├── output-template.md
    └── priority-rules.md
```

## Key Idea

Prompt work is information routing.

Do not ask only:

```text
Should this sentence go into the prompt?
```

Ask:

```text
Who needs this information?
When is it available?
Does it change model behavior, code behavior, tests, or user understanding?
What is the cheapest artifact that preserves its value?
```

## Pressure Cases

The `cases/` directory contains regression scenarios used to test whether the skill holds its boundary under realistic pressure:

1. Distill product backstory into runtime behavior.
2. Preserve safety and escalation constraints without expanding policy.
3. Extract dynamic runtime data instead of hard-coding samples.
4. Keep strict machine-readable output compatible with friendly prose.
5. Split overloaded prompts into staged calls.

## Status

This skill has been pressure-tested with local advisor runs against the included cases. The most important observed failure pattern was the agent treating helpful guesses as product contracts. The current version routes those guesses to `proposed-assumption` until the user, code, schema, or downstream consumer proves them.
