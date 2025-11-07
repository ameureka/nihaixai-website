# Repository Guidelines

## Project Structure & Module Organization
The repository root stores discussion notes (`codex-技巧.md`, `设计讨论.md`) and all UI deliverables inside `ui-code/`. Each page prototype lives in a named subdirectory (for example, `ui-code/倪海厦内容网站主页/`) and contains `code.html` alongside a `screen.png` reference capture. Keep any additional assets next to the page they support, and bundle shared resources in a clearly labeled folder. Archived design exports stay in `ui-code/stitch_.zip`; do not modify its contents directly—extract to a new folder when updating.

## Build, Test, and Development Commands
These pages are static Tailwind prototypes and run without a build step. Use `python3 -m http.server` from `ui-code` to serve the files locally, or open `code.html` directly in your browser. If you introduce tooling, document the command in this guide and supply an npm script or Make target.

## Coding Style & Naming Conventions
Follow the existing two-space indentation in `code.html` files and favor semantic HTML tags. Tailwind utility classes should mirror the current palette (`primary`, `background-light`, etc.) defined via the CDN config block—extend colors there before reusing hard-coded hex values. Name new directories with concise page descriptors, keeping screenshots as `screen.png` for quick visual diffs.

## Testing Guidelines
No automated tests currently exist; perform manual verification in at least one desktop and one mobile viewport. Capture updated screenshots when layout or styling changes so reviewers can compare against `screen.png`. Run a quick Lighthouse audit when adjusting performance-sensitive sections.

## Commit & Pull Request Guidelines
Use Conventional Commit prefixes (`feat:`, `fix:`, `style:`, `docs:`) and keep subject lines under 72 characters. Describe the specific page touched (for example, `feat: expand hero CTA on 人纪内容页面`). Pull requests should summarize the change scope, link any relevant discussion threads, and attach before/after screenshots from the affected directories. Note any manual accessibility or browser checks in the PR body.
