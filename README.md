# legado-book-source · 小说站书源自动制作 Skill

> 给 AI 助手装一个"书源工程师"技能，你只要丢一个小说网站网址给它，它就能自动帮你产出一份可以直接导入「阅读 App」的书源 JSON。

---

## 一、这东西是干什么的？（30 秒看懂）

### 先解释两个词

**Legado / 阅读 3.0**：一款 Android 上的免费小说阅读 App（开源、无广告、可自定义书源）。它本身不带书，需要你自己导入"书源"才能搜到书。

**书源**：一份 JSON 配置文件，相当于告诉阅读 App："去这个网站搜书、这个网站的正文在哪个位置、怎么翻页"。

### 这个 Skill 解决什么痛点？

自己手写书源要懂 CSS 选择器、正则表达式、JSON 格式，门槛很高。

装了这个 Skill 之后，你只要对 AI 助手说一句：

> "帮我给 https://www.xxx.com 做个书源"

AI 就会自动：
1. 打开这个网站，摸清它的页面结构
2. 找到搜索接口、书籍详情页、目录页、正文页的位置
3. 写出对应的规则
4. 在浏览器里验证每条规则真的能取到内容
5. 打包成一个可以直接导入阅读 App 的 JSON 文件

---

## 二、你需要准备什么？

| 项目 | 说明 |
|---|---|
| 一个支持 Skill 的 AI 助手 | 例如：豆包办公版 / 豆包电脑版（Agent 模式）。其他支持类似 Skill 机制的客户端也可以。 |
| 一台电脑 | Windows / Mac 都行，AI 需要用浏览器自动打开网站分析。 |
| 一个小说网站网址 | 你想做成书源的那个小说站，例如 `https://www.sudugu.cc`。 |
| 阅读 App | 手机上装好「阅读 3.0」（也叫 Legado），用来导入最终的 JSON。 |

> ⚠️ 不是手机上的豆包 App 直接用。这个 Skill 需要 AI 能操作电脑浏览器，所以必须在**电脑版 AI 助手**里使用。

---

## 三、怎么安装这个 Skill？

### 方法 A：手动下载（推荐新手）

1. 打开本仓库页面：https://github.com/2105569814-max/legado-book-source
2. 点绿色的 **`<> Code`** 按钮 → 点 **`Download ZIP`**
3. 解压下载下来的 zip，你会得到一个文件夹 `legado-book-source-main`
4. 把里面的 **`legado-book-source`** 文件夹整个复制出来
5. 找到你的 AI 助手的 Skill 目录：
   - **Windows**：
     ```
     C:\Users\你的用户名\AppData\Local\Doubao\User Data\Default\.doubao\agent_mode\workspace\.user_skills\
     ```
   - **Mac**：
     ```
     ~/Library/Application Support/Doubao/User Data/Default/.doubao/agent_mode/workspace/.user_skills/
     ```
   > 不知道"你的用户名"是什么？Windows 按 `Win+R`，输入 `%USERPROFILE%` 回车，地址栏里最后那个文件夹名就是。
6. 把复制出来的 `legado-book-source` 文件夹粘贴进去。

**装完长这样**：
```
.user_skills/
├── legado-book-source/        ← 就是这个文件夹
│   ├── SKILL.md
│   ├── references/
│   │   └── legado-syntax.md
│   └── assets/
│       └── book-source-template.json
├── 其他你已经装的 skill...
```

7. **重启 AI 助手**（关掉再打开），让它识别到新 Skill。

### 方法 B：用 git 命令（有基础再用）

打开终端（Windows 用 PowerShell，Mac 用终端），粘贴：

```bash
cd "你的 .user_skills 目录完整路径"
git clone https://github.com/2105569814-max/legado-book-source.git
```

---

## 四、怎么用？（一句话）

装好之后，在 AI 助手的对话框里直接说：

> **帮我给 https://www.xxx.com 做个书源**

就这一句。AI 会自动开始干活，整个过程大概 1~3 分钟，你会看到它：
- 自动打开浏览器访问那个网站
- 在网页里点来点去、按 F12 看元素
- 最后给你一个 `.json` 文件

### 也可以这样说

- "这个网站我搜书搜不到，帮我修一下书源"
- "这本书的正文乱码了，看看是哪条规则不对"
- "顺便把分类排行榜也做成发现页"

---

## 五、拿到 JSON 后，怎么导入阅读 App？

1. 把 AI 给你的 `.json` 文件**发到手机上**（微信文件传输、QQ、邮件、U盘都行）
2. 在手机上用文件管理器点开这个 `.json` 文件
3. 系统会弹出"打开方式"，选择 **「阅读」** App
4. 阅读 App 会自动识别并提示"是否导入书源"，点**确定**
5. 完成！去阅读 App 的"书源管理"里就能看到新加的源

> 💡 如果点开文件没有弹出"用阅读打开"的选项，那就：打开阅读 App → 底部「我的」→ 「书源管理」→ 右上角三个点 → 「本地导入」→ 选中那个 JSON 文件。

