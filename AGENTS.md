# Legado 书源规则 — AI 编写指南

本仓库内置了 Legado(开源阅读 App)书源规则的完整官方文档(见 `docs/`)。AI 在本仓库内编写或修改书源 JSON、订阅源、替换规则、朗读规则时,**必须遵循本文件与 `docs/` 中的规范**,不要凭印象编造语法或 API。

## 工作原则

1. **先判断站点形态,再选规则体系**:
   - 网页是服务端渲染的 HTML → 用 JSOUP 默认规则或 `@css:`(优先 CSS,可读性好)
   - 有 JSON 接口(浏览器 F12 网络面板可见)→ 用 `@json:` / `$.`,不要去解析 HTML
   - 页面靠 JS 渲染 → URL 加 `,{"webView": true}`
   - 接口加密 → 用 `<js>` 调 `java` 对象解密,参考 `docs/dev/js.md`
2. **规则是按前缀路由的字符串**,前缀决定解析器(见下表),写错前缀规则直接失效。
3. 拿不准的语法查 `docs/` 对应文档(索引见文末);常用 API 的准确签名以 `docs/dev/js.md` 为准。
4. 写完整套源后,按 docs/dev/debug.md 的思路自查:搜索 → 详情 → 目录 → 正文逐级验证。

## 一、规则前缀路由(最核心)

| 规则开头 | 解析器 | 示例 |
|:---|:---|:---|
| (无前缀) | JSOUP 默认规则 | `class.odd@tag.a@text` |
| `@css:` | CSS 选择器 | `@css:.book-list li@text` |
| `@json:` 或 `$.` | JSONPath | `$.data.books[*].title` |
| `@XPath:` 或 `//` | XPath | `//div[@class="box"]/a/@href` |
| `@js:` | JS,**只能放在其他规则最后**,前级结果存入 `result` | `tag.a@text@js:result.replace(" ","")` |
| `<js>...</js>` | JS,可放任意位置/单独使用 | `<js>java.ajax(url)</js>` |
| `:` 开头 | AllInOne 正则,仅限搜索列表/发现列表/详情预加载/目录列表 | `:<li><a[^"]+"([^"]*)">([^<]*)` |

## 二、默认规则(JSOUP Default)速查

每段用 `.` 分三部分:`类型.名称.位置`,段与段之间用 `@` 分隔,**最后一段是取值方式**:

```
class.odd.0@tag.a.0@text      // class="odd" 的第 1 个元素里第 1 个 <a> 的文本
tag.div.-1@text               // 最后一个 div 的文本(位置负数=倒数,0 起)
```

- 类型:`class` / `id` / `tag` / `text` / `children`
- 取值:`text` `ownText`(不含子元素) `textNodes` `href` `src` `html` `all`;CSS 里取属性用 `@content` 等
- 排除:`tag.div!0:2@text`(排除第 1、3 个 div,多个用 `:` 分隔)
- 列表倒置:列表规则最前面加 `-`,如 `-tag.dd@tag.a@text`
- 数组索引:`[0]` `[-1]` `[0:3]` `[0:10:2]` `[!0:2]` `[-1:0]`(反转)
- 尾部净化:`规则##正则##替换为`(替换为空可省略第二个 `##`)

## 三、连接符(仅限同种规则之间,不含 js 和正则)

| 符号 | 作用 |
|:---|:---|
| `\|\|` | 取第一个有值的结果(容错首选) |
| `&&` | 合并所有结果 |
| `%%` | 交替取值(多列表轮流) |

## 四、正则的三种形态(别混用)

| 形态 | 语法 | 适用位置 |
|:---|:---|:---|
| 全部匹配 AllInOne | `:正则`(加 `-:` 为倒序) | 仅搜索列表/发现列表/详情预加载/目录列表 |
| 单一匹配 OnlyOne | `##正则##替换###`(三个 `#` 结尾) | 仅上述位置**之外**(如详情页、正文) |
| 净化替换 | `##正则##替换`(两个 `#`) | 跟在其他规则后,循环替换;单独用等同对 `all` |

## 五、变量存取与模板

- 非 JS 规则:`@put:{key:"规则"}` 存,`@get:key` 取
- JS 里**不能用 `@get`**,只能 `java.put('key',v)` / `java.get('key')`
- 搜索/发现 URL 里:`{{key}}`(关键字)、`{{page}}`(页码,从 1 起),`{{}}` 内只允许 JS,如 `{{(page-1)*20}}`
- 第一页不带页码参数:`<,{{page}}>`(简写)或 `{{page - 1 == 0 ? "" : page}}`
- 其他规则中的 `{{}}`:默认按 JS 解析;要写其他规则需加标志头 `@@`(默认规则)、`@css:`、`@json:`/`$.`、`//`/`@xpath:`
- `{}` 单花括号是阅读 2.0 遗留语法(仅 JSONPath),**不要使用**

## 六、URL 参数

URL 后用 `,` 接 JSON 对象控制请求(相对路径自动基于 `bookSourceUrl` 拼接):

```
https://example.com/search,{
  "method": "POST",
  "charset": "gbk",
  "body": "searchkey={{key}}&page={{page}}",
  "headers": {"User-Agent": "Mozilla/5.0 ..."},
  "webView": false,
  "retry": 1
}
```

字段全集见 `docs/dev/url-options.md`。注意:JS 拼 URL 时 `body` 必须是字符串,动态值用 `String()` 强转。需要 JS 渲染的页面(含音频章节)用 `"webView": true`。

## 七、书源 JSON 最小骨架

