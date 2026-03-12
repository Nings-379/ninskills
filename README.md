# Agent-skills
  Agent Skill（代理技能）是 2025 年底至 2026 年流行起来的一种开放标准格式，旨在让 AI（如 Claude, ChatGPT, GitHub Copilot）具备特定领域的专业能力。
  
  Agent Skills = 用一个文件夹 + SKILL.md 文件，把某个能力打包给 AI Agent 使用。
  
  一个标准的 Agent Skill 通常是一个文件夹，包含以下核心文件：
```
    - SKILL.md (核心)：最重要的文件，定义了技能的名称、描述以及 AI 执行任务的具体指令（Prompt）。
    - scripts/ (可选)：存放可执行脚本（Python/JS 等），让 AI 真的能动起来。
    - references/ (可选)：存放参考文档或模版。
```
