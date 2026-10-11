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

## 许可治理（2026-10-11）

**决策：全组织统一 MIT**，署名沿用惯例 `Copyright (c) 2026 lilyco-42`。

### 已补

- 先补 9 个「README / Cargo.toml 已声明许可、只差许可文件」的仓 —— 零争议，只是让现实对上它自己的声明
- 再按决策批量补 **80 个**（直接提交默认分支）
- 结果：**MIT 37 → 124**，活跃仓无许可 **97 → 17**

### 排除：13 个仓 + 4 个 fork（**这是有据的决定，不是遗漏**）

13 个仓库本身含第三方代码/素材、或声明版权不属我们 —— 补 MIT 会**与仓库自己的声明冲突**：

| 仓库 | 依据 |
|---|---|
| `WSL2-Linux-Kernel` | Linux 内核分发，源码里 `MODULE_LICENSE("GPL")` |
| `character-theme-kit` | README「角色版权归原作者及发行方所有…禁止二次转售或商用」 |
| `ncmdump-android` | 基于原项目 `taurusxin/ncmdump`（Apache-2.0），直接复用其代码 |
| `sp-cdn-assets` | 《卫戍协议》游戏素材 CDN（第三方 IP） |
| `happy-sugar-life-live2d` | 第三方 IP 的 Live2D 素材 |
| `wechatdataanalysis-archive` | 第三方源码存档镜像（仓内有 `THIRD_PARTY_NOTICES.md`） |
| `xmake-mirror` | 第三方 xmake 包镜像 |
| `newland` | 含目录「官方库示例源码」 |
| `bridge_annotation_delivery` | 包装第三方 Semantic Segmentation Editor |
| `lptiyu-protocol-research` | README「研究对象的著作权归其运营方所有」 |
| `claude-max-antiban-guide` | 整理自他人公开文章（「已隐去原作者署名」）→ 非原创 |
| `lilyco-brand` | 演示视频含第三方音乐（KID《回到以后》） |
| `tsukishita` | 含 `Battle.flac/m4a` 疑似第三方音频 + `vendor/three.js` |

**另有 4 个 fork**（`1earn` / `LinyuKernel` / `claw-code` / `openBilibili-1`）——
fork 继承上游许可，**绝不能补**。

> 唯一能解除排除的路径是**换掉第三方素材/代码**（如 `lilyco-brand` 换自备 BGM）。
> 在那之前，这 17 个「无许可」是**正确状态**，不需要「补齐」。

## 最小 CI（家族仓库，2026-10-11）

命名家族（`ly*` / `l*` / `lilyco*`）里缺 CI 的仓库，统一挂最小 CI：

```yaml
name: ci
on:
  push: {branches: [main, master], tags: ["v*"]}
  pull_request:
  workflow_dispatch:
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !startsWith(github.ref, 'refs/tags/') }}
jobs:
  hygiene:
    uses: lilyco-42/.github/.github/workflows/lyco-hygiene.yml@main
    with:
      allow-large-files: "..."   # 仅大文件仓库需要，按实测扫描填
```

- 19 个仓库（排除 2 个 fork），**19/19 run 全绿**

### 未做：语言级 build / fmt / clippy

**当前工作约束是不做本地编译**，而语言级检查需要先确认「存量代码能过」，
否则等于凭空造红灯。实测数据（2026-10-11，本机 `rustfmt 1.10.0-stable`）：

- 家族 11 个 Rust 仓里**只有 2 个**（`lgame` / `lyco-model-bench`）`cargo fmt --all --check` 直接通过
- 9 个有格式化差异（`lyco_chat` 最大：171 块 / 19 文件）
- 2 个（`ly_motion` / `lyco_agent`）根目录没有 `Cargo.toml`，需先弄清 workspace 结构

⇒ 语言级 CI 应作为**独立一轮**：先 `cargo fmt --all` 一次性对齐，
再在能编译的环境验证 clippy，并在 CI 里**钉住 rustfmt 版本**避免版本漂移。

