# CLAUDE_CODE_USAGE_GUIDE.md - 与Claude Code协作规则与最佳实践

为了提高与Claude Code的协作效率，并避免Agent（Antigravity）过度“微管理”，特制定以下规则和最佳实践：

## 核心协作原则

1.  **高层次指令优先：** 始终尝试使用自然语言，以“做什么”为核心向Claude Code提出需求，而不是“怎么做”或提供详细的代码实现。
2.  **信任其实现能力：** 信任Claude Code能够根据高层次指令自主地实现代码逻辑，包括选择合适的方法、变量命名、代码结构等。
3.  **明确目标与上下文：** 在提出需求时，明确告知Claude Code当前的任务目标、涉及的文件、以及预期的输出或效果。
4.  **提供必要约束：** 如果有特定的库、函数或实现模式是必须遵守的，以简洁的语言明确指出，而不是提供完整的代码。
5.  **结果导向：** 专注于验证Claude Code的输出是否符合预期，而不是纠结于其实现细节。如果结果不符，再提供更具体的反馈。
6.  **避免重复劳动：** 除非明确要求，否则不应重复提供已被Claude Code接受或处理过的信息。

## 通过 `coding-agent` 调用 Claude Code 的最佳实践

当我（Agent）需要 Claude Code 来完成代码任务时，我会遵循以下流程：

### 0. 任务分解

-   **利用 `task-decomposer` 技能**：在将任务交给 Claude Code 之前，我会首先利用我已有的 `task-decomposer` 技能对复杂的编码任务进行详细的分解。
    -   **目的**：将一个大型的、模糊的开发任务拆解为一系列清晰、可执行的子任务，明确每个子任务的目标、输入、输出和依赖关系。
    -   **益处**：这有助于确保 Claude Code 每次处理的任务范围明确，提高代码生成的准确性和效率，并便于后续的进度跟踪和问题排查。

### 1. 任务准备

-   **创建独立工作目录**：为每个 Claude Code 任务创建一个独立的目录，确保环境隔离和文件管理清晰。
    ```tool_code
    exec(command="mkdir -p your_project_directory", ...)
    ```

### 2. 启动与指令下达

-   **启动 Claude Code 会话**：使用 `exec` 工具启动 Claude Code。
    -   **关键点**：必须使用 `pty=true`，因为 Claude Code 是交互式终端应用。
    -   **指令清晰**：在 `command` 参数中，用自然语言向 Claude Code 明确描述任务目标和要求（例如：“请用Python写一个简单的计数器类，包括初始化、递增、递减和获取当前值的方法。”）。
    -   **使用需求文档的命令示例** (2026-02-10 新增):
        ```bash
        bash pty:true workdir:~/Projects/myapp command:"claude 'Read REQUIREMENTS.md and implement the requirements. Run tests when done.'"
        ```
        这个示例展示了如何在调用 Claude Code 时，明确指示它首先读取工作目录下的 `REQUIREMENTS.md` 文件，然后根据其中的要求进行开发，并在完成后运行测试。

    ```tool_code
    exec(command="claude '你的代码生成/修改指令'", pty=true, workdir="your_project_directory", sessionId="<生成的sessionId>")
    ```

### 3. 实时监控与交互

-   **实时获取输出日志**：在 Claude Code 会话启动后，持续使用 `process action:log` 命令，传入 Claude Code 会话的 `sessionId`，实时获取其思考过程、建议和代码输出。
    ```tool_code
    process(action="log", sessionId="<Claude Code会话的ID>")
    ```
    这将确保我能“看到”Claude Code 的每一个动作和思考过程，从而进行及时的判断和响应。

-   **交互式确认与免确认原则**：
    -   **旧原则 (已废弃)**：当 Claude Code 询问是否执行某操作（例如创建文件、接受修改）时，它通常会提供数字选项（如 `1. Yes`, `2. No`）。Agent 需要根据情况，通过 `process action:submit` 命令来发送选择。
    -   **新原则 (2026-02-10 起生效)**：除非用户明确指示需要确认，否则在 Agent（赵红兵）与 Claude Code 协作进行开发的过程中，涉及到代码文件（创建、修改、删除等）的常规操作，Agent 不再向用户进行二次确认。Agent 将直接向 Claude Code 发送确认指令（如 `1`），并全程监控，只向用户汇报阶段性进展和最终结果，或在遇到异常/无法自主决策时寻求指示。此举旨在提高开发效率。
    ```tool_code
    process(action="submit", data="1", sessionId="<Claude Code会话的ID>")
    ```

