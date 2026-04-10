---
name: docusaurus-docs
description: Set up Docusaurus documentation with llms.txt plugin for any project. Use when the user wants to create project documentation, set up a docs site, generate LLM-friendly documentation, or add llms.txt/llms-full.txt support. Triggers on requests like "document this project", "create docs", "set up documentation", "add llms.txt".
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
---

# Docusaurus Documentation Setup with llms.txt

Generate a complete Docusaurus documentation site with LLM-friendly output (llms.txt + llms-full.txt) for any project.

For Docusaurus best practices, configuration standards, and troubleshooting, read [references/docusaurus-expert.md](references/docusaurus-expert.md).

## Steps

### 1. Create Docusaurus Site

```bash
npx create-docusaurus@latest docs classic --typescript
```

### 2. Install llms.txt Plugin

```bash
cd docs && npm install @signalwire/docusaurus-plugin-llms-txt --legacy-peer-deps
```

### 3. Configure docusaurus.config.ts

Update the config with:
- Project title, tagline, and metadata
- llms.txt plugin configuration
- i18n if multilingual support is needed
- Proper sidebar structure
- Theme colors matching the project's branding

**llms.txt plugin configuration:**

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

### 4. Clean Up Defaults

Remove default Docusaurus content:
- `docs/docs/tutorial-basics/`
- `docs/docs/tutorial-extras/`
- `docs/docs/intro.mdx` (if creating own intro.md)
- `docs/blog/`
- Default `HomepageFeatures` component

### 5. Analyze Codebase

Before writing documentation, thoroughly analyze the project:
- Read README, CLAUDE.md, AGENTS.md, build files, manifests
- Understand architecture, features, dependencies
- Map out the source code structure
- Identify key interfaces, classes, and data flows

### 6. Plan Documentation Structure

Create a logical sidebar structure. Typical categories:
- **Introduction** -- What the project is, key features
- **Getting Started** -- Prerequisites, setup, configuration
- **Architecture** -- Layers, patterns, data flow, DI
- **Features** -- One page per major feature
- **UI/Screens** -- User-facing components (if applicable)
- **Data Layer** -- Database, storage, APIs
- **API Reference** -- Interfaces, classes, method signatures

### 7. Write Documentation

Use **parallel agents** to write multiple doc sections simultaneously. Each agent should:
1. Read the relevant source files
2. Write well-structured Markdown with frontmatter (`title`, `description`, `sidebar_position`)
3. Include code snippets from the actual source
4. Add text-based diagrams for architecture/data flow
5. Escape `{curly braces}` in prose outside code blocks for MDX compatibility

### 8. Add i18n Support (Optional)

If multilingual:
1. Add locales to `docusaurus.config.ts` i18n config
2. Add `localeDropdown` to navbar items
3. Run `npx docusaurus write-translations --locale LOCALE`
4. Translate docs to `i18n/LOCALE/docusaurus-plugin-content-docs/current/`
5. Translate UI strings in `i18n/LOCALE/code.json`
6. Use `<Translate>` component in React pages (e.g., homepage)

### 9. Build and Verify

```bash
cd docs && npm run build
```

Verify:
- Build succeeds for all locales
- `llms.txt` and `llms-full.txt` are generated in the build output
- No broken links or MDX errors
- Test with `npm run serve` for multi-locale

## Important Notes

- **MDX escaping**: `{variable}` in prose (outside code blocks) must be `\{variable\}` -- Docusaurus treats them as JSX expressions
- **Dev server limitation**: `npm start` serves only one locale at a time. Use `npm run build && npm run serve` to test all locales
- **Frontmatter**: Always include `title`, `description`, `sidebar_position`
- **Code snippets**: Use actual code from the project, not generic examples
- **Parallel agents**: Spin up 5-8 agents for large codebases to write docs concurrently
- **llms.txt output**: After build, files are at `/llms.txt` (index) and `/llms-full.txt` (full content)
