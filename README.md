# aws-iam-least-privilege

An agent skill that derives the minimum AWS IAM permissions needed to deploy a Terraform plan.

## What it does

Given a Terraform plan JSON, the skill analyzes every resource being created or updated and outputs:

- A grouped, human-readable list of required IAM actions (by service)
- A ready-to-use IAM policy JSON document

## Usage

Ask your agent:

> "What are the minimum IAM permissions I need to run terraform apply?"

Then provide your plan JSON using one of three methods:

**Local Terraform**
```bash
terraform plan -out=plan.out
terraform show -json plan.out > plan.json
```

**HCP Terraform / Terraform Enterprise** — the skill uses `tfctl` to fetch the plan automatically from a workspace.

**Terraform MCP server** — the skill calls `get_plan_details` and `get_plan_json_output`.

## Structure

```
skills/aws-iam-least-privilege/
  SKILL.md                        # skill instructions (478 tokens)
  references/aws-service-map.md   # service mapping, pre-computed actions, tfctl commands
evals/aws-iam-least-privilege/
  eval.yaml                       # eval config and graders
  tasks/                          # 5 test tasks
  fixtures/                       # sample tfplan JSON files
```

## Evals

```bash
waza run evals/aws-iam-least-privilege/eval.yaml
```

Latest result: **5/5 tasks passed, aggregate score 1.00**. See [eval-results.md](eval-results.md) for details.
