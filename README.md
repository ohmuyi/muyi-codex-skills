# muyi-codex-skills

`muyi-codex-skills` 是一个面向 Codex 的自定义 skills 仓库，提供可直接安装和使用的实用 skill。本文档面向想要使用这套 skills 的读者，重点说明如何安装、更新、调用，以及当前已包含 skill 的核心能力和处理流程。

## 快速开始

安装整个仓库当前提供的全部 skills：

```text
$skill-installer install https://github.com/ohmuyi/muyi-codex-skills
```

安装完成后，重启 Codex。

默认情况下，skills 会被安装到：

```bash
~/.codex/skills/
```

## 安装说明

推荐使用 Codex 内置的 `$skill-installer`。这是最适合普通使用者的入口。

如果你想一次安装这套仓库中的全部 skills，直接使用上面的整仓安装命令即可。

当你把 GitHub 仓库根 URL 交给 `$skill-installer` 时，它会把这个 URL 当作一个 skills 仓库入口，而不是单个 skill 目录。对于这种仓库入口，安装器可能先检查本地已安装的 skill，再去远端探测仓库里哪些目录包含 `SKILL.md`，然后逐个安装未冲突的 skill。这属于正常行为，不表示安装失败。

如果你只想安装单个 skill，也可以指向具体目录安装。例如安装 `muyi-translate`：

```text
$skill-installer install https://github.com/ohmuyi/muyi-codex-skills/tree/main/skills/muyi-translate
```

如果你希望安装过程更确定、输出更直接，或者只想避免仓库探测提示，优先使用这种带具体 skill 路径的安装方式。

## 更新说明

当前没有单独列出的官方 `update` 指令，也没有专门用于更新 skill 的快捷键。需要同步 GitHub 仓库中的新版本时，直接再次运行同一条安装命令即可。

如果你是整仓使用者，继续运行这条命令：

```text
$skill-installer install https://github.com/ohmuyi/muyi-codex-skills
```

执行完成后，重启 Codex。

如果你只安装了某个单独的 skill，也可以继续使用对应目录 URL 重新安装，例如：

```text
$skill-installer install https://github.com/ohmuyi/muyi-codex-skills/tree/main/skills/muyi-translate
```

如果你的本地 Codex 提示同名 skill 已存在，或者没有立即识别到新版本，请按本地提示处理后重新执行对应命令。

## 如何使用

安装并重启 Codex 后，可以直接在提示词中显式调用：

```text
$muyi-translate 把这篇 Markdown 文章翻译成中文
```

也可以自然语言触发。`muyi-translate` 允许隐式触发，因此当任务明显是在做翻译、本地化、长文转换，且要求保留 Markdown 结构、链接和代码块时，Codex 通常可以自动选用它。

例如：

- 翻译这篇 README，保留链接和代码块
- 把这篇英文博客精翻成中文
- 把这个 URL 的文章翻译成自然中文

## 当前已包含的 skill

### `muyi-translate`

`muyi-translate` 是一个面向长文本和 Markdown 内容的翻译与本地化 skill，适合以下场景：

- 翻译文章、博客、README、普通文档
- 本地化 Markdown 内容，并保留标题层级、链接和代码块
- 处理长文，尤其是需要术语统一和稳定输出目录的情况
- 根据要求输出快翻、常规翻译或精翻版本

它的核心能力主要集中在四个方面。

第一是偏好与术语管理。skill 会优先查找 `EXTEND.md`，读取目标语言、模式、受众、文风、分块阈值以及用户自定义术语表，并将这些配置与内置 glossary、外部 glossary 文件合并，尽量保持术语统一。

第二是长文分块处理。对于达到阈值的 Markdown 内容，skill 会先整理术语，再调用 `scripts/main.ts` 做自动分块，把内容拆成多个 chunk，减少长文翻译时的上下文漂移。它优先使用 `bun`，如果系统没有 `bun` 但有 `npx`，则退回到 `npx -y bun`。

第三是多模式翻译。它支持 `quick`、`normal`、`refined` 三种模式，分别对应直接翻译、带分析的翻译，以及带批判审阅和润色的精翻流程。

第四是稳定输出。skill 会把所有中间文件和最终结果写入固定命名的输出目录，避免覆盖旧结果，并在结束后补做一次轻量的图片语言检查。

## `muyi-translate` 的处理步骤

`muyi-translate` 的工作流比较稳定，整体顺序如下：

1. 读取偏好配置和 glossary。
2. 判断输入来源是文件、URL 还是内联文本。
3. 将 URL 或内联文本物化为本地 Markdown 文件。
4. 在源文件旁边创建输出目录，目录名格式为 `{source-basename}-{target-lang}`。
5. 根据用户要求或默认配置选择 `quick`、`normal`、`refined` 模式。
6. 判断内容长度。短文直接处理，长文先抽取术语再分块。
7. 对每个 chunk 依次翻译，并按顺序合并。
8. 输出中间文件和最终 `translation.md`。
9. 复查 Markdown 结构、术语一致性、链接和代码块，并提示可能仍含源语言文字的图片。

不同输入类型会按固定方式处理：

- 文件：直接使用原文件
- URL：先抓取内容，再保存为 `translate/{slug}.md`
- 内联文本：保存为 `translate/{slug}.md`

输出目录会创建在源文件同级目录下，例如：

```text
posts/article.md -> posts/article-zh/
translate/ai-future.md -> translate/ai-future-zh/
```

如果输出目录已经存在，旧结果不会被覆盖，而是先被重命名为带时间戳的备份目录。

## `muyi-translate` 的输出文件

不同模式下，输出文件略有不同。

`quick` 模式只输出：

```text
translation.md
```

`normal` 模式输出：

```text
01-analysis.md
02-prompt.md
translation.md
```

`refined` 模式输出：

```text
01-analysis.md
02-prompt.md
03-draft.md
04-critique.md
05-revision.md
translation.md
```

这意味着你不仅能拿到最终译文，还能看到它的分析、提示词、初稿、批判审阅和修订过程。

## 仓库结构

本仓库当前结构如下：

```text
muyi-codex-skills/
  skills/
    muyi-translate/
  _templates/
    skill/
```

`skills/` 用于存放实际可安装的 skills。每个 skill 都是一个独立目录，至少包含 `SKILL.md`，通常还会包含 `agents/`、`scripts/`、`references/` 和 `assets/`。

`_templates/skill/` 是新建 skill 时可复用的模板目录，不会作为实际 skill 安装。
