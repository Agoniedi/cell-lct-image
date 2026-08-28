# Lumenverba 绘图

可安装 Agent 的技能，使用 Lumenverba 图像 API 生成图片。技能包内置仅依赖 Python 标准库的客户端，不依赖 MCP 常驻服务、本机固定路径或额外 Python 包。

## 安装

需要Python 3.11 或更高版本：

### 在对话中安装

新建一个对话，发送以下内容：

```text
请从 https://github.com/yixixi-yahaha/lumenverba-image/tree/v1.2.5/skills/lumenverba-image 安装 lumenverba-image 技能（当前最新稳定版 v1.2.5）。
```

### 使用命令安装

```powershell
npx.cmd skills add "https://github.com/yixixi-yahaha/lumenverba-image/tree/v1.2.5/skills/lumenverba-image" -g -y
```

当前最新稳定版为 `v1.2.5`。安装命令固定到发布标签；更新时请改用新的发布标签 URL，而不要继续使用旧版本链接。安装完成后，重新打开 Agent。技能会在文生图、参考图生图、海报、角色图、插画或包含指定清晰文字的图片请求中自动启用。

## 首次配置密钥

API 密钥来源于 [LumenVerba](https://lumenverba.cc/home) 网站。不要在聊天中发送 API 密钥。在 PowerShell 中粘贴并运行以下代码，输入时内容不会显示：

```powershell
$OutputEncoding = [Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
$secureKey = Read-Host "请输入 Lumenverba API 密钥" -AsSecureString
$plainKey = [System.Net.NetworkCredential]::new("", $secureKey).Password
[Environment]::SetEnvironmentVariable("LUMENVERBA_API_KEY", $plainKey, "User")
Remove-Variable plainKey
Write-Host "配置完成。请完全退出并重新打开 Agent，然后重新发送生图请求。"
```

技能固定使用 `https://api.lumenverba.cc/v1`，无需配置 API 地址。

## 干净卸载

仅删除技能目录不会清除首次配置时写入的用户环境变量。若不再使用本技能，可在对话中发送：

```text
请卸载 lumenverba-image（Lumenverba 绘图）技能，并删除用户环境变量 LUMENVERBA_API_KEY。不要显示密钥，也不要删除生成的图片或修改其他环境变量。完成后只报告技能目录和密钥是否仍存在，并提醒我完全退出并重新打开 Codex。
```

也可以在 PowerShell 中手动清理密钥：

```powershell
$OutputEncoding = [Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
[Environment]::SetEnvironmentVariable("LUMENVERBA_API_KEY", $null, "User")
Remove-Item Env:LUMENVERBA_API_KEY -ErrorAction SilentlyContinue
$keyStillExists = -not [string]::IsNullOrEmpty([Environment]::GetEnvironmentVariable("LUMENVERBA_API_KEY", "User"))
Write-Host "用户环境变量仍存在: $keyStillExists"
Remove-Variable keyStillExists
```

命令应显示 `用户环境变量仍存在: False`。已经运行的进程仍可能保留启动时继承的旧值，因此清理后必须完全退出 Agent、结束后台进程并重新打开。干净卸载不会删除此前生成的图片，也不会修改其他环境变量。

## 功能

- 文生图：根据提示词创建图片。
- 参考图生图：使用一张或多张本地参考图片生成新图。
- 文字生图：要求图中完整呈现指定、清晰可读的文字。

默认使用模型 `gpt-image-2`、尺寸 `1536x1024` 和质量 `standard`。可用模型为 `gpt-image-1`、`gpt-image-1.5`、`gpt-image-2`；可用尺寸为 `1024x1024`、`1536x1024`、`1024x1536`；可用质量为 `low`、`standard`、`high`。用户显式指定的参数始终优先。

## 批量生成

同一提示词可使用 `--count 2` 至 `--count 10` 在一次请求中生成多个版本，单次最多 10 张。多个不同提示词使用 `batch --prompt`，每批重复传入 2 至 4 个 `--prompt`，并发生成上限为 4 张；脚本会并发执行并按输入顺序输出成功图片路径。

### 在 Agent 中使用

同一提示词一次生成多张图片：

```text
请根据以下提示词生成 4 张不同版本的图片：雨夜中的上海街头，电影感摄影，霓虹灯倒映在路面上。
```

多个不同提示词并发生成：

```text
请同时生成两张图片：
1. 清晨薄雾中的江南水乡，写实摄影风格。
2. 火星基地内的植物温室，科幻概念艺术风格。
```

### 使用命令

```powershell
python "skills/lumenverba-image/scripts/lumenverba_image.py" generate --prompt "同一提示词" --count 2
python "skills/lumenverba-image/scripts/lumenverba_image.py" batch --prompt "第一张" --prompt "第二张"
```

全部成功时退出码为 `0`。部分失败时已成功图片仍保留并输出，失败批次项写入标准错误，退出码为 `1`。创建请求不会自动重试或切换生成方式。读取请求仅在首次出现 DNS 解析失败、TLS 连接失败、连接被拒绝或代理连接失败时最多自动重试 1 次；超时及其他网络错误不重试。

## 维护者发布验证

本节仅面向维护者；普通安装用户可跳过。每个发布标签前从 `main` 分支运行以下离线发布门禁：

```powershell
python -m unittest discover -s tests -v
python -m unittest discover -v
python -m compileall -q skills tests
python skills/lumenverba-image/scripts/lumenverba_image.py --help
python skills/lumenverba-image/scripts/lumenverba_image.py generate --help
python skills/lumenverba-image/scripts/lumenverba_image.py edit --help
python skills/lumenverba-image/scripts/lumenverba_image.py text --help
python skills/lumenverba-image/scripts/lumenverba_image.py batch --help
git diff --check HEAD^ HEAD
```

真实 API 冒烟测试是正式发布前的人工门禁，不属于 CI：使用独立低额度测试密钥生成一张无文字图片并验证 PNG；只有文字图相关变更才额外生成中文文字图，人工确认文字完整、清晰且没有额外文案。默认测试和 PR 检查不读取 API 密钥，也不访问 Lumenverba。
