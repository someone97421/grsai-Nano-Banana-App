# Nano Banana API 接口文档整理

> 整理时间：2026-04-22  
> 说明：原页面部分内容为前端动态渲染，静态抓取只能看到文档目录、端点名称和交互区。以下内容按公开页面可见端点、Apifox 页面、官方/社区示例与开源客户端枚举交叉整理。

---

## 1. 基础信息

### 1.1 服务节点

| 节点 | Base URL |
|---|---|
| 海外节点 | `https://api.grsai.com` / `https://grsaiapi.com` |
| 国内直连节点 | `https://grsai.dakka.com.cn` |

### 1.2 调用方式

完整接口地址 = `Base URL + Path`

示例：

```text
https://grsai.dakka.com.cn/v1/draw/nano-banana
https://grsai.dakka.com.cn/v1/draw/result
```

---

## 2. 认证方式

### Header

| 参数 | 位置 | 类型 | 必填 | 说明 |
|---|---|---:|---:|---|
| `Authorization` | Header | string | 是 | Bearer Token 认证 |
| `Content-Type` | Header | string | 是 | `application/json` |

示例：

```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

> Apifox 页面中也出现过 `token: <api-key>` 的 header 示例，但实际代码示例和 GrsAI 常规调用方式均使用 `Authorization: Bearer YOUR_API_KEY`。

---

## 3. 接口列表

| 名称 | Method | Path | 说明 |
|---|---|---|---|
| Nano Banana 绘画接口 | POST | `/v1/draw/nano-banana` | 提交文生图、图生图、图片编辑/融合任务 |
| 获取结果接口 | POST | `/v1/draw/result` | 通过任务 ID 查询异步任务结果 |

---

# 4. Nano Banana 绘画接口

## 4.1 Endpoint

```http
POST /v1/draw/nano-banana
```

完整 URL 示例：

```text
https://grsai.dakka.com.cn/v1/draw/nano-banana
```

---

## 4.2 请求参数

### Body JSON

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---:|---:|---|---|
| `model` | string | 是 | - | 使用的 Nano Banana 模型 |
| `prompt` | string | 是 | - | 文生图、编辑或融合描述 |
| `urls` | string[] | 否 | `[]` | 参考图 / 输入图 URL 列表；不传或空数组时为文生图，传入图片时为图生图/图片编辑/多图融合 |
| `aspectRatio` | string | 否 | `auto` | 输出宽高比 |
| `imageSize` | string | 否 | `1K` | 输出尺寸；主要用于 Pro / Pro VT / Pro CL / Banana 2 等支持尺寸控制的模型 |
| `webHook` | string | 否 | - | 回调地址；传 `"-1"` 表示立即返回任务 ID，然后使用结果接口轮询 |
| `shutProgress` | boolean | 否 | - | 是否关闭/隐藏流式进度；部分示例中用于只获取最终结果 |
| `cdn` | string | 否 | - | CDN 区域标识；部分客户端示例中使用 `zh` |

---

## 4.3 参数枚举值

### 4.3.1 `model`

| 值 | 说明 |
|---|---|
| `nano-banana-fast` | 快速版 / Banana 1 常用模型 |
| `nano-banana` | 标准版 |
| `nano-banana-pro` | Pro 版本，支持 `imageSize` |
| `nano-banana-pro-vt` | Pro VT 版本，支持 `imageSize` |
| `nano-banana-pro-cl` | Pro CL 渠道，支持 `imageSize` |
| `nano-banana-2` | Banana 2 版本，支持 `imageSize` |
| `nano-banana-2-cl` | Banana 2 CL 渠道 |
| `nano-banana-2-cl-4k` | Banana 2 CL 4K 渠道 |

> 注意：不同页面/客户端版本的模型列表可能略有差异。若接口返回“不支持模型”，以控制台“模型列表”或最新官方文档为准。

### 4.3.2 `aspectRatio`

完整枚举：

| 值 |
|---|
| `auto` |
| `1:1` |
| `16:9` |
| `9:16` |
| `4:3` |
| `3:4` |
| `3:2` |
| `2:3` |
| `5:4` |
| `4:5` |
| `21:9` |

### 4.3.3 `imageSize`

完整枚举：

| 值 | 说明 |
|---|---|
| `1K` | 1K 输出 |
| `2K` | 2K 输出 |
| `4K` | 4K 输出 |

### 4.3.4 `webHook`

| 值 | 说明 |
|---|---|
| `"-1"` | 异步模式：立即返回任务 ID，然后调用 `/v1/draw/result` 查询 |
| URL 字符串 | 任务完成后回调该地址 |

### 4.3.5 `shutProgress`

| 值 | 说明 |
|---|---|
| `true` | 关闭/隐藏进度，倾向于只返回最终结果 |
| `false` | 返回进度或流式进度 |

### 4.3.6 `status` 响应状态

| 值 | 说明 |
|---|---|
| `succeeded` | 任务成功 |
| `failed` | 任务失败 |
| `processing` | 处理中 |
| `pending` | 等待处理 |
| `running` | 运行中 |

> `processing` / `pending` / `running` 为常见任务状态归纳；实际返回以接口为准。

---

## 4.4 文生图请求示例

### cURL

```bash
curl --location --request POST 'https://grsai.dakka.com.cn/v1/draw/nano-banana' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --data-raw '{
    "model": "nano-banana-pro",
    "prompt": "生成一张猫咪海报，猫咪欢乐派对，仿佛要突破海报进入3D世界",
    "aspectRatio": "1:1",
    "imageSize": "1K",
    "shutProgress": false
  }'
