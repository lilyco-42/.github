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
