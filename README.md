# legado-ai

让 AI 写出符合 [Legado(开源阅读)](https://github.com/gedoor/legado) 规范的书源规则的规则库。

## 用法

把本仓库作为 AI 的工作区:任何支持 `AGENTS.md` 约定的 AI 编程工具(ZCode、Claude Code、Codex 等)进入本仓库后会自动加载 [AGENTS.md](AGENTS.md),获得:

- Legado 规则前缀路由与语法速查(默认规则 / CSS / JSONPath / XPath / JS / 正则 / 模板变量)
- 书源 JSON 字段骨架与写源工作流
- 高频踩坑清单
- 完整离线文档:[`docs/`](docs/README.md),镜像自 [legado-with-MD3 官方文档](https://hapelee.github.io/legado-with-MD3/)(Apache-2.0)

写源时只需对 AI 说"帮我写 xxx 网站的书源",AI 会依据内置文档产出符合规范的规则。

## 仓库结构

```
AGENTS.md        AI 工作规则 + 语法速查(核心,自动加载)
docs/            Legado 规则官方文档离线镜像(dev / guide / spec)
```

## 更新文档

见 [docs/README.md](docs/README.md) 的"更新方法"。

## 许可

- 本仓库代码与 AGENTS.md:MIT
- `docs/` 目录内容:来自 [HapeLee/legado-with-MD3](https://github.com/HapeLee/legado-with-MD3),遵循其 Apache-2.0 许可
