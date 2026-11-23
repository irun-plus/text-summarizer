# text-summarizer

一个最小可用的 WorkBuddy Skill：把长文本压缩成简洁摘要。

## 它能做什么

- 总结一段文字、提炼要点
- 读取 `.txt` / `.md` 文件内容并摘要
- 别人说「总结 / 摘要 / 提炼要点」时自动触发，未指定用途时默认按文字摘要处理

## 如何安装

### 方式一：用户级安装（所有项目都能用）

1. 下载本仓库：点页面右上角绿色 `Code` → `Download ZIP`（或 git clone）
2. 解压后，把里面的 `text-summarizer` 文件夹整个复制到：
   - Windows：`C:\Users\你的用户名\.workbuddy\skills\`
   - macOS / Linux：`~/.workbuddy/skills/`
3. 重启 WorkBuddy，Skill 即可生效

### 方式二：项目级安装（只在当前项目能用）

把 `text-summarizer` 文件夹放到项目的 `.workbuddy/skills/` 目录下。

## 怎么用

安装好后，直接对 WorkBuddy 说：

- 「用 text-summarizer 总结下面这段：……」
- 「帮我提炼这段文字的要点」
- 或者粘贴一段文字，说「总结」

Skill 会输出：**一句话总括 + 3～5 条要点**。

完整规则（触发条件、输入、步骤、输出、调用示例）见 [SKILL.md](SKILL.md)。

## 许可

自由使用、修改、分享。