```json
{
  "bookSourceUrl": "https://www.example.com",
  "bookSourceName": "示例源",
  "bookSourceGroup": "分组",
  "bookSourceType": 0,
  "searchUrl": "/search?key={{key}}&page={{page}}",
  "ruleSearch": {
    "bookList": "class.result@tag.item",
    "name": "tag.h3@text",
    "author": "tag.span.0@text",
    "bookUrl": "tag.a.0@href"
  },
  "ruleBookInfo": {
    "name": "[property=og:novel:book_name]@content",
    "author": "[property=og:novel:author]@content",
    "intro": "[property=og:description]@content",
    "coverUrl": "[property=og:image]@content",
    "tocUrl": "text.查看目录@href"
  },
  "ruleToc": {
    "chapterList": "class.list@tag.dd",
    "chapterName": "tag.a@text",
    "chapterUrl": "tag.a@href",
    "nextTocUrl": "text.下一页@href"
  },
  "ruleContent": {
    "content": "class.content@tag.p@text",
    "nextContentUrl": "text.下一页@href"
  }
}
```

字段全集(含发现 `ruleExplore`、`bookInfoInit`、`webJs`、`sourceRegex` 等)见 `docs/dev/source-fields.md`;完整实战例子见 `docs/dev/examples.md`。

## 八、写源工作流

1. **侦察**:浏览器 F12 → Network 面板,看搜索/目录/正文是 HTML 还是 XHR JSON,记下接口、参数、编码、必要 Headers。
2. **定体系**:JSON 接口用 `$.`;HTML 用 `@css:` 或默认规则;每条规则尽量用 `##净化` 去杂质。
3. **五组规则逐级写**:searchUrl + ruleSearch → ruleBookInfo → ruleToc → ruleContent,上一级的输出 URL 是下一级的输入。
4. **自查清单**:
   - 列表规则(bookList/chapterList)只在列表字段用;OnlyOne 正则(`###` 结尾)只在非列表字段用
   - `@js:` 是否放在规则最末;JS 里取变量是否用 `java.get`
   - 翻页:目录翻页用 `nextTocUrl`,正文翻页用 `nextContentUrl`,搜索翻页靠 `{{page}}`
   - GBK 站点是否在 URL 参数里标了 `"charset":"gbk"`
   - 封面/详情链接是相对路径时无需处理,Legado 会自动补全
5. **调试**:参考 `docs/dev/debug.md`,逐段验证,先通搜索再通正文。

## 九、高频踩坑

- `@js:` 不在最后 → 后面的规则不会执行;`<js></js>` 才能嵌中间
- JS 里写 `@get:key` 取不到值(只能 `java.get`)
- 连接符 `||`/`&&`/`%%` 跨规则类型使用会失效
- AllInOne 正则忘记以 `:` 开头 → 不生效;`###` OnlyOne 用在列表里 → 不生效
- 搜索 URL 忘写 `{{key}}`;POST body 拼接的数字没 `String()` 强转
- `{{}}` 里写默认规则却没加 `@@` 标志头
- 章节列表网站本身倒序 → `chapterList` 最前加 `-`
- 详情页规则可用 AllInOne 预加载(`bookInfoInit`)一次抓全,但只支持 `:` 正则或返回 JSON 对象的 JS

## 十、内置文档索引

| 路径 | 内容 | 什么时候读 |
|:---|:---|:---|
| `docs/dev/syntax.md` | 8 种规则语法详解 | 写任何规则前快速核对 |
| `docs/dev/source-fields.md` | 书源 JSON 全部字段 + 完整骨架 | 搭源结构时 |
| `docs/dev/url-options.md` | URL 参数(GET/POST/WebView/模板变量) | 写 searchUrl/tocUrl 等 |
| `docs/dev/rule.md` | 规则标志、jsLib、并发率、字体解密、图片解密等进阶项 | 源需要特殊配置时 |
| `docs/dev/js.md` | JS 环境:内置变量、`java.*` 全部方法、book/chapter/source/cookie/cache 对象 | 写 JS 规则时 |
| `docs/dev/xpath.md` | XPath 完整语法 | 用 XPath 时 |
| `docs/dev/regex.md` | 三种正则形态详解 | 用正则时 |
| `docs/dev/examples.md` | 完整源示例(CSS+正则 / XPath / JSONPath)与调试技巧 | 找参考写法时 |
| `docs/dev/request-headers.md` | 请求头/UA/代理配置 | 站点有反爬时 |
| `docs/dev/authentication.md` | 登录、loginUrl、JS 注入 Cookie | 站点需登录时 |
| `docs/dev/discovery-url.md` | 发现页 URL(分类数组)写法 | 配 exploreUrl 时 |
| `docs/dev/debug.md` | 源调试方法 | 验证/排错时 |
| `docs/guide/replace-rule.md` | 替换净化规则 | 写替换规则时 |
| `docs/guide/rss-source.md` | RSS 订阅源 | 写订阅源时 |
| `docs/dev/dict-rule.md` / `docs/dev/tts-rule.md` / `docs/dev/txt-toc.md` | 字典规则 / 在线朗读规则 / TXT 目录正则 | 对应特殊源类型 |
| `docs/spec/homepage-modules.md` / `docs/spec/related-books.md` / `docs/spec/mime-types.md` | 首页模块 / 关联书籍 / MIME 类型 | 对应扩展功能 |

文档镜像自 [HapeLee/legado-with-MD3](https://github.com/HapeLee/legado-with-MD3) 的 `docs/` 目录(Apache-2.0),在线版:<https://hapelee.github.io/legado-with-MD3/dev/syntax.html>。更新方法见 `docs/README.md`。
