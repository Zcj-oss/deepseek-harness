# 两台电脑聊天记录实时同步（Syncthing）

家里电脑 ⇄ 公司电脑，通过 **Syncthing**（免费、点对点、跨网络）双向同步 `.dsh\sessions`（聊天历史）。秒级同步；其中一台离线时，下次两台同时在线自动补齐。

## 家庭电脑（本机）—— 已配置完成 ✅

- Syncthing 已安装并运行（后台，开机自启）
- 共享文件夹：`dsh-sessions` → `C:\Users\A\.dsh\sessions`（双向，保留 5 个历史版本作安全网）
- 管理界面：浏览器开 `http://127.0.0.1:8384`
- **本机设备 ID**（公司电脑要填这个）：

```
5THM5SK-GHPAYT4-M7GVTBW-KDP7QOW-SZ42DVQ-EBPKMZK-KD7SLGU-ABE2BQM
```

## 公司电脑 —— 需要你操作

### 1. 安装 Syncthing
```powershell
winget install --id Syncthing.Syncthing -e --source winget
```
装完运行它（首次运行在浏览器打开 `http://127.0.0.1:8384` 管理界面）。

### 2. 添加远程设备（家庭电脑）
- 管理界面 → 右上角 **Add Remote Device**
- 设备 ID 粘贴：`5THM5SK-GHPAYT4-M7GVTBW-KDP7QOW-SZ42DVQ-EBPKMZK-KD7SLGU-ABE2BQM`
- 名称随意（如 "home"）

### 3. 添加共享文件夹
- **Add Folder**
- Folder ID：**`dsh-sessions`**（必须和家庭端一致）
- Folder Path：`C:\Users\<你的公司用户名>\.dsh\sessions`（没有就新建）
- Folder Type：**Send & Receive**（双向）
- 保存

### 4. 家庭端确认连接
家庭电脑的 Syncthing 管理界面会弹出"接受设备/文件夹"提示，点接受。两边开始同步。

> 同步密钥是"设备对设备"的：家庭端也需要加公司电脑的设备 ID（或接受家庭端的待确认请求）。如果公司端添加后家庭端没自动弹提示，把公司电脑的设备 ID 发给我，我在这边直接加上。

## 使用注意

- **避免两台电脑同时编辑同一个会话**：同一时间只用一边聊同一个会话（Syncthing 会保留冲突版本，不丢数据，但会多出 `.sync-conflict` 文件）
- 不同会话、先后使用都没问题（追加式 JSONL，同步很安全）
- 首次同步会传输约 10MB，之后只有增量
- 两台都要开着 Syncthing 才会同步（开机自启已设/建议设）

## 管理界面速查

- 家庭电脑：`http://127.0.0.1:8384`
- 公司电脑：装好后同样 `http://127.0.0.1:8384`
- 查看同步状态/冲突版本都在这里
