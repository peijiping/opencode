# 贡献指南 - OpenCode

我们希望能让您轻松地为 OpenCode 做出贡献。以下是最常见的会被合并的更改类型：

- Bug 修复
- 新增 LSP / 格式化工具
- LLM 性能改进
- 支持新的提供商
- 修复特定环境的问题
- 缺失的标准行为
- 文档改进

但任何 UI 或核心产品功能在实现之前，都必须经过核心团队的设计评审。

如果您不确定某个 PR 是否会被接受，请随时询问维护者，或查看带有以下标签的 issue：

- [`help wanted`](https://github.com/anomalyco/opencode/issues?q=is%3Aissue%20state%3Aopen%20label%3Ahelp-wanted)
- [`good first issue`](https://github.com/anomalyco/opencode/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22good%20first%20issue%22)
- [`bug`](https://github.com/anomalyco/opencode/issues?q=is%3Aissue%20state%3Aopen%20label%3Abug)
- [`perf`](https://github.com/anomalyco/opencode/issues?q=is%3Aopen%20is%3Aissue%20label%3A%22perf%22)

> [!NOTE]
> 忽略这些约束的 PR 很可能会被关闭。

想要接手一个 issue？留下评论，维护者可能会将其分配给您，除非是我们已经在处理的事项。

## 添加新的提供商

新的提供商通常不需要（如果确实需要的话也很少）代码更改，但如果您想添加对新提供商的支持，请先向以下仓库提交 PR：
https://github.com/anomalyco/models.dev

## 开发 OpenCode

- 要求：Bun 1.3+
- 从仓库根目录安装依赖并启动开发服务器：

  ```bash
  bun install
  bun dev
  ```

### 针对不同目录运行

默认情况下，`bun dev` 在 `packages/opencode` 目录中运行 OpenCode。要针对其他目录或仓库运行：

```bash
bun dev <directory>
```

要在 opencode 仓库自身的根目录中运行 OpenCode：

```bash
bun dev .
```

### 构建独立可执行文件

编译独立可执行文件：

```bash
./packages/opencode/script/build.ts --single
```

然后使用以下命令运行：

```bash
./packages/opencode/dist/opencode-<platform>/bin/opencode
```

将 `<platform>` 替换为您的平台（例如 `darwin-arm64`、`linux-x64`）。

- 核心模块：
  - `packages/opencode`：OpenCode 核心业务逻辑和服务器。
  - `packages/opencode/src/cli/cmd/tui/`：TUI 代码，使用 SolidJS 和 [opentui](https://github.com/sst/opentui) 构建。
  - `packages/app`：共享的 Web UI 组件，使用 SolidJS 构建。
  - `packages/desktop`：原生桌面应用，使用 Electron 构建（包装了 `packages/app`）。
  - `packages/plugin`：`@opencode-ai/plugin` 的源码。

### 理解 bun dev 与 opencode 的区别

在开发过程中，`bun dev` 等同于构建后的 `opencode` 命令。两者运行相同的 CLI 接口：

```bash
# 开发（从项目根目录）
bun dev --help           # 显示所有可用命令
bun dev serve            # 启动无头 API 服务器
bun dev web              # 启动服务器并打开 Web 界面
bun dev <directory>      # 在指定目录启动 TUI

# 生产环境
opencode --help          # 显示所有可用命令
opencode serve           # 启动无头 API 服务器
opencode web             # 启动服务器并打开 Web 界面
opencode <directory>     # 在指定目录启动 TUI
```

### 运行 API 服务器

启动 OpenCode 无头 API 服务器：

```bash
bun dev serve
```

默认情况下，这会在 4096 端口启动无头服务器。您可以指定不同的端口：

```bash
bun dev serve --port 8080
```

### 运行 Web 应用

在开发过程中测试 UI 更改：

1. **首先，启动 OpenCode 服务器**（参见上文[运行 API 服务器](#running-the-api-server)部分）
2. **然后运行 Web 应用：**

```bash
bun run --cwd packages/app dev
```

这会在 http://localhost:5173（或输出中显示的类似端口）启动本地开发服务器。大多数 UI 更改可以在此处测试，但服务器必须处于运行状态才能获得完整功能。

### 运行桌面应用

桌面应用是一个包装了 Web UI 的 Electron 应用程序。

在开发模式下运行桌面应用：

```bash
bun run --cwd packages/desktop dev
```

创建生产构建并打包应用：

```bash
bun run --cwd packages/desktop build
bun run --cwd packages/desktop package
```

> [!NOTE]
> 如果您更改了 API 或 SDK（例如 `packages/opencode/src/server/server.ts`），请运行 `./script/generate.ts` 以重新生成 SDK 和相关文件。

请尽量遵循[风格指南](./AGENTS.md)。

### 设置调试器

Bun 的调试功能目前还不够完善。我们希望本指南能帮助您完成设置并避开一些痛点。

调试 OpenCode 最可靠的方式是在终端中通过 `bun run --inspect=<url> dev ...` 手动运行，然后通过该 URL 附加调试器。其他方法可能导致断点映射不正确，至少在 VSCode 中如此（因人而异）。

注意事项：

- 如果您想运行 OpenCode TUI 并让断点在服务器代码中触发，您可能需要运行 `bun dev spawn` 而不是通常的 `bun dev`。这是因为 `bun dev` 在工作线程中运行服务器，断点可能无法正常工作。
- 如果 `spawn` 对您不起作用，您可以分别调试服务器：
  - 调试服务器：`bun run --inspect=ws://localhost:6499/ --cwd packages/opencode ./src/index.ts serve --port 4096`，然后使用 `opencode attach http://localhost:4096` 连接 TUI。
  - 调试 TUI：`bun run --inspect=ws://localhost:6499/ --cwd packages/opencode --conditions=browser ./src/index.ts`

其他技巧：

- 根据您的工作流程，您可能希望使用 `--inspect-wait` 或 `--inspect-brk` 代替 `--inspect`。
- 每次调用都指定 `--inspect=ws://localhost:6499/` 可能会很繁琐，您可以设置 `export BUN_OPTIONS=--inspect=ws://localhost:6499/` 来代替。

#### VSCode 设置

如果您使用 VSCode，可以使用我们的示例配置 [.vscode/settings.example.json](.vscode/settings.example.json) 和 [.vscode/launch.example.json](.vscode/launch.example.json)。

一些可能有问题的调试方法：

- 使用 `"request": "launch"` 的调试配置可能导致断点映射不正确，从而无法使用。
- 在 VSCode 的 `JavaScript Debug Terminal` 中运行 OpenCode 也会出现同样的问题。

尽管如此，您仍然可以尝试这些方法，因为它们可能对您有效。

## Pull Request 要求

### Issue 优先策略

**所有 PR 必须引用一个已有的 issue。** 在打开 PR 之前，请先创建一个描述 bug 或功能的 issue。这有助于维护者分类问题并防止重复工作。没有关联 issue 的 PR 可能会被关闭，恕不审查。

- 在 PR 描述中使用 `Fixes #123` 或 `Closes #123` 来关联 issue。
- 对于小修复，一个简短的 issue 即可 —— 提供足够的上下文让维护者理解问题即可。

### 通用要求

- 保持 pull request 短小精悍、重点突出。
- 解释问题以及您的更改为何能修复它。
- 在添加新功能之前，请确保代码库的其他地方没有已存在的实现。

### UI 更改

如果您的 PR 包含 UI 更改，请附带截图或视频展示更改前后的效果。这有助于维护者更快地审查，并让您更快地获得反馈。

### 逻辑更改

对于非 UI 更改（bug 修复、新功能、重构），请说明**您如何验证其有效性**：

- 您测试了什么？
- 审查者如何重现/确认该修复？

### 禁止 AI 生成的大段文本

冗长的 AI 生成的 PR 描述和 issue 是不被接受的，可能会被忽略。请尊重维护者的时间：

- 编写简短、重点突出的描述。
- 用自己的话解释更改了什么以及为什么。
- 如果您无法简洁地解释，您的 PR 可能太大了。

### PR 标题

PR 标题应遵循常规提交规范：

- `feat:` 新功能
- `fix:` bug 修复
- `docs:` 文档或 README 更改
- `chore:` 维护任务、依赖更新等
- `refactor:` 代码重构，不改变行为
- `test:` 添加或更新测试

您可以选择包含作用域以指明受影响的包：

- `feat(app):` app 包中的功能
- `fix(desktop):` desktop 包中的 bug 修复
- `chore(opencode):` opencode 包的维护

示例：

- `docs: update contributing guidelines`
- `fix: resolve crash on startup`
- `feat: add dark mode support`
- `feat(app): add dark mode support`
- `fix(desktop): resolve crash on startup`
- `chore: bump dependency versions`

### 风格偏好

以下并非强制要求，仅为一般性指导：

- **函数：** 除非拆分能带来明确的复用或组合优势，否则将逻辑保持在单个函数内。
- **解构：** 不要进行不必要的变量解构。
- **控制流：** 避免使用 `else` 语句。
- **错误处理：** 尽可能优先使用 `.catch(...)` 而不是 `try`/`catch`。
- **类型：** 使用精确的类型，避免使用 `any`。
- **变量：** 坚持使用不可变模式，避免使用 `let`。
- **命名：** 在保持描述性的前提下，选择简洁的单个单词标识符。
- **运行时 API：** 在合适的情况下使用 Bun 辅助方法，如 `Bun.file()`。

## 功能请求

对于全新的功能，请从设计讨论开始。创建一个 issue，描述问题、您提议的方法（可选）以及为什么它属于 OpenCode。核心团队将帮助决定是否推进；请等待批准，而不是直接提交功能 PR。

## 信任与担保系统

本项目使用 [vouch](https://github.com/mitchellh/vouch) 来管理贡献者信任。担保名单维护在 [`.github/VOUCHED.td`](.github/VOUCHED.td) 中。

### 工作原理

- **已担保用户**是被明确信任的贡献者。
- **被谴责用户**被明确屏蔽。被谴责用户的 issue 和 pull request 会被自动关闭。如果您被谴责，可以通过 [Discord](https://opencode.ai/discord) 联系维护者请求解除谴责。
- **其他所有人**都可以正常参与 —— 您不需要被担保也能创建 issue 或 PR。

### 维护者操作

具有写入权限的协作者可以在任何 issue 中通过评论管理担保名单：

- `vouch` — 为 issue 作者担保
- `vouch @username` — 为指定用户担保
- `denounce` — 谴责 issue 作者
- `denounce @username` — 谴责指定用户
- `denounce @username <reason>` — 谴责并附上理由
- `unvouch` / `unvouch @username` — 从名单中移除某人

更改会自动提交到 `.github/VOUCHED.td`。

### 谴责政策

谴责仅适用于反复提交低质量 AI 生成贡献、垃圾信息或其他恶意行为的用户。不用于意见分歧或无意的错误。

## Issue 要求

所有的 issue **必须**使用我们的 issue 模板之一：

- **Bug 报告** — 用于报告 bug（需要描述）
- **功能请求** — 用于建议增强（需要验证复选框和描述）
- **问题咨询** — 用于提问（需要问题内容）

不允许空白 issue。当新 issue 被创建时，自动化检查会验证它是否遵循模板并满足我们的贡献指南。如果 issue 不符合要求，您将收到一条评论，说明需要修复的内容，并拥有 **2 小时** 时间编辑该 issue。之后，它将被自动关闭。

Issue 可能因以下原因被标记：

- 未使用模板
- 必填字段留空或填写了占位符文本
- AI 生成的大段文本
- 缺少有意义的内容

如果您认为您的 issue 被错误标记，请告知维护者。
