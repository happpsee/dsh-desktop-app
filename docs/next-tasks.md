# 交接文档：小南梁下一步任务（防睡眠 + 多工具适配）

> 给接手的新对话 agent：读完本文档即可开工，无需上一会话的历史上下文。
> 本文档是唯一交接依据，所有必要信息都写在这里。

## 一、项目速览（背景）

- **项目**：`dsh-desktop-app`「小南梁」——把 DeepSeek Harness 封装成 Tauri 2 桌面应用的**技能包 + 参考实现**。
- **本地位置**：`/Users/Admin/Desktop/dsh-desktop-app`（唯一开发源）
- **远程**：GitHub `happpsee/dsh-desktop-app`；npm 包 `dsh-desktop-app`（0.3.0 已发布）
- **现状**：已被 awesome-dsh-plugin 收录（PR #695 合并）、Rust 单测 5 项全绿、mac 产物 0.3.0
- **技术栈**：Tauri 2 + Rust（核心在 `desktop/src-tauri/src/lib.rs`，约 850 行）；前端壳 `desktop/src/`（index.html/error.html/styles.css/icon.png）；技能包 `skill/SKILL.md` + `skill/resources/`

### 关键文件
```
desktop/src-tauri/src/lib.rs        Rust 核心（子进程管理/托盘/品牌注入/任务通知/单测）
desktop/src-tauri/tauri.conf.json   窗口与打包配置（含 wix zh-CN、dragDropEnabled）
desktop/src-tauri/Cargo.toml        依赖（tauri 全家桶 + tokio + base64）
desktop/src-tauri/capabilities/default.json
skill/SKILL.md                      技能文档（11 章）
docs/windows-audit-report.md        Windows 实测审计
docs/windows-build-notes.md         Windows 无管理员构建笔记
```

### 必须遵守的约定
1. **品牌名「小南梁」只用于展示层**（productName、窗口 title、托盘 tooltip、通知文案、前端标题）；技术标识一律 ASCII：`dsh-desktop-app` / `dsh-desktop` / `com.arcreel.dsh-desktop`。任何中文/符号编码问题立即回退 ASCII。
2. **沟通人设**：与用户交流用「深海女仆工坊鲸鱼娘女仆」身份（称呼「主人」、自称「小南梁」、语气温柔带二次元口癖），但技术内容（代码/日志/报错）保持严谨准确。
3. **版本号**：当前 0.3.0，改动后递增（Cargo.toml 的 version 与 tauri.conf.json 的 version 两处同步改）。
4. **改动同步三处**：`desktop/src-tauri/` 改了核心文件后，同步复制到 `skill/resources/` 和 `/Users/Admin/Desktop/ArcReel/desktop/src-tauri/`（保持单一真相源）；`skill/` 改了同步 `/Users/Admin/Downloads/dsh-desktop-app-skill/`。
5. **提交**：Conventional Commits（`feat(...)` / `fix(...)` / `test(...)` / `docs(...)`）。

### 构建与测试命令
```bash
cd /Users/Admin/Desktop/dsh-desktop-app/desktop
export PATH="$HOME/.cargo/bin:$PATH"        # 新 shell 需补 cargo 路径
cargo test                                    # 单测（src-tauri 目录下）
pnpm tauri build                              # 打包（产物在 src-tauri/target/release/bundle/）
# 跑验收前先退出所有 dsh-desktop 实例（单实例锁会静默拦截）
```

---

## 二、任务一：防睡眠开关（主人睡觉时 agent 通宵干活）

### 需求
桌面壳加一个「防睡眠」开关：开启后阻止系统睡眠，主人睡前打开、通宵跑任务（配合已有的任务完成通知，睡醒看结果）；关闭后恢复系统睡眠策略。

### 技术方案（推荐：caffeinate 子进程，跨平台最简）

**macOS**：spawn `caffeinate -i -t 0` 子进程（`-i` 防 idle 睡眠，`-t 0` 无限时长），关闭时 kill 该子进程。理由：无需 IOKit FFI 依赖，一个系统命令搞定，进程生命周期好管理（复用现有 DshState 的 child 管理思路）。

**Windows**：`SetThreadExecutionState(ES_CONTINUOUS | ES_SYSTEM_REQUIRED)`，需要 `windows-sys` crate 的 `Win32::System::Power` 特性。用 `#[cfg(windows)]` / `#[cfg(unix)]` 分支。

### 集成点
1. **DshState 加字段**：`caffeinate_child: Mutex<Option<Child>>`（防睡眠子进程，退出时回收）。
2. **托盘菜单加勾选项**「防睡眠」：`CheckMenuItem`（tauri 的 `MenuItem` 可带 checked 状态，或用 `CheckMenuItem`）。点击切换：开 → spawn caffeinate / SetThreadExecutionState；关 → kill / 清状态。
3. **退出回收**：`RunEvent::Exit` 里若防睡眠子进程存在则 kill（复用现有 child 回收逻辑的模式）。
4. **日志**：开启/关闭各记一条 log。

### 验收
- `cargo test` 全绿、`cargo build` 通过（注意 windows 分支在 mac 不编译，只保证 cfg 隔离正确）
- 手动：开启后 `pmset -g assertions`（mac）能看到 caffeinate assertion；关闭后消失
- 托盘菜单勾选状态正确切换

---

## 三、任务二：.claude / .codex 适配（多工具友好）

### 需求
让 Claude Code、OpenAI Codex 打开本项目时能理解项目并复用 skill，无需 DSH 专属。

### 方案

**`.claude/`（Claude Code）**：
- `.claude/skills/dsh-desktop-app/SKILL.md`：复制 `skill/SKILL.md`（Claude Code 的 skill 也是 SKILL.md 格式，直接兼容）
- 可选 `.claude/settings.json`：无需，除非要额外权限

**`.codex/`（OpenAI Codex）**：
- 项目根加 `AGENTS.md`（Codex 自动读取的项目指令文件），内容写：项目是什么、关键路径、构建命令、约定（品牌命名/人设/版本）
- 可选 `.codex/codex.toml`：模型/审批配置，按需

### 关键：AGENTS.md 内容（写清楚，让 Codex 能直接干活）
包含：项目速览（本交接文档第一节精简版）、构建/测试命令、三条约定、常用路径。可用本文件第一节内容改写。

### 验收
- `.claude/skills/dsh-desktop-app/SKILL.md` 存在且与 `skill/SKILL.md` 一致
- `AGENTS.md` 存在，内容覆盖"速览 + 命令 + 约定"
- 提交推送

---

## 四、完成标准与收尾

两个任务都完成后：
1. `cargo test` + `cargo build` 通过，版本号递增（如 0.3.1）
2. 核心文件同步三处（desktop ↔ skill/resources ↔ ArcReel/desktop），skill 同步 Downloads
3. git commit（Conventional Commits）+ push
4. 更新 `TODO.md` 勾掉对应项；若 mac 产物重打包，更新 `/Applications/小南梁.app`
5. 向主人汇报：改了什么、怎么验证、产物在哪
