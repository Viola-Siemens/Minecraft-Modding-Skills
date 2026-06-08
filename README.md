# Minecraft-Modding-Skills

If a developer wants to "distill" him/herself into Skills, there is no doubt that this will be the beginning of productivity improvement!

All skills here are licensed under [Apache 2.0 License](LICENSE).

## Installation

This repository provides reusable skills compatible with various AI coding agents. Each skill is defined by a `SKILL.md` file inside its own subdirectory under `skills/`.

### Prerequisites

Clone this repository to your local machine (or add it as a submodule):

```bash
git clone https://github.com/Viola-Siemens/Minecraft-Modding-Skills.git
cd your-repo
```

### Agent-Specific Setup

#### Claude Code

Claude Code loads skills from `~/.claude/skills/` (global) or `./.claude/skills/` (project-specific).

```bash
# Global installation
cp -r skills/* ~/.claude/skills/

# Or project-local (run inside your project root)
mkdir -p .claude/skills
cp -r /path/to/your-repo/skills/* .claude/skills/
```

#### Codex (OpenAI)

Codex skills are typically placed in `~/.codex/skills/` or defined via `codex.toml`.

```bash
# Global
cp -r skills/* ~/.codex/skills/

# Verify with
codex skills list
```

Alternatively, set the `CODEX_SKILLS_PATH` environment variable to point to the `skills/` directory.

#### Qwen Code (通义千问)

Qwen Code supports skills through a dedicated directory, usually `~/.qwen/skills/`.

```bash
cp -r skills/* ~/.qwen/skills/
```

#### Gemini CLI (Google)

For Gemini CLI, copy the skill folders to the default skills directory:

```bash
cp -r skills/* ~/.gemini/skills/
```

Or use the built-in import command (if available):

```bash
gemini skills install /path/to/your-repo/skills/minecraft-code-standards --consent
```

#### OpenClaw

OpenClaw loads skills from `~/.openclaw/skills/` or from a directory specified in `openclaw.json`.

```bash
cp -r skills/* ~/.openclaw/skills/
```

You can also set the `OPENCLAW_SKILLS_PATH` environment variable to the absolute path of the `skills/` folder.

### Verification

After copying, each agent should automatically detect the skills. For example, Claude Code will show available skills when you run:

```bash
claude skills list
```

### Updating

Pull the latest changes from the repository and re-run the copy command for your agent(s).

```bash
git pull
cp -r skills/* ~/.claude/skills/   # adjust path for your agent
```

### Notes

- Some agents may require a restart or a reload command to recognise newly added skills.
- If an agent does not have a predefined skills directory, consult its documentation – most allow you to configure a custom path via a configuration file or environment variable.
- Each skill is contained in its own folder. You may copy individual skills instead of the entire set.

For more details on writing your own skills, refer to the `SKILL.md` format used in this repository.
