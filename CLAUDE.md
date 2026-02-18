# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Multilingual (Italian/English) landing page for toto-castaldi.com showcasing three software projects: Docora, Lumio, and Helix. Built with Hugo, hosted on GitHub Pages.

## Commands

```bash
# Local development server with drafts
hugo server -D

# Production build
hugo --minify
```

## Architecture

- **Hugo** static site generator with built-in i18n
- **GitHub Pages** deployment (GitHub Actions CI/CD)
- **Custom domains**: toto-castaldi.com and www.toto-castaldi.com
- **Languages**: Italian (`it`) and English (`en`)

## Content

Projects are defined as data/content entries, each with:
- Name
- Short description (in both IT and EN)
- URL to subdomain (docora/lumio/helix.toto-castaldi.com)

## Planning

Project planning docs live in `.planning/` — managed by GSD workflow. See `.planning/PROJECT.md` for full context.
