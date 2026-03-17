# GitHub Actions

*GitHub Actions*---
https://github.com/anthropics/claude-code/tree/main/.skills/github-actions

发布日期：2025年

Published 2025

使用 GitHub Actions 构建 CI/CD 流水线——工作流、作业、步骤、触发器、缓存、构件、矩阵构建、密钥、环境、可复用工作流和自定义 Action。适用于自动化测试、构建、部署、代码质量检查或任何 GitHub 触发的自动化任务。

Build CI/CD pipelines with GitHub Actions — workflows, jobs, steps, triggers, caching, artifacts, matrix builds, secrets, environments, reusable workflows, and custom actions. Use when tasks involve automating tests, builds, deployments, code quality checks, or any GitHub-triggered automation.

# GitHub Actions

构建在每次推送、PR、定时计划或手动触发时直接在 GitHub 中运行的 CI/CD 流水线。

Build CI/CD pipelines that run on every push, PR, schedule, or manual trigger directly in GitHub.

## 工作流结构

Workflow Structure

工作流文件位于 `.github/workflows/` 目录下。每个 YAML 文件都是一个独立的流水线。

Workflows live in `.github/workflows/`. Each YAML file is an independent pipeline.

```yaml
# .github/workflows/ci.yml

name: CI                            # GitHub UI 中显示的名称 / Display name in GitHub UI

on:                                 # 触发器 / Triggers
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:                        # 取消重复运行 / Cancel duplicate runs
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:                             # 作业 ID / Job ID
    runs-on: ubuntu-latest          # 运行器操作系统 / Runner OS
    steps:
      - uses: actions/checkout@v4   # 克隆仓库 / Clone repo
      - uses: actions/setup-node@v4 # 安装 Node.js / Install Node.js
        with:
          node-version: 20
          cache: npm                # 缓存 npm 依赖 / Cache npm dependencies
      - run: npm ci                 # 安装依赖 / Install dependencies
      - run: npm test               # 运行测试 / Run tests
```

## 触发器

Triggers

```yaml
on:
  # 代码事件 / Code events
  push:
    branches: [main, develop]
    paths: ['src/**', 'package.json']    # 仅在特定文件变更时触发 / Only trigger on specific file changes
    tags: ['v*']                          # 标签推送（发布） / Tag pushes (releases)
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

  # 定时计划（cron） / Scheduled (cron)
  schedule:
    - cron: '0 8 * * 1'    # 每周一 UTC 8:00 / Every Monday at 8:00 UTC

  # 手动触发 / Manual trigger
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deploy target'
        required: true
        type: choice
        options: [staging, production]

  # 由其他工作流触发 / Triggered by other workflows
  workflow_call:
    inputs:
      node-version:
        type: string
        default: '20'
```

## 缓存

Caching

缓存依赖可以为每次运行节省 30-60 秒：

Caching dependencies cuts 30-60 seconds off every run:

```yaml
# 使用 setup-node 自动缓存 / Automatic cache with setup-node
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: npm       # 也支持：yarn, pnpm / Also supports: yarn, pnpm

# 手动缓存（任意目录） / Manual cache (any directory)
- uses: actions/cache@v4
  with:
    path: |
      node_modules
      .next/cache
    key: ${{ runner.os }}-deps-${{ hashFiles('package-lock.json') }}
    restore-keys: ${{ runner.os }}-deps-

# 为并行作业拆分保存/恢复 / Split save/restore for parallel jobs
# 作业 1：保存 / Job 1: save
- uses: actions/cache/save@v4
  with:
    path: node_modules
    key: modules-${{ hashFiles('package-lock.json') }}

# 作业 2+：恢复 / Job 2+: restore
- uses: actions/cache/restore@v4
  with:
    path: node_modules
    key: modules-${{ hashFiles('package-lock.json') }}
```

## 构件

Artifacts

在作业之间共享构建产物：

Share build outputs between jobs:

```yaml
# 上传 / Upload
- uses: actions/upload-artifact@v4
  with:
    name: build
    path: dist/
    retention-days: 1      # 节省存储成本 / Save storage costs

# 下载（在另一个作业中） / Download (in another job)
- uses: actions/download-artifact@v4
  with:
    name: build
    path: dist/
```

## 矩阵构建

Matrix Builds

跨多个版本/平台并行测试：

Test across multiple versions/platforms in parallel:

```yaml
jobs:
  test:
    strategy:
      fail-fast: false     # 某个作业失败时不取消其他作业 / Don't cancel other jobs if one fails
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

## 密钥与环境

Secrets and Environments

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production          # 如果已配置，则需要审批 / Requires approval if configured

    steps:
      - run: deploy --token ${{ secrets.DEPLOY_TOKEN }}

      # 环境特定的密钥 / Environment-specific secrets
      - run: echo "Deploying to ${{ vars.API_URL }}"
```

在以下位置设置密钥：Settings → Secrets and variables → Actions。

Set secrets at: Settings → Secrets and variables → Actions.

## 作业依赖与输出

