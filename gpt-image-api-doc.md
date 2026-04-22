# GPT Image API 接口文档整理

> 整理时间：2026-04-22  
> 说明：原 GrsAI GPT Image 文档页部分内容为前端动态渲染，静态抓取可见内容主要包括标题、节点区、两个端点、在线生成器和少量比例选项。以下内容结合公开页面、GrsAI 开源 ComfyUI 客户端与公开示例整理；若线上控制台返回字段与本文不同，以最新官方文档和实际 API 响应为准。

---

## 1. 基础信息

### 1.1 服务节点

| 节点 | Base URL |
|---|---|
| 海外节点 | `https://api.grsai.com` / `https://grsaiapi.com` / `https://grsai.com` |
| 国内直连节点 | `https://grsai.dakka.com.cn` |

### 1.2 调用方式

完整接口地址 = `Base URL + Path`

示例：

```text
https://grsai.dakka.com.cn/v1/draw/completions
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

---

## 3. 接口列表

| 名称 | Method | Path | 说明 |
|---|---|---|---|
| 图片生成 | POST | `/v1/draw/completions` | 提交 GPT Image 文生图 / 图生图 / 图片编辑任务 |
| 获取结果接口 | POST | `/v1/draw/result` | 通过任务 ID 查询异步任务结果 |

---

# 4. 图片生成接口

## 4.1 Endpoint

```http
POST /v1/draw/completions
```

完整 URL 示例：

```text
https://grsai.dakka.com.cn/v1/draw/completions
```

---

## 4.2 请求参数

### Body JSON

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---:|---:|---|---|
| `model` | string | 是 | - | 使用的 GPT Image 模型 |
| `prompt` | string | 是 | - | 文生图、编辑或改图指令 |
| `urls` | string[] | 否 | `[]` | 输入图 / 参考图 URL 列表；不传或空数组时为文生图，传入图片时为图生图/图片编辑 |
| `size` | string | 否 | `auto` | 输出比例/尺寸选项；GrsAI GPT Image 客户端使用该字段传递比例 |
| `variants` | integer | 否 | - | 一次请求返回的变体数量；开源客户端预留该字段 |
| `shutProgress` | boolean | 否 | `true` | 是否关闭/隐藏进度，倾向于只返回最终结果 |
| `cdn` | string | 否 | `zh` | CDN / 区域标识；国内直连示例中常用 `zh` |
| `webHook` | string | 否 | - | 回调地址；如使用 `"-1"`，一般表示返回任务 ID 后自行轮询结果接口 |

> 注意：GrsAI GPT Image 页面在线生成器中比例选项显示为 `auto`、`2:3`、`3:2`、`1:1`。开源客户端也使用 `size` 字段传这些值，而不是 `aspectRatio`。

---

## 4.3 参数枚举值

### 4.3.1 `model`

| 值 | 说明 |
|---|---|
| `sora-image` | GrsAI GPT Image / Sora Image 图像模型 |
| `gpt-image-1.5` | GPT Image 1.5 模型 |

### 4.3.2 `size`

| 值 | 说明 |
|---|---|
| `auto` | 自动比例 |
| `1:1` | 正方形 |
| `2:3` | 竖版 |
| `3:2` | 横版 |

### 4.3.3 `variants`

| 值 | 说明 |
|---:|---|
| `1` | 返回 1 个变体 |
| `2` | 返回 2 个变体 |
| `3` - `12` | 开源 ComfyUI 节点中 `num_images` 支持 1-12，但实现方式是并发多次请求，不一定等价于 API 的 `variants` 单请求批量参数 |

> 实际 API 是否接受 `variants: 3-12` 需以接口响应为准。开源 API 客户端只预留 `variants` 字段；ComfyUI 节点实际通过多次请求实现批量。

### 4.3.4 `urls`

| 数量 | 说明 |
|---:|---|
| `0` | 文生图 |
| `1` | 单图编辑 / 图生图 |
| `2` - `6` | 多图参考 / 多图编辑；开源 ComfyUI 节点支持上传 1-6 张参考图 |

### 4.3.5 `shutProgress`

| 值 | 说明 |
|---|---|
| `true` | 关闭/隐藏进度，倾向于直接返回最终结果 |
| `false` | 允许返回进度或流式进度 |

### 4.3.6 `cdn`

| 值 | 说明 |
|---|---|
| `zh` | 国内/中文区 CDN 标识，开源客户端默认使用 |

### 4.3.7 `webHook`

| 值 | 说明 |
|---|---|
| `"-1"` | 异步模式：返回任务 ID，然后调用 `/v1/draw/result` 查询 |
| URL 字符串 | 任务完成后回调该地址 |

### 4.3.8 响应状态 `status`

| 值 | 说明 |
|---|---|
| `succeeded` | 任务成功 |
| `failed` | 任务失败 |
| `processing` | 处理中 |
| `pending` | 等待处理 |
| `running` | 运行中 |

> 开源客户端对同步成功结果的判断是 `status == "succeeded"`；其它状态在轮询/异步场景中可能出现。

---

## 4.4 文生图请求示例

### cURL

```bash
curl --location --request POST 'https://grsai.dakka.com.cn/v1/draw/completions' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --data-raw '{
    "model": "gpt-image-1.5",
    "prompt": "生成一张高质量产品海报，白色背景，柔和棚拍光，画面中有一只透明玻璃香水瓶，瓶身印有简洁中文品牌名。",
    "size": "1:1",
    "shutProgress": true,
    "cdn": "zh"
  }'
