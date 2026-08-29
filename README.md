# Cell-lct Image 2 绘图

可安装 Agent 的技能，使用用户配置的 OpenAI-compatible Image 2 中转站生成图片，为 `cell-lct` 提供 PNG 参考图。技能包内置仅依赖 Python 标准库的客户端，不依赖 MCP 常驻服务、本机固定路径或额外 Python 包。

## 安装

需要 Python 3.11 或更高版本。

### 在对话中安装

新建一个对话，发送以下内容：

```text
请从 https://github.com/Agoniedi/cell-lct-image/tree/v1.2.5/skills/cell-lct-image 安装 cell-lct-image 技能（当前最新稳定版 v1.2.5）。
```

### 使用命令安装

```powershell
npx.cmd skills add "https://github.com/Agoniedi/cell-lct-image/tree/v1.2.5/skills/cell-lct-image" -g -y
```

安装完成后，重新打开 Agent。技能会在文生图、参考图生图、科研图或包含指定清晰文字的图片请求中自动启用。

## 首次配置

不要在聊天中发送中转站地址或 API 密钥。在 PowerShell 中粘贴并运行以下代码，输入时密钥内容不会显示：

```powershell
$OutputEncoding = [Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
$baseUrl = Read-Host "请输入 Image 2 中转站地址"
$secureKey = Read-Host "请输入 Image 2 API 密钥" -AsSecureString
$plainKey = [System.Net.NetworkCredential]::new("", $secureKey).Password
[Environment]::SetEnvironmentVariable("CELL_LCT_IMAGE_BASE_URL", $baseUrl, "User")
[Environment]::SetEnvironmentVariable("CELL_LCT_IMAGE_API_KEY", $plainKey, "User")
Remove-Variable plainKey, baseUrl
Write-Host "配置完成。请完全退出并重新打开 Agent，然后重新发送生图请求。"
```

中转站应提供 `/images/generations` 和 `/images/edits`。若接口实际路径包含 `/v1`，请将 `/v1` 一并写入 `CELL_LCT_IMAGE_BASE_URL`。

## 干净卸载

仅删除技能目录不会清除首次配置时写入的用户环境变量。若不再使用本技能，可在对话中发送：

```text
请卸载 cell-lct-image（Cell-lct Image 2 绘图）技能，并删除用户环境变量 CELL_LCT_IMAGE_BASE_URL 和 CELL_LCT_IMAGE_API_KEY。不要显示密钥，也不要删除生成的图片或修改其他环境变量。完成后只报告技能目录和配置是否仍存在，并提醒我完全退出并重新打开 Codex。
```

也可以在 PowerShell 中手动清理配置：

```powershell
$OutputEncoding = [Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
[Environment]::SetEnvironmentVariable("CELL_LCT_IMAGE_API_KEY", $null, "User")
[Environment]::SetEnvironmentVariable("CELL_LCT_IMAGE_BASE_URL", $null, "User")
Remove-Item Env:CELL_LCT_IMAGE_API_KEY -ErrorAction SilentlyContinue
Remove-Item Env:CELL_LCT_IMAGE_BASE_URL -ErrorAction SilentlyContinue
$configStillExists = -not [string]::IsNullOrEmpty([Environment]::GetEnvironmentVariable("CELL_LCT_IMAGE_API_KEY", "User")) -or -not [string]::IsNullOrEmpty([Environment]::GetEnvironmentVariable("CELL_LCT_IMAGE_BASE_URL", "User"))
Write-Host "用户环境变量仍存在: $configStillExists"
Remove-Variable configStillExists
```

命令应显示 `用户环境变量仍存在: False`。已经运行的进程仍可能保留启动时继承的旧值，因此清理后必须完全退出 Agent、结束后台进程并重新打开。干净卸载不会删除此前生成的图片，也不会修改其他环境变量。

## 功能

- 文生图：根据提示词创建图片。
- 参考图生图：使用一张或多张本地参考图片生成新图。
- 文字生图：要求图中完整呈现指定、清晰可读的文字。

默认使用模型 `gpt-image-2`、尺寸 `1536x1024` 和质量 `standard`。可用模型为 `gpt-image-1`、`gpt-image-1.5`、`gpt-image-2`；可用尺寸为 `1024x1024`、`1536x1024`、`1024x1536`；可用质量为 `low`、`standard`、`high`。用户显式指定的参数始终优先。生成的 PNG 可继续交给 `$cell-lct` 做矢量化。

## 批量生成

同一提示词可使用 `--count 2` 至 `--count 10` 在一次请求中生成多个版本，单次最多 10 张。多个不同提示词使用 `batch --prompt`，每批重复传入 2 至 4 个 `--prompt`，并发生成上限为 4 张；脚本会并发执行并按输入顺序输出成功图片路径。

### 使用命令

```powershell
python "skills/cell-lct-image/scripts/cell_lct_image.py" generate --prompt "同一提示词" --count 2
python "skills/cell-lct-image/scripts/cell_lct_image.py" batch --prompt "第一张" --prompt "第二张"
```

全部成功时退出码为 `0`。部分失败时已成功图片仍保留并输出，失败批次项写入标准错误，退出码为 `1`。创建请求不会自动重试或切换生成方式。读取请求仅在首次出现 DNS 解析失败、TLS 连接失败、连接被拒绝或代理连接失败时最多自动重试 1 次；超时及其他网络错误不重试。

## 维护者发布验证

本节仅面向维护者；普通安装用户可跳过。每个发布标签前从 `main` 分支运行以下离线发布门禁：

```powershell
python -m unittest discover -s tests -v
python -m unittest discover -v
python -m compileall -q skills tests
python skills/cell-lct-image/scripts/cell_lct_image.py --help
python skills/cell-lct-image/scripts/cell_lct_image.py generate --help
python skills/cell-lct-image/scripts/cell_lct_image.py edit --help
python skills/cell-lct-image/scripts/cell_lct_image.py text --help
python skills/cell-lct-image/scripts/cell_lct_image.py batch --help
git diff --check HEAD^ HEAD
```

真实 API 冒烟测试是正式发布前的人工门禁，不属于 CI：使用独立低额度测试密钥和自有中转站生成一张无文字图片并验证 PNG；只有文字图相关变更才额外生成中文文字图，人工确认文字完整、清晰且没有额外文案。默认测试和 PR 检查不读取 API 密钥，也不访问中转站。
