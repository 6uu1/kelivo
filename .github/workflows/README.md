# GitHub Actions Workflows

本目录包含自动化 CI/CD 工作流，帮助简化维护工作。

## 📋 工作流概览

### 🔄 自动同步与发布

| 工作流 | 触发条件 | 功能 |
|--------|----------|------|
| `auto-sync-upstream.yml` | 每天 10:00 (北京时间) | 自动检查上游更新，创建同步 PR 或 Issue |
| `auto-release.yml` | 推送 `v*.*.*` tag | 自动构建所有平台并发布 Release |
| `build-on-version-change.yml` | pubspec.yaml 版本变更 | 检测版本更新，自动创建 tag |

### 🔨 构建工作流

| 工作流 | 触发条件 | 功能 |
|--------|----------|------|
| `build-stable.yml` | 手动触发 | 完整多平台构建，支持发布 |
| `quick-build.yml` | Push/PR/手动 | 快速构建，用于验证 |
| `pr-check.yml` | PR 到 master | 代码检查、测试、格式验证 |

### 🔍 检查与扫描

| 工作流 | 触发条件 | 功能 |
|--------|----------|------|
| `dependency-check.yml` | 每周一 11:00 | 检查 Flutter 和依赖更新 |
| `security-scan.yml` | Push/PR/每周日 | 安全扫描、依赖审查 |

---

## 🚀 快速发布流程

### 方法 1: 修改版本号自动触发（推荐）

1. 修改 `pubspec.yaml` 中的 `version`
2. 提交并推送到 master
3. 自动检测版本变更，创建 tag
4. Tag 触发 `auto-release.yml` 构建发布

```bash
# 示例：从 1.1.11 升级到 1.1.12
# 修改 pubspec.yaml: version: 1.1.12+30
git add pubspec.yaml
git commit -m "chore: bump version to 1.1.12"
git push origin master
```

### 方法 2: 手动创建 Tag

```bash
git tag -a v1.1.12 -m "Release v1.1.12"
git push origin v1.1.12
```

### 方法 3: 手动触发工作流

1. 进入 Actions 页面
2. 选择 `build-stable.yml`
3. 勾选 `publish_release`
4. 填写 `release_tag` (如 `v1.1.12`)
5. 点击 Run workflow

---

## 🔄 同步上游

### 自动同步

每天北京时间 10:00 自动检查上游 `Chevey339/kelivo` 更新：
- 无冲突：自动创建 PR
- 有冲突：创建 Issue 提醒

### 手动同步

```bash
# 添加上游 remote（首次）
git remote add upstream https://github.com/Chevey339/kelivo.git

# 同步流程
git fetch upstream
git checkout master
git merge upstream/master
# 解决冲突（如有）
git push origin master
```

---

## 📦 快速验证构建

### 仅验证代码（不构建产物）

PR 会自动触发 `pr-check.yml`，进行：
- 代码格式检查
- 静态分析
- 单元测试
- 本地化检查

### 快速构建验证

触发 `quick-build.yml` 构建 Windows/Android 版本验证：
- 自动：Push 到 master
- 手动：Actions → quick-build.yml → Run workflow

---

## ⚙️ 配置项

### 必需的 Secrets

| Secret | 用途 |
|--------|------|
| `SILICONFLOW_KEY` | 应用内置 API Key |
| `SIGN_KEYSTORE_BASE64` | Android 签名证书 (Base64) |
| `KEYSTORE_PASSWORD` | 签名证书密码 |
| `KEY_ALIAS` | 签名密钥别名 |
| `KEY_PASSWORD` | 签名密钥密码 |

### 可调整的环境变量

在各 workflow 文件顶部：

```yaml
env:
  FLUTTER_VERSION: '3.35.7'  # Flutter 版本
  UPSTREAM_REPO: 'Chevey339/kelivo'  # 上游仓库
  UPSTREAM_BRANCH: 'master'  # 上游分支
```

---

## 🛠️ 常用命令

```bash
# 查看工作流运行状态
gh run list

# 查看特定工作流
gh run list --workflow=auto-sync-upstream.yml

# 手动触发工作流
gh workflow run auto-sync-upstream.yml

# 触发带参数的工作流
gh workflow run build-stable.yml -f build_windows=true -f publish_release=true -f release_tag=v1.1.12

# 查看运行日志
gh run view <run-id> --log
```

---

## 📅 自动化计划

| 时间 (北京时间) | 工作流 | 任务 |
|-----------------|--------|------|
| 每天 10:00 | auto-sync-upstream | 检查上游更新 |
| 每周一 11:00 | dependency-check | 检查依赖更新 |
| 每周日 12:00 | security-scan | 安全扫描 |
| 版本变更时 | build-on-version-change | 自动创建 tag |
| Tag 推送时 | auto-release | 构建发布 |

---

## ❓ 常见问题

### Q: 如何跳过 CI？

提交消息中包含 `[skip ci]` 或 `[ci skip]`：

```bash
git commit -m "docs: update readme [skip ci]"
```

### Q: 构建失败怎么办？

1. 查看 Actions 日志定位错误
2. 本地运行 `flutter analyze` 和 `flutter test`
3. 修复后重新推送

### Q: 如何更改上游仓库？

修改 `auto-sync-upstream.yml` 中的环境变量：

```yaml
env:
  UPSTREAM_REPO: 'your-org/your-repo'
  UPSTREAM_BRANCH: 'main'
```

### Q: 如何禁用某个工作流？

在 workflow 文件中注释掉 `on:` 部分，或删除该文件。
