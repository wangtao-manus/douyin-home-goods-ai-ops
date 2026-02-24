# Manus x 云手机：抖音全自动运营终极方案

**更新日期**：2026-02-24

**方案作者**：Manus AI

---

## 1. 核心架构：Manus + 阿里云无影云手机 + OpenClaw

本方案是目前实现抖音全自动运营的**最优解**，它彻底解决了境外 IP 限制、设备在线、手动操作等所有痛点。您只需完成一次性配置，后续所有操作均可由我（Manus）在云端发起，实现 7x24 小时无人值守自动化。

### 1.1. 工作流程

```mermaid
graph TD
    A[Manus 云端大脑] -- 1. 生成内容与指令 --> B{任务指令 (JSON)};
    B -- 2. 调用阿里云 API --> C[阿里云无影云手机 (国内IP)];
    C -- 3. 运行 OpenClaw Agent --> D[抖音 APP];

    subgraph C
        direction LR
        E[OpenClaw Agent] -- 模拟真人操作 --> F[抖音 APP];
    end

    D -- 4. 执行操作 --> G[发布视频/回复评论];
```

| 环节 | 执行者 | 说明 |
| :--- | :--- | :--- |
| **策略与内容** | **Manus** | 我负责生成选品报告、视频脚本、评论话术等。 |
| **指令下发** | **Manus** | 我将“发布视频”等指令通过 API 发送给阿里云。 |
| **指令执行** | **云手机 + OpenClaw** | 云手机内的 OpenClaw Agent 收到指令，模拟真人操作抖音 APP。 |
| **结果反馈** | **云手机** | 执行结果通过 API 返回给我，形成闭环。 |

### 1.2. 方案优势

- **完全自动化**：从内容生成到发布，无需任何人工干预。
- **规避风控**：使用国内数据中心的真实手机环境，IP干净，操作轨迹拟人化，风险极低。
- **7x24 小时在线**：云手机永不关机、永不断网。
- **高性价比**：相比组建人工团队，每月成本可忽略不计（约 ¥50-100）。

---

## 2. 实施步骤（约 30 分钟）

### 2.1. 购买阿里云无影云手机（OpenClaw 版）

这是最关键的一步，阿里云官方提供了预装 OpenClaw 的镜像，极大简化了部署流程。

1.  登录 [阿里云无影云手机控制台][1]。
2.  点击 **创建实例组**。
3.  在创建页面，进行如下关键配置：
    - **地域**：选择**中国大陆**任意地域。
    - **规格**：建议选择 **标准型 (cphone.4c8g.32g)** 或更高配置。
    - **镜像**：必须选择 **AI 镜像** → **无影 OpenClaw Image**。
    - **网络**：推荐选择 **云手机标准网络**，带宽 **5 Mbps** 即可。
4.  完成支付。您现在拥有了一台预装了 AI Agent 的云手机。

### 2.2. 配置云手机与 OpenClaw

#### 2.2.1. 获取阿里云 AccessKey

为了让 Manus 能够通过 API 控制您的云手机，需要创建一个 AccessKey。

1.  前往 [阿里云 RAM 控制台][2]。
2.  在左侧导航栏选择 **身份管理** → **用户**。
3.  创建一个新用户，并为其授予 `AliyunECPFullAccess` 权限。
4.  保存生成的 **AccessKey ID** 和 **AccessKey Secret**。

#### 2.2.2. 获取百炼 API Key

OpenClaw 需要一个大模型来理解指令，这里使用阿里云自家的百炼大模型。

1.  前往 [阿里云百炼大模型服务平台][3]。
2.  在左侧导航栏选择 **密钥管理**，创建一个新的 API Key。
3.  保存生成的 **API Key**（以 `sk-` 开头）。

#### 2.2.3. 在云手机中配置 Key

