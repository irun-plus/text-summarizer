<div align="center">

# text-summarizer

**一个最小可用的 WorkBuddy 技能 —— 把长文本压缩成简洁摘要**

![version](https://img.shields.io/badge/version-1.0.0-brightgreen)
![platform](https://img.shields.io/badge/platform-WorkBuddy-blue)
![license](https://img.shields.io/badge/license-MIT-orange)
![status](https://img.shields.io/badge/status-ready-success)

</div>

---

## 它解决什么问题

长文章、长报告、长对话看不过来？这个技能几秒钟帮你抓住重点：

- **一句话总括** —— 50 字内说清全文在讲什么
- **3～5 条要点** —— 关键结论、数据、行动项一目了然
- **自动触发** —— 说「总结」「摘要」「提炼要点」就生效，不用记复杂命令

## 快速开始

### 第一步：安装

| 方式 | 适用场景 | 操作 |
|------|----------|------|
| 用户级 | 所有项目都能用 | 把 `text-summarizer` 文件夹复制到 `~/.workbuddy/skills/`（Windows：`C:\Users\你的用户名\.workbuddy\skills\`），重启 WorkBuddy |
| 项目级 | 只在某个项目用 | 把 `text-summarizer` 文件夹放到项目的 `.workbuddy/skills/` 目录 |

也可以点仓库右上角绿色 **Code → Download ZIP** 下载整个仓库。

### 第二步：使用

安装好后，直接对 WorkBuddy 说：

> 用 text-summarizer 总结下面这段：
> 人工智能助手正在改变人们获取信息的方式……

### 第三步：看结果

```
一句话总括：AI 助手能快速摘要长文，但摘要仅供参考，关键决策需回查原文。

要点：
- 几秒内完成长文档阅读与摘要，提升信息获取效率
- 摘要可能遗漏细节，不适合作为唯一依据
- 专家建议将 AI 摘要用于初筛，重要决策回查原文
```

## 技能能做什么

| 输入 | 输出 |
|------|------|
| 直接粘贴的文字 | 一句话总括 + 3～5 条要点 |
| `.txt` / `.md` 文件路径 | 同上，自动读取文件内容 |
| 未指定用途的文字 | 默认按文字摘要处理 |

可选参数：摘要长度（默认 ≤150 字）、输出风格（默认要点式）、输出语言（默认与原文一致）。

## 项目结构

```
text-summarizer/
└── SKILL.md    # 技能定义：触发条件、输入、步骤、输出、调用示例
```

## 常见问题

**Q：怎么知道技能装好了？**
A：重启 WorkBuddy 后，说一句「总结这段文字」，按上面的格式输出就是成功了。

**Q：支持多长的文章？**
A：理论上不限，但极长的文档建议分段摘要，效果更好。

**Q：能改规则吗？**
A：能。编辑 `SKILL.md` 里的步骤和输出格式即可，比如把要点数改成 10 条。

## 许可

自由使用、修改、分享。

---

Made with WorkBuddy · 最小可用技能示例