---

## 六、做出来的书源长什么样？

以本作者实测的 `https://www.sudugu.cc` 为例，AI 会产出一份类似这样的 JSON（已做过简化）：

```json
[{
  "bookSourceName": "速读谷",
  "bookSourceUrl": "https://www.sudugu.cc",
  "searchUrl": "/modules/article/search.php,{\"method\":\"POST\",\"body\":\"searchkey={{key}}\"}",
  "ruleSearch": {
    "bookList": "@css:div.item",
    "name": "@css:div.itemtxt h1 a@text",
    "author": "@css:div.itemtxt p a@text##^作者：##"
  },
  "ruleToc": {
    "chapterList": "@css:div#list ul li a",
    "chapterName": "text",
    "chapterUrl": "href"
  },
  "ruleContent": {
    "content": "@css:div.con@html",
    "nextContentUrl": "@css:div.prenext a[href*='_']@href"
  }
}]
```

你不需要懂它，**能导入、能搜书、能看正文就成功了**。

---

## 七、目录结构说明

```
legado-book-source/
├── SKILL.md                        # AI 看的"操作手册"，告诉它怎么做书源
├── references/
│   └── legado-syntax.md            # Legado 规则语法完整速查表
└── assets/
    └── book-source-template.json  # 空模板，AI 做书源时从这里复制改造
```

| 文件 | 给谁看 | 作用 |
|---|---|---|
| `SKILL.md` | AI | 6 步工作流程：探首页 → 抓四类页面 → 识别 CMS → 验证选择器 → 组装 JSON → 校验交付 |
| `references/legado-syntax.md` | AI | 写规则时查语法（CSS/XPath/JSONPath/正则怎么写） |
| `assets/book-source-template.json` | AI | 填空用的空白模板，保证输出格式正确 |

---

## 八、常见问题 FAQ

### Q1：AI 说"这个网站我打不开"或"被反爬了"
换个时间再试，或者把网址前面加上 `www.` / 后面加上 `/` 再试一次。有些网站对自动化浏览器有限制。

### Q2：导入阅读 App 报错 `Expected BEGIN_ARRAY but was BEGIN_OBJECT`
说明 JSON 外面没有套 `[ ]`。本 Skill 的输出已经修过这个坑，但如果你手工改过 JSON，检查一下最外面是不是 `[{...}]` 而不是 `{...}`。

### Q3：书源导入后搜不到书
- 先看 AI 在说明里有没有提示"这个站不支持模糊搜索，要输完整书名"
- 换个完整书名再搜一次，比如直接搜"凡人修仙传"而不是"凡人"
- 阅读 App 里长按书源 → 「调试」→ 输入书名，能看到具体报错

### Q4：正文打开是空白 / 只有一半
大概率是网站正文分页了。AI 应该已经配好 `nextContentUrl`，如果还是不行，把报错截图发给 AI 让它修。

### Q5：正文里夹杂广告、"请收藏本站"之类的文字
把那段广告原文发给 AI，让它在 `ruleContent.content` 后面加一条 `##广告原文##` 清洗规则。

### Q6：封面图显示不出来
有些网站封面有防盗链。把 AI 给的 `coverUrl` 规则后面加上 Referer，例如：
```
"@css:div.item a img@src,{\"headers\":{\"Referer\":\"https://www.xxx.com\"}}"
```
（把 xxx 换成对应网站域名）

### Q7：这个 Skill 支持有声书吗？
默认做的是文字书源（`bookSourceType: 0`）。有声书需要嗅探音频 URL，规则更复杂，目前版本不覆盖，后续可能加。

---

## 九、它是怎么工作的？（好奇者看）

当你对 AI 说"帮我做个书源"时，AI 内部按这个流程走：

```
你给网址
  ↓
① 用浏览器打开首页，提取所有链接，归纳出 URL 规律
   （比如书籍页是 /123/，章节页是 /123/456.html）
  ↓
② 抓四类页面的真实 HTML：
   - 搜索结果页（POST 表单都带什么参数）
   - 书籍详情页（书名/作者/封面/简介在哪个 class 里）
   - 目录页（章节列表容器是什么）
   - 正文页（正文在哪个 div 里、有没有分页）
  ↓
③ 在浏览器里逐条跑 document.querySelectorAll() 验证
   每个选择器都真的能选中元素
  ↓
④ 从模板复制 JSON，把规则填进去
  ↓
⑤ 用 Python json.load() 校验是合法数组格式
  ↓
⑥ 交付 .json 文件给你
```

整个过程不需要你懂代码，AI 会自己搞定。

---

## 十、许可与致谢

- Legado 书源规则原文参考自 [yckceo 的 Legado 规则说明](https://www.yckceo.com/yuedu/tools/index/id/shuyuan.html)
- 本仓库代码以 MIT 协议发布，随便用、随便改、随便 fork
- 如果你觉得好用，欢迎点个 Star ⭐

有问题欢迎提 Issue，或者直接在这个仓库发 Discussion。