1.  返回无影云手机控制台，找到刚刚创建的实例，点击 **连接**。
2.  在云手机连接页面，点击右侧的 **SSH** 按钮，进入命令行终端。
3.  执行以下命令，将 `NEW_KEY` 替换为您上一步获取的百炼 API Key，一键完成配置并重启 OpenClaw。

    ```bash
    NEW_KEY="sk-xxxxxxxxxxxxxxxxxxxx" && sed -i 's/"apiKey": "DASHSCOPESK"/"apiKey": "'"$NEW_KEY"'"/g' /data/vendor/openclaw/home/.openclaw/openclaw.json;killall openclaw openclaw-gateway 2>/dev/null;/data/bin/openclaw gateway --port 18789 > /data/vendor/openclaw/tmp/openclaw_gateway.log 2>&1 &
    ```

至此，您的云手机 AI Agent 已配置完毕并开始运行。

---

## 3. Manus 远程控制代码模板

您无需关心这部分代码，这是我（Manus）在云端为您运行时使用的。您只需将 **阿里云 AccessKey** 和 **云手机实例 ID** 提供给我即可。

```python
# file: manus_control_cloud_phone.py
import os
import json
from aliyunsdkcore.client import AcsClient
from aliyunsdkecp.request.v20210914 import RunCommandRequest

# --- 您需要提供给 Manus 的信息 ---
ACCESS_KEY_ID = "YOUR_ACCESS_KEY_ID"       # 您的阿里云 AccessKey ID
ACCESS_KEY_SECRET = "YOUR_ACCESS_KEY_SECRET" # 您的阿里云 AccessKey Secret
INSTANCE_ID = "YOUR_INSTANCE_ID"           # 您的云手机实例 ID
REGION_ID = "cn-hangzhou"                  # 您的云手机所在地域
# -------------------------------------

class CloudPhoneController:
    def __init__(self):
        self.client = AcsClient(ACCESS_KEY_ID, ACCESS_KEY_SECRET, REGION_ID)

    def send_command_to_openclaw(self, prompt: str):
        """向云手机中的 OpenClaw 发送高级指令"""
        print(f"[INFO] 正在向云手机实例 {INSTANCE_ID} 发送指令...")
        print(f"[PROMPT] {prompt}")

        # 使用 OpenClaw 的命令行工具（TUI）发送指令
        # 这种方式比直接调用 webhook 更稳定
        command = f'openclaw tui --message "{prompt}"'

        request = RunCommandRequest.RunCommandRequest()
        request.set_InstanceId(INSTANCE_ID)
        request.set_CommandContent(command)
        request.set_ContentType("PlainText")

        try:
            response_str = self.client.do_action_with_exception(request)
            response = json.loads(response_str)
            print(f"[SUCCESS] 指令发送成功，任务 ID: {response.get('TaskId')}")
            return response
        except Exception as e:
            print(f"[ERROR] 指令发送失败: {e}")
            return None

# --- Manus 调用示例 ---
if __name__ == "__main__":
    controller = CloudPhoneController()

    # 示例1：自动发布视频
    # 前提：视频文件 '~/videos/new_video.mp4' 已通过 ADB 或其他方式上传到云手机
    video_publish_prompt = (
        "打开抖音，点击底部的加号按钮开始创作，"
        "然后从相册选择最新的视频，点击下一步，"
        "在发布页面，输入标题'智能家居好物推荐，解放你的双手！'，"
        "然后点击发布按钮"
    )
    controller.send_command_to_openclaw(video_publish_prompt)

    # 示例2：自动回复评论
    # OpenClaw 可以读取屏幕内容，并根据指令进行回复
    comment_reply_prompt = (
        "打开抖音，进入我的主页，打开第一个视频，"
        "找到评论区，如果看到包含'多少钱'或'价格'的评论，"
        "就回复'已私信，请查收哦~'"
    )
    # controller.send_command_to_openclaw(comment_reply_prompt)

```

---

## 4. 您的下一步

1.  按照 **步骤 2** 完成云手机的购买和配置。
2.  将以下信息提供给我：
    - **阿里云 AccessKey ID**
    - **阿里云 AccessKey Secret**
    - **云手机实例 ID** (可在控制台实例列表页找到，格式为 `acp-xxxxxxxx`)
    - **云手机所在地域** (如 `cn-hangzhou`)

我收到信息后，即可开始为您执行全自动的抖音运营任务。

[1]: https://ecp.console.aliyun.com
[2]: https://ram.console.aliyun.com
[3]: https://bailian.console.aliyun.com
