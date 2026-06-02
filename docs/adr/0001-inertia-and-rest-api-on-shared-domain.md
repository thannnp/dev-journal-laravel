# Serve the same domain through both Inertia and a REST API

The web UI is served with Inertia + React (controllers return `Inertia::render` props), while a separate REST API under `routes/api.php` exposes the same domain as JSON. Both interfaces share the same Eloquent models, Policies, Form Requests, and Action classes — controllers are thin protocol adapters that authorize, validate, call an Action, and return (Inertia response vs JSON resource). We accept the cost of maintaining two entry points because the explicit goal of this project is to *learn* Laravel, and seeing one domain served two ways is the clearest way to internalise where business logic belongs (in Actions, not controllers).

## Considered Options

- **Inertia only** — simplest; Laravel's recommended default. Rejected because it would never exercise API auth (Sanctum tokens), API Resources, or the discipline of keeping logic out of controllers.
- **REST API only (headless) + separate SPA** — common in industry, but loses the full-stack monolith experience that makes Laravel distinctive, and doubles the frontend setup.
- **Both, sharing the domain (chosen)** — more surface area than a real app this size needs, but each piece earns its place as a deliberate learning exercise rather than incidental complexity.

## Consequences

- The REST API is intentionally scoped to **Post** only for now (read = public, published-only; write = Sanctum-authenticated, same Policy as Inertia). Comments and Tags are not exposed via API yet.
- Any new domain behaviour must go in an Action (or model/Policy), never directly in a controller, or the two interfaces will drift.
