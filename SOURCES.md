# Sources : Tailwind CSS Skill Package

## Approved Sources

All skill content MUST be verified against these approved sources. No unverified blog posts or AI-generated content.

### Primary Sources

| Source | URL | Type | Last Verified |
|--------|-----|------|---------------|
| Tailwind CSS Official Docs (v3.4) | https://v3.tailwindcss.com/docs/installation | Official Documentation | Pending |
| Tailwind CSS Official Docs (v4) | https://tailwindcss.com/docs/installation | Official Documentation | Pending |
| Tailwind CSS GitHub Repo | https://github.com/tailwindlabs/tailwindcss | Source Code | Pending |
| Tailwind CSS Releases | https://github.com/tailwindlabs/tailwindcss/releases | Release Notes | Pending |
| Tailwind CSS Blog (v4 launch) | https://tailwindcss.com/blog/tailwindcss-v4 | Official Blog | Pending |
| Tailwind CSS Blog Index | https://tailwindcss.com/blog | Official Blog | Pending |
| Tailwind Plugins : Typography | https://github.com/tailwindlabs/tailwindcss-typography | Official Plugin | Pending |
| Tailwind Plugins : Forms | https://github.com/tailwindlabs/tailwindcss-forms | Official Plugin | Pending |
| Tailwind Plugins : Container Queries | https://github.com/tailwindlabs/tailwindcss-container-queries | Official Plugin | Pending |
| Tailwind Plugins : Aspect Ratio | https://github.com/tailwindlabs/tailwindcss-aspect-ratio | Official Plugin | Pending |
| Tailwind CSS v4 Upgrade Guide | https://tailwindcss.com/docs/upgrade-guide | Migration Guide | Pending |
| Tailwind CSS v4 Vite Plugin | https://github.com/tailwindlabs/tailwindcss/tree/main/packages/%40tailwindcss-vite | Source Code | Pending |
| Tailwind CSS v4 PostCSS Plugin | https://github.com/tailwindlabs/tailwindcss/tree/main/packages/%40tailwindcss-postcss | Source Code | Pending |
| tailwind-merge | https://github.com/dcastil/tailwind-merge | Companion Library | Pending |

### Secondary Sources (use only when primary is insufficient)

| Source | URL | Type | Last Verified |
|--------|-----|------|---------------|
| Tailwind UI Docs | https://tailwindcss.com/plus | Reference Patterns | Pending |
| Tailwind Discord Issues (via GitHub Discussions) | https://github.com/tailwindlabs/tailwindcss/discussions | Community Issues | Pending |
| Tailwind CSS Issues (anti-pattern mining) | https://github.com/tailwindlabs/tailwindcss/issues | Issue Tracker | Pending |

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
