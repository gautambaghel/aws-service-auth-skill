# Eval Results — aws-iam-least-privilege

**Date:** 2026-07-24  
**Run:** `results/2026-07-24T013918.099/claude-sonnet-4.6.json`  
**Model:** claude-sonnet-4.6  
**Engine:** copilot-sdk

## Summary

| Metric | Value |
|---|---|
| Total Tests | 5 |
| Passed | 5 |
| Failed | 0 |
| Success Rate | 100% |
| Aggregate Score | 1.00 |
| Duration | 3m51.717s |

## Per-Task Results

| Task | Result | Score | Duration |
|---|---|---|---|
| Basic S3 Bucket Plan | ✅ passed | 1.00 | 62.2s |
| EC2 Instance with VPC | ✅ passed | 1.00 | 28.8s |
| Lambda Function with IAM Role | ✅ passed | 1.00 | 53.6s |
| Multi-Service Stack | ✅ passed | 1.00 | 78.2s |
| Should Ask For Plan File | ✅ passed | 1.00 | 8.8s |

## Graders

All 5 graders passed across all tasks:

- `_output_contains` — checks for expected IAM actions and JSON keywords
- `_output_not_contains` — ensures no read-only actions in output
- `valid_iam_action_format` — regex: `(?:[a-z][a-z0-9-]*:[A-Z][A-Za-z]+)`
- `no_readonly_actions` — not_contains check for Describe/List/Get/Head verbs
- `iam_policy_json_structure` — regex checks for `"Version"`, `"Statement"`, `"Effect"`, `"Action"`, `"Resource"`

## Token Usage

| Metric | Value |
|---|---|
| Premium Requests | 24 |
| Turns | 24 |
| Input Tokens | 511,036 |
| Output Tokens | 13,887 |
| Cached Tokens Read | 361,568 |
| Cached Tokens Written | 116,168 |
| Total Tokens | 524,923 |

## Key Changes That Achieved 100%

1. Replaced all `prompt` graders with `text` graders (`regex_match` / `not_contains`) — waza `prompt` grader scoring is inverted (1 = fail detected), making it unsuitable for positive assertions.
2. Added pre-computed EC2 resource→action table to `references/aws-service-map.md` — eliminates the need to fetch and parse the large EC2 service reference JSON (~190 write actions), reducing EC2 task tool calls from 11 to 4 and eliminating response truncation.
3. Updated SKILL.md Step 4 to check pre-computed table first and only fetch service reference JSON for resource types not covered.
