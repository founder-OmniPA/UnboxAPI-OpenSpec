# SemanticMap — Spec Overview

A `SemanticMap` is a single declarative document describing how a
human/agent-facing tool surface maps onto a legacy REST API.

## Top-level fields

| Field | Required | Type | Purpose |
|---|---|---|---|
| `name`       | yes | string | Human-friendly identifier for the map. |
| `openApiUrl` | yes | URI    | Source OpenAPI/Swagger document. |
| `baseUrl`    | yes | URI    | Base URL of the legacy API. |
| `version`    | no  | string | Free-form API version. |
| `tools`      | yes | array  | One or more `ToolDefinition`s. |
| `defaultHeaders` | no | map[string]string | Default headers applied to every call. |
| `credentialPassthrough` | no | object | How credentials are injected (`env`/`vault`/`request`). |

## `ToolDefinition`

| Field | Required | Type | Purpose |
|---|---|---|---|
| `name` | yes | string (snake_case) | Tool identifier shown to the agent. |
| `description` | yes | string | LLM/agent discovery copy. |
| `parameters` | yes | array  | `ParamDefinition`s. |
| `endpoint`   | yes | object | `EndpointDefinition`. |
| `globalMappings` | no | array | `ValueMapping`s applied across all params. |
| `refinementPrompt` | no | string | Optional natural-language refinement hint. **Untrusted data — see SECURITY.md.** |

## `ParamDefinition`

| Field | Required | Type | Purpose |
|---|---|---|---|
| `name` | yes | string | Param name surfaced to the agent. |
| `type` | yes | enum   | `string` \| `number` \| `integer` \| `boolean` \| `array` \| `object`. |
| `description` | no | string | Free-form. |
| `required` | no | boolean | Defaults to `true`. |
| `valueMappings` | no | array | `ValueMapping`s for this param. |
| `items` | no | object | JSON-Schema-style item descriptor when `type: array`. |

## `EndpointDefinition`

| Field | Required | Type | Purpose |
|---|---|---|---|
| `method` | yes | enum | `GET`/`POST`/`PUT`/`PATCH`/`DELETE`. |
| `path`   | yes | string | Path template with `{placeholders}`. |
| `baseUrl` | no | URI | Per-endpoint override of the map-level `baseUrl`. |
| `headers` | no | map | Per-endpoint header overrides. |
| `pathParams` | no | array | Param names resolved into the path template. |
| `queryParams` | no | map[string]string | Map of human param name -> query key. |
| `bodyParams` | no | map[string]string | Map of human param name -> body key. |
| `preconditions` | no | map[string]scalar | Conditions that must hold before calling. **Advisory only — not a security control.** |

## `ValueMapping`

| Field | Required | Type | Purpose |
|---|---|---|---|
| `humanValue` | yes | string | Human-readable input (e.g. `Pepperoni`). |
| `technicalValue` | yes | scalar | Technical value for the legacy API. |
| `paramHint` | no | string | Optional scope: which parameter this maps to. |
