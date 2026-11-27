# Environment runbook: model-switch-e2e-1-phase1-to-3

## Bootstrap and ownership

DevOps Lead and Solution Architect review generated IaC and all remaining gaps before merge. This change set does not provision resources itself. Create encrypted, versioned remote state and its restricted IAM role first; apply dev/core before the shared ECR registries are used. Apply each environment through Atlantis after reviewed plans. Services run on ECS Fargate in each environment's cluster; set the ACM certificate and DNS for the load balancer before the first apply.

## Access and secrets

CI authenticates to AWS with GitHub OIDC, never static AWS keys. Scope the trust subject to the repository and its protected environment branches; scope ECR write to the generated image repositories. Grant ECR authorization-token access only as required by AWS. Atlantis uses a separate state/apply role. Human access uses SSO groups with environment-specific IAM roles; ECS Exec stays disabled; production access is audited and time limited.

Task definitions reference Secrets Manager entries by name, never values. Create the references listed in infra/PR.md before the first apply; only each service's execution role may read its own entries. After rotating a value, force a new deployment of the service so tasks read it. Never paste values in prompts, PRs or logs.

## Build (CI)

Every push to an environment branch (dev, qa, staging, prod) runs CI on that branch: tests, then an image build tagged <branch>-<commit>, pushed to the registry. CI records the image digest as the job output `digest` and in the artifact image-<component> (image.json). Configure real ECR registry addresses (ECR_REGISTRY) and the CI identity variables before the first build. Registries keep tags immutable, so a pushed image never changes.

## Deployment boundary

This change set ends at built, pushed images. Deploying them (workload manifests, promotion between environments, deployment approvals, health checks and rollback) is owned by the deployment side. infra/handoff.json lists what it needs: per environment the cluster and image digest file and secret references, per image its registry path, CI workflow and artifact, and per workload its port, health path and replica range. The ECS services are created here with bootstrap placeholder digests in infra/envs/<environment>/core/images.auto.tfvars.json; the deployment side writes real digests there and Atlantis applies them.

## Operations

Infrastructure rollback is a reviewed revert of the Terraform change, planned and applied through Atlantis. Database rollback requires a separate reviewed migration and backup restore plan; test restores in QA before incidents. Backups and restore ownership remain deployment-specific bootstrap requirements.

Alert rules are generated for each workload and environment. Connect the CloudWatch alarms (CPU; 5xx rate and p95 latency at the load balancer) to the team's on-call receiver and publish dashboard, tracing and incident links in the deployment record. HTTP services use latency/error alerts; workers use availability alerts. Logs go to CloudWatch Logs per service.

## Readiness evidence

Run `agent-engine phase4 check-environments --project-name model-switch-e2e-1-phase1-to-3` after merge/apply, using configured read-only source endpoints. It proves the reviewed change is merged on each environment's branch, applied by Atlantis per environment, and that CI succeeds; application health is the deployment side's evidence. Missing evidence remains not ready. Review environment-readiness.json alongside infra/PR.md before final G4 sign-off. Refresh evidence after every infra change; approval alone never proves provisioning.
