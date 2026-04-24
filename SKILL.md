---
name: skill-market-publisher
description: "Skill Market 发布助手：指导用户将本地 skill 打包、推送到 GitHub 并发布到 Skill Market。处理 CLI 登录、非交互式上传、rootUrl 格式、轨迹采集等完整流程。当用户要求发布 skill、上传到 Skill Market、skill-market-cli 报错时必须触发。"
model: kimi k2.6
tags: ["skill-market", "publish", "cli", "github", "upload"]
rootUrl: https://raw.githubusercontent.com/Alpaca233114514/skill-market-publisher/main/SKILL.md
---

# Skill Market Publisher

Complete workflow to publish a skill from local workspace to Skill Market.

## Prerequisites

1. `skill-market-cli` installed globally (`npm i -g skill-market-cli`)
2. Logged in to Skill Market (`skill-market-cli login` or `skill-market-cli token set <pat>`)
3. A skill directory with `SKILL.md` and optional `.skill-examples.json`

## Workflow

```
1. Validate skill directory structure
2. Push to GitHub (for rootUrl)
3. Ensure .skill-examples.json exists
4. Identify your own model (non-interactive only)
5. Run skill-market-cli upload --non-interactive
6. Verify upload success
```

## Phase 1: Validate & Prepare

Check the skill directory has required files:

```bash
ls /path/to/skill/
# Expected: SKILL.md
# Optional: scripts/, references/, .skill-examples.json
```

**SKILL.md frontmatter must include**:
- `name`: skill identifier
- `description`: one-line purpose
- `rootUrl`: HTTP(S) URL to raw SKILL.md (e.g. GitHub raw)

Example frontmatter:
```yaml
---
name: my-skill
description: "What this skill does"
rootUrl: https://raw.githubusercontent.com/user/repo/main/SKILL.md
---
```

## Phase 2: Push to GitHub

The Skill Market server requires `rootUrl` to be an accessible HTTPS link. GitHub raw URLs are recommended.

```bash
cd /path/to/skill
git init
git remote add origin https://github.com/<user>/<repo>.git
git add .
git commit -m "feat: add skill files"
git push -u origin main
```

**Critical**: The raw URL format is:
```
https://raw.githubusercontent.com/<user>/<repo>/<branch>/SKILL.md
```

Update `rootUrl` in SKILL.md after pushing, then commit again if needed.

## Phase 3: Prepare Usage Examples

Skill Market AI channel requires at least one usage example with AI trajectory (thinking / toolcall / message).

**Option A**: Pre-create `.skill-examples.json`:
```json
{
  "model": "kimi k2.6",
  "examples": [
    {
      "prompt": "用户会如何提问",
      "aiResponses": [
        { "type": "thinking", "content": "..." },
        { "type": "toolcall", "toolName": "...", "toolInput": {} },
        { "type": "message", "content": "..." }
      ],
      "model": "kimi k2.6"
    }
  ]
}
```

**Option B**: Let CLI auto-collect (requires interactive terminal).

## Phase 4: Upload

### Environment Auto-Detection

`skill-market-cli` detects your runtime environment and recommends the appropriate model automatically:

| Environment | Detected By | Recommended Model |
|-------------|-------------|-------------------|
| Claude Code | `CLAUDE_CODE` env | `claude-sonnet` |
| Kimi Code | `KIMI_CODE` env | `kimi` |
| Cursor | `CURSOR_TRACE_ID` / `CURSOR_ENV` env | `claude-sonnet` |
| VS Code | `VSCODE_CWD` / `TERM_PROGRAM=vscode` | `claude-sonnet` |
| Other / Unknown | — | `claude-sonnet` |

In **interactive mode**, the CLI presents a three-option selector:
1. **Detected model** (e.g., `kimi` for Kimi Code)
2. **Default model** (`claude-sonnet`)
3. **Custom input** (enter any model name)

You can always override with `-m <model>`. To disable auto-detection entirely, set `SMCLI_DISABLE_ENV_DETECT=1`.

### Pre-Upload: Agent Model Self-Identification (Non-Interactive)

