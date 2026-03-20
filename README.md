# muyi-codex-skills

`muyi-codex-skills` 是一个面向 Codex 的 skills 合集仓库。每个 skill 都放在独立目录中，便于单独安装、版本管理和后续扩展。

## 目录结构

```text
muyi-codex-skills/
  skills/
    muyi-translate/
  _templates/
    skill/
```

`skills/` 下存放真实可安装的 skills。每个 skill 都应该是一个独立目录，并至少包含 `SKILL.md`。

`_templates/skill/` 是新建 skill 时可直接复制的模板目录，不作为实际 skill 安装。

## 当前包含

- `muyi-translate`: 面向文章、文档、Markdown 和长文内容的翻译与本地化 skill

## 新增 skill 的约定

建议每个 skill 使用稳定且唯一的目录名，例如：

- `muyi-translate`
- `muyi-review`
- `muyi-browse`

每个 skill 内建议保持以下结构：

```text
skill-name/
  SKILL.md
  agents/openai.yaml
  scripts/
  references/
  assets/
```

## 安装示例

安装单个 skill：

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo <owner>/muyi-codex-skills \
  --path skills/muyi-translate
```

一次安装多个 skill：

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo <owner>/muyi-codex-skills \
  --path skills/muyi-translate skills/muyi-review skills/muyi-browse
```

安装后需要重启 Codex，才能识别新安装的 skills。
