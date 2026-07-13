# Extract + Graft — Design Taste Skills

Two companion agent skills that reverse-engineer a website's design taste (`extract`) and apply it to your own project (`graft`).

**`extract`** — Given a public URL, captures DOM data and screenshots, then runs a 4-step analysis pipeline (Measure → Pattern → Taste → Observer) to produce `{domain}.md` + `{domain}.json` with concrete design tokens AND the reasoning behind them.

**`graft`** — Given a previous `extract` output, diffs your project's tokens against the target taste, proposes a change plan, and — on approval — applies token-level edits only. Never touches content, layout, or business logic.

Core philosophy: **tokens without reasoning are useless to an agent.** "Border-radius is 2px" is a spec sheet. "Border-radius is 2px because the site signals precision over friendliness, and every rounded alternative was rejected" is taste.

---

## Quick Start

```bash
# 1. Install the skills (copy to your tool's skill directory — see below)
# 2. Install Playwright (optional, for MCP; the fallback script also handles this)
npx playwright install chromium

# 3. Extract a site's design taste
/extract https://linear.app

# 4. Apply it to your project
/graft linear.app
```

---

## Installation

### Claude Code

```bash
# Copy skills to Claude Code's skill directory
cp -r extract/ .claude/skills/extract/
cp -r graft/ .claude/skills/graft/
cp shared/taste-schema.json .claude/skills/shared/taste-schema.json

# Optional: Set up Playwright MCP for faster capture
claude mcp add playwright -s user -- npx -y @playwright/mcp@latest --isolated
```

### Cursor

```bash
# Copy skills to Cursor's skill directory
cp -r extract/ .cursor/skills/extract/
cp -r graft/ .cursor/skills/graft/
cp shared/taste-schema.json .cursor/skills/shared/taste-schema.json
```

### OpenCode

```bash
# Copy skills to OpenCode's skill directory
cp -r extract/ .opencode/skills/extract/
cp -r graft/ .opencode/skills/graft/
cp shared/taste-schema.json .opencode/skills/shared/taste-schema.json
```

### Google Antigravity (Gemini)

```bash
# Copy skills to Antigravity's skill directory
cp -r extract/ .agents/skills/extract/
cp -r graft/ .agents/skills/graft/
cp shared/taste-schema.json .agents/skills/shared/taste-schema.json
```

To set up Playwright MCP for Antigravity, add to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--isolated"]
    }
  }
}
```

### Codex CLI

```bash
# Copy skills to Codex's skill directory
cp -r extract/ .codex/skills/extract/
cp -r graft/ .codex/skills/graft/
cp shared/taste-schema.json .codex/skills/shared/taste-schema.json
```

### Global Installation (all tools)

To make the skills available across all projects, copy to your user-level skill directory:

| Tool | User-level path |
|------|----------------|
| Claude Code | `~/.claude/skills/` |
| Cursor | `~/.cursor/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Antigravity | `~/.gemini/config/skills/` |
| Codex CLI | `~/.codex/skills/` |

---

## Usage

### `/extract <url>`

Analyzes a single URL's design taste.

```
/extract https://linear.app
```

You'll be asked two setup questions:
1. **Export target** — which tool format to generate (Cursor, Windsurf, Claude Code, etc.)
2. **Crawl scope** — this page only, or explore 2–3 linked pages for a fuller read

**Output:**
- `linear.app.md` — human-readable Design Map + Taste DNA
- `linear.app.json` — structured data for downstream tools and `/graft`
- Export file in your selected tool's format (if not "skip")

### `/extract <url1> vs <url2>`

Comparative mode — analyzes both sites and produces a Delta section showing where they agree (category-standard) vs diverge (genuinely differentiating).

```
/extract https://stripe.com vs https://vercel.com
```

### `/graft <domain>`

Applies a previously extracted taste profile to your current project.

```
/graft linear.app
```

