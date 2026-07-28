---
title: AI 辅助贡献政策
authors: Wale Soyinka, Steven Spencer, Documentation Team
contributors: Steven Spencer, Ganna Zhyrnova
---

!!! note

    本 Rocky Linux 文档项目的 AI 辅助贡献政策基于并扩展了由 Fedora 项目制定的 [AI 辅助贡献政策](https://docs.fedoraproject.org/en-US/council/policy/ai-contribution-policy/)。本政策可能会有更改和修订。

您**可以(MAY)**使用 AI 辅助工具为 Rocky Linux 文档项目做出贡献，但必须遵循下述原则。

## 问责制(Accountability)

- 您**必须(MUST)**对自己的贡献负责。
- 为 Rocky Linux 做贡献意味着担保您提交内容的质量、许可证合规性和实用性。
- 所有贡献，无论来自人类作者还是借助大语言模型(LLMs)或其他生成式 AI 工具，都必须符合项目的收录标准。
- 贡献者始终是作者，并对这些贡献的全部内容负完全责任。

## 透明度(Transparency)

- 当您的贡献中有重要部分直接来自 AI 工具且未经任何修改时，您**必须(MUST)**披露您使用了 AI 工具。
- 对于 AI 工具的其他用途，您**应该(SHOULD)**在有用的情况下进行披露。
- 对语法和拼写纠正或语言清晰度改进的辅助工具的常规使用，无需披露。关于 AI 工具使用情况的信息将帮助我们评估其影响、建立新的最佳实践并调整现有流程。作者应在通常标明作者身份的地方进行披露。
- 对于在 git 中跟踪的贡献，推荐的方法是使用 `Assisted-by:` 提交消息尾部(trailer)。
- 对于贡献的内容，作者必须在文档前言或其他文档元数据部分包含披露信息。

示例：

  ```text
  ---
  title: 
  author: Steven Spencer
  contributors: Ganna Zhyrnova, Colussi Franco, tianci li, Wale Soyinka 
  ai-contributors: Claude (claude-sonnet-4-20250514), Gemini (gemini-2.5-pro)
  ---
  ```

## 贡献与社区评估

- AI 工具可以通过提供分析和建议来协助人类审查者。
- 您**不得(MUST NOT)**使用 AI 作为对贡献做出实质性或主观判断的唯一或最终仲裁者，也不得用于评估个人在社区中的身份（例如资金、领导角色或行为准则事务）。
- 这并不禁止使用自动化工具进行客观的技术验证，例如 CI/CD 流水线(CI/CD pipelines)、自动化测试或垃圾邮件过滤。
- 接受贡献的最终责任，即使由自动化系统实施，始终由授权该操作的人类贡献者承担。

## 大规模倡议

本政策不涵盖大规模倡议，这些倡议可能会显著改变项目的运作方式，或导致项目某些部分的贡献呈指数级增长。此类倡议需要与项目领导层单独讨论。

## 尊重现有贡献

- 您**不得(MUST NOT)**提交主要由其他贡献者的作品衍生或实质性重写的 AI 生成贡献。
- AI 辅助编辑**应该(SHOULD)**保留原作者的意图、语气和结构。

有关可能违反政策的担忧，应通过[此链接提交为 issue](https://github.com/rocky-linux/documentation/issues)

本文档中的关键词 "MAY"(可以)、"MUST"(必须)、"MUST NOT"(不得)和 "SHOULD"(应该)应按照 <https://datatracker.ietf.org/doc/html/rfc2119>[RFC 2119] 中的描述进行解释。
