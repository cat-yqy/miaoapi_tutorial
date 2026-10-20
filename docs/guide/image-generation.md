# 使用 GPT-Image-2 生成图片

首先需要安装 Python；如果不会，可以让 Codex 协助。

!!! warning "密钥分组"
    需要在 **GPT-Image-2** 分组下创建密钥，否则调用会失败。

## 示例代码

以下代码通过 MiaoAPI 的 Responses 接口调用图片生成工具，并保存中间图和最终图。将代码保存为 `generate.py`。

```python
import base64
import json
import os
from pathlib import Path

import httpx

BASE_URL = os.getenv("SUB2API_BASE_URL", "https://miaoapi.site/v1").rstrip("/")
API_KEY = os.environ["SUB2API_KEY"]
MODEL = os.getenv("SUB2API_MODEL", "gpt-5.5")
PROMPT = os.getenv("IMAGE_PROMPT", "机械龙，暗黑幻想风格")
OUTPUT_PATH = Path(os.getenv("OUTPUT_PATH", "dragon.png"))


def iter_sse_json_lines(response):
    for line in response.iter_lines():
        if not line or not line.startswith("data: "):
            continue

        data = line.removeprefix("data: ").strip()
        if data == "[DONE]":
            break

        try:
            yield json.loads(data)
        except json.JSONDecodeError:
            print(f"跳过非 JSON 事件: {data[:200]}")


def main():
    payload = {
        "model": MODEL,
        "input": PROMPT,
        "tools": [
            {
                "type": "image_generation",
                "partial_images": 3,
            }
        ],
        "stream": True,
    }

    last_image_b64 = None

    with httpx.stream(
        "POST",
        f"{BASE_URL}/responses",
        headers={
            "Authorization": f"Bearer {API_KEY}",
            "Content-Type": "application/json",
        },
        json=payload,
        timeout=300,
    ) as response:
        response.raise_for_status()

        for event in iter_sse_json_lines(response):
            event_type = event.get("type")

            if event_type == "response.image_generation_call.partial_image":
                index = event.get("partial_image_index", 0)
                last_image_b64 = event["partial_image_b64"]
                partial_path = OUTPUT_PATH.with_name(
                    f"{OUTPUT_PATH.stem}_partial_{index}{OUTPUT_PATH.suffix}"
                )
                partial_path.write_bytes(base64.b64decode(last_image_b64))
                print(f"保存中间图: {partial_path}")
            elif event_type == "response.completed":
                print("生成完成")
            elif event_type == "response.failed":
                raise RuntimeError(json.dumps(event, ensure_ascii=False))

    if not last_image_b64:
        raise RuntimeError("没有从 Sub2API stream 中拿到 partial_image_b64")

    OUTPUT_PATH.write_bytes(base64.b64decode(last_image_b64))
    print(f"保存最终图片: {OUTPUT_PATH}")


if __name__ == "__main__":
    main()

```

## 运行

先安装依赖：

```bash
pip install httpx
```

然后设置环境变量并运行：

```bash
export SUB2API_KEY="你的 sub2api key"
export IMAGE_PROMPT="你的生图 prompt"
export OUTPUT_PATH="test.png"
export SUB2API_MODEL="gpt-5.5"
python generate.py
```
