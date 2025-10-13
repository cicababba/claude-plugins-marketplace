# Contributing to Claude Code Plugins Marketplace

Thank you for your interest in contributing! This document provides guidelines for adding plugins to this marketplace.

## 🎯 What Can You Contribute?

- **New Plugins** - Complete plugin packages with commands, agents, and documentation
- **Plugin Improvements** - Enhancements to existing plugins
- **Documentation** - Better docs, examples, and guides
- **Bug Fixes** - Corrections to existing functionality

## 📋 Plugin Submission Guidelines

### 1. Plugin Structure

Your plugin must follow this structure:

```
your-plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── commands/                # Slash commands (optional)
│   └── command-name.md
├── agents/                  # Sub-agents (optional)
│   └── agent-name.md
├── hooks/                   # Hooks (optional)
│   └── hook-name.sh
└── README.md               # Plugin documentation
```

### 2. Plugin Metadata (`plugin.json`)

```json
{
  "name": "your-plugin-name",
  "description": "Clear, concise description of what your plugin does",
  "version": "1.0.0",
  "author": {
    "name": "Your Name",
    "email": "your.email@example.com"
  }
}
```

### 3. Documentation Requirements

Your plugin's README.md must include:

- **Purpose** - What problem does it solve?
- **Installation** - How to install the plugin
- **Usage** - Command examples and workflows
- **Commands** - List of all slash commands
- **Agents** - Description of included sub-agents
- **Examples** - Real-world usage scenarios

### 4. Quality Standards

- ✅ All commands must have clear descriptions
- ✅ Agents must specify their expertise and tools
- ✅ Code must be tested and functional
- ✅ Documentation must be in English
- ✅ Follow existing naming conventions
- ✅ No hardcoded paths or credentials

## 🔄 Contribution Process

### Step 1: Fork and Clone

```bash
git clone https://github.com/your-username/claude-plugins-marketplace
cd claude-plugins-marketplace
git checkout -b feature/your-plugin-name
```

### Step 2: Add Your Plugin

1. Create your plugin directory
2. Add all necessary files
3. Update `marketplace.json`:

```json
{
  "name": "claude-plugins-marketplace",
  "owner": {
    "name": "Marketplace Owner",
    "email": "owner@example.com"
  },
  "plugins": [
    {
      "name": "your-plugin-name",
      "source": "./your-plugin-name",
      "description": "Your plugin description"
    }
  ]
}
```

### Step 3: Test Your Plugin

```bash
# Test installation locally
/plugin marketplace add file:///absolute/path/to/claude-plugins-marketplace
/plugin install your-plugin-name

# Test all commands
/your-command
```

### Step 4: Submit Pull Request

Create a PR with:

- **Title**: `feat: Add [plugin-name] plugin`
- **Description**:
  - What does the plugin do?
  - What commands/agents are included?
  - Screenshots or examples (if applicable)
  - Testing performed

## ✅ Review Checklist

Before submitting, ensure:

- [ ] Plugin follows the required structure
- [ ] `plugin.json` is properly formatted
- [ ] README.md is comprehensive
- [ ] All commands work as documented
- [ ] No sensitive information is included
- [ ] Plugin name is unique and descriptive
- [ ] Version follows semantic versioning (1.0.0)
- [ ] License is specified (if different from marketplace)

## 🐛 Reporting Issues

Found a bug? Please open an issue with:

- **Plugin name** affected
- **Expected behavior**
- **Actual behavior**
- **Steps to reproduce**
- **Environment** (OS, Claude Code version)

## 💡 Suggesting Enhancements

Have an idea? Open an issue with:

- **Feature description**
- **Use case** - Why is it needed?
- **Proposed implementation** (if you have ideas)

## 📜 Code of Conduct

- Be respectful and constructive
- Focus on the problem, not the person
- Welcome newcomers and help them learn
- Give credit where credit is due

## 📞 Questions?

- Open a [Discussion](https://github.com/your-username/claude-plugins-marketplace/discussions)
- Check existing [Issues](https://github.com/your-username/claude-plugins-marketplace/issues)
- Review the [Claude Code Docs](https://docs.claude.com/en/docs/claude-code/overview)

---

Thank you for contributing to the Claude Code community! 🚀
