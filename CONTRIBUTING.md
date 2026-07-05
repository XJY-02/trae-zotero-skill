# 贡献指南

感谢你对 Zotero Skill for TRAE 项目的兴趣！我们欢迎各种形式的贡献。

## 贡献方式

- **报告 Bug** — 提交 Issue 描述问题和复现步骤
- **功能建议** — 分享你想要的新功能
- **文档改进** — 修正错别字、补充说明
- **代码贡献** — 提交 Pull Request

## 开发流程

1. Fork 本仓库
2. 创建你的功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交你的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启一个 Pull Request

## Skill 结构

```
.trae/skills/zotero/
└── SKILL.md    # Skill 的主要定义和文档
```

SKILL.md 包含 frontmatter 和正文两部分：

- **frontmatter**: `name` 和 `description` 字段
- **正文**: 详细的使用说明、API 文档、示例代码等

## 编码规范

- 使用中文编写文档（面向中文用户）
- API 示例同时提供 PowerShell（Windows）和 curl（macOS/Linux）版本
- 代码示例要简洁易懂
- 保持 Markdown 格式规范

## 提交 Pull Request

1. 确保你的更改与 Zotero Web API v3 兼容
2. 更新 README.md 和 SKILL.md 中的相关文档
3. 在 CHANGELOG.md 中添加变更记录
4. 描述清楚你的更改内容和原因

## 问题反馈

请通过 GitHub Issues 提交问题，并包含以下信息：

- 问题描述
- 复现步骤
- 预期行为
- 实际行为
- 环境信息（操作系统、TRAE 版本等）
