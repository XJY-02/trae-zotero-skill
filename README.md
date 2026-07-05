# Zotero Skill for TRAE

通过 Zotero Web API v3 管理你的 Zotero 文献库的 TRAE skill。支持条目、集合、标签的增删改查，以及文献导出和引用格式化。

## 功能特性

- **文献搜索** — 按标题、作者、全文搜索文献
- **条目管理** — 创建、读取、更新、删除文献条目
- **集合管理** — 管理文献收藏夹（Collections）
- **标签管理** — 给文献打标签、按标签筛选
- **文献导出** — 支持 BibTeX、RIS、CSV、CSL JSON 等多种格式
- **引用格式化** — APA、MLA、Chicago、IEEE、GB7714 等引用样式
- **群组库支持** — 可访问群组共享文献库
- **回收站管理** — 查看和恢复回收站条目

## 快速开始

### 1. 获取 Zotero API Key

1. 访问 [Zotero API Keys 设置页面](https://www.zotero.org/settings/keys/new)
2. 创建一个新的 API Key，根据需要勾选权限（建议勾选所有库的读写权限）
3. 记录 API Key 和页面上显示的 **User ID**（数字形式）

### 2. 设置环境变量

#### Windows (PowerShell)

```powershell
# 设置用户环境变量（永久生效）
[Environment]::SetEnvironmentVariable("ZOTERO_API_KEY", "your-api-key-here", "User")
[Environment]::SetEnvironmentVariable("ZOTERO_USER_ID", "1234567", "User")

# 使当前会话生效
$env:ZOTERO_API_KEY = [Environment]::GetEnvironmentVariable("ZOTERO_API_KEY", "User")
$env:ZOTERO_USER_ID = [Environment]::GetEnvironmentVariable("ZOTERO_USER_ID", "User")
```

#### macOS / Linux

```bash
# 添加到 ~/.bashrc 或 ~/.zshrc
echo 'export ZOTERO_API_KEY="your-api-key-here"' >> ~/.zshrc
echo 'export ZOTERO_USER_ID="1234567"' >> ~/.zshrc
source ~/.zshrc
```

> **注意**: `ZOTERO_USER_ID` 是可选项，skill 可以通过 API Key 自动获取。

### 3. 安装 Skill

将 `.trae/skills/zotero/` 目录复制到你的 TRAE 项目根目录下：

```
your-project/
└── .trae/
    └── skills/
        └── zotero/
            └── SKILL.md
```

或者直接将 `SKILL.md` 放入 `.trae/skills/zotero/` 目录中。

### 4. 验证安装

在 TRAE 中尝试以下指令：

- "帮我列出最近的文献"
- "搜索关于深度学习的论文"
- "显示我的所有收藏夹"

## 使用示例

### 搜索文献

```
帮我找一下关于不确定性量化的期刊论文
```

### 管理集合

```
显示 Uncertainty Quantification 收藏夹里的文献
```

### 添加新文献

直接告诉我文献的标题、作者、期刊等信息，我会帮你添加到 Zotero。

### 导出文献

```
把最近添加的 10 篇文献导出为 BibTeX
```

### 获取引用格式

```
给我这篇论文的 APA 引用格式
```

## API 参考

完整的 API 使用文档请参见 [SKILL.md](.trae/skills/zotero/SKILL.md)，包含：

- 条目操作（CRUD、搜索、分页）
- 集合操作
- 标签操作
- 群组库操作
- 引用格式化
- PowerShell 最佳实践
- 常见错误处理

## 支持的条目类型

| 类型 | 说明 |
|------|------|
| `book` | 书籍 |
| `bookSection` | 书籍章节 |
| `journalArticle` | 期刊论文 |
| `conferencePaper` | 会议论文 |
| `thesis` | 学位论文 |
| `report` | 报告 |
| `webpage` | 网页 |
| `newspaperArticle` | 报纸文章 |
| `magazineArticle` | 杂志文章 |
| `presentation` | 演示文稿 |
| `videoRecording` | 视频 |
| `audioRecording` | 音频 |
| `document` | 文档 |
| `note` | 笔记 |
| `attachment` | 附件 |

## 支持的导出格式

- `bibtex` — BibTeX
- `biblatex` — BibLaTeX
- `ris` — RIS
- `csv` — CSV
- `csljson` — CSL JSON
- `mods` — MODS
- `rdf_bibliontology` — Bibliographic Ontology RDF
- `rdf_dc` — Dublin Core RDF
- `rdf_zotero` — Zotero RDF

## 支持的引用样式

常用样式包括：`apa`、`mla`、`chicago-note-bibliography`、`ieee`、`gb7714-2015`、`nature` 等。

完整的样式列表请参见 [Zotero Style Repository](https://www.zotero.org/styles/)。

## 环境变量

| 变量名 | 必填 | 说明 |
|--------|------|------|
| `ZOTERO_API_KEY` | 是 | Zotero API Key |
| `ZOTERO_USER_ID` | 否 | Zotero 用户 ID（可自动获取） |

## 常见问题

### Q: 如何找到我的 User ID？

A: 访问 [Zotero API Keys 页面](https://www.zotero.org/settings/keys)，在页面顶部可以看到 "Your userID for use in API calls is XXXXXXX"。

### Q: API Key 安全吗？

A: API Key 只存储在你本地的环境变量中，不会上传到任何服务器。请妥善保管，不要提交到代码仓库。

### Q: 支持哪些 Zotero 库？

A: 支持个人用户库和群组库。群组库需要将路径前缀从 `/users/<userID>` 改为 `/groups/<groupID>`。

### Q: 单次可以操作多少条目？

A: 单次创建/更新/删除操作最多 50 个条目。读取操作每页最多 100 条，需要分页获取更多。

## License

MIT License

## 相关链接

- [Zotero 官网](https://www.zotero.org/)
- [Zotero Web API v3 文档](https://www.zotero.org/support/dev/web_api/v3/start)
- [Zotero Style Repository](https://www.zotero.org/styles/)
- [Zotero API Keys 设置](https://www.zotero.org/settings/keys)