```

### Python

```python
import requests

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://grsai.dakka.com.cn"

url = f"{BASE_URL}/v1/draw/completions"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}",
}

payload = {
    "model": "gpt-image-1.5",
    "prompt": "生成一张高质量产品海报，白色背景，柔和棚拍光，画面中有一只透明玻璃香水瓶，瓶身印有简洁中文品牌名。",
    "size": "1:1",
    "shutProgress": True,
    "cdn": "zh",
}

response = requests.post(url, json=payload, headers=headers, timeout=300)
print(response.json())
```

### JavaScript

```js
const API_KEY = "YOUR_API_KEY";
const BASE_URL = "https://grsai.dakka.com.cn";

async function generateImage() {
  const response = await fetch(`${BASE_URL}/v1/draw/completions`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${API_KEY}`,
    },
    body: JSON.stringify({
      model: "gpt-image-1.5",
      prompt: "生成一张高质量产品海报，白色背景，柔和棚拍光，画面中有一只透明玻璃香水瓶，瓶身印有简洁中文品牌名。",
      size: "1:1",
      shutProgress: true,
      cdn: "zh",
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
  "model": "gpt-image-1.5",
  "prompt": "保持人物脸部和服装一致，把背景改成高级时装摄影棚，柔和主光，浅灰色无缝背景，商业大片质感。",
  "urls": [
    "https://example.com/input-1.png"
  ],
  "size": "2:3",
  "shutProgress": true,
  "cdn": "zh"
}
```

---

## 4.6 多图参考请求示例

```json
{
  "model": "gpt-image-1.5",
  "prompt": "参考第一张人物脸部，第二张服装，第三张场景，生成一张统一风格的商业海报，保持人物身份一致。",
  "urls": [
    "https://example.com/ref-face.png",
    "https://example.com/ref-clothes.png",
    "https://example.com/ref-scene.png"
  ],
  "size": "1:1",
  "shutProgress": true,
  "cdn": "zh"
}
```

---

## 4.7 批量/多变体请求示例

### 单请求变体字段示例

```json
{
  "model": "gpt-image-1.5",
  "prompt": "生成两张不同构图的高端咖啡品牌海报，现代简约风格。",
  "size": "3:2",
  "variants": 2,
  "shutProgress": true,
  "cdn": "zh"
}
```

### 并发多次请求示例

开源 ComfyUI 客户端中的 `num_images` 更接近“并发发起多次请求”，而不是在单个 payload 中传入 `num_images`。如果你要稳定批量生产，建议用多次请求或异步任务池。

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://grsai.dakka.com.cn"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}",
}

payload = {
    "model": "gpt-image-1.5",
    "prompt": "生成一张电商主图，白底，产品居中，真实摄影。",
    "size": "1:1",
    "shutProgress": True,
    "cdn": "zh",
}

def submit_one():
    res = requests.post(
        f"{BASE_URL}/v1/draw/completions",
        headers=headers,
        json=payload,
        timeout=300,
    )
    res.raise_for_status()
    return res.json()

with ThreadPoolExecutor(max_workers=4) as pool:
    futures = [pool.submit(submit_one) for _ in range(4)]
    for f in as_completed(futures):
        print(f.result())
```

---

## 4.8 异步任务请求示例

如果接口支持 `webHook: "-1"`，可以先拿任务 ID，再调用结果接口轮询。

```json
{
  "model": "gpt-image-1.5",
  "prompt": "A premium perfume product poster, clean studio lighting, realistic glass material.",
  "size": "1:1",
  "webHook": "-1",
  "cdn": "zh"
}
```

---

## 4.9 Webhook 回调请求示例

```json
{
  "model": "gpt-image-1.5",
  "prompt": "A futuristic city at sunset, cinematic lighting, highly detailed.",
  "size": "3:2",
  "webHook": "https://your-server.com/callback",
  "cdn": "zh"
}
```

---

## 4.10 成功响应示例：同步 / 最终结果

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

多结果示例：

```json
{
  "id": "task_id",
  "status": "succeeded",
  "progress": 100,
  "results": [
    {
      "url": "https://example.com/generated-image-1.png"
    },
    {
      "url": "https://example.com/generated-image-2.png"
    }
  ]
}
```

---

## 4.11 异步响应示例

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

## 4.12 失败响应示例

```json
{
  "id": "task_id",
  "status": "failed",
  "error": "generation failed",
  "failure_reason": "generation failed"
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
| `id` | string | 是 | 图片生成接口返回的任务 ID |

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

def poll_result(task_id, max_attempts=60, interval=5):
    url = f"{BASE_URL}/v1/draw/result"

    for _ in range(max_attempts):
        response = requests.post(url, json={"id": task_id}, headers=headers, timeout=60)
        response.raise_for_status()
        result = response.json()

        # 兼容 data 包裹与顶层字段两种结构
        data = result.get("data", result)

        status = data.get("status")
        if status == "succeeded":
            return data

        if status == "failed":
            raise RuntimeError(f"Task failed: {data}")

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
    "error": "generation failed",
    "failure_reason": "generation failed"
  }
}
```

---

# 6. 完整枚举汇总

## 6.1 模型枚举

```text
sora-image
gpt-image-1.5
```

## 6.2 尺寸 / 比例枚举

```text
auto
1:1
2:3
3:2
```

## 6.3 参考图数量枚举

```text
0 张：文生图
1 张：图生图 / 单图编辑
2-6 张：多图参考 / 多图编辑
```

## 6.4 单请求变体数枚举

```text
1
2
```

> `3-12` 更适合通过并发多次请求实现，不建议直接假设 API 单请求支持。

## 6.5 布尔枚举

```text
true
false
```

## 6.6 任务状态枚举

```text
pending
processing
running
succeeded
failed
```

---

# 7. 最小可用调用模板

## 7.1 同步/直接获取结果

```python
import requests

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://grsai.dakka.com.cn"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}",
}

