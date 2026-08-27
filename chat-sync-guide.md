# 两台电脑聊天记录实时同步（Syncthing）

家里电脑 ⇄ 公司电脑，通过 **Syncthing**（免费、点对点、跨网络）双向同步 `.dsh\sessions`（聊天历史）。秒级同步；其中一台离线时，下次两台同时在线自动补齐。

## 公司电脑（本机）—— 已配置完成 ✅

- Syncthing 已安装并运行（后台，开机自启）
- 共享文件夹：`dsh-sessions` → `C:\Users\A\.dsh\sessions`（双向，保留 5 个历史版本作安全网）
- 管理界面：浏览器开 `http://127.0.0.1:8384`
- **公司端设备 ID**（家庭端需要添加它）：

```
5THM5SK-GHPAYT4-M7GVTBW-KDP7QOW-SZ42DVQ-EBPKMZK-KD7SLGU-ABE2BQM
```

## 家庭电脑 —— 需要你操作（推荐一键脚本）

把 `E:\deepseek-harness-desktop\setup-syncthing-home.ps1` 拷到**家庭电脑**运行：

```powershell
powershell -ExecutionPolicy Bypass -File setup-syncthing-home.ps1
```

脚本自动完成：装 Syncthing → 启动 → 建共享文件夹 `dsh-sessions`（指向 `%USERPROFILE%\.dsh\sessions`）→ 开机自启 → **打印家庭端设备 ID**。

把打印出来的**家庭端设备 ID** 发给我，我在公司端加上，配对完成、开始同步。

### 手动方式（4 步，不想用脚本时）

1. 安装：`winget install --id Syncthing.Syncthing`，运行（浏览器打开 `http://127.0.0.1:8384`）
2. **Add Remote Device** → 粘贴公司端设备 ID（上方 `5THM5SK-…`）
3. **Add Folder**：
   - Folder ID：**`dsh-sessions`**（必须和公司端一模一样）
   - Folder Path：`C:\Users\<家庭用户名>\.dsh\sessions`
   - Folder Type：**Send & Receive**
4. 公司端接受连接请求

## 使用注意

- **避免两台电脑同时编辑同一个会话**：同一时间只用一边聊同一个会话（Syncthing 会保留冲突版本，不丢数据，但会多出 `.sync-conflict` 文件）
- 不同会话、先后使用都没问题（追加式 JSONL，同步很安全）
- 首次同步会传输约 10MB，之后只有增量
- 两台都要开着 Syncthing 才会同步（两端都已设/建议设开机自启）

## 管理界面速查

- 家庭电脑：`http://127.0.0.1:8384`
- 公司电脑（本机）：`http://127.0.0.1:8384`
- 查看同步状态/冲突版本都在这里
