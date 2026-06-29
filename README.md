# windows-utf8-shell

一个给 Codex 使用的轻量 skill，用来降低 Windows PowerShell 环境下的中文和 UTF-8 编码问题。

## 为什么需要它

Codex 在 Windows 上执行命令时，经常会通过 PowerShell 读取文件、拼接脚本、运行 Python/Node、调用模型 API 或生成报告。

这些默认路径在处理中文时并不总是安全：

- `Get-Content` 默认读取方式可能不符合 UTF-8 预期
- PowerShell here-string、`echo`、管道传递内联脚本时，中文 prompt 可能在进入 Python 或 API 前已经损坏
- PowerShell 5.1 和 PowerShell 7 对 `-Encoding utf8` 的行为不完全一致
- 通过重定向、`Out-File`、未指定编码的写入方式生成中文文件时，可能出现 BOM、乱码或问号
- JSON 请求内容如果在 shell 层被破坏，模型收到的就不是原始中文

这类问题很容易被误判为“模型没理解中文”“API 返回异常”或“文件内容本来就坏了”。  
这个 skill 的目标是让 Codex 在触碰中文、UTF-8 文件、JSON 和 API 请求内容时，优先使用更稳的读取、写入和传输方式。

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
