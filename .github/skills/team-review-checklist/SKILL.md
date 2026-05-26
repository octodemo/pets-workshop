---
name: team-review-checklist
description: Team review rules for pull requests. Use when reviewing API-related changes to enforce endpoint test coverage and Microsoft REST API error envelope compliance.
license: MIT
---

# Team Review Checklist

Use this skill when reviewing pull requests that add or modify API endpoints.
It defines two focused checks so reviews stay consistent and low-noise.

## Rule 1 — New API endpoints must include tests

### Flag when

A pull request adds a new HTTP route, handler, or API endpoint (for example:
Flask route, Express handler, FastAPI path operation, ASP.NET controller
action, Express Router) **and** the same pull request does not add or modify a
corresponding test.

Recognized test locations include:

- `tests/`
- `test/`
- `__tests__/`
- files matching `*_test.*`, `*.test.*`, `*.spec.*`, or `test_*.*`

### Review comment to post

> This pull request adds a new API endpoint but does not include a
> corresponding test. Per our team review checklist, every new endpoint
> needs at least one test that exercises the happy path and the primary
> error path.

### Why

Every new endpoint expands the service's public surface area.
Tests prevent regressions when models, middleware, or adjacent endpoints change.

### Acceptable example

For a new `POST /api/pets/{id}/adopt` endpoint, add a companion test that covers:

- 200 path: adopt succeeds for an existing, non-adopted pet.
- 404 path: adopting a non-existent pet returns the documented error response.

## Rule 2 — Error responses must use the Microsoft REST API error envelope

### Flag when

A pull request adds or modifies an endpoint that returns an error response
(any non-2xx status code) **and** the error body does not follow this envelope:

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "target": "string",
    "details": [
      {
        "code": "string",
        "message": "string",
        "target": "string"
      }
    ]
  }
}
```

Minimum required fields are `error.code` and `error.message`.
`target` and `details` are optional.

### Non-compliant examples

```python
# ❌ Flat string error with no envelope.
return jsonify({"error": "Pet not found"}), 404
```

```python
# ❌ Bare string body.
return "Pet not found", 404
```

```python
# ❌ Message at top level, no error object.
return jsonify({"message": "Pet not found"}), 404
```

### Compliant example

```python
# ✅ Microsoft REST API error envelope.
return jsonify({
    "error": {
        "code": "PetNotFound",
        "message": "Pet not found.",
        "target": "id"
    }
}), 404
```

### Review comment to post

> This error response does not follow the Microsoft REST API error envelope.
> Error bodies on this service must include an `error` object containing
> at least `code` and `message`, and optionally `target` and `details`.
> Please update this response to match the envelope before merging.

### Why

Consistent error shapes let clients implement robust, reusable error handling
without endpoint-specific parsing logic.

## Comment quality rules

- If both rules fail, create separate comments so each issue can be resolved
  independently.
- Keep comments specific, actionable, and tied to changed code.
- Avoid duplicate comments for the same issue in the same file.

## Out of scope

Do **not** use this skill for general code style or formatting checks.
Do **not** use this skill for broad security/performance audits.
Keep this skill focused on API test coverage and error envelope compliance.
