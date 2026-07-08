# Output Template

Use these headings in this order unless the user explicitly asks for raw prompt text only.

Start directly with `## Intent Check`. Do not add a preface such as "I'll apply the skill."

```md
## Intent Check
- Goal:
- Uses:
- Outputs:
- Must not:
- Main risk if misunderstood:

## Boundary Summary
- LLM call:
- Runtime job:
- Downstream consumer:
- Key failure mode:

## Boundary Ledger
| Information | Primary role | Final handling | Why |
| --- | --- | --- | --- |

## Runtime Prompt
### Instructions
...

### Runtime Inputs
...

### Output Format
...

### Safety and Uncertainty Rules
...

## Runtime Variables
| Variable | Source | Required? | Notes |
| --- | --- | --- | --- |

## Output Contract
- Schema / JSON shape:
- Parser or tool expectations:
- Required failure / uncertainty fields:
- If unspecified, say `not specified` and list proposed defaults separately as assumptions.
- If fields or enum values are specified, do not add to them unless marked as a proposed assumption outside the contract.
- Keep proposed assumptions outside the formal prompt output format unless the user accepts them.

## Change Note
- Preserved:
- Distilled:
- Moved out:
- Removed:
```

Notes:

- Omit `Change Note` for a brand-new prompt.
- If the user asks for only raw prompt text, still use this template internally before collapsing the response.
- Keep the ledger compact. Include contested or high-risk facts first.
- Do a contradiction scan before returning: instructions, variable notes, and output contract must not disagree.
- Check that each ledger `Final handling` claim is actually reflected in the returned artifact.
- If `Intent Check` reveals material ambiguity, ask for confirmation before finalizing the prompt.
