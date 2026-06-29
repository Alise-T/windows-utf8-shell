# windows-utf8-shell

一个给 Codex 使用的轻量 skill，用来降低 Windows PowerShell 环境下的中文和 UTF-8 编码问题。

## 为什么需要它

Codex 在 Windows PowerShell 里默认读取或传递中文内容时，容易把文本变成乱码。

即使某次手动改了读取方式，换线程、上下文压缩或重新开始任务后，Codex 也可能回到默认命令。结果就是同一个中文乱码问题反复出现。

这不只会影响文件读取和报告生成。调用模型 API 时，如果中文 prompt 或 JSON 请求内容在 shell 层已经损坏，模型收到的就不是原始中文。codex自己排查这类问题也会浪费时间和 token。

本 skill 的作用是把更稳的读取、写入和传输方式固定下来，当 Codex 在 Windows 上处理中文、UTF-8 文件、JSON、Markdown 或 API 请求内容时，优先避开容易破坏中文的 PowerShell 路径。

## 主要解决的问题

- 读取或写入中文 Markdown、YAML、JSON、JSONL 时出现乱码
- PowerShell here-string、`echo`、重定向导致中文 prompt 被破坏
- 调用模型 API 时，中文请求内容变成 `??`、乱码或空回复
- Windows PowerShell 5.1 和 PowerShell 7 对 `-Encoding utf8` 行为不一致
- Codex 在处理中文文件、报告、日志、API 请求内容时误判为模型问题

## 适合场景

- 在 Windows 上用 Codex 读取或修改中文项目文件
- 生成包含中文的 Markdown、YAML、JSON、JSONL 报告
- 调试模型 API 请求，尤其是中文 prompt、JSON body、日志输出
- 排查 `??`、`锟斤拷`、`鈥`、`Ã`、`Â`、`mojibake` 等编码异常

## 安装方式

把本仓库放到 Codex skills 目录下：

```text
%USERPROFILE%\.codex\skills\windows-utf8-shell
```

目录结构应类似：

```text
windows-utf8-shell/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── scripts/
    └── utf8_prelude.ps1
```

当 Codex 在 Windows PowerShell 中处理中文、UTF-8 文件、JSON、Markdown、模型 API 请求内容时，会读取 `SKILL.md` 中的规则。

## 手动加载 PowerShell 预处理脚本

如果需要手动使用，可以在命令前加载：

```powershell
$codexHome = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { Join-Path $env:USERPROFILE ".codex" }
. (Join-Path $codexHome "skills\windows-utf8-shell\scripts\utf8_prelude.ps1")
```

它会设置 Python、PowerShell 控制台和管道输出的 UTF-8 相关环境。

## 工作准则

本 skill 不负责修复已经损坏的文本。它的作用是让 Codex 在读取、写入、传输中文内容前，优先选择更安全的命令和编码路径。

- 中文 prompt、JSON 请求内容、Markdown 文本不要通过 PowerShell here-string 或 `echo` 直接传递
- 稳定文件优先用 `apply_patch`
- 生成报告或精确 UTF-8 文件时优先用 Python 写入
- PowerShell 读取文件时使用 `-LiteralPath -Raw -Encoding UTF8`
- 发现乱码时，先怀疑 shell 编码链路，再怀疑模型或 API

## License

MIT
