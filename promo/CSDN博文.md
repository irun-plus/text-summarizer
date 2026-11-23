---
title: 从零写一个 WorkBuddy Skill：最小可用 text-summarizer 全流程（附踩坑记录）
tags: [WorkBuddy, AI Agent, Skill开发, 开源, 效率工具]
category: AI工具实战
---

# 从零写一个 WorkBuddy Skill：最小可用 text-summarizer 全流程（附踩坑记录）

> 摘要：本文记录了我从零创建一个最小可用 WorkBuddy 技能（文字摘要助手）的完整过程——从 SKILL.md 的五要素设计、校验打包，到 GitHub 上传、市场发布字段配置，再到推广素材准备。中间踩了不少坑（代理失效、DNS 解析失败、API 提交 422），一并记录，供想上手 Skill 开发的朋友参考。全程零代码，一个 Markdown 文件就能搞定。

## 一、为什么写这个 Skill

WorkBuddy 的 Skill 机制，本质是把"一套做事方法"沉淀成可复用的手册，让 AI 按规矩执行。我选了最经典的入门场景——**文字摘要**：

- 输入：一段长文本，或 `.txt` / `.md` 文件路径
- 输出：一句话总括（≤50 字）+ 3~5 条要点
- 触发：用户说"总结/摘要/提炼要点"，或没指定用途但需要压缩文字

选它的理由很简单：需求真实、逻辑清晰、五要素齐全，是练手"最小可用 Skill"的最佳范本。

## 二、Skill 的核心：一个 SKILL.md

整个技能只有一个文件 `SKILL.md`，由 YAML frontmatter + Markdown 正文组成。frontmatter 里 `name` 和 `description` 决定 AI 何时调用它：

```yaml
---
name: text-summarizer
description: 将长文本压缩为简洁摘要。当用户要求总结、摘要、提炼要点，或未指定用途但需要压缩一段文字时，应使用本 Skill。
agent_created: true
---
```

正文必须回答四个问题，这就是"五要素"里的后四个：

| 要素 | 内容 |
|------|------|
| 触发条件 | 什么时候用这个技能 |
| 输入 | 必填什么、可选什么、默认值 |
| 步骤 | 获取→清洗→提取→生成→自检 |
| 输出 | 一句话总括 + 3~5 条要点 |
| 调用示例 | 一个完整的用户输入→预期输出 |

**写作要点**：用祈使句/不定式（"读取文本""提取要点"），不要用第二人称"你应该"。因为这是写给另一个 AI 实例看的操作手册，不是给人看的说明文。

