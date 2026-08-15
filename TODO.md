# TODO / Roadmap

> 发布后的收尾与后续方向。完成项勾选后随 commit 更新。

## 质量与合流

- [ ] **补 Rust 单元测试**：探测逻辑（find_dsh_bin / find_node / find_dsh_bin_js）、
      `version_key` semver 排序、端口就绪判断、SpawnError 分类——目标是 CI 可跑
- [ ] **Windows 侧最终合流**：让 Windows 机器拉取本仓库 0.2.5+ 代码重跑
      acceptance.ps1，确认 mac 仓库代码在 Windows 实测通过（消除"两线代码"状态）
- [ ] **Releases 补 Windows 产物**：0.2.5 的 .msi / -setup.exe 上传至 Releases

## 已知限制（v1 遗留）

- [ ] **品牌注入刷新丢失**：窗口内 Cmd+R 刷新后样式还原，需 webview on_navigation
      钩子自动重注入
- [x] **Web GUI 任务完成通知**：本地 HTTP 桥 + busy→idle 检测 + Dock 角标/跳动
      （0.3.0 实现；端到端实测待用户退出实例后验证）
- [ ] **托盘点击验收**（Windows 无头环境未实测）：需要真机手工确认

## 生态化（可选方向）

- [x] **dsh.bundle 化改造**：已做成 `dsh plugin add` 可安装的 bundle（index.js +
      cordis.patch.yml + package.json dsh.bundle），npm 发布 0.3.0，awesome-dsh-plugin
      收录（PR #695 已合并，dsh-market 自动聚合）
- [ ] **中文技术社区文章**：《把 DeepSeek Harness 包成桌面应用》实操文
      （掘金/知乎/B 站），挂仓库链接
- [x] **图像理解接入**：dsh-vision-router 插件已安装并验证（页面注入 ✓，视觉工具
      已挂载，OVH 免费链 + 可选自有视觉模型）
