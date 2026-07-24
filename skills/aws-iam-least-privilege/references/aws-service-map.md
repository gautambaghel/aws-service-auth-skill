# ARN Scoping Rules (Step 6)

Never use `"Resource": "*"` as a final answer. Scope to the narrowest ARN pattern:

- Use known values from the plan (bucket names, role names, AMI IDs, etc.)
- For IDs unknown at plan time, use a wildcard suffix:
  `arn:aws:ec2:REGION:ACCOUNT_ID:instance/*`
- Replace `REGION` / `ACCOUNT_ID` with actual values when known from the plan
  (region from provider config, account from existing ARNs in `prior_state`)
- Remind the user to replace `<account-id>` placeholders before use

Common patterns:

| Service | ARN pattern |
|---|---|
| S3 bucket | `arn:aws:s3:::BUCKET_NAME` |
| S3 objects | `arn:aws:s3:::BUCKET_NAME/*` |
| IAM role | `arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME` |
| Lambda | `arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME` |
| DynamoDB | `arn:aws:dynamodb:REGION:ACCOUNT_ID:table/TABLE_NAME` |
| SQS | `arn:aws:sqs:REGION:ACCOUNT_ID:QUEUE_NAME` |
| EC2 instance | `arn:aws:ec2:REGION:ACCOUNT_ID:instance/*` |
| EC2 VPC | `arn:aws:ec2:REGION:ACCOUNT_ID:vpc/*` |
| EC2 security group | `arn:aws:ec2:REGION:ACCOUNT_ID:security-group/*` |
| Log group | `arn:aws:logs:REGION:ACCOUNT_ID:log-group:LOG_GROUP_NAME:*` |

---

# Plan Acquisition (Step 1)

## Path A — Local Terraform
```bash
terraform plan -out=plan.out
terraform show -json plan.out > plan.json
```
Share the `plan.json` file or paste its contents.

## Path B — HCP Terraform / Terraform Enterprise via tfctl
Use when user mentions HCP Terraform, Terraform Cloud, TFE, or a workspace name.
See "tfctl Commands" section below.

## Path C — Terraform MCP server
Use when user doesn't want to install a binary.
Use tools `get_plan_details` then `get_plan_json_output` (returns same JSON format).

If unclear, ask: "Are you using HCP Terraform or Terraform Enterprise? Share
workspace name and org and I'll pull the plan. Otherwise paste your
`terraform show -json plan.out` output."

---

# Create/Deploy Verb List (Step 5)

Include an action whose name starts with any of:
`Create`, `Put`, `Run`, `Launch`, `Start`, `Register`, `Publish`, `Upload`,
`Add`, `Attach`, `Associate`, `Allocate`, `Enable`, `Set`, `Tag`, `Update`,
`Import`, `Activate`, `Deploy`, `Build`, `Provision`

---

# Service Reference API (Step 4)

Index: `https://servicereference.us-east-1.amazonaws.com`
Returns `[{ "service": "...", "url": "..." }]` — use the `url` field.

Per-service JSON: `https://servicereference.us-east-1.amazonaws.com/v1/{service}/{service}.json`

Each action entry:
```json
{ "Name": "CreateBucket", "Annotations": { "Properties": { "IsWrite": true } } }
```

---

# AWS Resource Type → Service Mapping

Use this table in Step 3 to map Terraform resource types to AWS service
identifiers. For any `aws_*` type not listed, derive the service from the
resource type prefix: `aws_{service}_*`.

| Terraform resource type(s) | AWS service |
|---|---|
| `aws_s3_bucket`, `aws_s3_object`, `aws_s3_bucket_*`, `aws_s3_bucket_policy` | `s3` |
| `aws_instance`, `aws_security_group`, `aws_vpc`, `aws_subnet`, `aws_route_table`, `aws_internet_gateway`, `aws_nat_gateway`, `aws_eip`, `aws_network_interface`, `aws_key_pair`, `aws_launch_template` | `ec2` |
| `aws_autoscaling_group`, `aws_launch_configuration` | `autoscaling` + `ec2` |
| `aws_lambda_function`, `aws_lambda_permission`, `aws_lambda_alias`, `aws_lambda_event_source_mapping` | `lambda` |
| `aws_iam_role`, `aws_iam_policy`, `aws_iam_role_policy`, `aws_iam_role_policy_attachment`, `aws_iam_instance_profile`, `aws_iam_user`, `aws_iam_group` | `iam` |
| `aws_dynamodb_table` | `dynamodb` |
| `aws_db_instance`, `aws_rds_cluster`, `aws_db_subnet_group`, `aws_db_parameter_group`, `aws_db_option_group` | `rds` |
| `aws_cloudwatch_log_group`, `aws_cloudwatch_metric_alarm`, `aws_cloudwatch_dashboard` | `logs` + `cloudwatch` |
| `aws_sqs_queue`, `aws_sqs_queue_policy` | `sqs` |
| `aws_sns_topic`, `aws_sns_topic_subscription`, `aws_sns_topic_policy` | `sns` |
| `aws_api_gateway_rest_api`, `aws_api_gateway_*`, `aws_apigatewayv2_*` | `apigateway` |
| `aws_kms_key`, `aws_kms_alias` | `kms` |
| `aws_secretsmanager_secret`, `aws_secretsmanager_secret_version` | `secretsmanager` |
| `aws_ssm_parameter`, `aws_ssm_document` | `ssm` |
| `aws_lb`, `aws_alb`, `aws_lb_listener`, `aws_lb_target_group`, `aws_alb_*` | `elasticloadbalancing` |
| `aws_ecr_repository`, `aws_ecr_lifecycle_policy` | `ecr` |
| `aws_ecs_cluster`, `aws_ecs_task_definition`, `aws_ecs_service` | `ecs` |
| `aws_eks_cluster`, `aws_eks_node_group`, `aws_eks_addon` | `eks` |
| `aws_elasticache_cluster`, `aws_elasticache_replication_group`, `aws_elasticache_subnet_group` | `elasticache` |
| `aws_cloudfront_distribution`, `aws_cloudfront_origin_access_identity` | `cloudfront` |
| `aws_route53_record`, `aws_route53_zone`, `aws_route53_health_check` | `route53` |
| `aws_kinesis_stream`, `aws_kinesis_firehose_delivery_stream` | `kinesis` + `firehose` |
| `aws_glue_job`, `aws_glue_crawler`, `aws_glue_database` | `glue` |
| `aws_sfn_state_machine` | `states` |
| `aws_ses_*` | `ses` |
| `aws_cognito_*` | `cognito-idp` |
| `aws_opensearch_domain`, `aws_elasticsearch_domain` | `es` |
| `aws_mq_broker` | `mq` |

