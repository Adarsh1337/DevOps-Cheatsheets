# GitLab CI/CD Cheatsheet

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [.gitlab-ci.yml Basics](#gitlab-ciyml-basics)
- [Common Templates](#common-templates)
- [Runners](#runners)
- [Artifacts & Cache](#artifacts--cache)
- [Environments & Deployments](#environments--deployments)
- [Security & Secrets](#security--secrets)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Overview

GitLab CI/CD is GitLab's built-in continuous integration and deployment system.
Pipelines are defined in a .gitlab-ci.yml file stored at the root of your repository.

## Quick Start

1. Create .gitlab-ci.yml in your repository root
2. Register a Runner (shared or specific)
3. Push code and watch pipelines run in GitLab > CI/CD > Pipelines

```yaml
# .gitlab-ci.yml (minimal example)
stages:
  - build
  - test
  - deploy

build:
  stage: build
  image: node:20-alpine
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

test:
  stage: test
  image: node:20-alpine
  script:
    - npm test -- --ci --reporters=default --reporters=junit
  artifacts:
    reports:
      junit: junit.xml

deploy:
  stage: deploy
  image: alpine:3.20
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  environment:
    name: production
    url: https://example.com
  script:
    - ./scripts/deploy.sh
```

## .gitlab-ci.yml Basics

- stages: Ordered list of pipeline stages
- jobs: Define a job per stage with image, script, rules, artifacts
- rules/only/except: Control when jobs run
- needs: Define job dependencies for faster pipelines
- variables: Define environment variables

```yaml
variables:
  NODE_ENV: production

lint:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm run lint
  rules:
    - changes:
        - src/**/*.js
```

## Common Templates

### Docker Build and Push

```yaml
.docker-login:
  image: docker:27
  services: [docker:27-dind]
  variables:
    DOCKER_DRIVER: overlay2
  before_script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin "$CI_REGISTRY"

container:build:
  stage: build
  extends: [.docker-login]
  script:
    - docker build -t "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA" .
    - docker push "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA"
  artifacts:
    reports:
      dotenv: image.env
```

### Terraform IaC

```yaml
terraform:init:
  image: hashicorp/terraform:1.7
  script:
    - terraform init -input=false

terraform:plan:
  stage: test
  image: hashicorp/terraform:1.7
  script:
    - terraform plan -input=false -out tfplan
  artifacts:
    paths: [tfplan]
```

## Runners

- Shared Runners: Provided by GitLab instance
- Specific Runners: Registered to your project/group

```bash
# Register a runner (shell example)
sudo gitlab-runner register \
  --non-interactive \
  --url https://gitlab.com/ \
  --registration-token $REG_TOKEN \
  --executor docker \
  --docker-image alpine:3.20 \
  --description "project-runner" \
  --tag-list "docker,linux" \
  --run-untagged=false
```

## Artifacts & Cache

```yaml
artifacts:
  paths:
    - dist/
  expire_in: 1 week

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
```

## Environments & Deployments

```yaml
deploy:staging:
  stage: deploy
  environment:
    name: staging
    url: https://staging.example.com
  script: ["./scripts/deploy-staging.sh"]
  only: ["merge_requests", "main"]
```

## Security & Secrets

- Use masked variables in Settings > CI/CD > Variables
- Do not commit secrets; use CI/CD variables, Vault, or Kubernetes secrets
- Restrict protected variables to protected branches/tags

```yaml
variables:
  KUBE_CONFIG: "$KUBE_CONFIG"

deploy:k8s:
  stage: deploy
  image: bitnami/kubectl:1.30
  script:
    - mkdir -p ~/.kube
    - echo "$KUBE_CONFIG" | base64 -d > ~/.kube/config
    - kubectl apply -f k8s/
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

## Best Practices

1. Keep jobs small and cache dependencies effectively
2. Use rules and needs to reduce pipeline time
3. Pin images to known-good versions
4. Use environments and review apps for MRs
5. Fail fast; surface logs and test reports

## Troubleshooting

### Common Issues

| Symptom | Cause | Fix |
|--------|-------|-----|
| Job stuck in pending | No runners available | Register a runner or check tags |
| Docker build fails | Missing Docker-in-Docker | Add docker:dind service |
| Secrets not available | Variable not protected | Protect branch or variable |

### Debugging Tips

```yaml
variables:
  FF_SCRIPT_SECTIONS: "true"

.debug:
  before_script:
    - echo "CI_DEBUG_TRACE=$CI_DEBUG_TRACE"

print:env:
  stage: test
  image: alpine:3.20
  script:
    - env | sort
  rules:
    - if: $CI_PIPELINE_SOURCE == "push"
```

---

Last Updated: October 2025
Version: GitLab 16.x
