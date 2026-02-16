#### baidu_search (百度千帆智能搜索)

- **描述**: 直接调用 `/Users/fengxiao/.openclaw/workspace/projects/fox-skills/baidu_search_wrapper/main.py` 脚本，通过百度千帆 AI Search API 进行智能搜索。支持多种搜索模式、模型指定及丰富参数，提供 Markdown 或 JSON 格式结果。**现在进行网络搜索时，优先使用此脚本。**
- **使用方法**: 通过 `exec` 工具直接运行 Python 脚本，并传入命令行参数。
- **参数说明**:
    - `query`: 位置参数，搜索问题。不传且非交互终端时从 stdin 读一行。
    - `--api {web_summary|chat}`: 默认 `chat`；用 `chat` 时要同时加 `--model`。
    - `--model MODEL`: 仅 `--api chat` 时必填，如 `ernie-4.5-turbo-32k`、`deepseek-r1`。
    - `--api-key KEY`: API Key，不传则用环境变量或脚本默认。
    - `--instruction TEXT`: 人设指令，约束输出风格。
    - `--timeout N`: 超时秒数，默认 60；chat 时内部会按 120 处理。
    - `--json`: 输出完整 JSON；不传则输出 Markdown 摘要+来源。
    - `--search-source`: 仅 chat：`baidu_search_v1` / `baidu_search_v2`（默认 `v2`）。
    - `--deep-search`: 仅 chat：开启深度搜索。
    - `--no-reasoning`: 仅 chat：关闭深度思考。
    - `--recency {week|month|semiyear|year}`: 仅 chat：只搜这段时间内的网页。
- **使用示例**:
  ```bash
  # 高性能版，直接问一个问题
  python3 /Users/fengxiao/.openclaw/workspace/projects/fox-skills/baidu_search_wrapper/main.py "北京有哪些必去景点"

  # 高性能版，输出 JSON
  python3 /Users/fengxiao/.openclaw/workspace/projects/fox-skills/baidu_search_wrapper/main.py "今天北京天气" --json

  # 智能搜索生成，指定模型
  python3 /Users/fengxiao/.openclaw/workspace/projects/fox-skills/baidu_search_wrapper/main.py "近日油价调整" --api chat --model ernie-4.5-turbo-32k

  # 深度搜索 + 最近一个月网页
  python3 /Users/fengxiao/.openclaw/workspace/projects/fox-skills/baidu_search_wrapper/main.py "Python 3.12 新特性" --api chat --model ernie-4.5-turbo-32k --deep-search --recency month

  # 从 stdin 读问题
  echo "北戴河景区介绍" | python3 /Users/fengxiao/.openclaw/workspace/projects/fox-skills/baidu_search_wrapper/main.py --json
  ```