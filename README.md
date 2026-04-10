# Docusaurus Skills

Agent-agnostic skills for automating Docusaurus documentation setup with LLM-friendly output (llms.txt). Compatible with Claude Code, OpenAI Codex, and other AI coding agents.

## Skills

### docusaurus-docs

Sets up a complete Docusaurus documentation site from scratch, including:

- Docusaurus site creation with TypeScript and classic template
- `@signalwire/docusaurus-plugin-llms-txt` plugin for generating `llms.txt` and `llms-full.txt`
- Automated codebase analysis and documentation planning
- Multi-language (i18n) support
- Parallel documentation generation across multiple sections
- Proper MDX escaping and frontmatter configuration
- Build verification and testing

### docusaurus-docs-cicd

Everything from `docusaurus-docs`, plus automated CI/CD via GitHub Actions:

- GitHub Actions workflows that detect code changes and trigger documentation updates
- Support for multiple AI providers:
  - Claude Code (via `anthropics/claude-code-action@v1`)
  - OpenAI Codex (via `openai/codex-action@v1`)
- Intelligent triggering — only rebuilds docs when relevant source files change
- Infinite loop prevention by excluding the docs folder from workflow triggers
- Automated pull requests for documentation updates
- Full customization for default branch, file extensions, language, and project-specific rules

## Installation

Copy the desired skill folder into your agent's skills directory:

```bash
# For basic Docusaurus setup
cp -r skills/docusaurus-docs ~/.claude/skills/

# For Docusaurus with CI/CD automation
cp -r skills/docusaurus-docs-cicd ~/.claude/skills/
```

## Usage

Once installed, the skills are available as commands in your AI coding agent:

- `/docusaurus-docs` — Set up a Docusaurus docs site with llms.txt support
- `/docusaurus-docs-cicd` — Set up Docusaurus docs with automated CI/CD pipeline

## Requirements

- Node.js 18.0.0+
- An AI coding agent (Claude Code, OpenAI Codex, or similar)
- GitHub repository (for CI/CD skill)
- `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` in GitHub Secrets (for CI/CD skill)

## License

MIT
