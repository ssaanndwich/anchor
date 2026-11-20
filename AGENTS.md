# AGENTS.md

Instructions for AI coding agents (Claude Code, Cursor, and others) working in this repository. Humans should read README.md first; this file is the agent-facing contract.

## Project overview

Apex Atelier is a custom design system built on IBM's Carbon Design System (v11), plus two consumers of it:

- **`packages/theme`**: the Apex brand theme. It takes Carbon's White and Gray 100 themes and swaps the brand color for copper. It is the only place brand values live.
- **`apps/web`**: the reference storefront (React 19, Vite, TypeScript, `@carbon/react`). Customers pick a GT Series vehicle and order mods and upgrades.
- **`packages/mcp-server`**: a Model Context Protocol server that exposes the design system to AI agents through four tools: `docs_search`, `code_example`, `icon_search` and `token_lookup`.

The architecture is agentic: agents use the design system through the MCP server's tools, and the same tools are deployed so other agents and clients can use them remotely.

## Repository layout

```
apps/
  web/                  Storefront (Vite + React + @carbon/react)
    src/views/          Showroom, Configurator, Orders
    src/data/catalog.ts Vehicles, upgrade options, prices, fitment rules
packages/
  theme/                Apex Sass theme: $apex-light, $apex-dark, button tokens
    src/index.scss
  mcp-server/           MCP server (stdio locally, Streamable HTTP when deployed)
    src/server.ts       Registers tools
    src/tools/          docs.ts, code.ts, icons.ts, tokens.ts
    src/data/           components.json, icons.json
.github/workflows/      CI (ci.yml) and deploy (deploy.yml)
Dockerfile              Builds web + MCP server into one image
AGENTS.md               This file
```

## Setup and commands

Node 22 or later, with npm workspaces. Run everything from the repo root.

```bash
npm install                          # install all workspaces
npm run dev -w apps/web              # storefront at http://localhost:5173
npm run build                        # typecheck and build every workspace
npm run lint                         # oxlint
npm test                             # unit tests (all workspaces)
npm run dev -w packages/mcp-server   # MCP server over stdio
npx @modelcontextprotocol/inspector npx tsx packages/mcp-server/src/server.ts
```

Before you finish any task, `npm run build` and `npm test` must pass. Don't report a task as done if either fails.

## Design system rules (must follow)

These rules match Carbon's own guidelines and the Carbon MCP `code_audit`. If you break one, the PR will be rejected.

1. **Use Carbon components first.** Reach for `@carbon/react` before writing custom UI: Tile, RadioTile, SelectableTile, Tabs, ContentSwitcher, Modal, DataTable, Tag, InlineNotification, UI Shell (Header, SideNav). Only build a custom component if no Carbon component fits, and say why in the PR.
2. **Never hard-code colors.** Use Carbon tokens: Sass `theme.$link-primary`, or CSS `var(--cds-link-primary)`. Hex values are allowed only in `packages/theme`.
3. **Change the brand only in `packages/theme`.** It overrides these tokens and nothing else: `interactive`, `border-interactive`, `icon-interactive`, `link-*`, `focus`, `background-brand`, plus the `--cds-button-primary*` and `--cds-button-tertiary*` custom properties. Leave layers, text, borders and support colors as Carbon's.
4. **No inline styles.** Don't use `style={{…}}`. Put styles in SCSS with a class name, using `@carbon/react/scss/spacing`, `theme` and `type`.
5. **Use Carbon's layout and type scale.** Lay pages out with `Grid` and `Column` (sm 4, md 8, lg 16; spans 1 to 16, never 0). Use `withRowGap` when a grid wraps onto multiple rows. Hide a column at certain breakpoints with a class, not a span of 0. For text, use Carbon type styles (`type.type-style('heading-04')`), not custom font sizes.
6. **Give every Button an explicit `kind`.** Use one primary action per view.
7. **Keep Tags for states that need attention.** Finished or neutral states are plain text.

## Accessibility (must follow)

- **Contrast:** text must meet WCAG 2.1 AA (4.5:1 for text, 3:1 for large text and UI). If you change a brand color, recompute every pairing and update the contrast table in README.md.

  | Pairing | Contrast |
  | --- | --- |
  | White on copper `#A3471F` | 6.0:1 |
  | Copper on White | 6.0:1 |
  | Light copper `#F2A27A` on Gray 100 | 8.8:1 |

- **Labels:** icon-only controls need an `aria-label`. Illustrations need `role="img"` and a description.
- **Keyboard:** everything must work by keyboard. Don't remove focus outlines.
- **Both themes:** check every UI change in the White and Gray 100 themes, and at 390px (phone) and 1440px (desktop) widths.

## Working with the MCP servers

Two Carbon MCP servers can be available to agents. Use the right one:

| Server | Use it for |
| --- | --- |
| IBM's hosted Carbon MCP (`https://mcp.carbondesignsystem.com/mcp`, IBMid sign-in) | Official Carbon guidance, component examples and `code_audit` |
| This repo's `packages/mcp-server` | Apex-specific data: brand tokens, our component catalog and icons |

**When building UI:**
- Look up the component with `code_search` (IBM) or `code_example` (ours) before writing it.
- Run IBM's `code_audit` on every changed `.tsx` and `.scss` file before opening a PR. Fix all errors. Explain any warnings you leave.

**When changing the MCP server:**
- Every tool returns `{ content: [{ type: "text" as const, text }] }` and sets `isError: true` on failures.
- Validate input with zod schemas in `server.ts`. Don't trust raw arguments.
- Tools are read-only. Never add a tool that writes files, runs shell commands or makes network calls on the user's behalf.
- `token_lookup` reads values from `@carbon/themes` and `@carbon/layout`. Don't copy token values into JSON.
- When adding a component to `src/data/components.json`,
