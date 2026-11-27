# Branching and CI

One repository holds the application code and its infrastructure (infra/). Each environment has its own long-lived branch:

feature/* → dev → qa → staging → prod → main

| Environment | Branch | Who approves a pull request into it |
|---|---|---|
| dev | `dev` | Code review |
| qa | `qa` | QA Lead |
| stg | `staging` | Client UAT sign-off |
| prod | `prod` | Recorded go/no-go plus DevOps Lead |
| release record | `main` | DevOps Lead; tag every release on `main` with an immutable release tag |

Feature branches start from `dev` and return to it through reviewed pull requests. Changes move up one environment branch at a time through pull requests; never commit directly to an environment branch, and never merge downwards (fixes go to `dev` and travel up). Protect every environment branch and `main`: pull requests only, required reviews and CI/policy checks, no administrator bypass.

CI runs on every pull request (tests) and on every push to an environment branch (tests, then an image built from that branch, tagged `<branch>-<commit>`). Images are rebuilt per environment branch, not copied, so pin dependencies and base images for reproducible builds. Each environment's Terraform lives in `infra/envs/<environment>/core`; Atlantis plans and applies it only for pull requests into that environment's branch. Deploying the built images is owned by the deployment side (see `infra/handoff.json`).

