# skill-market-cli upload Options

| Option | Short | Required | Default | Description |
|--------|-------|----------|---------|-------------|
| `--name` | `-n` | Yes* | frontmatter | Skill name |
| `--description` | `-d` | Yes* | frontmatter | Skill description/purpose |
| `--tags` | `-t` | No | `['general']` | Comma-separated tags |
| `--model` | `-m` | No | `'deepseek-chat'` | Recommended model |
| `--yes` | `-y` | No | false | Skip final confirmation only |
| `--non-interactive` | — | No | false | Skip ALL prompts (Agent/CI mode) |

*Required in `--non-interactive` mode if not present in SKILL.md frontmatter.

## Non-Interactive Behavior

When `--non-interactive` is set:

1. **name/description missing**: Error exit with helpful message
2. **model missing**: Defaults to `'deepseek-chat'`
3. **tags missing**: Defaults to `['general']`
4. **rootUrl missing**: Defaults to local `file://` path (often rejected by server)
5. **usageExamples missing**: Error exit unless `.skill-examples.json` exists
6. **Final confirmation**: Always skipped

## Global Options

| Option | Description |
|--------|-------------|
| `--mode <env>` | Server environment (production/development/test) |
| `-v, --version` | Show CLI version |
