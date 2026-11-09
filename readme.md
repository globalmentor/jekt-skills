# Jekt™ Skills

[Jekt](https://jekt.dev/) is a context architecture for software projects: it keeps a project's tickets, plans, designs, and minutes as plain files in its repository, where people and AI assistants work with them directly. This repository provides the Jekt skill, which teaches AI agents to find and use that context and to carry out Jekt operations such as creating tickets, writing plans, and updating minutes. Jekt is developed and stewarded by [GlobalMentor, Inc.](https://www.globalmentor.com/), which publishes this skill.

The skill follows the [Agent Skills](https://agentskills.io/) format and works with agents that support it, including GitHub Copilot, Claude Code, Codex, and Cursor.

## Installation

Install the skill in each repository that uses Jekt, so that it is available to everyone working there, including cloud agents. To use it in all your repositories, install it for your user instead. Use one installation method for each agent; a skill installed twice gives the agent two copies of it.

### Skill Installers

With [`skills`](https://github.com/vercel-labs/skills), which requires Node.js:

```bash
npx skills add globalmentor/jekt-skills
```

With the [GitHub CLI](https://cli.github.com/), version 2.90 or later:

```bash
gh skill install globalmentor/jekt-skills jekt
```

Both install into the current repository by default; commit the installed skill so that collaborators get it. To install for your user, add `-g` to the `skills` command or `--scope user` to the `gh skill` command.

### Plugins

In Claude Code:

```text
/plugin marketplace add globalmentor/jekt-skills
/plugin install jekt@jekt-skills
```

In GitHub Copilot CLI:

```bash
copilot plugin install globalmentor/jekt-skills
```

In VS Code: [Install in VS Code](https://vscode.dev/redirect?url=vscode%3A%2F%2Fchat-plugin%2Finstall%3Fsource%3Dglobalmentor%252Fjekt-skills%26plugin%3Djekt), or run **Chat: Install Plugin from Source** with `https://github.com/globalmentor/jekt-skills`.

To recommend the plugin to everyone working in a repository, add the following to its `.github/copilot/settings.json` or `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "jekt-skills": {
      "source": {
        "source": "github",
        "repo": "globalmentor/jekt-skills"
      }
    }
  },
  "enabledPlugins": {
    "jekt@jekt-skills": true
  }
}
```

### Manual Installation

Copy `skills/jekt/` into a folder your agent reads skills from, such as `.agents/skills/` in your repository or `~/.agents/skills/` for your user.

## Getting Started

In a repository where the skill is installed, ask your agent to initialize Jekt, for example: "Initialize Jekt in this repository." To learn what Jekt is and how it works, see [jekt.dev](https://jekt.dev/).

## Updating

Run `npx skills update` or `gh skill update`, or update the plugin in your agent. Releases are tagged `v{version}`; to install a particular release, name its tag, as in `npx skills add globalmentor/jekt-skills@v0.1.0` or `gh skill install globalmentor/jekt-skills jekt@v0.1.0`.

## License

The Jekt skill is licensed under the [Apache License 2.0](license.md). "Jekt" and the Jekt logo are trademarks of GlobalMentor, Inc.