The skill will:
1. Inventory your project's current design tokens
2. Diff them against `linear.app.json`
3. Show a change plan with per-line reasoning
4. Wait for your approval before making any edits
5. Apply token-level changes only (never touches content or logic)

---

## Playwright Setup

### MCP (recommended for supported tools)

Playwright MCP provides faster, lower-token-cost browser interaction. It's optional — the skills work without it.

| Tool | Setup command |
|------|--------------|
| Claude Code | `claude mcp add playwright -s user -- npx -y @playwright/mcp@latest --isolated` |
| Antigravity | Add to `~/.gemini/settings.json` mcpServers (see above) |
| Cursor | Configure in Cursor's MCP settings panel |
| Others | MCP configuration varies — check tool docs |

### Standalone Fallback (no MCP needed)

If MCP isn't configured, the extract skill automatically uses `capture-fallback.mjs` — a standalone Playwright script. Prerequisites:

```bash
# Install Playwright's bundled Chromium (one-time, ~100MB)
npx playwright install chromium
```

The `--isolated` flag (MCP) or fresh browser context (fallback) ensures you're always analyzing the public page, never an authenticated state.

---

## File Structure

```
extract/
├── SKILL.md                        # Entry point: triggers, setup, pipeline orchestration
├── references/
│   ├── extract-dom.js              # Browser-injected DOM extractor
│   ├── capture-fallback.mjs        # Standalone Playwright script (MCP fallback)
│   ├── step1-measure.md            # Step 1 prompt: pure measurement
│   ├── step2-pattern.md            # Step 2 prompt: pattern recognition
│   ├── step3-taste.md              # Step 3 prompt: taste principles + confidence
│   ├── step4-observer.md           # Step 4 prompt: quality gate + final output
│   ├── anti-slop-wordlist.md       # Banned generic terms
│   └── export-formats.md           # Per-tool export specs (10 targets)

graft/
├── SKILL.md                        # Entry point: inventory → diff → plan → apply
├── references/
│   ├── inventory-project.md        # How to detect/scan local token sources
│   ├── diff-and-plan.md            # Diffing logic + change plan format
│   ├── token-boundary-guard.md     # What counts as "token-level" vs not
│   └── anti-slop-wordlist.md       # Banned generic terms

shared/
└── taste-schema.json               # Canonical JSON schema both skills use
```

---

## Known Limitations

- **Login pages**: Pages behind authentication return the login form, not the target page. Use the public marketing page URL instead.
- **Cloudflare/bot blocks**: Some sites show a verification page to headless browsers. Try a different page on the same domain, or provide a direct URL to a specific page.
- **SPA hydration**: Heavy single-page apps (Figma, Notion) may still be hydrating after the wait window. This is flagged as a caveat in the output rather than silently producing bad data.
- **CSS custom property names**: DOM extraction resolves variables to computed values — original `--variable-name` references aren't recoverable. The extract captures the values, not the naming convention.
- **Inline styles**: `graft` cannot safely operate on projects with heavily inlined styles without first proposing a token-extraction step. It flags this rather than attempting risky scattered edits.
- **Viewport dependency**: All measurements are from a 1440×900 desktop viewport. Mobile layouts are not captured, but mobile design systems usually share the same tokens.
- **Motion capture limitations**: Only 1–2 nav links and 1–2 buttons are hovered for interaction capture. Complex hover menus or reveal-on-hover patterns may be missed.

---

## Anti-Slop Policy

Both skills mechanically enforce a ban on generic AI adjectives as genuine descriptors:

> `clean, modern, sleek, visually appealing, user-friendly, intuitive, seamless, elegant, minimalist, polished, refined, sophisticated, premium feel`

Every claim in the output must be backed by a specific px/hex/ratio value and cited evidence. The test: **"Could I have written this without seeing this specific website?"** If yes, it's slop — rewrite it.

---

## License

MIT. DOM extractor adapted from [senlindesign/taste-skill](https://github.com/senlindesign/taste-skill) (MIT).
