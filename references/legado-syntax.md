# Legado 书源语法速查

## 规则写法总览

Legado 支持四种解析引擎，靠前缀区分：

| 前缀 | 引擎 | 示例 |
|---|---|---|
| 无（默认） | Jsoup Default | `tag.div.0@text`、`class.book@href` |
| `@css:` | Jsoup CSS | `@css:div.item a@text` |
| `@XPath:` 或 `//` | XPath | `//div[@class="item"]/a/text()` |
| `@json:` 或 `$.` | JSONPath | `$.books[*].name` |
| `@js:` | JavaScript | 放在其他规则最后 |

## Default 规则语法

格式：`类型.名称.位置@获取内容`
- 类型：`class`、`id`、`tag`、`text`、`children`
- 位置：从 0 开始，负数倒序，`!` 排除，`[0,2:5,-1]` 数组写法
- `@` 分隔多段
- `@` 最后一段是获取内容：`text`、`ownText`、`href`、`src`、`html`、`textNodes`

## 连接符（同种规则间）

- `&&` 合并所有取到的值
- `||` 取第一个有值的
- `%%` 交替取值（列表 1 第 1 个 → 列表 2 第 1 个 → ...）

## 正则净化

- **跟在其他规则后**：`##正则##替换内容`，循环替换
- **OnlyOne（独立用）**：`##正则##替换内容###`，只替换第一个匹配
- **AllInOne（只能在 bookList/chapterList/exploreList 用）**：以 `:` 开头，如 `:href="(/read/\d+/)">([^<]+)` 然后 `$1` `$2` 引用分组

## URL 写法

### GET
```
https://example.com/search?q={{key}}&page={{page}}
```

### POST
```
/search.php,{"method":"POST","body":"k={{key}}&page={{page}}"}
```

### 带编码 / 请求头
```
/search.php,{"charset":"gbk","headers":{"User-Agent":"Mozilla/5.0 ..."}}
```

### 变量
- `{{key}}`：搜索关键词
- `{{page}}`：页码，从 1 开始
- `{{}}` 里默认是 JS，可用 `{{book.name}}`、`{{chapter.title}}`
- 非 `{{}}` 上下文里要用其他引擎需加前缀 `@@`（Default）、`@css:`、`@XPath:`

## 关键字段位置

### 书源基本
- `bookSourceUrl`：根 URL，唯一标识
- `bookSourceName`：显示名
- `bookSourceGroup`：分组
- `bookSourceType`：0=文字，1=有声
- `bookUrlPattern`：书籍 URL 正则，识别用

### 搜索 ruleSearch
- `bookList`：结果列表循环容器
- `name` / `author` / `kind` / `coverUrl` / `intro` / `lastChapter` / `bookUrl`

### 发现 ruleExplore + exploreUrl
- exploreUrl 格式：`名称::URL` 多个用换行或 `&&` 分隔
- 或 JSON 数组 `[{"title":"玄幻","url":"/xuanhuan/{{page}}.html"}]`

### 详情 ruleBookInfo
- 同上 + `tocUrl`（目录页 URL，留空则用 bookUrl）
- `bookInfoInit`：预处理（AllInOne 正则或 JS 返回 JSON）

### 目录 ruleToc
- `chapterList`：章节列表容器，`-` 开头倒序
- `chapterName`：章节名
- `chapterUrl`：章节链接
- `nextTocUrl`：目录下一页
- `updateTime`：章节更新时间

### 正文 ruleContent
- `content`：正文内容
- `nextContentUrl`：正文下一页（分页拼接）
- `sourceRegex`：嗅探正则（有声书用）
- `webJs`：WebView 里执行的 JS

## 常用 JS 工具函数

```
java.ajax(url)              异步请求
java.getCookie(tag, key)    读 cookie
java.base64Decode(str)     base64 解码
java.md5Encode(str)         MD5
java.timeFormat(ts)         时间戳格式化
java.htmlFormat(str)        HTML 转文本
```

## 调试技巧

Legado 搜索框输入：
- `++https://详情页URL`：调试详情页
- `--https://正文页URL`：调试正文页
- `搜索关键词`：调试搜索