Job Dependencies and Outputs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.ver.outputs.version }}
    steps:
      - id: ver
        run: echo "version=$(node -p 'require(\"./package.json\").version')" >> $GITHUB_OUTPUT

  deploy:
    needs: build                    # 在 build 之后运行 / Runs after build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # 仅在 main 分支上 / Only on main branch
    steps:
      - run: echo "Deploying version ${{ needs.build.outputs.version }}"
```

## 常见模式

Common Patterns

### 代码检查 + 测试 + 构建 + 部署

Lint + Test + Build + Deploy

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm test

  build:
    needs: [lint, test]            # 两者都必须通过 / Both must pass
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with: { name: build, path: dist/ }

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with: { name: build, path: dist/ }
      - run: npx vercel deploy --prod --token ${{ secrets.VERCEL_TOKEN }}
```

### Docker 构建与推送

Docker Build and Push

```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### PR 评论附带结果

PR Comment with Results

```yaml
- uses: actions/github-script@v7
  with:
    script: |
      await github.rest.issues.createComment({
        owner: context.repo.owner,
        repo: context.repo.repo,
        issue_number: context.issue.number,
        body: '✅ Build succeeded! Preview: https://...'
      });
```

### 通过 SSH 部署

Deploy via SSH

```yaml
- uses: appleboy/ssh-action@v1
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: deploy
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd /opt/app
      git pull origin main
      npm ci --production
      npm run build
      pm2 restart app
```

## 可复用工作流

Reusable Workflows

跨仓库共享工作流：

Share workflows across repos:

```yaml
# .github/workflows/shared-ci.yml（在共享仓库中）
# .github/workflows/shared-ci.yml (in a shared repo)
on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: '20'
    secrets:
      npm-token:
        required: false

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: ${{ inputs.node-version }}, cache: npm }
      - run: npm ci
      - run: npm test
```

```yaml
# .github/workflows/ci.yml（在使用方仓库中）
# .github/workflows/ci.yml (in consuming repo)
jobs:
  ci:
    uses: org/shared-workflows/.github/workflows/shared-ci.yml@main
    with:
      node-version: '20'
```

## 服务（CI 中的数据库）

Services (Databases in CI)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports: ['5432:5432']
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s

      redis:
        image: redis:7
        ports: ['6379:6379']

    env:
      DATABASE_URL: postgresql://test:test@localhost:5432/testdb
      REDIS_URL: redis://localhost:6379

    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

## 计费

Billing

- **公共仓库**：无限分钟数，免费
- **私有仓库（免费版）**：每月 2,000 分钟
- **Linux 运行器**：1 倍计费
- **macOS 运行器**：10 倍计费（请谨慎使用）
- **Windows 运行器**：2 倍计费

- **Public repos**: unlimited minutes, free
- **Private repos (free tier)**: 2,000 minutes/month
- **Linux runners**: 1x multiplier
- **macOS runners**: 10x multiplier (use sparingly)
- **Windows runners**: 2x multiplier

## 最佳实践

Guidelines

- **使用 `concurrency` 配合 `cancel-in-progress`** ——始终添加此配置以避免在过时的运行上浪费分钟数
- **积极使用缓存** —— `actions/cache` 可用于 node_modules、构建产物、Docker 层。每次运行节省 30-60 秒。
- **矩阵构建中使用 `fail-fast: false`** ——查看所有失败，而不仅仅是第一个
- **不要在工作流文件中存储密钥** ——使用 GitHub Secrets。切勿在日志中 `echo` 密钥。
- **将 Action 版本固定到 SHA** —— `uses: actions/checkout@abc123` 比 `@v4` 更安全（防止供应链攻击）
- **使用 `needs` 控制作业顺序** ——默认并行执行。仅在存在实际依赖时才添加 `needs`。
- **构件会过期** ——将 `retention-days` 设置为所需的最小值。默认 90 天会浪费存储空间。
- **使用 `if: github.ref == 'refs/heads/main'`** ——保护部署作业，防止从 PR 意外部署到生产环境
- **使用可复用工作流统一组织标准** ——将通用 CI 模式提取到共享仓库中

- **`concurrency` with `cancel-in-progress`** — always add this to avoid wasting minutes on outdated runs
- **Cache aggressively** — `actions/cache` for node_modules, build outputs, Docker layers. Saves 30-60s per run.
- **`fail-fast: false` in matrix builds** — see all failures, not just the first one
- **Don't store secrets in workflow files** — use GitHub Secrets. Never `echo` secrets in logs.
- **Pin action versions to SHA** — `uses: actions/checkout@abc123` is more secure than `@v4` (prevents supply chain attacks)
- **Use `needs` for job ordering** — parallel by default. Add `needs` only when there's a real dependency.
- **Artifacts expire** — set `retention-days` to the minimum needed. Default 90 days wastes storage.
- **`if: github.ref == 'refs/heads/main'`** — guard deploy jobs to prevent accidental production deploys from PRs
- **Reusable workflows for org-wide standards** — extract common CI patterns into a shared repo
