# Sindri — project guide for Claude Code

Sindri is Sonni's personal React component library. It is the UI foundation for all of her new web development, and a public showcase of her React/UI skills. Named after the dwarf smith of Norse mythology who forged Mjölnir and Draupnir — craftsmanship is the point.

The repo is **empty as of 2026-10-13**. This file records the decisions and direction agreed during planning so work can start from them. Items marked **Open** still need Sonni's call — ask before acting on them.

## Purpose and audience

- **Public, but not for others to use.** The repo and docs site are public so people can read the code. There is no support commitment, no npm publishing, and no API-stability promise to outsiders. The README says so.
- **Consumers are Sonni's own apps.** Consumed through the pnpm workspace, or as a git dependency from separate app repos.
- **Showcase.** The deployed docs site is the portfolio piece, so docs quality, theming and accessibility matter as much as the components.
- **APIs may break freely.** Refactor aggressively when real app usage shows a better shape.

## Stack (decided)

- React + TypeScript (strict)
- Tailwind CSS (v4), with design tokens as CSS variables
- Headless primitives underneath; styling is ours
- Apps that consume Sindri: Vite + TanStack Router / TanStack Query (chosen over Next.js)
- Testing: Vitest + Testing Library, Playwright
- Backend for those apps is Kotlin/Spring Boot — not relevant inside this repo

## Headless primitive layer

- **Recommended: Base UI** (`@base-ui/react`). Stable v1.0 in 2026, from the creators of Radix, Floating UI and Material UI, actively maintained, broad component set, data attributes style cleanly with Tailwind.
- **Alternative: React Aria Components.** Choose this only if accessibility depth is to be the headline of the showcase; more code per component.
- **Not chosen:** Headless UI (too small a component set), Radix (releases have slowed; Base UI is effectively its successor).
- **Open:** confirm Base UI with Sonni before scaffolding.
- Do not build Sindri by copying shadcn/ui components. Borrowing patterns is fine; shadcn output is instantly recognisable and undermines the showcase.

## Repository structure (planned)

pnpm monorepo:

```
packages/ui        # the library (Vite library mode or tsup)
apps/docs          # Storybook — the public docs site / portfolio
apps/playground    # small Vite + TanStack Router app using the library like a real app
```

## Component architecture

**Tokens first.** Colour, radius, spacing, typography and motion are CSS variables consumed by Tailwind. Theming and dark mode work from day one. Components never use raw hex values or arbitrary Tailwind colours.

**Variants.** Use `tailwind-variants` or `cva` together with `tailwind-merge`, so a consumer's `className` overrides safely.

**Three tiers:**
1. Primitives — Button, Input, Checkbox, Switch, Badge
2. Composites on the headless layer — Dialog, Popover, Select, Combobox, Menu, Tabs, Toast, Tooltip
3. Stack-aware patterns — DataTable on TanStack Table, form fields wired to TanStack Form. This tier is what makes the library distinctly Sonni's.

**Build order (suggested):** tokens → Button → Dialog end to end (component, story, tests, docs) as the reference pattern, then the rest.

## API conventions (these also make it AI-friendly)

- **One vocabulary everywhere.** The same prop names (`variant`, `size`, `tone`, …) with the same allowed values on every component. Never `color` on one and `intent` on another.
- **Strict string-literal unions** for variant props, so the type checker rejects invented values.
- **JSDoc on every component and prop**, including a one-line "use this when / use X instead when". It feeds editor tooltips, the `.d.ts` files, and the Storybook components manifest.
- **One composition pattern.** Compound parts (`Dialog.Root`, `Dialog.Trigger`, …) for composites; flat props for primitives. Don't mix the two within a component.
- **Never re-export the headless library raw.** Consumers only ever see Sindri's API.
- **`className` is always accepted and merged** through `tailwind-merge`.

## Every component ships with

- The component, with full JSDoc
- A Storybook story covering every variant and state; stories are the canonical usage examples
- Vitest + Testing Library tests, including an `axe` accessibility check
- Playwright visual snapshots of its stories

## AI-friendliness

The library should be easy for AI agents both to extend and to consume.

- **This file** guides agents working inside the repo.
- **A consumer guide** for app repos — an `AGENTS.md` snippet or a Claude skill kept in this repo and installed in each app. It says: use Sindri before writing UI; never hand-roll buttons or inputs; use tokens, not raw Tailwind colours; here are the patterns for forms and tables.
- **Storybook MCP** — `@storybook/addon-mcp`, with the components manifest enabled. It builds the manifest from stories, prop types and JSDoc, and lets agents look up components, generate stories and run tests. Still a preview feature, React-only (which suits us). The consumer guide should tell agents to check the MCP before using any prop, and never to guess props.
- **Machine-readable docs** on the public site: `llms.txt`, `llms-full.txt`, and a plain-markdown version of each docs page.
- **Guardrails that let agents self-correct:** ESLint rules forbidding direct `@base-ui` imports in apps and raw colour values; strict TypeScript; `axe` in tests. One command runs typecheck + lint + test.
- **Skip for now:** a bespoke MCP server, or a shadcn-style registry. Storybook MCP and readable types cover discovery. Revisit only if a real gap appears.

## Releases

No npm publishing and no Changesets. Keep a CHANGELOG and tag releases — enough to show versioning discipline.

## Showcase extras

- Keep decision records (ADRs) in `docs/adr/` — why Base UI, why tokens, the API conventions, the AI-friendliness approach. Reviewers read these closely.
- The docs site should have a theme and dark-mode toggle, live examples, and a page explaining the AI-ready design.
- Possible docs tagline drawing on Draupnir, Odin's ring that drips copies of itself: reusable by design.

## Open decisions

- Confirm Base UI vs React Aria.
- Package name: proposed `@skrymer/sindri` (workspace only, never published).
- License: MIT (reads as confident, costs nothing) vs reference-only. Not yet chosen.
- `tailwind-variants` vs `cva`.
- Docs site domain.
- Storybook vs Ladle for docs — Storybook is preferred because of its MCP addon.

## Working agreement for agents

- Before finishing any change, run the single check command (typecheck + lint + test) once it exists.
- Follow the API conventions above for every new component; if a convention doesn't fit, raise it rather than diverging.
- When a new decision is made, record it here or as an ADR.
