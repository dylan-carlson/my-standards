# my-standards

My personal coding standards and conventions, packaged as skills for Claude Code.

This repo is a Claude Code plugin (`my-standards`, v1.0.0) that also serves as its own plugin marketplace (`my-standards-marketplace`).

## Repository layout

```
.
├── .claude-plugin/
│   ├── marketplace.json   # Marketplace manifest listing the my-standards plugin
│   └── plugin.json        # Plugin manifest (name, description, version, author)
└── skills/
    └── api-endpoints/
        └── SKILL.md       # Conventions for building API endpoints
```

### `.claude-plugin/plugin.json`

Defines the plugin:

| Field | Value |
|---|---|
| `name` | `my-standards` |
| `version` | `1.0.0` |
| `author` | Dylan Carlson |
| `description` | My personal coding standards and conventions, packaged as skills for Claude Code. |

### `.claude-plugin/marketplace.json`

Defines a marketplace named `my-standards-marketplace` (owner: Dylan Carlson) with one entry, the `my-standards` plugin, sourced from the repo root (`"source": "."`).

## Installation

From Claude Code, add this repo as a marketplace, then install the plugin:

```
/plugin marketplace add <path-or-git-url-to-this-repo>
/plugin install my-standards@my-standards-marketplace
```

## Skills

### `api-endpoints`

Guides the creation of new API endpoints. Claude uses it when adding routes, handlers, or controllers to a project. If the project already does something differently, the skill tells Claude to follow the project and point out the difference.

It covers:

- **File layout**: one folder per resource at `src/routes/<resource>/`, with separate files for routes, handlers, service, validation schema, and tests. Every router is registered in `src/routes/index.ts` under `/api/v1`.
- **Naming**: plural kebab-case URL paths, HTTP methods as the verb, kebab-case files, camelCase verb-first handlers (`listUsers`, `createUser`), PascalCase schemas with a suffix (`CreateUserBody`), and camelCase JSON fields.
- **Error handling**: validate all input first (`400` / `VALIDATION_ERROR`), throw typed errors from services (`NotFoundError`, `ConflictError`, and so on), and let a single central error middleware build responses. Unknown errors become `500` / `INTERNAL_ERROR` with no internals leaked. Includes a status code reference table.
- **Response shape**: a standard envelope with either `data` (plus `meta`, including pagination) or `error` (`code`, `message`, `details`), never both. Timestamps are ISO 8601 UTC and IDs are strings.
- **Pre-commit checklist**: route registration, schemas, auth, typed errors, envelope and status codes, tests (success, `400`, `404`, auth failures), passing tests, lint, and type checks, updated API docs, and a clean diff.

See [`skills/api-endpoints/SKILL.md`](skills/api-endpoints/SKILL.md) for the full rules.
