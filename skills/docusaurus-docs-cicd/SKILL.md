---
name: docusaurus-docs-cicd
description: Set up Docusaurus documentation with llms.txt plugin AND CI/CD automation via GitHub Actions. Use when the user wants automated documentation updates, CI/CD for docs, GitHub Actions documentation pipeline, or self-updating docs. Triggers on requests like "automate documentation", "set up docs CI/CD", "auto-update docs on PR", "documentation pipeline".
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
---

# Docusaurus Documentation Setup with llms.txt + CI/CD Automation

Generate a complete Docusaurus documentation site with LLM-friendly output and automated CI/CD that keeps docs synchronized with code changes.

For Docusaurus best practices, configuration standards, and troubleshooting, read [references/docusaurus-expert.md](references/docusaurus-expert.md).

## Part 1: Docusaurus Setup

Follow all steps from the `docusaurus-docs` skill (Steps 1-9), then continue with Part 2 below.

### Quick Reference -- Core Setup

```bash
# 1. Create site
npx create-docusaurus@latest docs classic --typescript

# 2. Install llms.txt plugin
cd docs && npm install @signalwire/docusaurus-plugin-llms-txt --legacy-peer-deps

# 3. Configure, clean defaults, write docs, build
# (see docusaurus-docs skill for full details)
```

**llms.txt plugin config for docusaurus.config.ts:**

```typescript
plugins: [
  [
    '@signalwire/docusaurus-plugin-llms-txt',
    {
      content: {
        enableMarkdownFiles: true,
        enableLlmsFullTxt: true,
        includeDocs: true,
        includePages: false,
        includeBlog: false,
      },
      siteTitle: 'PROJECT_NAME',
      siteDescription: 'PROJECT_DESCRIPTION',
    },
  ],
],
```

## Part 2: CI/CD Automation

### Step 1: Choose Your AI Provider

| | Claude Code | OpenAI Codex |
|--|-------------|-------------|
| **GitHub Action** | `anthropics/claude-code-action@v1` | `openai/codex-action@v1` |
| **API Key Secret** | `ANTHROPIC_API_KEY` | `OPENAI_API_KEY` |
| **Agent file** | `.claude/agents/docusaurus-expert.md` | `AGENTS.md` in repo root |
| **Console** | https://console.anthropic.com/ | https://platform.openai.com/ |

### Step 2: Create the Agent Configuration

The CI/CD pipeline needs an agent file that tells the AI how to update documentation. Copy the content from [references/docusaurus-expert.md](references/docusaurus-expert.md) into your repo.

**For Claude Code**, create `.claude/agents/docusaurus-expert.md`:

```bash
mkdir -p .claude/agents
cp references/docusaurus-expert.md .claude/agents/docusaurus-expert.md
```

**For Codex**, create `AGENTS.md` in the repo root:

```markdown
# Agents

## Docusaurus Expert

You are a Docusaurus expert. Read the full instructions at
[.claude/agents/docusaurus-expert.md](.claude/agents/docusaurus-expert.md)
for configuration standards, content organization, and troubleshooting.

When updating documentation:
1. Find the Docusaurus documentation folder (docs/)
2. Update documentation for any changed functionality
3. Add new documentation for new features
4. Update API references if function signatures changed
5. Ensure all code examples match the current implementation
6. Escape {curly braces} in prose outside code blocks for MDX compatibility
```

**For both agents**: Keep both files in your repo. Claude Code reads `.claude/agents/`, Codex reads `AGENTS.md`. The docusaurus-expert content is agent-agnostic.

### Step 3: Create GitHub Actions Workflow

Choose the workflow matching your provider.

#### Option A: Claude Code Workflow

Create `.github/workflows/docusaurus-auto-docs.yml`:

```yaml
name: Docusaurus Documentation Automation

on:
  pull_request:
    branches:
      - main  # CUSTOMIZE: your default branch
    paths:
      # CUSTOMIZE: file types that trigger doc updates
      - '**.kt'
      - '**.java'
      - '**.js'
      - '**.ts'
      - '**.jsx'
      - '**.tsx'
      - '**.py'
      - '**.swift'
      - '**.go'
      - '**.rs'
      # Exclude paths that should NOT trigger docs
      - '!.github/**'
      - '!**/node_modules/**'
      - '!**/dist/**'
      - '!**/build/**'
      - '!docs/**'  # CRITICAL: prevents infinite PR loops

jobs:
  auto-document:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      id-token: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get changed files
        id: changed
        run: |
          git fetch origin ${{ github.event.pull_request.base.ref }}:${{ github.event.pull_request.base.ref }}
          CHANGED_FILES=$(git diff --name-only ${{ github.event.pull_request.base.ref }}...HEAD | tr '\n' ' ')
          echo "files=$CHANGED_FILES" >> $GITHUB_OUTPUT

      - name: Update documentation
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Read and follow the instructions in .claude/agents/docusaurus-expert.md

            Changed files in this pull request:
            ${{ steps.changed.outputs.files }}

            ## Requirements
            1. Find the Docusaurus documentation folder
            2. Update documentation for any changed functionality
            3. Add new documentation for new features
            4. Update API references if function signatures changed
            5. Ensure all code examples match the current implementation
            6. Rebuild llms.txt and llms-full.txt

            ## Project-specific rules
            # CUSTOMIZE: Update these for your project
            - Documentation language: English
            - Follow existing documentation structure and style
            - Include code snippets from actual source files
            - Escape {curly braces} in prose for MDX compatibility

            Focus on documenting the changes found in the modified files above.
          claude_args: "--max-turns 15 --dangerously-skip-permissions"

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: "docs: automated documentation update"
          title: "Documentation Update"
          body: |
            Automated documentation update based on pull request changes.
            **Changed files:**
            ```
            ${{ steps.changed.outputs.files }}
            ```
          branch: docs/auto-${{ github.sha }}
          base: main
```

