# 本机作品汇总：用 pi 与 dsh 造出来的东西

> 统计时间：2026-10 · 机器：macOS（Apple Silicon）· 配套展示页：[`index.html`](index.html)

过去半年，我把需求用中文说给两个跑在本机的编程助手——**pi**（终端代理，支持 RPC）和 **dsh**（桌面 / Web / headless 多形态）——由它们读仓库、改代码、跑测试、打包、装进 `/Applications`。下面是产出的软件、工具与技能，以及每条产物的构建轨迹。

| 指标 | 数值 |
| --- | --- |
| 成品 | **12** 件（桌面 4 / 本地服务 3 / 设备与系统 5） |
| 代码量 | **154,694 行**（Rust 147,896 + JS/TS 6,798） |
| 构建会话 | **258** 次（pi 190 + dsh 68） |
| 已装进 `/Applications` | **4** 个 |
| 自建 skill | **13** 个 |
| GitHub 仓库 | **8** 个（treq、maildesk、RaptorQR、utools-fd-search、toval、artifeeds、tbas、utools-selfie），其余 4 件仅在本机 |

### 公开仓库（8）

| 仓库 | 对应成品 | 说明 | 最近更新 |
| --- | --- | --- | --- |
| [tuean/treq](https://github.com/tuean/treq) | treq | 本地优先的桌面 API 客户端（Rust + GPUI） | 2026-10-14 |
| [tuean/maildesk](https://github.com/tuean/maildesk) | maildesk | 原生 macOS 邮件客户端，手写 IMAP/SMTP，无 WebView | 2026-10-03 |
| [tuean/RaptorQR](https://github.com/tuean/RaptorQR) | RaptorQR · Host | fork 自 `infrost/RaptorQR`，自建 `host/` 屏幕捕获接收端 | 2026-10-16 |
| [tuean/utools-fd-search](https://github.com/tuean/utools-fd-search) | utools-fd-search | uTools 插件：fd + rg + fzf 文件搜索 | 2026-09-15 |
| [tuean/toval](https://github.com/tuean/toval) | toval | 个人创作宇宙工作台（Rust + axum + SQLite） | 2026-10-19 |
| [tuean/artifeeds](https://github.com/tuean/artifeeds) | artifeeds | RSS / Atom / JSON Feed 桌面阅读器（Rust + GPUI） | 2026-10-19 |
| [tuean/tbas](https://github.com/tuean/tbas) | TBAS | Edge 标签页快照（CLI + 扩展） | 2026-10-19 |
| [tuean/utools-selfie](https://github.com/tuean/utools-selfie) | utools-selfie | uTools 摄像头录像插件（macOS） | 2026-10-19 |

其余 4 件（TVA、tblogs、gov-reports、pdf2str）没有远端仓库，只在本机。

---

## 一、桌面应用（4）

| 项目 | 是什么 | 技术栈 | 体量 / 状态 | GitHub 仓库 | 构建轨迹 |
| --- | --- | --- | --- | --- | --- |
| **treq** | 本地优先的 API 调试客户端，界面参考 Insomnia，工作区是磁盘上可进 Git 的 YAML | Rust + GPUI | 33.3k 行 · 21 commits · 已装 `/Applications` | [tuean/treq](https://github.com/tuean/treq) | pi 5 个会话（71 MB）；README 自述「绝大部分代码由 Claude Code / pi 编写」 |
| **maildesk** | 原生 macOS 邮件客户端，IMAP4rev1 / SMTP / MIME / HTML 子集全部手写，无 WebView | Rust + GPUI + SQLite | 28.8k 行 · 51 commits（10-02 → 10-16）· 已装 `/Applications` | [tuean/maildesk](https://github.com/tuean/maildesk) | dsh 9 个会话（含早期 `email-desktop` 目录） |
| **artifeeds** | 面向「上千个源、又快又省」的 RSS / Atom / JSON Feed / OPML 阅读器 | Rust + GPUI + SQLite | 18.0k 行 · 已装 `/Applications` | [tuean/artifeeds](https://github.com/tuean/artifeeds) | pi 项目记忆 + dsh 2 个会话 |
| **TVA** | 按住说话，把语音指令交给本机 pi / dsh / opencli 执行 | Rust + GPUI + whisper.cpp | 5.0k 行 · `dist/TVA.app` | —（仅本机） | 驱动 pi/dsh 的前端；本机会话库**没有**它的构建记录，单独标注 |

**几个关键设计**

- **treq**：285 个测试函数；响应区有状态码 / 耗时 / 大小、JSON 路径提示；请求体行高可调。
- **maildesk**：3.6 万封邮件实测 94 MB 物理内存、空闲 0.3% CPU、5.4 MB 二进制。
- **artifeeds**：条件请求（未更新只花 304、零字节正文）+ 自适应轮询 + 全部队列有硬上限；`artifeeds suggest` 用本机 pi 把任意网页变成订阅，一个 OPML 可落地 102 个源 / 12 个分组。
- **TVA**：whisper tiny 出实时预览（0.15s）、small 出终稿（0.9s）；六步状态全部来自真实事件（`agent_start` / `tool_execution_start` …）。

---

## 二、本地服务与内容（3）

| 项目 | 是什么 | 技术栈 | 体量 | GitHub 仓库 | 构建轨迹 |
| --- | --- | --- | --- | --- | --- |
| **toval** | 个人创作宇宙工作台：故事、角色、事件、章节长在一张图上，AI 生成前留修订快照 | Rust + axum + askama + HTMX + SQLite | 38.3k 行（最大） | [tuean/toval](https://github.com/tuean/toval) | dsh **20 个会话**，本机最密集 |
| **tblogs** | 两段式博客系统：本地 `tblogs-admin` 发布，服务器 `tblogs-server` 只读 | Rust + axum + SQLite + askama | 7.7k 行 · 10 commits | —（有本地 Git，无远端） | dsh 3 个会话；`deploy/` 含 systemd + Caddy/nginx + cloudflared + DDNS |
| **gov-reports** | 2000–2026 共 27 份《政府工作报告》Markdown 语料 + RAG | Python + Markdown | 27 份 · 50.4 万字 | —（仅本机） | dsh 3 个会话；原始 HTML 快照可离线重跑 |

- **toval**：`universe → entity / era / event / story → chapter` 的图模型；单文件二进制，只监听 `127.0.0.1`，运行期不需要 Node。
- **tblogs**：服务器**没有任何写作入口**，被攻破也改不了文章；无前端构建、无 CDN。
- **gov-reports**：中央站点改版后缺失的 2001、2008–2012 共 6 年，用省级政府官方转载补齐并逐条标注来源。

---

## 三、设备与系统小工具（5）

| 项目 | 是什么 | 技术栈 | 体量 / 状态 | GitHub 仓库 | 构建轨迹 |
| --- | --- | --- | --- | --- | --- |
| **RaptorQR · Host** | 用屏幕上的动画二维码在无网络的两机之间搬文本和文件；新增屏幕捕获接收端 | Rust（gpui / 控制台） | host 16.8k 行 · 93 commits · universal · 已装 `/Applications` | [tuean/RaptorQR](https://github.com/tuean/RaptorQR)（fork） | pi 3 + dsh 2 个会话；GitHub Actions 出 Windows 构建 |
| **TBAS** | 把 Edge 分组标签页存成可检索、可选择性恢复、可安全关闭的快照 | Node CLI + Edge 扩展 | 5.3k 行 JS · 4 套测试 + e2e | [tuean/tbas](https://github.com/tuean/tbas) | dsh 1 个会话 |
| **utools-fd-search** | uTools 插件：fd（文件名）+ rg（内容）+ fzf（模糊排序） | JS / uTools | 1.5k 行 · 38 KB `.upxs` | [tuean/utools-fd-search](https://github.com/tuean/utools-fd-search) | pi 4 个会话 |
| **utools-selfie** | uTools 摄像头录像：CamHelper.app 把 ffmpeg 变成独立 TCC 实体 | JS + ffmpeg | 9 KB `.upxs` | [tuean/utools-selfie](https://github.com/tuean/utools-selfie) | pi 1 个会话 |
| **pdf2str** | PDF → 纯文本的一行命令，扫描件走 OCR | Rust + liteparse | 58 行 | —（仅本机） | `RustroverProjects/pdf2str` |

- **RaptorQR**：收到的文本不落盘、直接进系统剪贴板；文件写 `~/Downloads`；macOS 版支持多显示器与拖拽框选。
- **TBAS**：记录窗口 / 分组 / 颜色 / 折叠 / 固定页 / 顺序；「保存并关闭」按 80 MB/标签估算可释放数百 MB；导出离线 HTML 报告。

---

## 四、Agent 工具链

### 4.1 命令行脚本（`~/bin`）

| 脚本 | 作用 |
| --- | --- |
| `dsh-remote` / `win-task` | 跨机派单：任务走 ssh 的 stdin，远端 Windows 上的 dsh headless 执行；中文、引号、换行不被 shell 转义破坏 |
| `ui-unwedge` / `ui-wakeguard` | macOS 窗口服务器失步（Dock 卡住、空间切不动）的诊断与逐级复位；用 `sysctl kern.waketime` 做唤醒探测，命中才重诊断 |
| `dlp-toggle` | 终端管控（联软 / 微步）的临时开关；`bootout` 只卸载当前会话，重启自愈 |
| `wb-otel-env` | 从登录快照读账号，注入 WorkBuddy 的 OTel 上报环境变量（LaunchAgent 开机执行） |

### 4.2 pi 扩展

- `deepseek-balance.ts`：会话启动即拉余额，之后每 5 分钟刷新到状态栏。

### 4.3 自建 skill（13 个）

| skill | 作用 |
| --- | --- |
| `jira-issue-chain-create` | 按模板建「需求→同行评审→特性→故事→子任务」五层 Jira 单链 |
| `jira-cmmi-daily-check` | 读 CMMI 不符合项清单，按规则逐条改单并回读验证 |
| `jira-version-create` | 按日期生成 ISMP 的「故事 / 特性」两个发布版本 |
| `ismp-account-role-export` | 把 ISMP 导出的角色-菜单表加工成域账户审计清单 |
| `eboss-meeting-room-book` | 查空闲会议室并预约 / 改约（opencli 驱动 Edge） |
| `broker-chat-compliance-check` | 用本地 DeepSeek 对券商坐席聊天记录做合规质检 |
| `dashi-ppt` | 生成可离线打开、可浏览器编辑的 HTML 演示，导出 PPTX / PDF |
| `macos-windowserver-wedge-triage` | macOS UI 卡死分诊与恢复 |
| `macos-modern-cli-shell-setup` | 装齐现代 CLI 替代品并配好 zsh alias / 全局 AGENTS.md |
| `mem-skill-macos` | 「系统数据」膨胀定位与安全清理 |
| `pi-llm-cost-audit` | 从 session jsonl 聚合每日 / 每会话 token 与花费 |
| `split-large-excel-to-zip` | 大 Excel（含加密）按固定行数拆分并打包 |
| `iterm2-cmdv-image-paste` | 让 ⌘V 在 TUI 里既能粘文本又能贴图 |

### 4.4 dsh 侧

- 68 个会话、9 个市场插件（auto-memory、rewind、better-sidebar、notifier…）、任务板与子代理。
- 彩蛋：桌面宠物「D老师」，亲密度 484 / 396 轮。

---

## 五、构建方式

```
说需求 → 读仓库 → 小步改 → 装上去 → 回写文档
```

1. **说需求**：一句话、一段抱怨、一张截图，不写规格。
2. **读仓库**：agent 先读 README、测试与现有代码，形成约束。
3. **小步改**：一次一件事，跑测试、跑截图自检。
4. **装上去**：bundle → `/Applications`，真实用一轮。
5. **回写文档**：把取舍写进 README / AGENTS.md，下一次会话继承。

| | pi | dsh |
| --- | --- | --- |
| 形态 | 终端代理 | 桌面 / Web / headless |
| 特长 | `--mode rpc` 可被其它程序调用（TVA 即如此）；项目记忆 + 全局 AGENTS.md | 任务板、子代理、技能、插件市场；远程部署 |
| 适合 | 在一个仓库里长时间磨一个东西 | 把一整套流程跑完、多项目并行 |
| 记录 | 190 个会话 · 592 MB | 68 个会话 · 109 MB |

---

## 六、口径与证据

- **代码行数**：`src/` 与 `crates/` 下 `.rs` 行数合计，不含 `target/` 与依赖；JS/TS 同理。合计 154,694 行。
- **会话数**：pi 取 `~/.pi/agent/sessions/<工作区>/*.jsonl`（190 个）；dsh 取 `~/.dsh/sessions/<工作区>/`（68 个）。目录名即当时的工作目录。
- **装机**：以 `/Applications` 下实际存在为准（treq、maildesk、artifeeds、RaptorQR Receiver）。
- **公开仓库**：以 `git remote` 指向 `github.com/tuean` 为准（treq、maildesk、RaptorQR、utools-fd-search、toval、artifeeds、tbas、utools-selfie）。
- **归属**：会话目录只说明「在该项目里工作过」，不等于逐行归属；treq 的 README 有作者自述作为直接证据，其余按会话密度与提交节奏推断。
- **例外**：TVA 驱动 pi / dsh，但本机会话库里没有它的构建记录，因此不计入 pi/dsh 产物。
- **截图**：来自各仓库 `docs/`，非本次补拍。

---

## 七、怎么看

```bash
open ~/RustroverProjects/showoff/index.html      # 可视化展示页（单文件 + assets/）
open ~/RustroverProjects/showoff/showcase.pdf    # 同一份内容的 PDF 版（13 页，A4）
open ~/RustroverProjects/showoff/poster.png      # 首屏海报
```

页面支持按类别筛选，包含 treq / TBAS 的实拍截图、各项目的体量与构建轨迹，以及口径说明；
有 GitHub 仓库的 8 张卡片右上角带 `GitHub ↗` 入口。
