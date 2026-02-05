# Claude Skills Repository

This repository contains my custom Claude skills. These skills work across Claude.ai (web/desktop), Claude Code (CLI), and the Claude API.

## Repository Structure

```
claude-skills/
├── README.md
├── playwright-automation/
│   └── SKILL.md
└── [other-skills]/
    └── SKILL.md
```

## How Skills Sync Across Environments

### Claude.ai Web + Claude Desktop App
- **Sync**: Automatic between web and desktop app
- **Setup**: Upload once via Settings > Capabilities
- **Update**: Re-upload when you make changes

### Claude Code (CLI)
- **Sync**: Manual per machine via symlinks
- **Setup**: Clone this repo and symlink skills
- **Update**: Pull latest from git (symlinks automatically get updates)

### Claude API
- **Sync**: Manual upload via Console or API
- **Setup**: Upload via Claude Console
- **Update**: Re-upload when you make changes

## Initial Setup

### 1. Clone This Repository

```bash
# Clone to your preferred location
git clone <your-repo-url> ~/repos/claude-skills
cd ~/repos/claude-skills
```

### 2. Set Up for Claude Code (CLI)

On each machine where you use Claude Code:

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Symlink each skill
ln -s ~/repos/claude-skills/playwright-automation ~/.claude/skills/playwright-automation

# Verify
ls -la ~/.claude/skills
```

**Note**: Adjust the repo path (`~/repos/claude-skills`) to match where you cloned it.

### 3. Set Up for Claude.ai (Web/Desktop)

#### Install the Packaging Script Dependencies

First, ensure you have Python available. Then navigate to the skill-creator examples:

```bash
# The packaging script is available in Claude's skill-creator examples
# You can use it directly from there or copy it to your repo
```

#### Package a Skill

```bash
# Package the skill into a .skill file (which is just a zip with .skill extension)
python /mnt/skills/examples/skill-creator/scripts/package_skill.py ~/repos/claude-skills/playwright-automation

# This creates playwright-automation.skill in the current directory
```

#### Upload to Claude.ai

1. Go to [claude.ai](https://claude.ai)
2. Click your profile → **Settings**
3. Navigate to **Capabilities**
4. Ensure **Code execution and file creation** is enabled
5. Scroll to **Skills** section
6. Click **Upload skill**
7. Upload the `playwright-automation.skill` file
8. The skill is now available in both Claude.ai web and Claude Desktop app

## Adding New Machines

When you set up a new machine:

```bash
# 1. Clone the repo
git clone <your-repo-url> ~/repos/claude-skills

# 2. Symlink skills for Claude Code
mkdir -p ~/.claude/skills
ln -s ~/repos/claude-skills/playwright-automation ~/.claude/skills/playwright-automation
# Repeat for each skill

# 3. For Claude.ai web/desktop - no action needed, skills already uploaded
```

## Updating Skills

### For Claude Code (CLI)

```bash
# On any machine, edit the skill
cd ~/repos/claude-skills/playwright-automation
# Edit SKILL.md

# Commit and push
git add -A
git commit -m "Update playwright skill with new patterns"
git push

# On other machines, just pull
cd ~/repos/claude-skills
git pull
# The symlinks automatically point to the updated files
```

### For Claude.ai (Web/Desktop)

After updating a skill in the repo:

```bash
# 1. Package the updated skill
cd ~/repos/claude-skills
python /mnt/skills/examples/skill-creator/scripts/package_skill.py playwright-automation

# 2. Re-upload to Claude.ai
# Go to Settings > Capabilities > Skills
# Remove old version (click X)
# Upload new .skill file
```

## Creating New Skills

```bash
# Navigate to this repo
cd ~/repos/claude-skills

# Use the skill-creator init script
python /mnt/skills/examples/skill-creator/scripts/init_skill.py my-new-skill --path .

# This creates:
# my-new-skill/
#   ├── SKILL.md (edit this)
#   ├── scripts/ (optional)
#   ├── references/ (optional)
#   └── assets/ (optional)

# Customize the skill, then commit
git add my-new-skill
git commit -m "Add new skill: my-new-skill"
git push
```

Then follow the setup steps above to make it available in Claude Code and Claude.ai.

## Validation

Before uploading skills to Claude.ai, validate them:

```bash
python /mnt/skills/examples/skill-creator/scripts/quick_validate.py ~/repos/claude-skills/playwright-automation
```

The packaging script also validates automatically.

## Tips

- **Keep skills small**: Only include what Claude needs to know
- **Test locally first**: Use Claude Code to test skills before uploading to web
- **Version control everything**: Commit frequently with clear messages
- **Document triggers clearly**: The `description` field in frontmatter determines when Claude loads the skill
- **Use gerund naming**: Name skills like "doing-something" (e.g., "playwright-automation", "pdf-processing")

## Resources

- [Claude Skills Documentation](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [Skill Authoring Best Practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices)
- [Agent Skills Standard](https://github.com/anthropics/agent-skills)

## Troubleshooting

### Skill not triggering in Claude Code
- Check that the skill is symlinked: `ls -la ~/.claude/skills`
- Verify the symlink points to the right location: `readlink ~/.claude/skills/playwright-automation`
- Try explicitly mentioning the skill in your prompt: "Use the playwright-automation skill to..."

### Skill not appearing in Claude.ai
- Ensure Code execution is enabled in Settings > Capabilities
- Check that you uploaded the .skill file (not just the folder)
- Try removing and re-uploading the skill

### Changes not reflecting
- **Claude Code**: Pull latest changes from git
- **Claude.ai**: Re-package and re-upload the skill