#### Option B: OpenAI Codex Workflow

Create `.github/workflows/docusaurus-auto-docs.yml`:

```yaml
name: Docusaurus Documentation Automation

on:
  pull_request:
    branches:
      - main  # CUSTOMIZE: your default branch
    paths:
      # CUSTOMIZE: file types that trigger doc updates
      - '**.kt'
      - '**.java'
      - '**.js'
      - '**.ts'
      - '**.jsx'
      - '**.tsx'
      - '**.py'
      - '**.swift'
      - '**.go'
      - '**.rs'
      # Exclude paths that should NOT trigger docs
      - '!.github/**'
      - '!**/node_modules/**'
      - '!**/dist/**'
      - '!**/build/**'
      - '!docs/**'  # CRITICAL: prevents infinite PR loops

jobs:
  auto-document:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get changed files
        id: changed
        run: |
          git fetch origin ${{ github.event.pull_request.base.ref }}:${{ github.event.pull_request.base.ref }}
          CHANGED_FILES=$(git diff --name-only ${{ github.event.pull_request.base.ref }}...HEAD | tr '\n' ' ')
          echo "files=$CHANGED_FILES" >> $GITHUB_OUTPUT

      - name: Update documentation
        uses: openai/codex@v1
        with:
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}
          prompt: |
            Read the AGENTS.md file and the docusaurus-expert instructions it references.

            Changed files in this pull request:
            ${{ steps.changed.outputs.files }}

            Update the Docusaurus documentation in docs/ to reflect these changes.
            Follow the existing structure and style. Escape {curly braces}
            in prose for MDX compatibility. Rebuild with npm run build to
            verify llms.txt and llms-full.txt are regenerated.

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: "docs: automated documentation update"
          title: "Documentation Update"
          body: |
            Automated documentation update based on pull request changes.
            **Changed files:**
            ```
            ${{ steps.changed.outputs.files }}
            ```
          branch: docs/auto-${{ github.sha }}
          base: main
```

### Step 4: Configure GitHub Secrets

1. Go to **Settings > Secrets and Variables > Actions**
2. Add the secret for your chosen provider:
   - **Claude Code**: `ANTHROPIC_API_KEY` (from https://console.anthropic.com/)
   - **Codex**: `OPENAI_API_KEY` (from https://platform.openai.com/)

### Step 4: Configure GitHub Permissions

Go to **Settings > Actions > General > Workflow permissions**:
- Enable "Read repository contents and packages permissions"
- Enable "Allow GitHub Actions to create and approve pull requests"

## How It Works

```
Developer opens PR with code changes
        |
        v
GitHub Action triggers (filtered by file paths)
        |
        v
AI agent reads changed files + existing docs
        |
        v
Docusaurus-expert context guides doc updates
        |
        v
llms.txt + llms-full.txt are regenerated
        |
        v
New PR created: "Documentation Update"
        |
        v
Team reviews and merges the docs PR
```

## Important Notes

### Infinite Loop Prevention
The `!docs/**` path exclusion in the workflow trigger is **critical**. Without it, the documentation PR triggers another documentation update, creating an endless loop.

### Customization Checklist
Before deploying, update these `# CUSTOMIZE` markers:
- [ ] Default branch name (`main` vs `master` vs other)
- [ ] File extensions that trigger updates (match your tech stack)
- [ ] Docusaurus folder path in the exclusion (`!docs/**`)
- [ ] Documentation language
- [ ] Project-specific rules in the prompt
- [ ] Reviewer team (optional)

### llms.txt Plugin
The `@signalwire/docusaurus-plugin-llms-txt` plugin generates:
- `/llms.txt` -- Index of all docs with titles and descriptions
- `/llms-full.txt` -- Complete documentation as single plaintext file
- Individual `.md` files per page in the build output

These are regenerated on every build, including CI/CD builds.

### MDX Escaping
`{variable}` patterns in prose (outside code blocks) must be escaped as `\{variable\}` -- Docusaurus MDX treats them as JSX expressions and the build will fail.

### Dev Server vs Build Server
`npm start` serves only one locale. Use `npm run build && npm run serve` to test all locales and verify llms.txt generation.
