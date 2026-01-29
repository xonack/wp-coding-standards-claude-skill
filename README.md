# claude-skill-wp-coding-standards

Comprehensive WPCS enforcement skill covering WordPress-Core, WordPress-Extra, WordPress-VIP standards, Yoda conditions, sanitization/escaping, nonce verification, and auto-fix workflows.

## Installation

### Claude Code Plugin Marketplace

```bash
/plugin install https://github.com/xonack/claude-skill-wp-coding-standards
```

### Manual Installation

Copy the skill file to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/wp-coding-standards
cp skills/wp-coding-standards/SKILL.md ~/.claude/skills/wp-coding-standards/SKILL.md
```

## Usage

Once installed, the skill activates automatically when working on relevant WordPress tasks. You can also invoke it directly:

```
/wp-coding-standards
```

## Structure

```
claude-skill-wp-coding-standards/
├── README.md
├── LICENSE
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── wp-coding-standards/
        └── SKILL.md
```

## Contributing

PRs welcome. Please follow the [Agent Skills specification](https://agentskills.io/specification) for SKILL.md format.

## License

MIT
