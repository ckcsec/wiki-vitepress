# Usage Guide

This guide explains how to use CKCsec Wiki as an English reader while the project continues its internationalization work.

## Reading The Site

- Use the language menu in the navigation bar to switch between Chinese and English.
- Start from the English homepage for project-level information.
- Use the Chinese site for the deepest technical article coverage today.
- Use local search to find article titles, categories, and project pages.

## Contributing

The project welcomes improvements that make the knowledge base easier to understand and maintain.

Good first contributions include:

- fixing typos or broken links
- improving Markdown formatting
- adding source references
- translating high-value Chinese entry pages into English
- proposing clearer sidebar or category organization

## Local Development

Install dependencies and start the VitePress server:

```bash
npm install
npm run docs:dev
```

Build the static site before submitting larger changes:

```bash
npm run docs:build
```

The default Chinese site lives under `docs/`. English pages live under `docs/en/`.
