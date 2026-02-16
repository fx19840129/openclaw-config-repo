# markdown_writer 技能操作指南

## 描述
`markdown_writer` 是一个通用的 Markdown 文件写入工具，支持新建、覆盖、追加内容，并可灵活添加标题。它直接调用 OpenClaw 注入的 `read` 和 `write` 工具进行文件操作，确保了稳定性和可靠性。

## 使用场景
- 需要创建新的 Markdown 文档时。
- 需要更新或覆盖现有 Markdown 文档内容时。
- 需要向现有 Markdown 文档追加内容时。
- 需要标准化 Markdown 文档的标题格式时。

## 使用方法
通过 `sessions_spawn` 调用 `markdown_writer` 技能。

### 参数

-   **`file_path`** (字符串, **必填**):
    *   **描述**: 目标 Markdown 文件的绝对路径。例如：`/Users/fengxiao/.openclaw/workspace/my_new_doc.md`。
    *   **重要提示**: 务必使用绝对路径，以避免文件创建或查找错误。

-   **`content`** (字符串, **必填**):
    *   **描述**: 要写入 Markdown 文件的内容。可以包含 Markdown 格式的文本。

-   **`title`** (字符串, 可选):
    *   **描述**: 如果提供，将在文件顶部添加一个 H1 级别的标题（例如 `# 我的标题`）。
    *   **默认值**: `""` (空字符串)。
    *   **注意**:
        *   当 `append` 为 `False` (覆盖模式) 且 `title` 不为空时，`title` 将作为文件内容的起始标题。
        *   当 `append` 为 `True` (追加模式) 时，如果文件原本是空的，`title` 会生效；如果文件已经有内容，`title` 会被忽略，以保持追加内容的连贯性。

-   **`append`** (布尔值, 可选):
    *   **描述**: 控制文件写入模式。
    *   **`True`**: 将 `content` 追加到现有文件内容的末尾。如果文件不存在，则创建新文件并写入 `content` (此时 `title` 如果提供且文件为空，也会生效)。
    *   **`False`**: (默认值) 覆盖现有文件内容。如果文件不存在，则创建新文件。

### 使用示例

#### 1. 创建一个新的 Markdown 文件并添加标题

```python
sessions_spawn(
    skill="markdown_writer",
    params={
        "file_path": "/Users/fengxiao/.openclaw/workspace/projects/fox-skills/test_docs/new_report.md",
        "title": "2026年2月12日市场分析报告",
        "content": "### 今日大盘\n今日沪深两市震荡上行，创业板指表现强势...\n\n### 热点板块\n人工智能、新能源汽车概念股领涨..."
    }
)
```

#### 2. 向现有 Markdown 文件追加内容

```python
sessions_spawn(
    skill="markdown_writer",
    params={
        "file_path": "/Users/fengxiao/.openclaw/workspace/projects/fox-skills/test_docs/new_report.md",
        "content": "\n## 后市展望\n预计市场短期内将保持震荡，关注后续政策导向...",
        "append": True
    }
)
```

#### 3. 覆盖现有 Markdown 文件（不带标题）

```python
sessions_spawn(
    skill="markdown_writer",
    params={
        "file_path": "/Users/fengxiao/openclaw/workspace/projects/fox-skills/test_docs/new_report.md",
        "content": "这是一个全新的文件内容，覆盖了之前的所有内容。"
    }
)
```

## 注意事项
-   **绝对路径**: 始终使用文件的绝对路径，以确保技能能够准确找到并操作文件。
-   **错误处理**: 技能会返回操作结果的字符串，如果出现错误，会包含详细的错误信息。请注意检查返回结果。
-   **并发操作**: 避免对同一个文件进行高并发的写入或追加操作，这可能导致文件内容损坏或写入异常。

#### `message` 工具发送文件规范 (参见独立指南)

-   **描述**: 关于如何使用 `message` 工具发送本地文件的详细指南，请参考专门的文档。
-   **查看指南**: 请参考 `/Users/fengxiao/.openclaw/workspace/docs/skills_guides/OpenClaw_Feishu_File_Sending_Guide.md`

### 验证注释 (2026-02-12 新增)
本指南已于 2026-02-12 进行了内容验证和更新，确保所有信息准确无误。

## 文档操作策略与经验总结 (2026-02-12 新增)

在进行文档（Markdown 文件）操作时，尤其是在涉及删除、修改特定段落时，根据目前的经验，推荐以下策略：

1.  **优先级：OpenClaw 内置 `write` 工具进行“读-改-写”**：
    *   **操作流程**：
        1.  使用 `read` 工具完整读取目标文件的所有内容。
        2.  在内部对读取到的文本内容进行精确的修改（删除特定行、替换特定段落等）。
        3.  使用 `write` 工具将修改后的**完整内容**一次性覆盖写入到目标文件。
    *   **优点**：
        *   **高可靠性**：这是最稳妥的修改方式，完全避免了 `edit` 工具对 `oldText` 匹配度要求高、`newText` 不能为空等限制，也避免了子代理在复杂文本处理时的“玄学”问题。
        *   **精确控制**：可以确保修改结果与预期完全一致，不易出错。
    *   **适用场景**：任何需要精确修改文档内容的场景，尤其是删除大段内容或进行复杂文本重构时。

2.  **`edit` 工具的使用限制**：
    *   `edit` 工具需要 `oldText` 和 `newText` 参数都提供，并且 `oldText` 必须与文件中现有文本**完全匹配**。
    *   `newText` **不能是空字符串**来表示删除。如果需要删除内容，通常需要用一个空格 `' '` 或换行符 `'\n'` 来代替，但这并非真正的“删除”，而是替换为不可见字符。
    *   **建议**：仅在需要对文件中**精确已知且较短的文本进行替换**时使用 `edit` 工具。不建议用于删除大段内容或模糊匹配的场景。

3.  **`markdown_writer` 技能的使用**：
    *   `markdown_writer` 技能的底层也是调用 `read` 和 `write` 工具。
    *   **建议**：主要用于创建新文件、向文件末尾追加内容，或整体覆盖文件内容。对于删除或修改特定内部段落的复杂场景，仍然建议优先采用“读-改-写”策略。

4.  **与 `coding-agent` 协作进行文档修改**：
    *   `coding-agent` 在代码生成和修改方面表现优秀，但在**精确的文本内容删除和段落重构**方面，其表现可能不如预期稳定。
    *   **建议**：将 `coding-agent` 的职责严格限制在**代码编写、修改、调试、审查**等核心代码任务上。对于文档内容修改，除非用户明确要求 `coding-agent` 处理，否则优先采用“读-改-写”策略，以减少不确定性。

**教训回顾**：
*   **网关稳定性是基础**：之前的 `OpenClaw Gateway` 崩溃导致命令积压和执行异常，这是所有操作的前提。务必优先确保网关稳定。
*   **工具选择要谨慎**：理解每个工具的特性和限制，避免错误地使用工具（例如用 `edit` 尝试删除）。
*   **指令理解要精准**：避免“过度解读”用户指令，尤其是在“阅读/了解”和“开发/修改”之间。

希望这份指南能让我未来在文档操作上更加得心应手，避免再犯低级错误！
