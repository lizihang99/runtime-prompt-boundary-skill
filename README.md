# Agent-Written Prompt Boundary

When an AI coding agent builds a software feature that calls an LLM, it may also write the prompt for that product LLM call. This skill helps the agent keep that prompt clean: only the information the product model needs at runtime goes into the prompt.

## 中文简介

当你用 AI agent 开发软件时，agent 有时会顺手写产品里调用大模型的 prompt。

这个 skill 解决的就是这件事：**让 agent 分清哪些信息该写进真正发给大模型的 prompt，哪些只是开发过程里的背景、实现细节、示例数据、测试用例或临时猜测。**

它不是普通的“帮你润色 prompt”。它更像一个边界检查器：

- 产品模型运行时必须知道的，写进 prompt
- 每次调用才有的数据，变成变量
- 下游代码要解析的结构，写成输出契约
- 只是在开发时用来验证的例子，放到测试或 eval case
- 还没被需求、代码或 schema 证明的想法，放到 `proposed-assumption`
- 对运行时模型没用、甚至会污染判断的内容，移出 prompt

这样可以避免 agent 为了“更完整”，把开发讨论、业务背景、实现方案、样例数据或没有证据的字段/枚举/安全规则，一股脑塞进产品运行时 prompt。

## Why This Exists

Coding agents are good at turning discussion into code. But when the code includes an LLM call, the agent may blur three different audiences:

- the human developer explaining the feature
- the coding agent implementing the feature
- the product LLM that will receive the runtime prompt

Those audiences need different information. A fact that helps the coding agent may be noise, leakage, or policy drift for the product model.

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
