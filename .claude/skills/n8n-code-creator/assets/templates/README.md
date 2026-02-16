# n8n Code 节点提示词模板

本目录包含 n8n Code 节点代码生成所需的提示词模板。

## 文件说明

### 核心模板

- **`prompt-templates.md`**
  - 用于与用户沟通和澄清需求的提示词模板
  - 帮助快速构建有效的需求确认问题
  - 确保生成的代码符合用户期望

## 使用方式

### 在 SKILL.md 中引用

```markdown
**For prompt construction**:
- `assets/templates/prompt-templates.md` - Templates for clarifying requirements
```

### 实际使用场景

当需要向用户确认以下信息时，参考此模板：
- 输入数据结构的确认
- 转换逻辑的细节
- 输出格式的规范
- 边界情况的处理方式

### 与其他参考文件的关系

- **`assets/templates/prompt-templates.md`** - 需求沟通模板（本文件）
- **`references/code-patterns.md`** - 40+ 转换模式库（参考资料）
- **`references/best-practices.md`** - 最佳实践指南（参考资料）

## 目录结构说明

根据 Claude Agent Skill 规范：
- **`assets/templates/`** - 存放可直接使用的模板文件
- **`references/`** - 存放文档、说明、参考资料

本目录（`assets/templates/`）存放的是**可直接复用的提示词模板**，用于标准化与用户的交互流程。
