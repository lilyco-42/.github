# lyco 生态组织配置

## lyco-hygiene reusable workflow

架构守则的可执行检查（守则见 lilyco `docs/ECOSYSTEM_ROADMAP.md`）：

- 守则 2：git 依赖必须锁 rev/tag（扫描 Cargo.toml 里的 `branch = "main/master"`）
- 大文件不入库（默认阈值 2MB，可配豁免前缀）

用法（任意生态仓库）：

```yaml
jobs:
  hygiene:
    uses: lilyco-42/.github/.github/workflows/lyco-hygiene.yml@main
    with:
      allow-large-files: "assets/,docs/banner"
```

## lyco-sync-stars reusable workflow

星数自动同步：`docs/banner.svg` 的角标、个人主页 README 的卡片行里写死的 `★N`，
每天比对一次，**只有真的不一致才提交**（2026-09-23 一次性发现 5 处过期）。

用法（任意生态仓库的 `.github/workflows/sync-stars.yml`）：

```yaml
name: sync stars
on:
  schedule: [{ cron: "17 3 * * *" }]   # 每天 11:17 GMT+8
  workflow_dispatch:
jobs:
  sync:
    permissions: { contents: write }
    uses: lilyco-42/.github/.github/workflows/lyco-sync-stars.yml@main
```

匹配规则：

1. `**[名字](https://github.com/<owner>/<repo>)** ★N` → 取**那个仓**的真实星数（个人主页 README）
2. 文件里剩下的裸 `★N` → 取**本仓**的真实星数（banner 角标）

注意：GitHub 会在仓库 60 天无活动后暂停定时任务。本 workflow 只在星数变化时才提交，
长期无变化被暂停时，手动 Run workflow 一次即可恢复。
