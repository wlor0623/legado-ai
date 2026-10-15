# 内置文档

本目录镜像自 [HapeLee/legado-with-MD3](https://github.com/HapeLee/legado-with-MD3) 仓库的 `docs/` 目录(原项目采用 Apache-2.0 许可),是 Legado 书源规则的官方文档源文件(markdown),供 AI 离线查阅。

- 在线版:<https://hapelee.github.io/legado-with-MD3/>
- 规则语法详解:<https://hapelee.github.io/legado-with-MD3/dev/syntax.html>

## 目录结构

```
docs/
├── dev/               开发文档(核心)
│   ├── syntax.md          规则语法详解 ★ 最常用
│   ├── source-fields.md   源字段速查(书源 JSON 全部字段)
│   ├── url-options.md     URL 参数详解
│   ├── rule.md            源规则帮助(规则标志/jsLib/字体解密等)
│   ├── js.md              JS 变量和函数(java.* API 全集)
│   ├── xpath.md           XPath 路径表达式
│   ├── regex.md           正则表达式三种形态
│   ├── examples.md        源示例
│   ├── request-headers.md 请求头配置
│   ├── authentication.md  认证与登录
│   ├── discovery-url.md   发现 URL 配置
│   ├── debug.md           源调试
│   ├── dict-rule.md       字典规则
│   ├── tts-rule.md        在线朗读规则
│   ├── txt-toc.md         TXT 目录正则
│   └── index.md           开发文档索引
├── guide/             使用入门
│   ├── book-source.md     书源导入管理
│   ├── replace-rule.md    替换净化规则
│   ├── rss-source.md      RSS 订阅源
│   └── index.md
└── spec/              配置规范
    ├── homepage-modules.md  首页模块配置
    ├── related-books.md     关联书籍配置
    ├── mime-types.md        MIME 类型参考
    └── index.md
```

## 更新方法

从上游仓库拉取最新版(在仓库根目录执行):

```bash
BASE="https://raw.githubusercontent.com/HapeLee/legado-with-MD3/main/docs"
for f in dev/syntax dev/source-fields dev/url-options dev/rule dev/js dev/xpath \
         dev/regex dev/examples dev/request-headers dev/authentication \
         dev/discovery-url dev/debug dev/dict-rule dev/tts-rule dev/txt-toc \
         guide/book-source guide/replace-rule guide/rss-source \
         spec/homepage-modules spec/related-books spec/mime-types; do
  mkdir -p "docs/$(dirname "$f")"
  curl -sfL --retry 5 "$BASE/$f.md" -o "docs/$f.md" && echo "OK $f" || echo "FAIL $f"
done
```

> 注意:上游 `docs/` 里还有一些 app 本身的开发计划文档(`track-*`、`mad-modernization-plan.md` 等),与书源规则无关,不纳入镜像。