完整的 SKILL.md 见仓库：[github.com/irun-plus/text-summarizer](https://github.com/irun-plus/text-summarizer)

## 三、校验与打包

WorkBuddy 自带 `skill-creator`，初始化、校验、打包都有现成脚本：

```bash
# 初始化骨架
python init_skill.py text-summarizer --path ./workspace

# 校验（检查 frontmatter、目录结构、描述质量）
python quick_validate.py ./text-summarizer

# 打包成 zip
python package_skill.py ./text-summarizer ./dist
```

校验通过会输出 `Skill is valid!`。注意：默认的打包脚本会把 `.git` 目录一起打进去，所以我用 Python 手动打了个纯净包，只含 `text-summarizer/SKILL.md`，体积从 19KB 降到 1.5KB。

## 四、上传 GitHub：踩坑最密集的环节

这部分是我花时间最多的地方，记录下来帮大家避坑。

### 坑 1：浏览器 OAuth 登录超时

Git Credential Manager 触发浏览器登录时，用 `127.0.0.1` 做回调。我第一次把认证命令的输出管道截断（`| head`），导致本地回调服务提前关闭，浏览器访问 `127.0.0.1` 直接超时。

**教训**：认证流程不要截断进程，给足超时时间。

### 坑 2：环境代理失效，curl / git 全部超时

环境里注入了一个 `http_proxy=http://127.0.0.1:5083`，但本地并没有这个代理服务，所有 HTTP 请求都卡在连接代理上。

```bash
curl: (28) Failed to connect to 127.0.0.1 port 5083 after 21037 ms
```

### 坑 3：git 的 libcurl DNS 解析失败

绕过代理后，git push 又报：

```
fatal: unable to access ...: getaddrinfo() thread failed to start
```

这是环境里 git 自带 libcurl 的线程问题，curl 命令也反复段错误。

### 解决方案：改用 PowerShell 的 .NET HTTP 栈

既然 git/curl 都不通，我直接用 GitHub REST API 完成所有操作：

```powershell
$headers = @{ Authorization = "Bearer $token"; 'User-Agent' = 'WorkBuddy' }

# 建仓库
$body = @{ name='text-summarizer'; private=$false } | ConvertTo-Json
Invoke-RestMethod -Uri 'https://api.github.com/user/repos' -Method Post -Headers $headers -Body $body

# 提交文件
$body = @{ message='Add skill'; content=[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($text)); branch='main' } | ConvertTo-Json
Invoke-RestMethod -Uri 'https://api.github.com/repos/{user}/{repo}/contents/SKILL.md' -Method Put -Headers $headers -Body $body

# 建 Release 并上传 zip 附件
$assetHeaders = @{ Authorization = "Bearer $token"; 'Content-Type' = 'application/zip' }
Invoke-RestMethod -Uri "https://uploads.github.com/repos/{user}/{repo}/releases/{id}/assets?name=text-summarizer.zip" -Method Post -Headers $assetHeaders -Body $bytes
```

PowerShell 的 `Invoke-RestMethod` 走 .NET 网络栈，不受上面那些问题影响，一次成功。最后用 SHA256 比对确认本地和远端文件字节级一致。

### 坑 4：API 更新文件必须带 SHA

更新已有文件时，若 body 里不带当前 blob 的 `sha`，GitHub 返回 422：

```powershell
$cur = Invoke-RestMethod -Uri '.../contents/SKILL.md' -Headers $headers
$body = @{ message='update'; content=$base64; sha=$cur.sha; branch='main' }
```

同理，更新 Release 附件要先删旧 asset 再上传。

## 五、为技能市场发布做准备

查到的发布渠道有两个：**WorkBuddy 开放平台**和**腾讯 SkillHub**。它们对 SKILL.md 的 frontmatter 有额外要求，缺字段会被拒绝解析：

```yaml
version: 1.0.0
display_name: 文字摘要助手
display_name_en: Text Summarizer
description_zh: 把长文本压缩为简洁摘要……
description_en: Compress long text into a concise summary……
```

注意 `name` 只能用小写字母、数字和连字符（kebab-case）。我把这些字段和原有字段共存，一份 SKILL.md 同时满足本地使用和市场上传。

SkillHub 还支持 CLI 发布和绑定 GitHub 自动导入，适合后续频繁更新的场景。

## 六、推广素材

Skill 写好只是第一步，让别人用起来才是关键。我准备了：

- **README**：彩色徽章 + 安装/使用/FAQ，别人点进仓库一眼看懂
- **Release zip**：一键安装包，下载解压即可
- **宣传卡片**：HTML 页面，浏览器打开截图就能发社交平台
- **多平台文案**：朋友圈、小红书、知乎、开发者群各一版

一个体会：对这类小工具，**上架技能市场比发朋友圈有效得多**——市场是目标用户聚集地，别人搜"摘要"就能看到；朋友圈只是熟人捧场。优先级建议：市场 > 技术社区 > 朋友圈。

## 七、总结

回顾整个过程：

1. 一个 SKILL.md 就能做出最小可用 Skill，门槛比想象中低
2. 五要素（触发/输入/步骤/输出/示例）是核心，写清楚 AI 就能正确执行
3. 环境网络问题多时，GitHub API + PowerShell 是可靠的兜底路径
4. 打包、README、Release、市场字段，这些"周边"决定别人能不能真的用起来

这个 Skill 已开源，欢迎 star、试用、提 issue：
**https://github.com/irun-plus/text-summarizer**

---

*本文由 WorkBuddy 辅助创作，记录了真实的开发过程。*
