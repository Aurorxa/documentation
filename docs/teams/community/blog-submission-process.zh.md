# Rocky Linux 博客提交流程 (Blog Submission Process)

本文档涵盖从构思到推广，在 Rocky Linux 网站上撰写和发布博客文章的全流程。

---

## 项目看板 (Project Board)

所有博客工作都在 [Rocky Linux 社区项目看板](https://github.com/orgs/rocky-linux/projects/14/views/2) 上进行追踪。每篇文章在开始撰写之前都应有一个对应的 issue。

### 列定义

| 状态 | 含义 |
|--------|---------| 
| **To Do** (待办) | 主题已确定，所需信息已收集并记录，已分配给一位作者 |
| **In Progress** (进行中) | 正在撰写中 |
| **Blocked** (阻塞) | 正在等待另一方提供信息或意见 |
| **Review** (审核中) | 初稿已完成，正在进行社区反馈流程 |
| **Approved** (已批准) | 所有修订已完成，等待发布 |
| **Done** (已完成) | 已发布 |

---

## 逐步流程

### 1. 确定主题

主题可以来自任何地方：社区需求、安全事件、版本里程碑、贡献者故事或调查反馈。如果你有想法，在创建 issue 之前，先在 [Mattermost](https://chat.rockylinux.org) 上向 `~Community` 频道提出，特别是当你不确定是否已有人在处理时。

### 2. 创建 Issue

在[项目看板](https://github.com/orgs/rocky-linux/projects/14/views/2)上创建一个 issue。issue 标题应清晰地描述该文章（例如 "Blog: Contributor Spotlight: Jane Doe"）。将状态设置为 **To Do**，如果已确定作者，则分配给他。

### 3. 撰写初稿

在 Google Docs 或其他支持评论和建议的协作平台上撰写。这样可以在文章移至 GitHub 之前方便他人留下反馈。

遵循为 Rocky Linux 博客文章确立的风格约定：

- 专业、友好、直截了当的语气
- 标题使用 sentence case (句首大写)，不使用 em dash (长破折号)
- 引人入胜的开头，不使用填充短语或现在分词引导句
- 软件包名称、命令和仓库引用使用 inline code (行内代码) 格式
- 避免过度使用粗体或滥用列表符号；在自然流畅的地方使用散文体

### 4. 修订

对初稿进行拼写检查和语法检查。如果需要发现拗口的措辞，可以大声朗读。确保所有技术细节（软件包名称、版本字符串、CVE 编号、命令）都是准确的，且未被转述。

### 5. 发布到 Mattermost 征求反馈

在 [Mattermost](https://chat.rockylinux.org) 上的 `~Community` 频道分享初稿链接并征求反馈。将 issue 移至项目看板的 **Review** 状态。在继续推进之前，给社区一个合理的反馈时间窗口。

### 6. 根据反馈进行修订

整合社区审核的反馈。如果文章正在等待特定人员或团队的反馈，将 issue 移至 **Blocked** 状态，直到收到相关信息。

### 7. 准备 Markdown 文件

将最终稿按照以下结构转换为 `.md` 文件：

**Frontmatter (前置元数据)**（必填，分隔符必须存在）：

title: "使用 sentence case 的文章标题"

date: "YYYY-MM-DD"

author: "你的名字"

**文件名约定：**

YYYY-MM-DD-简短描述性-slug.md

将文件放置在内容仓库的 `news/` 目录中。

### 8. 通过 GitHub 提交

内容仓库为 [rocky-linux/rockylinux.org-content](https://github.com/rocky-linux/rockylinux.org-content)。

1. 在创建任何新分支之前，先使用 GitHub 的 **Sync fork** (同步 fork) 按钮同步你的 fork
2. 创建一个名为 `news/你的文章-slug` 的新分支
3. 将你的 `.md` 文件添加到 `news/` 目录中
4. 针对 `rocky-linux/rockylinux.org-content` 上的 `main` 分支发起 pull request (拉取请求)
5. 在 PR 正文中添加一段关于该文章的简短纯文本描述
6. 请求相应的技术或社区审核者进行审核
7. 处理所有审核反馈并根据需要更新分支
8. 一旦 PR 获得批准并准备好合并，将 issue 移至 **Approved**

### 9. 发布后的推广

文章上线后，在 Mattermost 的 `~Community` 频道中请社区团队 (Community Team) 在社交渠道（LinkedIn、Bluesky、Mastodon）上进行推广。提供已发布的 URL 以及你有的话任何建议的推广文案。

将 issue 移至 **Done**。

---

## 提示 (Tips)

- 不要直接提交到上游仓库的 `main` 分支；始终从你 fork 上的分支进行开发
- 在提交之前验证原始文件中存在 frontmatter 分隔符 (`---`)；从富文本编辑器复制时很容易丢失这些分隔符
- 双连字符 (`--`) 不能替代标点符号，且不能出现在代码块之外
- `news/` 目录使用文件名中的日期作为发布日期；确保 frontmatter 中的日期与文件名日期一致