---

# Pre-computed Common Actions (skip Step 4 fetch for these)

For these resource types, use the actions below directly — do NOT fetch the
service reference JSON. Always prefix with `ec2:`.

| Resource type | Minimum write actions |
|---|---|
| `aws_vpc` | `ec2:CreateVpc`, `ec2:ModifyVpcAttribute`, `ec2:CreateTags` |
| `aws_subnet` | `ec2:CreateSubnet`, `ec2:ModifySubnetAttribute`, `ec2:CreateTags` |
| `aws_internet_gateway` | `ec2:CreateInternetGateway`, `ec2:AttachInternetGateway`, `ec2:CreateTags` |
| `aws_route_table` | `ec2:CreateRouteTable`, `ec2:CreateRoute`, `ec2:AssociateRouteTable`, `ec2:CreateTags` |
| `aws_security_group` | `ec2:CreateSecurityGroup`, `ec2:AuthorizeSecurityGroupIngress`, `ec2:AuthorizeSecurityGroupEgress`, `ec2:CreateTags` |
| `aws_instance` | `ec2:RunInstances`, `ec2:CreateTags` |
| `aws_eip` | `ec2:AllocateAddress`, `ec2:AssociateAddress`, `ec2:CreateTags` |
| `aws_nat_gateway` | `ec2:CreateNatGateway`, `ec2:CreateTags` |
| `aws_key_pair` | `ec2:ImportKeyPair`, `ec2:CreateTags` |
| `aws_launch_template` | `ec2:CreateLaunchTemplate`, `ec2:CreateTags` |
| `aws_network_interface` | `ec2:CreateNetworkInterface`, `ec2:AttachNetworkInterface`, `ec2:CreateTags` |

---

# Always-Include Permissions (Step 5 special cases)

Include these regardless of the verb filter when the relevant resource types
appear in the plan:

| Condition | Always include |
|---|---|
| Any of: `aws_lambda_function`, `aws_ecs_task_definition`, `aws_ecs_service`, `aws_instance`, `aws_glue_job`, `aws_sfn_state_machine`, `aws_autoscaling_group` | `iam:PassRole` |
| `aws_eks_cluster`, `aws_elasticache_*`, `aws_rds_cluster`, `aws_db_instance`, `aws_lb`, `aws_ecs_service` | `iam:CreateServiceLinkedRole` |
| `aws_lambda_function` | `logs:CreateLogGroup`, `logs:CreateLogDelivery`, `logs:PutLogEvents` |
| `aws_kms_key` with `enable_key_rotation = true` | `kms:EnableKeyRotation` |

---

# tfctl Commands for Fetching Plan JSON (Step 1B)

```bash
# Get the current run ID for a workspace
tfctl api /organizations/{organization}/workspaces/WORKSPACE_NAME \
  --jq '.data.relationships.["current-run"].data.id'

# Fetch plan JSON by run ID (follows 307 redirect automatically)
tfctl api /runs/RUN_ID/plan/json-output --json

# Or fetch by plan ID directly
tfctl api /plans/PLAN_ID/json-output --json
```

A `204` response means the plan hasn't finished yet — wait and retry.

---

# Service Reference API

Index URL: `https://servicereference.us-east-1.amazonaws.com`

Returns a JSON array of `{ "service": "...", "url": "..." }` entries.
Fetch individual service files from the `url` field, e.g.:
`https://servicereference.us-east-1.amazonaws.com/v1/s3/s3.json`

Each service JSON has a top-level `"Actions"` array. Each action:
```json
{
  "Name": "CreateBucket",
  "Annotations": {
    "Properties": { "IsWrite": true, "IsList": false }
  }
}
```
