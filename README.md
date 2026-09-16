# skills

Agent Skills I build and use, kept in one place so they're versioned, shareable, and installable anywhere.

They're plain `SKILL.md` files, so they work with Claude Code, Codex, Cursor, Windsurf, and every other agent the [`skills`](https://github.com/vercel-labs/skills) CLI supports.

## Install

```bash
# all of them
npx skills add 00-Aakash-00/skills

# just one
npx skills add 00-Aakash-00/skills --skill <name>

# see what's in here first
npx skills add 00-Aakash-00/skills --list
```

Add `-g` to install globally instead of into the current project.

## The skills

| Skill | What it's for |
| --- | --- |
| [`ux-guidelines`](skills/ux-guidelines/SKILL.md) | The four screen states (loading, error, empty, success), graceful degradation, action feedback, and forgiving forms. Behavioral UX — the un-happy paths generated code skips. |
| [`qa-guidelines`](skills/qa-guidelines/SKILL.md) | Human-simulating end-to-end QA: an independent subagent drives the real surface (web, mobile, desktop, CLI) through the full branching tree of user flows instead of trusting the tests the implementer wrote. For AI products it reads answers and traces — never keyword or regex checks. |

## Layout

Each skill is a self-contained directory:

```
skills/<name>/
  SKILL.md        # frontmatter + the instructions the agent loads
  *.md            # reference files SKILL.md links to, loaded on demand
```

## License

MIT — use them, fork them, change them.
