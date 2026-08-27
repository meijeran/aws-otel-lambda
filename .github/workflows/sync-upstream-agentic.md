---
on: 
  schedule: weekly on monday around 06:00
  workflow_dispatch:

runs-on: ubuntu-latest

permissions:
  contents: read

safe-outputs:
  create-pull-request:
    title-prefix: "[sync]"

  create-issue:
    title-prefix: "[sync]"
    
---


# Update from upstream

Sync with the upstream `aws-observability/aws-otel-lambda:main` repository  and open a pull request against the `main` branch of this repository for human approval rather than pushing directly, so the PR Build workflow (.github/workflows/pr-build.yml) must succeed before a reviewer merges it.
When something fails then create an issue in this repository with the error message and a link to the upstream commit that failed to sync.