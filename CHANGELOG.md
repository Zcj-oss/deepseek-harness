# DeepSeek Harness 桌面端 更新日志

桌面端位于 `E:\deepseek-harness-desktop\`，本文件记录从安装到当前的全部改动。

---

## 一、桌面端创建（基础）

- 新建 Electron 桌面壳（独立于 deepseek-harness 仓库）：自动拉起 `dsh web` 服务 + 独立窗口
- 桌面快捷方式「DeepSeek Harness 桌面端」
- electron-builder 打包 portable 单文件 exe（`release\DeepSeek-Harness-Desktop-0.1.0.exe`）
- 服务复用：3080 已有服务则直接复用，无则自动拉起；窗口关闭时回收子进程

## 二、黑屏问题修复（3 个根因）

1. **仓库 node_modules 陈旧**：缺 `@deepseek-ai/dsh-client-ui-directory-picker-native` 链接，`dsh web` 启动即崩溃 → 仓库跑 `pnpm install` 修复
2. **Electron 内置 Node 过旧**（20.18.3 < 22.15，缺 `node:zlib` zstd API）→ 桌面端改用**系统 Node** 拉起服务
3. **加载时序**：服务启动早期返回空文档 → 新增 **API 就绪探针**（`POST /api/session.list` 返回 200 才切窗口）
- 启动失败时弹窗附**服务日志尾部**，便于定位

## 三、启动提速

- 用**预编译 `apps/cli/lib/bin.js`** 替代源码+tsx 拉起服务：冷启动 **51.6s → 约 3s**（8 倍提速；源码模式仅作缺失时回退）
- **启动闪屏**：窗口立即出现"正在启动…"，服务就绪后自动切换
- 轮询精度收紧：500ms→200ms、750ms→250ms

## 四、托盘常驻

- 系统托盘图标（蓝色 D），双击/点图标可唤回窗口
- 托盘菜单：**打开 / 设置… / 退出（同时停止服务）**
- 点关闭按钮 = 隐藏到托盘（服务继续后台运行），热启动约 1 秒
- 修复"关窗后点桌面图标没反应"（second-instance 未调用 show）

## 五、设置窗口

- 入口：窗口右下角 **"⚙ 桌面设置"** 按钮（余额角标正上方）+ 托盘"设置…"
- 选项：
  - **开机自启动**（写入 Windows 启动项，即时生效）
  - **显示余额角标**（开关）
  - **余额刷新间隔**（1~60 分钟，默认 3）
- 设置持久化到 `%APPDATA%\dsh-desktop\config.json`，即时生效
- 齿轮按钮位置修正：避开 harness 左下角设置入口，改到右下角

## 六、DeepSeek API 余额角标

- 窗口右下角显示实时余额（`https://api.deepseek.com/user/balance`）
- API Key 读取顺序：环境变量 → `$DSH_HOME/.credentials.yaml` → 仓库 `.env`（**Key 只在主进程**，不注入页面）
- 刷新：启动 + 每 3 分钟 + 窗口聚焦（30s 防抖）+ **左键点击立即刷新**
- **右键余额角标** → 用系统浏览器打开 DeepSeek API 平台（platform.deepseek.com/usage）

## 七、前台唤醒修复

- 后台启动/开机自启时窗口被其它窗口遮挡 → 页面就绪时 `show + moveTop + focus` 强制置前

## 八、Git 版本管理

- 安装 Git 2.55.0.3 + GitHub CLI 2.97.0（已登录账号 Zcj-oss）
- `E:\deepseek-harness` 初始化 git 仓库：**7467 个源文件**提交 `f067f02 snapshot 2026-08-20`
- 版本标签：**`v-snapshot-2026-08-20`**
- 私有远程仓库：**https://github.com/Zcj-oss/deepseek-harness**（main + 标签已推送）
- **git 代理配置**：`http.proxy 127.0.0.1:7897`（GitHub 直连被墙，走 Clash）
- 字节级保真：`core.autocrlf=false`，源码指纹 `403455BC…55DF3D` 核对一致

## 九、自动更新（新电脑收版本）

- **检测**：启动 20 秒后 + 每 30 分钟，`git ls-remote --tags origin` 对比最新日期标签（约定 `v-YYYY-MM-DD`）与本地版本
- **提示**：有新版 → 右下角绿色 **"⬆ 新版本 v-… 可用，点击更新"**
- **更新**：点击自动 `git fetch → git checkout <tag> → pnpm install → pnpm run build`（进度实时显示，构建约 10 分钟）→ 弹窗**立即重启**生效
- 兜底：GitHub 连不上 / 仓库有本地改动时提示原因，不打扰

## 十、新电脑迁移

- **`setup-new-machine.ps1`**：一键配置（Node 检查 → pnpm → 仓库确认/克隆 → 装依赖 → 构建 → API Key → 桌面配置）
- **`harness-version.ps1`**：版本指纹生成/核对（`-Verify` 输出 MATCH/MISMATCH）
- **《迁移到新电脑.md》**：完整迁移 + 版本一致性指南
- 远程仓库方案：两台机器锁同一 commit/tag（`git checkout v-snapshot-2026-08-20`）

---

## 当前版本状态

- 桌面端 exe：`release\DeepSeek-Harness-Desktop-0.1.0.exe`（约 69MB）
- 仓库快照：`f067f02` / `v-snapshot-2026-08-20`（已推送 GitHub 私有仓库）
- 自检：复用路径 ~7s / 拉起路径 ~9s，两条路径 DOM 与截图均正常

## 使用提示

- 升级本机仓库后：`git add -A && git commit -m "…" && git tag v-<日期> && git push origin main --tags`
- 新电脑：桌面端每 30 分钟自动检测，看到绿色角标点一下即自动更新
- 访问 GitHub 需要代理开启（Clash 127.0.0.1:7897）