### 4. 代码审查与优化 (集成 `requesting-code-review` 技能)

-   **主动代码审查**：在 Claude Code 完成代码编写或修改后，我会**主动触发 `requesting-code-review` 技能**。
    -   **触发时机**：在完成主要任务、实现核心功能，或在代码即将合并到主分支之前。
    -   **目的**：利用 `requesting-code-review` 技能，从外部视角（模拟人类代码审查员或另一个 AI 审查 Agent）来验证代码的质量、逻辑、规范性和是否符合要求。
    -   **流程**：我将向 `requesting-code-review` 技能提供 Claude Code 编写的代码以及任务上下文，等待其返回审查结果。
    -   **反馈与迭代**：根据审查结果，我将决定是直接接受代码，还是将审查意见反馈给 Claude Code 进行进一步的修改和优化。

### 5. 任务收尾

-   **文件验证与获取**：在 Claude Code 完成所有操作并退出会话后，我会在指定的工作目录中验证和获取最终生成或修改的文件。
    ```tool_code
    read(file_path="your_project_directory/your_file.py")
    ```

## 文件与目录管理规范 (2026-02-10 新增)

为确保项目结构清晰和需求管理有序，特制定以下文件与目录管理规范：

-   **日常测试或临时脚本**：统一存放在 `workspace/script/` 目录下。
-   **临时脚本的需求文档**：存放在 `workspace/script/requirement/` 目录下。
-   **具体项目的需求文档**：在该项目自己的根目录下创建 `requirement` 文件（或目录）来管理。例如，`your_project_directory/REQUIREMENTS.md` 或 `your_project_directory/docs/spec.md`。

Agent（赵红兵）在将任务分配给 Claude Code 时，会确保需求文件位于 Claude Code `workdir` 的可访问路径内，并在 Claude Code 的 `prompt` 中明确指示其先读取该需求文件再进行开发。

## Agent 与 Claude Code 协作的沟通红线 (2026-02-10 更新)

1.  **需求传递规范：**
    *   **禁止直接粘贴需求内容：** 除非需求内容极短且无文件存储必要，否则一律禁止将完整的需求文档内容直接作为 `sessions_spawn` 的 `task` 参数。
    *   **强制文件路径引用：** 启动 `coding-agent` 时，必须通过 `sessions_spawn` 的 `task` 参数，明确告知 `coding-agent` 需求文档的文件路径，并指示它自行读取。
    *   **明确读取指令：** 在 `task` 参数中，除了提供文件路径，还需包含明确的指令，例如“请自行读取该文件”，以确保 `coding-agent` 知道要从文件中获取需求。

2.  **模型与思考模式：**
    *   **默认模型优先：** 除非用户明确指定，否则 `coding-agent` 应使用其默认模型进行开发。
    *   **思考模式按需开启：** 根据用户指令，灵活控制 `coding-agent` 的 `thinking` 模式（`on` 或 `off`），以便用户实时监控或减少输出干扰。

3.  **任务反馈机制：**
    *   `coding-agent` 完成任务后，应总结其工作成果和代码提交情况，并返回给主会话。

4.  **强制验证文件创建/修改**: 所有由 `coding-agent` 完成的文件创建或修改任务，我必须要求其在完成后，**通过 `ls -F <完整绝对路径>` 或 `cat <完整绝对路径>` 命令，验证文件确实已在要求路径下创建或修改，并输出验证结果**。

4.  **强制验证文件创建/修改**: 所有由 `coding-agent` 完成的文件创建或修改任务，我必须要求其在完成后，**通过 `ls -F <完整绝对路径>` 或 `cat <完整绝对路径>` 命令，验证文件确实已在要求路径下创建或修改，并输出验证结果**。

---
*上次更新时间：2026-02-10*