payload = {
    "model": "gpt-image-1.5",
    "prompt": "生成一张极简风格咖啡品牌海报，产品居中，真实摄影，中文标题清晰。",
    "size": "1:1",
    "shutProgress": True,
    "cdn": "zh",
}

res = requests.post(
    f"{BASE_URL}/v1/draw/completions",
    headers=headers,
    json=payload,
    timeout=300,
)

print(res.json())
```

---

## 7.2 异步轮询

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
    "model": "gpt-image-1.5",
    "prompt": "生成一张极简风格咖啡品牌海报，产品居中，真实摄影，中文标题清晰。",
    "size": "1:1",
    "webHook": "-1",
    "cdn": "zh",
}

submit_res = requests.post(
    f"{BASE_URL}/v1/draw/completions",
    headers=headers,
    json=submit_payload,
    timeout=300,
).json()

task_id = submit_res.get("id") or submit_res.get("data", {}).get("id")
print("task_id:", task_id)

while True:
    result_res = requests.post(
        f"{BASE_URL}/v1/draw/result",
        headers=headers,
        json={"id": task_id},
        timeout=60,
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

# 8. 注意事项

1. GrsAI GPT Image 页面可见端点为 `/v1/draw/completions` 和 `/v1/draw/result`。
2. GPT Image 客户端中比例字段使用 `size`，支持 `auto` / `1:1` / `2:3` / `3:2`。
3. GPT Image 模型枚举可用 `sora-image` / `gpt-image-1.5`。
4. `urls` 为空时为文生图；传入图片 URL 时用于图生图、改图、多图参考。
5. 开源 ComfyUI 节点支持最多 6 张输入图。
6. 开源 ComfyUI 节点的 `num_images` 是并发多次请求，不是明确的 API Body 字段；如需批量生成，建议业务端实现队列/并发池。
7. 返回结果通常使用 `results[].url`；也建议兼容顶层 `url` 或 `data.results[].url`。
8. 返回图片 URL 可能有有效期限制，生成完成后建议及时下载归档。
9. 如果接口返回 SSE 风格文本，例如 `data: {...}`，客户端需去掉 `data: ` 前缀再解析 JSON。
10. 如果接口返回“不支持模型/参数”，以控制台最新模型列表或官方页面为准。

