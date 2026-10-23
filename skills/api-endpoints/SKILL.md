---
description: Guides the creation of new API endpoints. Use when adding routes, handlers, or controllers to the project.
---

# API Endpoints

Follow these conventions whenever you add or change an API endpoint. If the project already does something differently, match the project and point out the difference to the user.

## Where route files live

- Each resource gets its own folder: `src/routes/<resource>/`.
- Inside it:
  - `<resource>.routes.ts`: maps HTTP methods and paths to handlers. Keep it to routing only, with no business logic.
  - `<resource>.handlers.ts`: request handlers. They parse input, call the service, and shape the response.
  - `<resource>.service.ts`: business logic and data access. It never touches `req` or `res`.
  - `<resource>.schema.ts`: request validation schemas (body, params, query).
  - `<resource>.test.ts`: tests for the endpoint.
- Register every resource router in `src/routes/index.ts`, mounted under `/api/v1`.

## Naming conventions

- **URL paths**: plural, kebab-case nouns. Use `/api/v1/user-profiles/:id`, not `/getUserProfile`.
- **HTTP methods carry the verb**:
  - `GET` for a list or a single item
  - `POST` to create
  - `PATCH` for a partial update
  - `PUT` for a full replace
  - `DELETE` to remove
- **Files and folders**: kebab-case.
- **Handlers**: camelCase, verb first, named for the action: `listUsers`, `getUser`, `createUser`, `updateUser`, `deleteUser`.
- **Schemas**: PascalCase with a suffix, such as `CreateUserBody`, `UserParams`, `ListUsersQuery`.
- **JSON fields**: camelCase in both requests and responses.

## Error handling pattern

- Validate every input (params, query, body) against its schema before any other work. A validation failure returns `400` with `VALIDATION_ERROR`.
- Throw typed errors from services instead of returning error values: `NotFoundError`, `ValidationError`, `ConflictError`, `UnauthorizedError`, `ForbiddenError`. Each carries a status code and a stable machine-readable `code`.
- Handlers do not catch errors just to format them. Wrap them in the shared async handler so errors reach the central error middleware.
- The central error middleware is the only place that turns errors into HTTP responses.
- Unknown errors become `500` with `INTERNAL_ERROR`. Log the full error and stack on the server. Never send stack traces, SQL, or internal messages to the client.
- Use status codes precisely:

  | Status | When |
  |---|---|
  | `200` | Read or update succeeded |
  | `201` | Created (include a `Location` header) |
  | `204` | Delete succeeded, no body |
  | `400` | Invalid input |
  | `401` | Not authenticated |
  | `403` | Authenticated but not allowed |
  | `404` | Not found |
  | `409` | Conflict |
  | `422` | Request is well formed but a business rule failed |
  | `500` | Unexpected server error |

## Response shape

Every JSON response uses the same envelope.

Success:

```json
{
  "data": { "id": "123", "name": "Ada" },
  "meta": {}
}
```

Paginated list:

```json
{
  "data": [ { "id": "123" } ],
  "meta": { "page": 1, "pageSize": 20, "total": 57 }
}
```

Error:

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User 123 was not found.",
    "details": []
  }
}
```

- A response has either `data` or `error`, never both.
- `error.code` is UPPER_SNAKE_CASE and stays the same over time. Clients depend on it.
- `error.message` is readable by humans and safe to show to users.
- `error.details` holds per-field validation problems: `{ "field": "email", "issue": "must be a valid email" }`.
- Timestamps are ISO 8601 strings in UTC. IDs are strings.

## Before committing

Every new or changed endpoint needs all of the following:

- [ ] A route registered in `src/routes/index.ts` under the versioned prefix.
- [ ] A validation schema for every params, query, and body input.
- [ ] Authentication and authorization applied, or a comment explaining why the endpoint is public.
- [ ] Errors thrown as typed errors and handled only by the central middleware.
- [ ] Responses that use the standard envelope and the correct status codes.
- [ ] Tests that cover:
  - the success path
  - a validation failure (`400`)
  - not found (`404`), where it applies
  - unauthorized or forbidden, where it applies
- [ ] Passing tests, lint, and type checks.
- [ ] Updated API docs (OpenAPI spec or README) with the method, path, request, and response examples.
- [ ] No secrets, debug logging, or commented-out code left in the diff.
