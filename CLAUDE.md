# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog built with Astro, deployed to Azure Static Web Apps. The site displays a collection of blog posts with a clean, minimalist design.

## Key Commands

**Development:**
```bash
npm run dev        # Start dev server on port 3001
npm start          # Alternative dev server (default port)
```

**Build:**
```bash
npm run build      # Build for production (outputs to dist/)
npm run preview    # Preview production build
```

## Architecture

**Content Collections:**
- Blog posts are managed via Astro's content collections in `src/content/blog/`
- Collection schema defined in `src/content/config.ts` with required fields: `title` (string) and `date` (Date)
- Posts are markdown files with frontmatter

**Routing:**
- Homepage: `src/pages/index.astro` - Lists all blog posts sorted by date (newest first)
- Blog posts: `src/pages/posts/[...slug].astro` - Dynamic route using `getStaticPaths()` with `prerender: true`
- Posts are statically generated at build time from the content collection

**Layouts:**
- `src/layouts/Layout.astro` - Base layout with global styles, ViewTransitions, and SEO meta tags
  - Takes `tabTitle` prop for page title
  - Includes Google site verification only on homepage
- `src/layouts/PostLayout.astro` - Wrapper for blog posts
  - Takes `frontmatter` prop with title and date
  - Includes back navigation and formatted date display

**Date Formatting:**
Both index and post pages use consistent date formatting:
```typescript
const dateOptions = {
  day: "2-digit",
  month: "short",
  year: "numeric",
} as const;
```

**Styling:**
- Global styles in Layout.astro with CSS custom properties
- Dark theme (`--accent`, `--accent-light`, `--accent-dark`)
- Scoped component styles in individual .astro files

## Deployment

- Auto-deploys to Azure Static Web Apps on push/PR to main branch
- Build output directory: `dist/`
- GitHub Actions workflow: `.github/workflows/azure-static-web-apps-nice-river-0756fda03.yml`

## TypeScript Configuration

Uses Astro's strict TypeScript configuration via `"extends": "astro/tsconfigs/strict"`.
