---
name: "zotero"
description: "通过 Zotero Web API 管理文献库，支持搜索、增删改查条目、集合、标签等操作。当用户需要操作 Zotero 文献库、搜索文献、管理收藏夹或标签时调用此 skill。"
---

# Zotero Skill

通过 Zotero Web API v3 管理你的 Zotero 文献库。支持条目（items）、集合（collections）、标签（tags）等的增删改查操作。

## 配置要求

使用此 skill 前，需要设置以下环境变量：

| 环境变量 | 必填 | 说明 |
|----------|------|------|
| `ZOTERO_API_KEY` | 是 | Zotero API Key，在 [Zotero 设置页面](https://www.zotero.org/settings/keys/new) 创建 |
| `ZOTERO_USER_ID` | 否 | Zotero 用户 ID，可通过 API Key 自动获取 |

### Windows 设置环境变量

```powershell
# 设置用户环境变量（永久生效）
[Environment]::SetEnvironmentVariable("ZOTERO_API_KEY", "your-api-key", "User")

# 验证
$env:ZOTERO_API_KEY = [Environment]::GetEnvironmentVariable("ZOTERO_API_KEY", "User")
```

### macOS / Linux 设置环境变量

```bash
# 添加到 ~/.bashrc 或 ~/.zshrc
export ZOTERO_API_KEY="your-api-key"
export ZOTERO_USER_ID="1234567"
```

## 获取 API Key 和 User ID

1. 访问 [Zotero API Keys 设置](https://www.zotero.org/settings/keys/new)
2. 创建一个新的 API Key，勾选需要的权限（建议勾选所有库的读写权限）
3. 记录 API Key
4. 在同一页面可以看到你的 **User ID**（数字形式，不是用户名）

### 验证 API Key

```powershell
# Windows PowerShell
Invoke-RestMethod -Uri "https://api.zotero.org/keys/$env:ZOTERO_API_KEY" -Headers @{"Zotero-API-Version" = "3"}
```

```bash
# macOS / Linux
curl "https://api.zotero.org/keys/$ZOTERO_API_KEY" -H "Zotero-API-Version: 3"
```

## 通用请求规范

### API 基础信息

- **基础地址**: `https://api.zotero.org`
- **API 版本**: 3
- **认证方式**: HTTP Header `Zotero-API-Key` 或 `Authorization: Bearer`

### 请求 Headers

所有 API 请求必须包含以下 header：

```
Zotero-API-Version: 3
Zotero-API-Key: <ZOTERO_API_KEY>
```

### 用户库路径前缀

```
/users/<ZOTERO_USER_ID>
```

> 如果未设置 `ZOTERO_USER_ID`，先调用 `/keys/<API_KEY>` 接口获取 userID。

### 群组库路径前缀

```
/groups/<groupID>
```

## 条目 (Items) 操作

### 1. 获取条目列表

```
GET /users/<userID>/items
```

**常用参数：**

| 参数 | 值 | 默认值 | 说明 |
|------|-----|--------|------|
| `q` | string | null | 快速搜索（标题、作者等） |
| `qmode` | `titleCreatorYear`, `everything` | `titleCreatorYear` | 搜索模式，`everything` 包含全文 |
| `itemType` | string | null | 按条目类型筛选，如 `book`, `journalArticle` |
| `tag` | string | null | 按标签筛选，支持 `tag=foo&tag=bar` (AND), `tag=foo \|\| bar` (OR) |
| `sort` | `dateAdded`, `dateModified`, `title`, `creator`, `date`, `publisher` | `dateModified` | 排序字段 |
| `direction` | `asc`, `desc` | 取决于 sort | 排序方向 |
| `limit` | 1-100 | 25 | 每页数量 |
| `start` | integer | 0 | 起始偏移量 |
| `include` | `data`, `bib`, `citation` | `data` | 返回内容，可逗号分隔多个 |
| `includeTrashed` | 0, 1 | 0 | 是否包含回收站条目 |
| `format` | `json`, `bib`, `bibtex`, `ris`, `csv` 等 | `json` | 返回格式 |

**PowerShell 示例：**

```powershell
$headers = @{
  "Zotero-API-Version" = "3"
  "Zotero-API-Key" = $env:ZOTERO_API_KEY
}
$userId = $env:ZOTERO_USER_ID

# 搜索标题含 "machine learning" 的条目
Invoke-RestMethod -Uri "https://api.zotero.org/users/$userId/items?q=machine+learning&qmode=titleCreatorYear&limit=20&sort=dateModified&direction=desc" -Headers $headers
```

### 2. 获取单个条目

```
GET /users/<userID>/items/<itemKey>
```

### 3. 获取条目的子条目（附件、笔记等）

```
GET /users/<userID>/items/<itemKey>/children
```

### 4. 创建新条目

```
POST /users/<userID>/items
Content-Type: application/json
```

**请求体示例（JSON 数组，最多 50 个）：**

```json
[
  {
    "itemType": "book",
    "title": "我的书籍",
    "creators": [
      {
        "creatorType": "author",
        "firstName": "San",
        "lastName": "Zhang"
      }
    ],
    "tags": [
      { "tag": "重要" },
      { "tag": "已读" }
    ],
    "collections": [
      "<collectionKey>"
    ],
    "date": "2024",
    "publisher": "出版社名称",
    "ISBN": "978-xxxxxxxxx"
  }
]
```

**PowerShell 示例：**

```powershell
$body = @(
  @{
    itemType = "journalArticle"
    title = "论文标题"
    creators = @(@{ creatorType = "author"; firstName = "San"; lastName = "Zhang" })
    publicationTitle = "期刊名称"
    volume = "10"
    issue = "2"
    pages = "100-120"
    date = "2024"
    DOI = "10.xxxx/xxxxx"
    tags = @(@{ tag = "待读" })
  }
) | ConvertTo-Json -Depth 10

Invoke-RestMethod -Uri "https://api.zotero.org/users/$userId/items" -Method Post -Headers @{"Zotero-API-Version"="3"; "Zotero-API-Key"=$env:ZOTERO_API_KEY; "Content-Type"="application/json"} -Body $body
```

### 5. 更新条目

**完整更新 (PUT)：**

```
PUT /users/<userID>/items/<itemKey>
Content-Type: application/json
```

提交完整的条目数据（含 `version` 字段）。

**部分更新 (PATCH)：**

```
PATCH /users/<userID>/items/<itemKey>
Content-Type: application/json
If-Unmodified-Since-Version: <当前版本号>
```

只提交需要修改的字段。

**PowerShell 示例：更新条目标题**

```powershell
$body = @{
  title = "新标题"
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://api.zotero.org/users/$userId/items/<itemKey>" -Method Patch -Headers @{"Zotero-API-Version"="3"; "Zotero-API-Key"=$env:ZOTERO_API_KEY; "Content-Type"="application/json"; "If-Unmodified-Since-Version"="<version>"} -Body $body
```

### 6. 删除条目

```
DELETE /users/<userID>/items/<itemKey>
If-Unmodified-Since-Version: <版本号>
```

**批量删除（最多 50 个）：**

```
DELETE /users/<userID>/items?itemKey=<key1>,<key2>,<key3>
If-Unmodified-Since-Version: <库版本号>
```

### 7. 获取回收站条目

```
GET /users/<userID>/items/trash
```

### 8. 导出为指定格式

使用 `format` 参数导出为不同格式：

| 格式值 | 说明 |
|--------|------|
| `bibtex` | BibTeX |
| `biblatex` | BibLaTeX |
| `ris` | RIS |
| `csv` | CSV |
| `csljson` | CSL JSON |
| `mods` | MODS |

**示例：导出为 BibTeX**

```
GET /users/<userID>/items?format=bibtex&limit=100
```

## 集合 (Collections) 操作

### 1. 获取所有集合

```
GET /users/<userID>/collections
```

### 2. 获取顶级集合

```
GET /users/<userID>/collections/top
```

### 3. 获取单个集合

```
GET /users/<userID>/collections/<collectionKey>
```

### 4. 获取子集合

```
GET /users/<userID>/collections/<collectionKey>/collections
```

### 5. 获取集合内的条目

```
GET /users/<userID>/collections/<collectionKey>/items
```

支持与条目列表相同的搜索、排序、分页参数。

### 6. 创建集合

```
POST /users/<userID>/collections
Content-Type: application/json
```

**请求体：**

```json
[
  {
    "name": "集合名称",
    "parentCollection": "<父集合Key，顶级则省略或设为 false>"
  }
]
```

### 7. 更新集合

```
PUT /users/<userID>/collections/<collectionKey>
Content-Type: application/json
```

### 8. 删除集合

```
DELETE /users/<userID>/collections/<collectionKey>
If-Unmodified-Since-Version: <版本号>
```

### 9. 将条目添加到集合

通过更新条目的 `collections` 字段实现：

```
PATCH /users/<userID>/items/<itemKey>
Content-Type: application/json

{
  "collections": ["<collectionKey1>", "<collectionKey2>"]
}
```

## 标签 (Tags) 操作

### 1. 获取所有标签

```
GET /users/<userID>/tags
```

**参数：**

| 参数 | 值 | 说明 |
|------|-----|------|
| `q` | string | 搜索标签名 |
| `qmode` | `contains`, `startsWith` | 搜索模式 |
| `sort` | `title`, `numItems` | 排序方式 |
| `direction` | `asc`, `desc` | 排序方向 |
| `limit` | integer | 数量限制 |

### 2. 获取条目关联的标签

```
GET /users/<userID>/items/<itemKey>/tags
```

### 3. 获取集合内的标签

```
GET /users/<userID>/collections/<collectionKey>/tags
```

### 4. 给条目添加/移除标签

通过更新条目实现：

```
PATCH /users/<userID>/items/<itemKey>
Content-Type: application/json

{
  "tags": [
    { "tag": "标签1" },
    { "tag": "标签2" }
  ]
}
```

### 5. 删除标签

```
DELETE /users/<userID>/tags?tag=<标签名>
If-Unmodified-Since-Version: <库版本号>
```

## 群组 (Groups) 操作

### 1. 获取用户所属群组

```
GET /users/<userID>/groups
```

### 2. 群组库操作

群组库的操作与用户库相同，只需将路径前缀从 `/users/<userID>` 替换为 `/groups/<groupID>`。

## 格式化引用和书目

### 获取格式化引用

```
GET /users/<userID>/items/<itemKey>?include=citation&style=apa
```

### 获取格式化书目

```
GET /users/<userID>/items/<itemKey>?include=bib&style=chicago-note-bibliography
```

**常用 style 参数：**

| 样式名 | 说明 |
|--------|------|
| `apa` | APA 格式 |
| `mla` | MLA 格式 |
| `chicago-note-bibliography` | 芝加哥注释书目格式 |
| `ieee` | IEEE 格式 |
| `gb7714-2015` | GB/T 7714-2015 中文格式 |
| `nature` | Nature 格式 |

## 条目类型参考

常见条目类型（itemType）：

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

## 响应头信息

重要的响应头：

- `Total-Results`: 匹配结果总数（用于分页）
- `Last-Modified-Version`: 库/对象的当前版本号（用于写操作）
- `Link`: 分页链接（`rel=next`, `rel=last`, `rel=prev`, `rel=first`）

## PowerShell 最佳实践

### 1. 统一 Headers 和配置

```powershell
$apiKey = $env:ZOTERO_API_KEY
$userId = $env:ZOTERO_USER_ID
$baseUrl = "https://api.zotero.org"

$headers = @{
  "Zotero-API-Version" = "3"
  "Zotero-API-Key" = $apiKey
}
```

### 2. 自动获取 User ID（如果未设置）

```powershell
if (-not $userId) {
  $keyInfo = Invoke-RestMethod -Uri "$baseUrl/keys/$apiKey" -Headers $headers
  $userId = $keyInfo.userID
}
```

### 3. 处理分页

当 `Total-Results` 大于 `limit` 时，循环获取所有结果：

```powershell
$allItems = @()
$start = 0
$limit = 100

do {
  $response = Invoke-WebRequest -Uri "$baseUrl/users/$userId/items?limit=$limit&start=$start" -Headers $headers
  $items = $response.Content | ConvertFrom-Json
  $allItems += $items
  $total = [int]$response.Headers['Total-Results']
  $start += $limit
} while ($start -lt $total)
```

### 4. 获取版本号

写操作前先获取当前版本号：

```powershell
$response = Invoke-WebRequest -Uri "$baseUrl/users/$userId/items/<itemKey>" -Headers $headers
$version = $response.Headers['Last-Modified-Version']
$item = $response.Content | ConvertFrom-Json
```

### 5. 构造 JSON 数组（重要！）

PowerShell 的 `ConvertTo-Json` 在处理只有一个元素的数组时会自动展开为单个对象，导致 API 返回 "Uploaded data must be a JSON array" 错误。**创建/更新条目时必须手动构造 JSON 字符串**：

```powershell
# 错误方式（单元素会被展开）
$body = @(@{ itemType = "book"; title = "Test" }) | ConvertTo-Json -Depth 10

# 正确方式：手动构造 JSON 字符串
$body = '[
  {
    "itemType": "book",
    "title": "我的书籍",
    "creators": [
      { "creatorType": "author", "firstName": "San", "lastName": "Zhang" }
    ]
  }
]'

# 或者使用 -AsArray 标志（PowerShell 7+）
$body = (@(@{ itemType = "book"; title = "Test" }) | ConvertTo-Json -Depth 10 -AsArray)
```

### 6. PowerShell 5 中文/UTF-8 编码问题（重要！）

**PowerShell 5（Windows 自带的 powershell.exe）默认使用系统本地编码（如 GBK），而非 UTF-8。** 使用 `ConvertTo-Json` + `Invoke-RestMethod` 发送包含中文/非 ASCII 字符的请求时，中文字符会被损坏（变成 `?`）。

**表现：** 集合名称、标签、摘要等中的中文全部变成问号。

**根本原因：**
- `ConvertTo-Json` 输出的字符串是系统本地编码（GBK）
- `Invoke-RestMethod` 发送请求时不声明 `charset=utf-8`
- Zotero API 收到非 UTF-8 字节后，无法识别的字符被替换为 `?`

**解决方案：使用 .NET HttpWebRequest 手动发送 UTF-8 编码的请求**

```powershell
# 错误方式：ConvertTo-Json + Invoke-RestMethod 会导致中文乱码
$body = @{ name = "我的收藏" } | ConvertTo-Json
Invoke-RestMethod -Uri "$baseUrl/users/$userId/collections" -Method Post -Headers $headers -Body $body
# 结果：集合名称变成 "????"

# 正确方式：手动构造 JSON 并用 .NET 类发送 UTF-8 请求
$jsonBody = '{"name": "我的收藏"}'
$bodyBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonBody)

$request = [System.Net.HttpWebRequest]::Create("$baseUrl/users/$userId/collections")
$request.Method = "POST"
$request.Headers.Add("Zotero-API-Version", "3")
$request.Headers.Add("Zotero-API-Key", $apiKey)
$request.ContentType = "application/json; charset=utf-8"
$request.ContentLength = $bodyBytes.Length

$stream = $request.GetRequestStream()
$stream.Write($bodyBytes, 0, $bodyBytes.Length)
$stream.Close()

$resp = $request.GetResponse()
$respStream = $resp.GetResponseStream()
$reader = New-Object System.IO.StreamReader($respStream, [System.Text.Encoding]::UTF8)
$result = $reader.ReadToEnd()
```

**读取响应时也需要注意：** 用 `Invoke-RestMethod` 读取含中文的响应时，PowerShell 控制台显示可能乱码，但实际数据可能是正确的。验证方式：

```powershell
# 用 WebClient 获取原始字节再用 UTF-8 解码，确保看到真实数据
$webClient = New-Object System.Net.WebClient
$webClient.Headers.Add("Zotero-API-Version", "3")
$webClient.Headers.Add("Zotero-API-Key", $apiKey)
$responseBytes = $webClient.DownloadData("$baseUrl/users/$userId/items/<itemKey>")
$responseStr = [System.Text.Encoding]::UTF8.GetString($responseBytes)
```

> **注意：** PowerShell 7+（`pwsh`）默认使用 UTF-8，不存在此问题。

### 7. 单条目导出的正确方式

单条目导出格式（`format=bibtex`、`format=ris` 等）不能直接在 `/items/<itemKey>` 上使用，需要使用多条目接口加 `itemKey` 参数：

```powershell
# 错误：单条目路径不支持 format 参数
# GET /users/<userID>/items/<itemKey>?format=bibtex

# 正确：使用多条目接口 + itemKey 参数
Invoke-RestMethod -Uri "$baseUrl/users/$userId/items?itemKey=<itemKey>&format=bibtex" -Headers $headers
```

同样，获取单条目的格式化引用和书目也推荐使用此方式：

```powershell
# 获取 APA 引用
$result = Invoke-RestMethod -Uri "$baseUrl/users/$userId/items?itemKey=<itemKey>&include=citation&style=apa" -Headers $headers
$citation = $result[0].citation
```

## 常见错误

| 状态码 | 说明 | 处理方式 |
|--------|------|----------|
| 400 | 请求格式错误 | 检查 JSON 格式和字段 |
| 401 | API key 无效 | 验证 API key 是否正确设置 |
| 403 | 权限不足 | 检查 API key 权限范围 |
| 404 | 资源不存在 | 检查 itemKey/collectionKey |
| 409 | 库被锁定 | 稍后重试 |
| 412 | 版本不匹配 | 重新获取最新版本后重试 |
| 413 | 请求体过大 | 减少单次提交数量（最多 50 个） |
| 428 | 缺少版本号 | 添加 `If-Unmodified-Since-Version` 或 `version` 字段 |

## 使用场景示例

### 场景 1：搜索文献

用户说"帮我找一下关于深度学习的论文" → 使用 `/items?q=深度学习&qmode=everything&itemType=journalArticle`

### 场景 2：列出某集合的文献

用户说"显示我的机器学习收藏夹里的文献" → 先获取集合列表找到对应 key，再用 `/collections/<key>/items`

### 场景 3：添加新文献

用户提供文献信息 → 构造 JSON，使用 `POST /items` 创建

### 场景 4：给文献打标签

用户说"给这篇文献加上已读标签" → 使用 `PATCH /items/<key>` 更新 tags 字段

### 场景 5：导出文献

用户说"把这些文献导出为 BibTeX" → 使用 `format=bibtex` 参数

### 场景 6：获取引用格式

用户说"给我这篇论文的 APA 引用格式" → 使用 `include=citation&style=apa`
