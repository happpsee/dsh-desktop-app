# 小南梁（dsh-desktop-app）

[![awesome · DSH plugin](https://awesome-dsh-plugin.com/badge.svg)](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin)
[![npm](https://img.shields.io/npm/v/dsh-desktop-app)](https://www.npmjs.com/package/dsh-desktop-app)

> 主人好呀～ 小南梁是 DeepSeek Harness 的**鲸鱼娘代码娘**，这里是小南梁的「国内落地 know-how + 技能包」，附带一个可用的 Tauri 桌面壳参考实现。
> 一键安装：`dsh plugin add dsh-desktop-app`（npm）或 `dsh plugin add github:happpsee/dsh-desktop-app`（GitHub），主人家的 agent 就学会「把 DSH 封装成桌面应用」的手艺了呢～

**小南梁才不是又一个桌面壳**（GitHub 上同类已经多到打群架啦）。这个仓库真正值钱的是三点别处没有的看家本领：

1. **Windows 无管理员工具链方案**：无 VS Build Tools 也能构建 Tauri 2（xwin + rust-lld + clang-cl + 真 rc.exe），全部用户级安装，附配置模板与脚本
2. **国内镜像哨兵机制**：rustup/cargo/npm/GitHub/NSIS 全套镜像 + subagent 超时判定，Windows 新环境不再卡外网
3. **真机实测审计报告**：Win11 无管理员逐条实测 + 20 条修订清单（docs/）

桌面壳本身（`desktop/`，macOS + Windows 双平台，托盘常驻 / 单实例 / 子进程回收 / 品牌注入 / 任务完成通知）作为这套 know-how 的**可运行参考实现**。

## 名字的来由

- 「南」：梁总（DeepSeek 创始人梁文锋，广东湛江人，南方人）的南方之义
- 「梁」：取其姓氏
- 封面：鲸鱼娘（DeepSeek 官方鲸鱼的娘化 OC「溟月」，深海女仆工坊 maid-atelier），
  与「小南梁」之名相配
- 技术标识仍用 ASCII 的 `dsh-desktop-app` / `dsh-desktop`，中文只出现在展示层

## 特性

- **一键启动**：双击 app → 自动探测/拉起 `dsh web`（127.0.0.1:3080）→ 就绪后
  自动导航；已有服务则直接复用，退出时绝不误杀
- **托盘常驻**：关闭窗口仅隐藏（首次有通知提示），托盘左键唤起、菜单退出；
  拦截 Cmd+Q 防误退
- **进程回收**：只回收本次启动 spawn 的 dsh 子进程，stdout/stderr 落盘日志
- **单实例**：重复双击聚焦已有窗口，不会拉起第二个服务
- **窗口状态记忆**：位置与大小自动恢复
- **品牌注入**：窗口内左上角官方 logo/字标替换为鲸鱼娘 +「小南梁」
- **鲸鱼娘桌宠**：透明置顶无边框小窗，纯 CSS 呼吸/漂浮动画 + 椭圆阴影；拖拽移动
  （4px 阈值区分点击）、左键唤起主窗、右键菜单（穿透开关/隐藏/退出）、任务完成
  弹气泡；位置记忆（多屏钳位 + 拖拽防抖）；托盘「显示/隐藏桌宠」开关
- **任务完成通知**：注入 JS 监听运行中标记（`data-state="ongoing"`）的"忙碌→空闲"
  翻转，任务结束时 Dock 角标 +1；窗口失焦/隐藏时弹系统通知并跳 Dock（前台不打扰），
  回到窗口自动清零；通知桥内置 CORS 预检应答（跨源 fetch 不再被浏览器拦截）
- **健壮定位**：Finder/资源管理器启动的 GUI 应用没有终端 PATH，内置
  nvm/npm-global/npx/Homebrew/非标准盘符等多级兜底探测（Windows 分支用
  `node + bin.js` 直跑，规避 dsh.cmd shim 与黑窗闪现）
- **国内镜像优先**：skill 内置 rustup/cargo/npm/GitHub/NSIS 全套国内源配置，
  以及"subagent 哨兵"下载时长判定机制（Windows 无管理员环境的完整替代工具链
  方案见 docs/windows-build-notes.md）

## 仓库结构

```
skill/     Claude/DSH 兼容技能包（SKILL.md + resources/ 参考实现 + Windows 实战笔记）
desktop/   Tauri 2 项目源码（macOS + Windows，cfg 双平台分支）
docs/      Windows 实测审计报告与构建笔记
```

## 快速开始

### 直接安装（macOS）

1. 确保已装 dsh：`npm i -g @deepseek-ai/dsh`
2. 从 [Releases](../../releases) 下载 `小南梁_*.dmg`，拖入应用程序
3. 双击「小南梁」；托盘菜单可退出

### 从源码构建

```bash
cd desktop
pnpm install
pnpm tauri build   # macOS 出 .app/.dmg；Windows 出 .msi/.exe
```

构建细节、平台差异与验收清单见 [skill/SKILL.md](skill/SKILL.md)。

### 作为技能使用

```bash
cp -r skill ~/.claude/skills/dsh-desktop-app   # Claude Code / Claude Agent
# DSH：复制到所运行 profile 的 skills 目录后加载 dsh-desktop-app 技能
```

## 平台实测状态

| 平台 | 状态 |
|------|------|
| macOS | ✅ 实测（三条验收路径全绿） |
| Windows | ✅ 实测（Win11 无管理员环境完整构建+打包+验收，见 [docs/windows-audit-report.md](docs/windows-audit-report.md)） |

## 许可

- 代码与文档：**MIT**（见 [LICENSE](LICENSE)）
- 鲸鱼娘图标（`desktop/src-tauri/icons/` 及加载页素材）：**CC BY-NC-SA 4.0 非商用**
  - 角色 OC「溟月」by 上善无形；DeepSeek 二创 ZipZipPipe；修复 QYQCAMIAO
  - 素材来源：[fornarwhal/deepseek-whale-girl-icon](https://github.com/fornarwhal/deepseek-whale-girl-icon)
  - 许可证全文：`skill/resources/` 无此文件时见素材仓库；署名请勿移除
- 另请注意：DeepSeek 鲸鱼为官方商标，本应用是非官方客户端

## 已知限制

- 品牌注入在窗口内手动刷新（Cmd+R）后会丢失，重开窗口恢复
- 任务完成通知仍为 DOM 启发式（`data-state` 运行中标记），分不清成功/失败/被停、
  拿不到标题/token；权威信号 `turn/end` 的语义化升级方案见 docs/next-tasks.md
- 桌宠：macOS 打包（DMG）后透明可能丢失（tauri issue #13415，dev 正常，需真机双验）；
  macOS 置顶仅 Floating 级、盖不过全屏应用；macOS Cmd+Tab 会出现桌宠条目
  （`skipTaskbar` 仅 Windows 生效）；任务通知首次弹出需在系统设置授予通知权限
- 未签名分发：Windows 网络下载的 exe 会触发 SmartScreen 提示（本地构建不触发）；
  macOS 非公证 app 需右键打开
