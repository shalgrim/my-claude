# my-claude

Version-controlled Claude Code configuration: global settings (CLAUDE.md), custom commands, and skills.

Designed to work alongside a Dropbox-synced `~/.claude/` directory. The repo holds the parts worth version controlling, and symlinks connect them into the Dropbox directory that `~/.claude` points to.

## Repository Structure

```
my-claude/
├── CLAUDE.md                     # Global Claude Code settings
├── README.md
├── commands/                     # Custom slash commands
│   ├── capture-notes.md
│   ├── create-flash-card.md
│   └── quiz.md
└── skills/                       # Claude skills
    └── playwright-automation/
        └── SKILL.md
```

## How It Works

`~/.claude/` is a symlink to `~/Dropbox/homedotclaude/`, which syncs non-version-controlled state (history, cache, etc.) across machines via Dropbox.

Within that Dropbox directory, three symlinks point back into this repo:

```
~/Dropbox/homedotclaude/
├── CLAUDE.md  → ~/repos/my-claude/CLAUDE.md
├── commands/  → ~/repos/my-claude/commands/
├── skills/    → ~/repos/my-claude/skills/
└── [other files managed by Dropbox only]
```

## Setup

### Prerequisites

- `~/.claude/` symlinked to `~/Dropbox/homedotclaude/` (or equivalent)
- Git installed

### On a New Machine

```bash
# 1. Clone the repo
git clone <your-repo-url> ~/repos/my-claude

# 2. Remove any existing files/dirs that will be replaced by symlinks

## WARNING THESE COMMANDS CAN DO SOME SERIOUS DAMANGE MAKE SURE YOU KNOW WHAT YOU'RE DOING!
rm -f ~/Dropbox/homedotclaude/CLAUDE.md
rm -rf ~/Dropbox/homedotclaude/commands
rm -rf ~/Dropbox/homedotclaude/skills

# 3. Create symlinks
ln -s ~/repos/my-claude/CLAUDE.md ~/Dropbox/homedotclaude/CLAUDE.md
ln -s ~/repos/my-claude/commands ~/Dropbox/homedotclaude/commands
ln -s ~/repos/my-claude/skills ~/Dropbox/homedotclaude/skills

# 4. Verify
ls -la ~/Dropbox/homedotclaude/CLAUDE.md
ls -la ~/Dropbox/homedotclaude/commands
ls -la ~/Dropbox/homedotclaude/skills
```

### Updating Across Machines

After making changes on one machine:

```bash
# On the machine where you made changes
cd ~/repos/my-claude
git add -A && git commit -m "Describe your changes" && git push

# On other machines
cd ~/repos/my-claude
git pull
# Symlinks automatically pick up the updated files
```

## Adding New Content

### New Command

Add a markdown file to `commands/`:

```bash
# Create the command file
cat > ~/repos/my-claude/commands/my-command.md << 'EOF'
Your command prompt here...
EOF

# Commit
cd ~/repos/my-claude
git add commands/my-command.md
git commit -m "Add my-command"
git push
```

### New Skill

Add a directory under `skills/`:

```bash
mkdir ~/repos/my-claude/skills/my-new-skill
# Create SKILL.md in that directory with the skill definition

cd ~/repos/my-claude
git add skills/my-new-skill
git commit -m "Add my-new-skill"
git push
```

No new symlinks needed - the `skills/` directory symlink covers all skills.

## Using Skills in Claude.ai Web/Desktop

Skills can also be uploaded to Claude.ai for use in the web and desktop apps. This only applies to **skills** - CLAUDE.md and commands are Claude Code (CLI) features only and have no equivalent in the web app.

To upload a skill:

1. Zip the skill folder (the zip should contain the skill folder as its root):
   ```bash
   cd ~/repos/my-claude/skills
   zip -r playwright-automation.zip playwright-automation/
   ```
2. Go to [claude.ai](https://claude.ai)
3. Navigate to **Customize > Skills**
4. Upload the `.zip` file
5. Delete the `.zip` file afterward - it's a disposable artifact, the repo is the source of truth

When you update a skill, you'll need to re-zip and re-upload it to Claude.ai. Claude Code picks up changes automatically via symlinks.

## Resources

- [How to create custom Skills - Claude Help Center](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills)
- [Extend Claude with skills - Claude Code Docs](https://code.claude.com/docs/en/skills)
