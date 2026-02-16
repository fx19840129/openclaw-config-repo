# OpenClaw Feishu 文件发送指南

## `message` 工具发送文件注意事项

在使用 `message` 工具发送本地文件时，请务必使用 **`filePath`** 参数来指定文件路径，而不是 `file_path`。

**正确用法：**
```python
message(action='send', to='[接收者ID]', message='[消息内容]', filePath='/path/to/your/file.ext')
```

**错误用法示例 (请避免)：**
```python
# 错误：使用了 file_path 参数
message(action='send', to='[接收者ID]', message='[消息内容]', file_path='/path/to/your/file.ext')
```

**教训：**
- 仔细核对工具文档中的参数名。
- 在遇到类似问题时，优先检查参数拼写和大小写。

---

**此次任务总结：**

**远程服务器 `8.155.160.8` 的 PostgreSQL 已经成功升级到 15 版本，并且数据库 `english_vocab_db` 及用户 `english_user` 都已配置完成，`schema.sql` 和 `data.sql` 也已成功导入。最终，英语词汇生成脚本 `english_practice_cli.py` 成功运行，并生成了 `day-2.docx` 文件，也已成功发送给你。**