Before executing `--non-interactive` upload, the Agent **must ask itself "what model am I?"** and pass the exact answer via `-m`. This ensures the uploaded skill's `model` field precisely matches the environment where it will be used.

**How to self-identify**:

1. Based on your system context, capabilities, and self-knowledge, determine your exact model name.
2. Pass that exact name via `-m`.

**Examples**:
- If you recognize yourself as **Kimi k2.6** → `-m "kimi k2.6"`
- If you recognize yourself as **Claude Sonnet** → `-m "claude-sonnet"`
- If you recognize yourself as **GPT-4o** → `-m "gpt-4o"`
- If you are **uncertain** → omit `-m`; the CLI will auto-detect

**Example command** (Agent identifying itself as Kimi k2.6):
```bash
skill-market-cli upload /path/to/skill \
  -n "skill-name" \
  -d "Skill description" \
  -t "tag1,tag2,tag3" \
  -m "kimi k2.6" \
  --non-interactive
```

The Agent should perform this self-identification **before every non-interactive upload**. Do not map or transform the model name—use your self-identified exact model name.

### Non-Interactive Mode (Recommended for Agents)

Use `--non-interactive` to skip all prompts. Missing fields use defaults or error out:

```bash
skill-market-cli upload /path/to/skill \
  -n "skill-name" \
  -d "Skill description" \
  -t "tag1,tag2,tag3" \
  -m "<self-identified-model>" \
  --non-interactive
```

**Field behavior in --non-interactive**:
| Field | Source | Fallback |
|-------|--------|----------|
| name | `-n` arg → frontmatter | **Error exit** |
| description | `-d` arg → frontmatter | **Error exit** |
| model | `-m` arg → frontmatter → auto-detect | `claude-sonnet` (env-detected) |
| tags | `-t` arg → frontmatter | `['general']` |
| rootUrl | frontmatter | `file://...` (often rejected by server) |
| usageExamples | `.skill-examples.json` | Auto-collect if prompts exist; **Error** if completely missing |

### Interactive Mode

For manual use when terminal supports interactive input:

```bash
skill-market-cli upload /path/to/skill -y
```

The CLI will prompt for missing fields. When reaching the model selection step, you will see a **three-option selector** based on your detected environment:

```
选择推荐模型（用于案例采集与提交，建议与线上一致）：
❯ 使用识别模型: claude-sonnet (Claude Code)
  使用默认模型: claude-sonnet
  其他（手动输入）
```

Select **Detected model** to use the environment-recommended model, **Default model** for the global fallback, or **Custom input** to specify any model name (e.g., `gpt-4o`, `claude-opus`).

## Phase 5: Troubleshooting

### Error: `ERR_USE_AFTER_CLOSE` / readline closed
**Cause**: Inquirer interactive prompts fail in non-TTY environments (AI agents, CI).
**Fix**: Use `--non-interactive` flag and pre-provide all fields via CLI args or frontmatter.

### Error: `rootUrl 格式无效`
**Cause**: `file://C:\Users\...` Windows paths are malformed or server rejects `file://` protocol.
**Fix**: Use `https://raw.githubusercontent.com/.../SKILL.md` as rootUrl.

### Error: `无法访问该链接: TLS handshake timeout`
**Cause**: Skill Market server cannot reach GitHub due to network issues.
**Fix**: Use `https://raw.githubusercontent.com/...` instead of `https://github.com/...`. If still failing, retry later or use an alternative CDN/mirror.

### Error: `缺少用户案例及轨迹`
**Cause**: No `.skill-examples.json` and no usageExamples in SKILL.md.
**Fix**: Create `.skill-examples.json` with at least one example containing `prompt`, `aiResponses`, and `model`.

### Error: `上传出错 400 / 500`
Check response data for specific field validation errors. Common issues:
- `tags` is empty array
- `usageExamples` is empty array
- `rootUrl` is not http(s)

## Reference

- `references/cli-options.md` - Full CLI option matrix
- `references/example-json-schema.md` - `.skill-examples.json` schema
