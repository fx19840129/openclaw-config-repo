# baidu_search_wrapper 技能操作指南

## 描述
`baidu_search_wrapper` 是一个封装了百度千帆智能搜索生成接口的 OpenClaw 技能。它支持两种主要的搜索模式：高性能版（web_summary）和可指定模型的聊天版（chat/completions）。该技能旨在提供强大、灵活的百度搜索能力，并支持多种高级参数配置，如人设指令、深度搜索、时间筛选等。

## 核心功能
- **高性能搜索（web_summary）**: 快速获取搜索结果的摘要。
- **智能聊天搜索（chat/completions）**: 可指定 AI 模型（如 Ernie、DeepSeek 等），进行更精确、更深度的搜索，并支持深度思考、时间范围过滤等。
- **灵活的参数控制**: 支持 API Key、人设指令、超时设置、搜索引擎版本选择、深度搜索/思考开关、时间筛选等。
- **多格式输出**: 支持输出完整的 JSON 数据或格式化后的 Markdown 摘要与来源。

## 使用场景
- 需要快速获取特定问题的搜索摘要。
- 需要根据不同模型进行定制化、深度化的搜索。
- 需要对搜索结果进行格式化展示（Markdown）或进行程序化处理（JSON）。
- 需要集成百度搜索能力到其他自动化任务或技能中。

## 使用方法
通过 `sessions_spawn` 调用 `baidu_search_wrapper` 技能。

### 参数

-   **`query`** (字符串, **必填**):
    *   **描述**: 搜索问题或关键词。
    *   **注意**: 即使是位置参数，通过 `sessions_spawn` 调用时也需以关键字参数形式传递。

-   **`api`** (字符串, 可选, 默认: `web_summary`):
    *   **描述**: 指定使用的接口。
    *   **可选值**: `web_summary` (高性能版，响应快，不指定模型) 或 `chat` (智能搜索生成，需同时指定 `--model`)。

-   **`model`** (字符串, **仅当 `api` 为 `chat` 时必填**):
    *   **描述**: 指定 AI 模型名。
    *   **示例**: `ernie-4.5-turbo-32k`, `ernie-4.5-turbo-128k`, `deepseek-r1`, `deepseek-v3`。

-   **`api_key`** (字符串, 可选):
    *   **描述**: 百度千帆 API Key。如果不提供，将优先尝试读取环境变量 `QIANFAN_API_KEY`，否则使用脚本内默认 Key。

-   **`instruction`** (字符串, 可选):
    *   **描述**: 人设指令，用于约束模型输出风格与语气。

-   **`timeout`** (整数, 可选, 默认: 60):
    *   **描述**: 请求超时秒数。`web_summary` 模式下默认 60 秒，`chat` 模式下内部默认 120 秒。

-   **`json`** (布尔值, 可选, 默认: `False`):
    *   **描述**: 如果设置为 `True`，则输出完整的 JSON 格式结果；否则输出格式化后的 Markdown 摘要与来源。

-   **`search_source`** (字符串, 仅当 `api` 为 `chat` 时生效, 默认: `baidu_search_v2`):
    *   **描述**: 搜索引擎版本。
    *   **可选值**: `baidu_search_v1` 或 `baidu_search_v2` (推荐使用 v2)。

-   **`deep_search`** (布尔值, 仅当 `api` 为 `chat` 时生效, 默认: `False`):
    *   **描述**: 如果设置为 `True`，则开启深度搜索（可能会返回更多结果，调用次数可能增加）。

-   **`no_reasoning`** (布尔值, 仅当 `api` 为 `chat` 时生效, 默认: `False`):
    *   **描述**: 如果设置为 `True`，则关闭深度思考（对 DeepSeek-R1、文心 X1 等模型生效）。

-   **`recency`** (字符串, 仅当 `api` 为 `chat` 时生效, 可选):
    *   **描述**: 仅检索指定时间范围内的网页。
    *   **可选值**: `week`, `month`, `semiyear`, `year`。

### 使用示例

#### 1. 使用高性能版搜索（默认）

```python
sessions_spawn(
    skill="baidu_search_wrapper",
    params={
        "query": "2024年全球经济增长预测"
    }
)
```

#### 2. 使用高性能版搜索，并输出 JSON 格式

```python
sessions_spawn(
    skill="baidu_search_wrapper",
    params={
        "query": "OpenClaw 最新功能",
        "json": True
    }
)
```

#### 3. 使用聊天版搜索，指定模型和深度搜索

```python
sessions_spawn(
    skill="baidu_search_wrapper",
    params={
        "query": "Python 异步编程的最佳实践",
        "api": "chat",
        "model": "deepseek-r1",
        "deep_search": True,
        "recency": "year"
    }
)
```

#### 4. 使用聊天版搜索，带人设指令

```python
sessions_spawn(
    skill="baidu_search_wrapper",
    params={
        "query": "讲一个关于编程的笑话",
        "api": "chat",
        "model": "ernie-4.5-turbo-32k",
        "instruction": "你是一个幽默风趣的程序员，喜欢用梗和段子来解释技术问题。"
    }
)
```

## 注意事项
-   **API Key**: 确保 `QIANFAN_API_KEY` 环境变量已设置，或在调用时通过 `api_key` 参数传入有效的百度千帆 API Key。
-   **模型选择**: 在 `chat` 模式下，务必选择一个有效的模型名称。
-   **错误处理**: 技能会返回操作结果的字符串，如果出现错误，会包含详细的错误信息。请注意检查返回结果。
-   **调用限制**: 请注意百度千帆接口的调用频率和配额限制。
