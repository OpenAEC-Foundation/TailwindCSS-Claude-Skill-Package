# Sources : Tailwind CSS Skill Package

## Approved Sources

All skill content MUST be verified against these approved sources. No unverified blog posts or AI-generated content.

### Primary Sources

| Source | URL | Type | Last Verified |
|--------|-----|------|---------------|
| Tailwind CSS Official Docs (v3.4) | https://v3.tailwindcss.com/docs/installation | Official Documentation | 2026-05-19 |
| Tailwind CSS Official Docs (v4) | https://tailwindcss.com/docs/installation | Official Documentation | Pending |
| Tailwind CSS v4 Vite install | https://tailwindcss.com/docs/installation/using-vite | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 Next.js install | https://tailwindcss.com/docs/installation/framework-guides/nextjs | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 functions and directives | https://tailwindcss.com/docs/functions-and-directives | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 theme variables | https://tailwindcss.com/docs/theme | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 colors | https://tailwindcss.com/docs/colors | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 padding/spacing | https://tailwindcss.com/docs/padding | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 dark mode | https://tailwindcss.com/docs/dark-mode | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 responsive design | https://tailwindcss.com/docs/responsive-design | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 source detection | https://tailwindcss.com/docs/detecting-classes-in-source-files | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 utility-first philosophy | https://tailwindcss.com/docs/utility-first | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 styling with utility classes | https://tailwindcss.com/docs/styling-with-utility-classes | Official Documentation | 2026-05-19 |
| Tailwind CSS v4 adding custom styles | https://tailwindcss.com/docs/adding-custom-styles | Official Documentation | 2026-05-19 |
| Tailwind CSS v3 configuration | https://v3.tailwindcss.com/docs/configuration | Official Documentation | 2026-05-19 |
| Tailwind CSS v3 plugin API | https://v3.tailwindcss.com/docs/plugins | Official Documentation | 2026-05-19 |
| Tailwind CSS v3 dark mode | https://v3.tailwindcss.com/docs/dark-mode | Official Documentation | 2026-05-19 |
| Tailwind CSS v3 variants | https://v3.tailwindcss.com/docs/hover-focus-and-other-states | Official Documentation | 2026-05-19 |
| Tailwind CSS GitHub Repo | https://github.com/tailwindlabs/tailwindcss | Source Code | Pending |
| Tailwind CSS Releases | https://github.com/tailwindlabs/tailwindcss/releases | Release Notes | Pending |
| Tailwind CSS Blog (v4 launch) | https://tailwindcss.com/blog/tailwindcss-v4 | Official Blog | 2026-05-19 |
| Tailwind CSS Blog (JIT origin) | https://tailwindcss.com/blog/just-in-time-the-next-generation-of-tailwind-css | Official Blog | 2026-05-19 |
| Tailwind CSS Blog Index | https://tailwindcss.com/blog | Official Blog | Pending |
| Tailwind Plugins : Typography | https://github.com/tailwindlabs/tailwindcss-typography | Official Plugin | 2026-05-19 |
| Tailwind Plugins : Forms | https://github.com/tailwindlabs/tailwindcss-forms | Official Plugin | 2026-05-19 |
| Tailwind Plugins : Container Queries | https://github.com/tailwindlabs/tailwindcss-container-queries | Official Plugin | 2026-05-19 |
| Tailwind Plugins : Aspect Ratio | https://github.com/tailwindlabs/tailwindcss-aspect-ratio | Official Plugin | Pending |
| Tailwind CSS v4 Upgrade Guide | https://tailwindcss.com/docs/upgrade-guide | Migration Guide | 2026-05-19 |
| Tailwind CSS v4 Vite Plugin | https://github.com/tailwindlabs/tailwindcss/tree/main/packages/%40tailwindcss-vite | Source Code | Pending |
| Tailwind CSS v4 PostCSS Plugin | https://github.com/tailwindlabs/tailwindcss/tree/main/packages/%40tailwindcss-postcss | Source Code | Pending |
| tailwind-merge | https://github.com/dcastil/tailwind-merge | Companion Library | 2026-05-19 |

### Secondary Sources (use only when primary is insufficient)

| Source | URL | Type | Last Verified |
|--------|-----|------|---------------|
| Tailwind UI Docs | https://tailwindcss.com/plus | Reference Patterns | Pending |
| Tailwind Discord Issues (via GitHub Discussions) | https://github.com/tailwindlabs/tailwindcss/discussions | Community Issues | Pending |
| Tailwind CSS Issues (anti-pattern mining) | https://github.com/tailwindlabs/tailwindcss/issues | Issue Tracker | 2026-05-19 |
| Issue 18136 : dynamic class names not detected | https://github.com/tailwindlabs/tailwindcss/issues/18136 | Anti-pattern source | 2026-05-19 |
| Issue 16346 : @apply broken in v4 without @reference | https://github.com/tailwindlabs/tailwindcss/issues/16346 | Anti-pattern source | 2026-05-19 |
| Issue 16287 : Next.js catch-all glob escape | https://github.com/tailwindlabs/tailwindcss/issues/16287 | Anti-pattern source | 2026-05-19 |
| Issue 16733 : v4.0.8 Astro regression | https://github.com/tailwindlabs/tailwindcss/issues/16733 | Anti-pattern source | 2026-05-19 |
| Issue 19825 : Turbopack arbitrary value miss | https://github.com/tailwindlabs/tailwindcss/issues/19825 | Anti-pattern source | 2026-05-19 |
| Issue 9401 : decimal class purge | https://github.com/tailwindlabs/tailwindcss/issues/9401 | Anti-pattern source | 2026-05-19 |

## Verification Rules

1. **Primary sources ONLY**: Official docs > source code > official tutorials
2. **NEVER use**: Random blog posts, unverified StackOverflow answers, AI-generated content without verification
3. **Version-check**: Ensure source matches target version (v3.4 vs v4). Both supported per skill where divergence matters.
4. **Date-check**: Note last verification date per source
5. **Cross-reference**: If official docs are sparse, verify against source code in `tailwindlabs/tailwindcss` repo
6. **WebFetch**: ALWAYS use WebFetch to verify against latest official documentation (D-006)

## Source Addition Protocol

When discovering a new source during research:
1. Verify it's official or maintained by core team
2. Add to appropriate table above
3. Set "Last Verified" to current date
4. Record in LESSONS.md if the source revealed significant insights
