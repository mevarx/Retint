# ReTint

ReTint is a cloneable and installable agentic skill designed for AI coding assistants. It allows your agent to reverse-engineer any website's design taste (`extract`) and apply it directly to your own codebase (`graft`).

- **`extract`** — Given a public URL, captures DOM data and screenshots, then runs a 4-step analysis pipeline (Measure → Pattern → Taste → Observer) to produce `{domain}.md` + `{domain}.json` with concrete design tokens AND the reasoning behind them.
- **`graft`** — Given a previous `extract` output, diffs your project's tokens against the target taste, proposes a change plan, and — on approval — applies token-level edits only. Never touches content, layout, or business logic.

Core philosophy: **tokens without reasoning are useless to an agent.** "Border-radius is 2px" is a spec sheet. "Border-radius is 2px because the site signals precision over friendliness, and every rounded alternative was rejected" is taste.

---

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/mevarx/Retint.git

# 2. Install Playwright browser dependencies (for the extraction fallback)
npx playwright install chromium

# 3. Copy the skill directories to your agentic coding environment (see IDE steps below)
```

---

## IDE-Specific Setup & Installation

To use ReTint, copy the skill folders to the location where your specific coding tool expects to find custom instructions, prompts, or rules:

### 1. Cursor
Cursor uses custom `.mdc` system rules inside your workspace directory.
```bash
# Create the local rules directory if it doesn't exist
mkdir -p .cursor/rules

# Copy the skill folders to your Cursor workspace rules/skills
cp -r ReTint/extract/ .cursor/rules/extract/
cp -r ReTint/graft/ .cursor/rules/graft/
cp ReTint/shared/taste-schema.json .cursor/rules/shared/taste-schema.json
```
*Note: Cursor will automatically pick up the custom definitions to guide visual design output.*

### 2. Windsurf
Windsurf rules reside inside the `.windsurf/rules` folder.
```bash
# Create the rules folder
mkdir -p .windsurf/rules

# Copy the skills
cp -r ReTint/extract/ .windsurf/rules/extract/
cp -r ReTint/graft/ .windsurf/rules/graft/
cp ReTint/shared/taste-schema.json .windsurf/rules/shared/taste-schema.json
```

### 3. Claude Code
Claude Code automatically discovers custom skills inside the workspace-level `.claude/skills/` directory.
```bash
# Create the local skills directory
mkdir -p .claude/skills

# Copy ReTint skills
cp -r ReTint/extract/ .claude/skills/extract/
cp -r ReTint/graft/ .claude/skills/graft/
cp ReTint/shared/taste-schema.json .claude/skills/shared/taste-schema.json

# Optional: Set up Playwright MCP for faster page captures
claude mcp add playwright -s user -- npx -y @playwright/mcp@latest --isolated
```

### 4. GitHub Copilot & VS Code
For GitHub Copilot, custom instructions are placed inside the `.github/` folder.
```bash
# Create the GitHub configuration directory
mkdir -p .github

# Copy the skills to keep them in context
cp -r ReTint/extract/ .github/extract/
cp -r ReTint/graft/ .github/graft/
cp ReTint/shared/taste-schema.json .github/taste-schema.json
```
*Add a reference to `.github/copilot-instructions.md` directing Copilot to use these skills during design changes.*

### 5. Google Gemini / Antigravity
Gemini's Antigravity CLI and IDE agents automatically discover workspace and global skills.
- **Workspace-level**: Place under `.agents/skills/`
- **Global-level**: Place under `~/.gemini/config/skills/`

```bash
# Workspace setup
mkdir -p .agents/skills
cp -r ReTint/extract/ .agents/skills/extract/
cp -r ReTint/graft/ .agents/skills/graft/
cp ReTint/shared/taste-schema.json .agents/skills/shared/taste-schema.json
```

To configure Playwright MCP globally, add the following to `~/.gemini/settings.json`:
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

### 6. Bolt & Lovable (Browser-based AI IDEs)
For browser-based builders like Bolt or Lovable, paste the prompt guidelines directly into project settings or instructions:
- **Bolt**: Save the visual directives inside `.bolt/prompt`.
- **Lovable**: Paste the directives and anti-pattern blocks directly into the **Project Knowledge** pane in the dashboard.

---

## Usage

### 1. `/extract <url>`
Analyzes a single URL's design taste.
```
/extract https://linear.app
```
**Output:**
- `linear.app.md` — human-readable Design Map + Taste DNA
- `linear.app.json` — structured data for downstream tools and `/graft`
- Tool-specific export file (Cursor, Windsurf, Gemini, etc.)

### 2. `/extract <url1> vs <url2>`
Comparative mode — analyzes both sites and produces a Delta section showing where they agree (category-standard) vs diverge (genuinely differentiating).
```
/extract https://stripe.com vs https://vercel.com
```

### 3. `/graft <domain>`
Applies a previously extracted taste profile to your current project.
```
/graft linear.app
```
The skill will inventory your project's current design tokens, diff them against `linear.app.json`, show a change plan with per-line reasoning, and wait for approval before making any edits.

---

## File Structure

```
ReTint/
├── extract/
│   ├── SKILL.md                        # Entry point: triggers, setup, pipeline orchestration
│   └── references/
│       ├── extract-dom.js              # Browser-injected DOM extractor
│       ├── capture-fallback.mjs        # Standalone Playwright script (MCP fallback)
│       ├── step1-measure.md            # Step 1 prompt: pure measurement
│       ├── step2-pattern.md            # Step 2 prompt: pattern recognition
│       ├── step3-taste.md              # Step 3 prompt: taste principles + confidence
│       ├── step4-observer.md           # Step 4 prompt: quality gate + final output
│       ├── anti-slop-wordlist.md       # Banned generic terms
│       └── export-formats.md           # Per-tool export specs (10 targets)
├── graft/
│   ├── SKILL.md                        # Entry point: inventory → diff → plan → apply
│   └── references/
│       ├── inventory-project.md        # How to detect/scan local token sources
│       ├── diff-and-plan.md            # Diffing logic + change plan format
│       ├── token-boundary-guard.md     # What counts as "token-level" vs not
│       └── anti-slop-wordlist.md       # Banned generic terms
└── shared/
    └── taste-schema.json               # Canonical JSON schema both skills use
```

---

## Anti-Slop Policy

Both skills mechanically enforce a ban on generic AI adjectives as genuine descriptors:
> `clean, modern, sleek, visually appealing, user-friendly, intuitive, seamless, elegant, minimalist, polished, refined, sophisticated, premium feel`

Every claim in the output must be backed by a specific px/hex/ratio value and cited evidence. The test: **"Could I have written this without seeing this specific website?"** If yes, it's slop — rewrite it.

---

## License

MIT. DOM extractor adapted from [senlindesign/taste-skill](https://github.com/senlindesign/taste-skill) (MIT).
