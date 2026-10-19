---
name: legado-book-source
description: 为小说阅读 App（Legado / 阅读3.0）制作书源 JSON。当用户给出小说网站网址并要求"做书源/配置书源/导入书源/分析这个小说站/抓正文规则"，或需要调试已有书源规则时使用。覆盖网站结构探查、CSS/XPath 规则编写、JSON 组装与校验全流程。
---

# Legado 书源制作

## 核心原则

- **先看真 HTML，再写规则**。不要凭 URL 猜结构，必须用浏览器打开首页、详情页、目录页、正文页，用 `document.querySelectorAll` 验证每个选择器确实能命中元素。
- **输出必须是 JSON 数组** `[{...}]`，不是单个对象 `{...}`。这是导入报错 `Expected BEGIN_ARRAY but was BEGIN_OBJECT` 的头号原因。
- **每条规则都要在浏览器里跑一遍** `querySelectorAll(sel).length` > 0，再写进 JSON。

## 工作流程

### 第 1 步：探查首页 URL 模式

用 browser-use 打开用户给的网址，执行 JS 提取所有 `<a>` 的 href：

```python
import aha_browser_use as bu, re
bu.navigate(url)
import time; time.sleep(5); bu.wait_for_load(timeout=20)
html = bu.js("return document.documentElement.outerHTML")
links = re.findall(r'<a[^>]+href="([^"]*)"[^>]*>(.*?)</a>', html, re.S)
for href, text in links[:80]:
    t = re.sub(r'<[^>]+>', '', text).strip()[:30]
    print(f"{t}  ->  {href}")
```

从输出归纳 URL 模板：
- 书籍详情页：通常 `/{id}/`、`/book/{id}.html`、`/{cat}_{id}/`
- 章节页：通常 `/{bookId}/{chapterId}.html`、`/{bookId}/{chapterId}_{page}.html`
- 搜索：找 `<form action="...search...">` 看 method 和参数名
- 分类/排行：作为发现页 exploreUrl

### 第 2 步：抓取四类页面的真实 HTML

依次导航并保存 HTML 片段，重点看容器 class/id：

1. **搜索结果页**：用 XMLHttpRequest 同步 POST 表单，观察结果列表容器
2. **书籍详情页**：找书名、作者、分类、封面、简介、最新章节、目录入口
3. **目录页**：章节列表容器，注意是否分页
4. **正文页**：正文容器 class、翻页链接（含 `_2.html` 之类分页必须配 nextContentUrl）

同步 POST 测试搜索：

```python
result = bu.js("""
var x = new XMLHttpRequest();
x.open('POST', '/modules/article/search.php', false);
x.setRequestHeader('Content-Type','application/x-www-form-urlencoded');
x.send('searchkey='+encodeURIComponent('完整书名')+'&其他隐藏字段=值');
return x.responseText;
""")
```

### 第 3 步：识别 CMS 类型并套规则

常见小说站 CMS 特征：

| CMS | 特征路径 | 典型容器 |
|---|---|---|
| 杰奇 CMS | `/modules/article/search.php`、`/fenlei/1/1/` | `div.item`、`div.itemtxt`、`div#list.dir.clear` |
| 笔趣阁 | `/search.php?q=`、`/8_12345/` | `div#list`、`div.bookbox`、`#content` |
| 苹果 CMS | `/index.php/ajax/suggest`、JSON API | JSONPath `$.books[*]` |
| 自定义 | 看 og:novel meta 标签 | `[property=og:novel:book_name]` |

### 第 4 步：在浏览器里验证每个选择器

写好候选规则后，逐条跑：

```python
for name, sel in [
  ("书名", "div.itemtxt h1 a"),
  ("作者", "div.itemtxt p a[href*='author']"),
  ("目录", "div#list ul li a"),
]:
    cnt = bu.js('return document.querySelectorAll("' + sel + '").length')
    sample = bu.js('var e=document.querySelectorAll("' + sel + '")[0]; return e?e.textContent.trim().slice(0,40):"NONE"')
    print(f"[{cnt}] {name}: {sample}")
```

必须出现 `[1]` 或更高计数，且 sample 文本符合预期。封面取 `@src` 时 textContent 为空是正常的。

### 第 5 步：组装 JSON

从 `assets/book-source-template.json` 复制模板，替换字段。关键提醒：

- `bookSourceUrl` = 用户给的根域名
- `searchUrl`：POST 搜索写成 `/path,{"method":"POST","body":"k={{key}}&..."}`
- `ruleSearch.bookList`、`ruleExplore.bookList` 必须指向**循环容器**，不是单个元素
- 作者文本常带"作者："前缀，用 `##^作者：##` 清洗
- 搜索结果里 bookUrl 常带 `#dir` 锚点，用 `##\\#dir$##` 清掉
- 正文分页：`nextContentUrl` 指向"下一页"链接选择器，最后一页取不到值时 Legado 自动停止
- 章节正文里的 `<script>` 混淆用 `content` 规则后加 `##<script.*?</script>##` 清除

### 第 6 步：校验并交付

```python
import json
data = json.load(open(path, encoding="utf-8"))
assert isinstance(data, list), "必须是数组"
print("书源数:", len(data), "| 名称:", data[0]["bookSourceName"])
```

校验通过后用 `present_files` 交付 `.json` 文件。告诉用户：
- 导入方式：Legado → 书源管理 → + → 粘贴/文件导入
- 搜索注意事项（是否支持模糊搜索、是否需完整书名）
- 已知限制（正文分页、反爬、VIP 等）

## 常见坑位速查

1. **导入报 BEGIN_ARRAY**：文件是 `{...}` 而非 `[{...}]`
2. **搜索无结果**：检查是否漏了隐藏字段（`action=login`、`searchtype=articlename` 等），打开浏览器表单看 `<input type="hidden">`
3. **目录只有最新 3 章**：完整目录可能在详情页下方 `div#list`，或单独的目录页，翻页规则 `nextTocUrl` 要配
4. **正文乱码**：URL 后加 `,{"charset":"gbk"}`
5. **正文被截断**：网站分页了，配 `nextContentUrl`
6. **选择器在网页里能选中、在 Legado 里取不到**：Legado 的 `@css:` 用的是 Jsoup CSS 选择器，不支持 `:nth-of-type` 的复杂变体时，改用 XPath `//` 开头
7. **封面图裂**：给 `coverUrl` 规则加 Referer header，或正文图片规则里手动拼 headers

## 详细参考

- Legado 完整语法（CSS/XPath/JSONPath/JS/正则、URL 参数、变量）见 `references/legado-syntax.md`
- 输出模板见 `assets/book-source-template.json`
