# CGE-P Labs

GRC engineering lab work for the GRC Engineering Club's CGE-P track: compliance controls written as code, with the audit evidence committed next to the code that produced it.

Author: Seraphim Omisade, GRC Analyst. ISO 27001 and ISO 42001 Lead Auditor.

## Lab 2.3: Compliant S3 primitive

A reusable Terraform module that deploys an Amazon S3 bucket with a dedicated access-log bucket and enforces NIST SP 800-53 controls at deploy time.

| Control | What enforces it | Where the evidence is |
|---|---|---|
| SC-28 Protection of information at rest | AES-256 server-side encryption on both buckets | `evidence/lab-2-3/state.json`: `aws_s3_bucket_server_side_encryption_configuration.*`; output `encryption_algorithm` |
| AC-3 Access enforcement | All four S3 public access block settings enabled on both buckets | `state.json`: `aws_s3_bucket_public_access_block.*` |
| CM-6 Configuration settings | Versioning enabled; four required compliance tags applied to every resource through provider default tags | `state.json`: `aws_s3_bucket_versioning.primary`, `tags_all` |
| AU-12 Audit record generation | Server access logging to a separate bucket under `access-logs/` | `state.json`: `aws_s3_bucket_logging.primary` |
| AU-9 Protection of audit information | Log bucket is encrypted, blocks all public access, and only grants write to the S3 log delivery group | `state.json`: `aws_s3_bucket.log`, `aws_s3_bucket_acl.log` |

Input validation rejects nonconforming project names and any environment other than `dev`, `staging`, or `prod` before a plan is produced.

Evidence was captured with `terraform show -json` against the saved plan and the applied state.

## Repository layout

```
terraform/primitives/compliant-s3/   Module source (main.tf, variables.tf, outputs.tf)
evidence/lab-2-3/                    Machine-readable plan and state evidence
```

## Run it

```
cd terraform/primitives/compliant-s3
terraform init
terraform plan -var project_name=cgep-lab -var environment=dev -out tfplan
terraform apply tfplan
terraform destroy -var project_name=cgep-lab -var environment=dev
```

Requires Terraform 1.6 or later and AWS credentials with S3 permissions. Deploys to us-east-1.
