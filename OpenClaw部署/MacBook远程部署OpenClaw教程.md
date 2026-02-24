# 教程：在 MacBook 上远程部署 OpenClaw 并与 Manus AI 联通

**版本**：1.0
**目标**：通过安全的私有网络，允许我（Manus AI）远程向您 MacBook 上的 OpenClaw 发送指令，实现抖音账号的全自动化运营。
**制作人**：Manus AI

---

## Part 1: 架构与核心原理

我们将使用 **Tailscale** 这款工具，在我的云端服务器和您的 MacBook 之间，建立一个点对点的加密私有网络。这相当于为我们之间开辟了一条“内部安全通道”，我可以通过这条通道，合法且安全地访问您 MacBook 上的 OpenClaw 服务，而无需将您的电脑暴露在公共互联网上。

**最终工作流：**

```
Manus AI (云端) --> Tailscale 私有网络 --> MacBook (您的本地网络) --> OpenClaw --> 本地浏览器 --> 抖音创作者中心
```

---

## Part 2: MacBook 端部署与配置（全程约15分钟）

请在您的 MacBook 上，按照以下步骤操作：

### 第 1 步：安装环境依赖 (Homebrew & Node.js)

打开您 MacBook 上的“**终端 (Terminal)**”应用，执行以下命令：

1.  **安装 Homebrew** (如果已有请跳过):
    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```

2.  **安装 Node.js v22** (OpenClaw 官方要求):
    ```bash
    brew install node@22
    ```

### 第 2 步：安装 OpenClaw

继续在终端中，执行 OpenClaw 官方一键安装脚本：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

安装过程中，它会自动完成引导配置。结束后，您可以执行 `openclaw doctor` 来检查安装是否成功。

### 第 3 步：安装并配置 Tailscale

1.  **从 App Store 安装 Tailscale**: 在 MacBook 的 App Store 中搜索“Tailscale”并安装。

2.  **登录 Tailscale**: 打开 Tailscale 应用，使用您的 Google、Apple 或其他账号登录。这会将您的 MacBook 加入到您的私有网络中。

3.  **获取 MacBook 的 Tailscale IP**: 登录成功后，点击菜单栏的 Tailscale 图标，您会看到一串 `100.x.x.x` 格式的 IP 地址，**请复制并保存好这个 IP 地址**，这是您 MacBook 在私有网络中的唯一身份。

### 第 4 步：配置 OpenClaw，开启“远程遥控模式”

这是最关键的一步。我们需要修改 OpenClaw 的配置文件，让它通过 Tailscale 接收我的指令。

1.  **打开配置文件**: 在终端执行以下命令，用文本编辑器打开配置文件：
    ```bash
    open ~/.openclaw/openclaw.json
    ```

2.  **修改配置文件**: 将文件内容修改为如下配置。这会同时**开启 Tailscale 服务**和**Webhook 远程指令接收服务**。

    ```json
    {
      "gateway": {
        "bind": "loopback",
        "tailscale": {
          "mode": "serve"
        },
        "auth": {
          "mode": "token",
          "token": "YOUR_SECRET_TOKEN"
        }
      },
      "hooks": {
        "enabled": true,
        "token": "YOUR_SECRET_TOKEN",
        "path": "/hooks",
        "allowRequestSessionKey": true
      },
      "browser": {
        "enabled": true
      }
    }
    ```

    > **重要提示**：请将上面两个 `YOUR_SECRET_TOKEN` 替换为您自己的**一个足够复杂的密码**。这个密码将作为我和您之间通信的“暗号”。

### 第 5 步：重启 OpenClaw 并验证

1.  **重启 OpenClaw Gateway** 使配置生效:
    ```bash
    openclaw gateway restart
    ```

2.  **验证远程访问**: 在您的**手机**上，确保也安装并登录了同一个 Tailscale 账号，然后打开手机浏览器，访问 `http://<你MacBook的Tailscale IP>:18789`。如果您能看到 OpenClaw 的 Dashboard 登录界面，则证明远程通路已成功打通！

---

## Part 3: 交付与联调

恭喜您！您的 MacBook 现在已经是一台可以被我远程安全调度的“AI 自动化执行节点”。

请将以下两项信息提供给我，我即可开始远程操控 OpenClaw 执行任务：

1.  **您 MacBook 的 Tailscale IP 地址** (如: `100.x.x.x`)
2.  **您在配置文件中设置的 `YOUR_SECRET_TOKEN`**

我收到信息后，会通过类似以下的指令，从我的云端服务器向您的 MacBook 发出第一条测试指令，例如：

```bash
# 我将在我的服务器上执行此命令
curl -X POST http://<您的Tailscale IP>:18789/hooks/agent \
  -H 'Authorization: Bearer <您的Secret Token>' \
  -H 'Content-Type: application/json' \
  -d '{"message":"打开浏览器，访问 douyin.com"}'
```

如果一切顺利，您将看到您 MacBook 上的浏览器被自动唤起，并打开抖音首页。至抖音首页。至此，我们的远程协作链路便已完全建立！
