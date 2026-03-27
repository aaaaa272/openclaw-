# OpenClaw 部署与飞书接入完全指南 (Windows 10)

> 🚨 **安全警告 (2026年3月)**
> 目前针对 OpenClaw 的恶意软件和钓鱼攻击较为泛滥。**请绝对不要**从不明链接、非官方的 GitHub 仓库或伪造的 Docker 部署工具中下载。请严格遵循本教程，仅通过官方的 npm/pnpm 命令行获取。

---

## 阶段一：环境准备与核心部署

OpenClaw 强依赖于 Node.js，且原生支持 Windows 环境，我们可以直接通过 PowerShell 完成安装。

### 1. 安装 Node.js
1. 前往 [Node.js 官网](https://nodejs.org/)。
2. 下载并安装最新的 **LTS (长期支持) 版本** 的 Windows 安装包。
3. 按下 `Win + X`，选择“Windows PowerShell”，验证安装（版本需 >= v22.12.0）：
   ```powershell
   node --version
   ```

### 2. 安装 OpenClaw CLI
1. **以管理员身份运行 PowerShell**（避免全局安装时遇到权限报错）。
2. 执行全局安装命令：
   ```powershell
   npm install -g openclaw@latest
   ```
3. 验证安装是否成功：
   ```powershell
   openclaw --version
   ```

### 3. 运行初始化与守护进程
安装完成后，初始化工作区并将其注册为后台服务，确保 AI 助手保持在线：
```powershell
openclaw onboard --install-daemon
```
> **提示**：按照屏幕交互提示选择工作区路径（默认在 `%USERPROFILE%\.openclaw`）和你的大模型提供商（输入对应的 API Key）。

---

## 阶段二：飞书端“数字员工”配置

使用飞书官方的 WebSocket 长连接方案，无需配置复杂的内网穿透即可实现稳定通讯。

### 1. 创建飞书自建应用
1. 浏览器登录 [飞书开放平台](https://open.feishu.cn/app)。
2. 点击 **“创建企业自建应用”**，填写应用名称与图标。
3. 在左侧菜单 **“凭证与基础信息”** 中，记录下 **App ID** 和 **App Secret**（⚠️ 妥善保密）。

### 2. 配置应用权限
1. 在左侧菜单点击 **“应用能力” -> “机器人”**，点击 **“启用”** 并保存。
2. 在左侧菜单点击 **“安全设置” -> “权限管理”**。
3. 点击右上角的 **“批量导入权限”**，粘贴以下代码：
   ```json
   {
     "scopes": {
       "tenant": [
         "im:message.send_as_bot",
         "im:message:readonly",
         "im:message.p2p_msg:readonly",
         "im:chat.members:bot_access",
         "aily:file:read",
         "aily:file:write"
       ]
     }
   }
   ```

### 3. 发布应用
在左侧菜单点击 **“版本管理与发布” -> “创建版本”**，填写版本号（如 `1.0.0`），保存并 **“申请发布”**。*(注：未发布状态下无法建立长连接)*

---

## 阶段三：连接服务与安全验证

回到你的 Windows PowerShell 终端，完成最后的插件安装与绑定。

### 1. 安装飞书官方插件
执行以下命令，该工具将自动配置 WebSocket 模式：
```powershell
npx -y @larksuite/openclaw-lark-tools install
```
> **提示**：根据向导提示，依次填入之前记录的 **Feishu App ID** 和 **Feishu App Secret**。

### 2. 重启网关服务
使刚刚安装的插件配置生效：
```powershell
openclaw gateway restart
```

### 3. 安全验证（私信配对）
为防止未授权调用大模型 API，必须完成首次私信配对绑定：
1. 打开飞书客户端，搜索你的机器人名称，发送一条私信（例如：“你好”）。
2. 机器人会回复一段提示并提供一个 **配对码 (Pairing Code)**（例如 `Y7A9B`）。
3. 回到 PowerShell 终端，输入以下命令完成授权绑定（将 `Y7A9B` 替换为你的实际验证码）：
   ```powershell
   openclaw pairing approve feishu Y7A9B
   ```

---

## 附录：常用运维命令

在日常使用和调试中，你可能会用到以下命令：

* **查看系统状态**：`openclaw status`
* **查看代理日志**：`openclaw history`
* **手动重启服务**：`openclaw gateway --port 18789 --verbose`
