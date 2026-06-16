+++
title = "AWS Cost Hygiene: S3 Lifecycle Savings at Careem"
date = 2026-06-16
+++

**Case study repo:** [aws-cost-infra-case-study](https://github.com/muhammadahmed-01/aws-cost-infra-case-study)

Cloud spend often hides in defaults: log buckets that never expire, idle compute nobody owns, Terraform changes that merge without a cost diff. This page summarizes two real workstreams: **S3 lifecycle savings at Careem** and **SageMaker pricing in Infracost**.

### Careem: S3 log storage

Application and service logs accumulated in S3 Standard storage without tiering or expiration. After lifecycle policies (transition to IA/Glacier, expire beyond retention, noncurrent version cleanup):

| Metric | Result |
|--------|--------|
| Storage volume | **83% to 88% reduction** |
| Monthly savings | **~$1k** (measured) |

Reference Terraform pattern: [s3-log-lifecycle.tf](https://github.com/muhammadahmed-01/aws-cost-infra-case-study/blob/main/examples/terraform/s3-log-lifecycle.tf)

Full checklist (EC2/EKS right-sizing, idle resources, tagging): [INFRA-HYGIENE-CHECKLIST.md](https://github.com/muhammadahmed-01/aws-cost-infra-case-study/blob/main/docs/INFRA-HYGIENE-CHECKLIST.md)

### Infracost: SageMaker endpoint configuration

**GitHub PR:** [infracost/infracost#3567](https://github.com/infracost/infracost/pull/3567)

Three billing paths mapped for `aws_sagemaker_endpoint_configuration`:

* **Provisioned instances:** `instance_type` and `initial_instance_count`, plus optional hosting storage
* **Serverless inference:** GB-second compute and provisioned concurrency when configured
* **Shadow variants:** Parallel shadow production variants costed separately from live traffic

Longer write-up: [Shift-Left FinOps: Adding SageMaker Support to Infracost](/projects/infracost-sagemaker/)

### Sample audit artifact

Illustrative monthly waste table (format only, not Careem billing): [sample-waste-audit.md](https://github.com/muhammadahmed-01/aws-cost-infra-case-study/blob/main/docs/sample-waste-audit.md)
