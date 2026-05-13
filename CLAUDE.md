# CLAUDE.md

Guidance for Claude Code (and other AI assistants) working in this repo.

## Project

Static marketing site for **4 By Grace** church, served at `4bygrace.info`.

- Plain HTML + CSS, no framework, no build step.
- Hosted on **Cloudflare Pages**, auto-deployed from GitHub.
- Production branch: `main` → live at `4bygrace.info`.

## File map

| Path          | Purpose                                                  |
| ------------- | -------------------------------------------------------- |
| `index.html`  | Single page. All copy and structure lives here.          |
| `styles.css`  | All styling. Uses CSS custom properties (see `:root`).   |
| `_headers`    | Cloudflare Pages response headers (CSP, caching, HSTS).  |
| `robots.txt`  | Crawler directives.                                      |
| `.gitignore`  | Ignores Node/build artifacts (none currently produced).  |

There is no `package.json`, no bundler, no test suite. Edits ship as-is.

## Deploy workflow

Cloudflare Pages watches this repo. Three environments map to three branch types:

1. **Dev preview — feature branch.** Pushing to any non-`main` branch (e.g. `claude/<slug>`) auto-deploys to a unique Cloudflare Pages preview URL of the form `<branch-slug>.<project>.pages.dev`. Use this to eyeball edits before opening a PR.
2. **PR preview — pull request.** Opening a PR against `main` causes Cloudflare to comment on the PR with a per-commit preview URL. Use this for review.
3. **Production — `main`.** Merging the PR triggers the production deploy to `4bygrace.info`. Do not push directly to `main`.

Branch slug rule: Cloudflare lowercases the branch name and replaces `/` and other non-alphanumerics with `-`. So `claude/card-code-edit-preview-u9zmP` becomes something like `claude-card-code-edit-preview-u9zmp`.

If `dev.4bygrace.info` or `preview.4bygrace.info` ever get configured as Pages custom-domain aliases, document the mapping here.

## Conventions

### HTML
- Single page; sections are `<section id="...">` so the hero CTAs (`#visit`, `#times`) anchor-link to them. Keep IDs stable if you rename sections — update both the section and the `href`.
- Keep the document semantic: `header.hero` → `main` → `footer`. New content goes inside a new `<section>` in `main`.

### CSS
- All design tokens live in `:root` (`--bg`, `--paper`, `--accent`, `--text`, fonts). Prefer adjusting tokens over hardcoding colors/fonts in rules.
- Fonts are loaded from Google Fonts (`EB Garamond` for headings, `Inter` for body). If you add a new weight, update the `<link>` in `index.html` too.
- Mobile-friendly by default — `clamp()` on the h1, `max-width` on `main`. Don't add fixed widths.

### `_headers` and CSP
The Content-Security-Policy in `_headers` is strict:
- `script-src 'self'` — no inline scripts, no third-party JS. If you need JS, add it as a `.js` file in the repo.
- `style-src 'self' https://fonts.googleapis.com` — inline `<style>` is blocked. Put styles in `styles.css`.
- `font-src 'self' https://fonts.gstatic.com` — Google Fonts is whitelisted; any other font host needs to be added here.
- `img-src 'self' data:` — external image hosts need to be added before you can hotlink images.

When adding a feature that requires loosening CSP, update `_headers` in the same commit and explain why in the commit message.

### Cache headers
`_headers` caches `styles.css` for 1 day and HTML for 5 minutes. If you rename CSS or add new assets that need different cache rules, update `_headers`.

## Working on this repo as an AI

- **Always develop on a feature branch**, never commit to `main` directly. Branch name `claude/<short-slug>` is the convention.
- **One change per PR.** This repo is small enough that PRs should be focused (a section, a style pass, a copy edit), not omnibus.
- **Verify locally before pushing** by opening `index.html` in a browser, or by serving the directory (`python3 -m http.server`) and visiting `localhost:8000`. There's no build step to run.
- **After pushing,** the Cloudflare branch-preview URL is the source of truth for "did it render correctly." Wait for the deploy to finish (~30–60s) and load that URL.
- **No third-party scripts or trackers** — keeping CSP `script-src 'self'` is intentional.
- **Don't add a build tool** (Vite/webpack/etc.) without a reason. The whole point of this repo is that it stays a no-build static site.
- **Keep copy reverent and plain.** This is a church site; avoid marketing-speak and avoid emoji unless asked.

## Common tasks

- **Edit copy:** modify `index.html`. No other file changes needed.
- **Tweak design:** prefer editing `:root` tokens in `styles.css`. For new components, add classed elements and a CSS block in the relevant section of `styles.css` (file is ordered roughly top-to-bottom of the page).
- **Add a section:** add `<section id="newid">…</section>` in `index.html`; add styles in `styles.css` if needed; consider whether the hero CTAs should link to it.
- **Add an asset (image, icon):** place at repo root or in a new subdirectory, reference with a relative path, and verify `_headers` CSP `img-src` covers the source.