```

### Python

```python
import requests

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://grsai.dakka.com.cn"

url = f"{BASE_URL}/v1/draw/nano-banana"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}",
}

payload = {
    "model": "nano-banana-pro",
    "prompt": "生成一张猫咪海报，猫咪欢乐派对，仿佛要突破海报进入3D世界",
    "aspectRatio": "1:1",
    "imageSize": "1K",
    "shutProgress": False,
}

response = requests.post(url, json=payload, headers=headers)
print(response.json())
```

### JavaScript

```js
const API_KEY = "YOUR_API_KEY";
const BASE_URL = "https://grsai.dakka.com.cn";

async function generateImage() {
  const response = await fetch(`${BASE_URL}/v1/draw/nano-banana`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${API_KEY}`,
    },
    body: JSON.stringify({
      model: "nano-banana-pro",
      prompt: "生成一张猫咪海报，猫咪欢乐派对，仿佛要突破海报进入3D世界",
      aspectRatio: "1:1",
      imageSize: "1K",
      shutProgress: false,
    }),
  });

  return await response.json();
}

generateImage().then(console.log);
```

---

## 4.5 图生图 / 图片编辑请求示例

```json
{
  "model": "nano-banana-pro",
  "prompt": "保持人物一致性，把场景改成赛博朋克街头，霓虹灯，高级电影感",
  "urls": [
    "https://example.com/input-1.png"
  ],
  "aspectRatio": "1:1",
  "imageSize": "2K",
  "shutProgress": false
}
```

---

## 4.6 多图融合请求示例

```json
{
  "model": "nano-banana-pro",
  "prompt": "融合多张参考图，保持主体脸部一致，生成电商主视觉海报，高级摄影棚布光",
  "urls": [
    "https://example.com/ref-1.png",
    "https://example.com/ref-2.png",
    "https://example.com/ref-3.png"
  ],
  "aspectRatio": "4:5",
  "imageSize": "2K"
}
```

---

## 4.7 异步任务请求示例

当 `webHook` 设置为 `"-1"` 时，接口通常会立即返回任务 ID，需要再调用结果接口查询。

```json
{
  "model": "nano-banana-pro",
  "prompt": "A cute cat poster, joyful party, 3D breakthrough effect",
  "aspectRatio": "1:1",
  "imageSize": "1K",
  "webHook": "-1"
}
```

---

## 4.8 Webhook 回调请求示例

```json
{
  "model": "nano-banana-pro",
  "prompt": "A futuristic city at sunset, cinematic lighting",
  "aspectRatio": "16:9",
  "imageSize": "2K",
  "webHook": "https://your-server.com/callback"
}
```

---

## 4.9 成功响应示例：同步 / 最终结果

```json
{
  "id": "task_id",
  "status": "succeeded",
  "progress": 100,
  "results": [
    {
      "url": "https://example.com/generated-image.png"
    }
  ]
}
```

也可能返回单图 URL 结构：

```json
{
  "id": "task_id",
  "status": "succeeded",
  "progress": 100,
  "url": "https://example.com/generated-image.png"
}
```

---

## 4.10 异步响应示例

```json
{
  "data": {
    "id": "task_id"
  }
}
```

或：

```json
{
  "id": "task_id",
  "status": "pending"
}
```

---

## 4.11 失败响应示例

```json
{
  "id": "task_id",
  "status": "failed",
  "error": "generation failed"
}
```

---

# 5. 获取结果接口

## 5.1 Endpoint

```http
POST /v1/draw/result
```

完整 URL 示例：

```text
https://grsai.dakka.com.cn/v1/draw/result
```

---

## 5.2 请求参数

### Body JSON

| 参数 | 类型 | 必填 | 说明 |
|---|---:|---:|---|
| `id` | string | 是 | 绘画接口返回的任务 ID |

---

## 5.3 请求示例

### cURL

```bash
curl --location --request POST 'https://grsai.dakka.com.cn/v1/draw/result' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --data-raw '{
    "id": "task_id"
  }'
```

### Python

```python
import requests
import time

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://grsai.dakka.com.cn"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}",
}

def poll_result(task_id, max_attempts=12, interval=5):
    url = f"{BASE_URL}/v1/draw/result"

    for _ in range(max_attempts):
        response = requests.post(url, json={"id": task_id}, headers=headers)
        data = response.json()

        # 有些返回结构会包在 data 里
        payload = data.get("data", data)

        status = payload.get("status")
        if status == "succeeded":
            return payload
        if status == "failed":
            raise RuntimeError(f"Task failed: {payload}")

        time.sleep(interval)

    raise TimeoutError("Polling timeout")

result = poll_result("task_id")
print(result)
```

---

## 5.4 成功响应示例

```json
{
  "data": {
    "id": "task_id",
    "status": "succeeded",
    "progress": 100,
    "results": [
      {
        "url": "https://example.com/generated-image.png"
      }
    ]
  }
}
```

或：

```json
{
  "id": "task_id",
  "status": "succeeded",
  "progress": 100,
  "results": [
    {
      "url": "https://example.com/generated-image.png"
    }
  ]
}
```

---

## 5.5 处理中响应示例

```json
{
  "data": {
    "id": "task_id",
    "status": "processing",
    "progress": 45
  }
}
```

---

## 5.6 失败响应示例

```json
{
  "data": {
    "id": "task_id",
    "status": "failed",
    "error": "generation failed"
  }
}
```

---

# 6. Gemini 官方接口格式兼容

文档页显示“支持 Gemini 官方接口格式”。公开示例中常见调用方式是将 Gemini 图像模型接口替换为 GrsAI Base URL，并把模型名替换为 Nano Banana 模型名。

示例 URL：

```text
https://grsai.dakka.com.cn/v1beta/models/nano-banana-fast:streamGenerateContent
```

常见替换：

| Gemini 原模型 / 调用习惯 | GrsAI 替换 |
|---|---|
| `gemini-2.5-flash-image` | `nano-banana-fast` |
| Gemini Base URL | GrsAI Base URL |

---

# 7. 完整枚举汇总

## 7.1 模型枚举

```text
nano-banana-fast
nano-banana
nano-banana-pro
nano-banana-pro-vt
nano-banana-pro-cl
nano-banana-2
nano-banana-2-cl
nano-banana-2-cl-4k
```

## 7.2 宽高比枚举

```text
auto
1:1
16:9
9:16
4:3
3:4
3:2
2:3
5:4
4:5
21:9
```

## 7.3 图片尺寸枚举

```text
1K
2K
4K
```

## 7.4 布尔枚举

```text
true
false
```

## 7.5 任务状态枚举

```text
pending
processing
running
succeeded
failed
```

---

# 8. 最小可用调用模板

## 8.1 同步/直接获取结果

```python
import requests

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://grsai.dakka.com.cn"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}",
}

payload = {
    "model": "nano-banana-pro",
    "prompt": "生成一张高质量猫咪海报，3D突破画面效果",
    "aspectRatio": "1:1",
    "imageSize": "1K",
    "shutProgress": True,
}

res = requests.post(
    f"{BASE_URL}/v1/draw/nano-banana",
    headers=headers,
    json=payload,
)

print(res.json())
```

## 8.2 异步轮询

```python
import requests
import time

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://grsai.dakka.com.cn"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}",
}

submit_payload = {
    "model": "nano-banana-pro",
    "prompt": "生成一张高质量猫咪海报，3D突破画面效果",
    "aspectRatio": "1:1",
    "imageSize": "1K",
    "webHook": "-1",
}

submit_res = requests.post(
    f"{BASE_URL}/v1/draw/nano-banana",
    headers=headers,
    json=submit_payload,
).json()

task_id = submit_res.get("id") or submit_res.get("data", {}).get("id")
print("task_id:", task_id)

while True:
    result_res = requests.post(
        f"{BASE_URL}/v1/draw/result",
        headers=headers,
        json={"id": task_id},
    ).json()

    data = result_res.get("data", result_res)
    status = data.get("status")
    progress = data.get("progress")

    print("status:", status, "progress:", progress)

    if status == "succeeded":
        print("result:", data)
        break

    if status == "failed":
        raise RuntimeError(data)

    time.sleep(5)
```

---

# 9. 注意事项

1. `imageSize` 主要对 Pro / Pro VT / Pro CL / Banana 2 等模型生效；普通模型可能忽略该参数。
2. `urls` 为空或不传时通常是文生图；传入图片 URL 时用于图生图、编辑或融合。
3. 返回图片 URL 可能有有效期限制，建议生成后及时下载保存。
4. 如果使用 `webHook: "-1"`，需要保存返回的任务 ID 并调用 `/v1/draw/result` 轮询。
5. 如果返回结构包在 `data` 字段中，客户端应兼容 `response["data"]` 和顶层字段两种结构。
6. 如果接口返回 `data: {...}` 形式的 SSE 文本，需要先去掉 `data: ` 前缀再解析 JSON。
