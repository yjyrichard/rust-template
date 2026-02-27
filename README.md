# Rust 项目模板

Rust项目模板，修复了 `cargo generate` 的 `cliff.toml` 兼容问题，并整理了国内镜像配置说明。

## 快速开始

```bash
cargo generate your-github-username/template
```

生成后进入项目目录：

```bash
cd your-project-name
cargo run
```

## 环境设置

### 1. 安装 Rust

#### macOS / Linux

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### Windows

从 [rustup.rs](https://rustup.rs) 下载 `rustup-init.exe` 并运行。

安装时建议选择 `1) Proceed with standard installation`，会自动安装 Visual Studio Build Tools（需勾选"使用 C++ 的桌面开发"）。

### 2. 配置国内镜像（推荐）

国内直接访问 crates.io 和 rustup 速度很慢，建议配置镜像。

#### Rustup 镜像

设置环境变量（Windows 在"系统环境变量"中新建，macOS/Linux 写入 `~/.bashrc` 或 `~/.zshrc`）：

```bash
export RUSTUP_DIST_SERVER=https://rsproxy.cn
export RUSTUP_UPDATE_ROOT=https://rsproxy.cn/rustup
```

Windows 用户在"用户变量"中新建：

| 变量名 | 变量值 |
|--------|--------|
| `RUSTUP_DIST_SERVER` | `https://rsproxy.cn` |
| `RUSTUP_UPDATE_ROOT` | `https://rsproxy.cn/rustup` |

#### Cargo 镜像（Sparse 稀疏索引，最快）

编辑 `~/.cargo/config.toml`（Windows 路径为 `C:\Users\你的用户名\.cargo\config.toml`），写入：

```toml
[source.crates-io]
replace-with = 'rsproxy-sparse'

[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"

[net]
git-fetch-with-cli = true
```

> 备选镜像（如果 rsproxy 不可用）：
>
> ```toml
> [source.crates-io]
> replace-with = 'ustc'
>
> [source.ustc]
> registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"
> ```

### 3. 安装开发工具

```bash
# 项目模板生成工具
cargo install cargo-generate

# 代码检查与安全审计
cargo install --locked cargo-deny

# 拼写检查
cargo install typos-cli

# 自动生成 changelog
cargo install git-cliff

# 增强测试工具
cargo install cargo-nextest --locked
```

### 4. 安装 pre-commit

pre-commit 用于在 git commit 前自动运行代码检查（格式化、clippy、测试等）。

```bash
pipx install pre-commit
```

安装后在项目目录运行：

```bash
pre-commit install
```

### 5. 推荐 VSCode 插件

- rust-analyzer: Rust 语言支持（必装）
- Even Better TOML: TOML 文件支持
- Error Lens: 内联错误提示
- crates: Cargo.toml 依赖版本提示
- GitLens: Git 增强
- Rust Test Lens: 快速运行测试
- TODO Highlight: TODO 高亮

## 项目文件结构详解

```
.
├── .github/
│   └── workflows/
│       └── build.yml            # GitHub Actions CI/CD 配置
├── assets/
│   ├── README.md                # 资源文件说明
│   └── juventus.csv             # 示例数据集（用于课程练习）
├── src/
│   └── main.rs                  # 程序入口文件
├── .gitignore                   # Git 忽略规则
├── .pre-commit-config.yaml      # pre-commit 钩子配置
├── _typos.toml                  # typos 拼写检查配置
├── Cargo.lock                   # 依赖版本锁定文件（自动生成）
├── Cargo.toml                   # Rust 项目核心配置
├── CHANGELOG.md                 # 变更日志（由 git-cliff 自动生成）
├── cargo-generate.toml          # cargo-generate 模板配置
├── cliff.toml                   # git-cliff changelog 生成规则
├── deny.toml                    # cargo-deny 依赖安全审计配置
└── README.md                    # 本文件
```

### 核心文件

#### `Cargo.toml` — 项目配置文件

Rust 项目的核心配置，类似于 Node.js 的 `package.json` 或 Python 的 `pyproject.toml`。定义了项目名称、版本、Rust edition，以及所有依赖。运行 `cargo build` 或 `cargo run` 时，Cargo 会读取此文件。

#### `Cargo.lock` — 依赖锁定文件

自动生成，记录了所有依赖的精确版本号。确保团队成员和 CI 环境使用完全相同的依赖版本。不要手动编辑，由 Cargo 自动维护。

#### `src/main.rs` — 程序入口

Rust 可执行项目的入口文件，包含 `main()` 函数。所有程序从这里开始执行。

### 工具配置文件

#### `.pre-commit-config.yaml` — 提交前自动检查

配置了 git commit 前自动运行的检查项，包括：
- 代码格式化检查（`cargo fmt`）
- 编译检查（`cargo check`）
- Clippy 静态分析（`cargo clippy`）
- 依赖安全审计（`cargo deny`）
- 拼写检查（`typos`）
- 单元测试（`cargo nextest`）
- 通用检查（行尾空格、换行符、merge 冲突标记等）

任何一项检查失败都会阻止提交，确保代码质量。

#### `deny.toml` — 依赖安全审计

`cargo-deny` 的配置文件，用于检查项目依赖的安全性和合规性：
- 漏洞检查：依赖是否有已知安全漏洞（来自 RustSec 数据库）
- 许可证检查：依赖的开源许可证是否符合要求（默认允许 MIT、Apache-2.0 等）
- 重复依赖检查：是否引入了同一个 crate 的多个版本
- 来源检查：依赖是否来自可信的 registry

运行方式：`cargo deny check`

#### `cliff.toml` — 自动生成 changelog

`git-cliff` 的配置文件。它会根据你的 git commit 记录，按照 [Conventional Commits](https://www.conventionalcommits.org/) 规范自动生成格式化的变更日志。commit 消息会被自动分类为 Features、Bug Fixes、Documentation 等。

运行方式：`git cliff`

#### `_typos.toml` — 拼写检查配置

`typos` 工具的配置文件。它会扫描代码中的英文拼写错误。可以在 `[default.extend-words]` 中添加白名单词汇（比如项目特有的缩写），在 `[files]` 中排除不需要检查的文件。

运行方式：`typos`

#### `cargo-generate.toml` — 模板生成配置

`cargo-generate` 的配置文件。指定了在生成项目时需要跳过模板变量替换的文件。`cliff.toml` 中的 Tera 模板语法 `{{ }}` 会与 `cargo-generate` 的变量替换语法冲突，因此需要排除。

### CI/CD 配置

#### `.github/workflows/build.yml` — GitHub Actions 自动化流水线

推送代码或创建 PR 时自动触发，依次执行：
1. 代码格式检查（`cargo fmt`）
2. 编译检查（`cargo check`）
3. Clippy 静态分析
4. 运行所有测试（`cargo nextest`）

当推送 `v*` 格式的 tag（如 `v1.0.0`）时，还会自动：
5. 用 `git-cliff` 生成 changelog
6. 创建 GitHub Release

### 其他文件

#### `.gitignore` — Git 忽略规则

告诉 Git 哪些文件不需要纳入版本控制。默认忽略 `/target` 目录（Rust 编译输出，类似于 Java 的 `/build` 或 Node.js 的 `/node_modules`）。

#### `CHANGELOG.md` — 变更日志

由 `git-cliff` 根据 git commit 记录自动生成，记录项目的版本变更历史。不需要手动编辑。

#### `assets/` — 资源文件夹

存放项目需要的静态资源文件。模板中包含了一个 `juventus.csv` 示例数据集，用于课程练习。
