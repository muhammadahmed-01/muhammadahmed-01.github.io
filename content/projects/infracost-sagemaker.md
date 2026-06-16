+++
title = "Shift-Left FinOps: Adding SageMaker Support to Infracost"
date = 2026-05-10
description = "Infracost Go PR for SageMaker endpoints, serverless inference, shadow variants, and async Terraform pricing."
+++

**GitHub PR:** [infracost/infracost#3567](https://github.com/infracost/infracost/pull/3567) (open, awaiting upstream review)

"Shift-Left FinOps" is about moving cost visibility as far forward in the development lifecycle as possible. This project involved contributing to **Infracost**, a tool that sits in the CI/CD pipeline to show engineers the "price tag" of their Terraform changes before they hit production.

### The Technical Challenge
AWS SageMaker pricing is complex because it involves multiple billing dimensions that are often opaque in Terraform code. My contribution adds pricing support for `aws_sagemaker_endpoint_configuration`, covering:

* **Inference Variants:** Support for both provisioned instance-based hosting and serverless configurations.
* **Shadow Production Variants:** Mapping costs for test models running in parallel with production traffic—a common source of "bill shock."
* **Async & Data Capture:** Accounting for the overhead of asynchronous inference configurations and data capture fees for model monitoring.

### Implementation Details
Working in the **Go** codebase of Infracost, I had to map complex Terraform resource attributes to the AWS Price List API. This required:
1. **Resource Mapping:** Translating nested configuration blocks into cost components.
2. **Logic for Serverless vs. Provisioned:** Handling the conditional logic where pricing changes from "per hour" to "per GB/second" based on the `serverless_config` block.
3. **Unit Testing:** Ensuring that the pricing engine accurately calculates the monthly estimate across various edge cases (e.g., provisioned concurrency).

### Key Learnings
Contributing to a FinOps tool is a masterclass in the "financial underbelly" of the cloud. It forces you to realize that as a software engineer, you are effectively a "financial architect." Every line of infrastructure code is a spending decision. By building visibility into the PR level, we empower engineers to be proactive rather than reactive.