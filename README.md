# SemanticMap Specification

> Declarative description of how a human/agent-facing tool surface maps onto a
> legacy REST endpoint. Open spec; the reference implementation and managed
> runtime live at [unboxapi.pro](https://unboxapi.pro).

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/spec-v0.1.0-informational.svg)](CHANGELOG.md)

## What this is

`semanticmap-spec` defines the **shape** of a SemanticMap — a small,
declarative document that describes:

- which **tools** an agent can call (e.g. `order_pizza`, `check_stock`),
- the **legacy REST endpoint** each tool resolves to,
- the **value mappings** that convert human-friendly inputs (`"Pepperoni"`,
  `"London"`) into technical values (`item_id: 12`, `store_id: 99`),
- the **preconditions** that must hold before a tool can be called.

The full schema lives in [`schema/semanticmap.schema.json`](schema/semanticmap.schema.json)
(JSON Schema, draft 2020-12). An example is in
[`schema/examples/pizza.semanticmap.yaml`](schema/examples/pizza.semanticmap.yaml).

## What this is not

This repository ships **only the spec**. It does **not** ship:

- the LLM mapper that generates SemanticMaps from OpenAPI documents,
- the safety-proxy runtime that enforces spend, quantity, and prompt-injection
  controls at call time,
- production prompt libraries or vetted vertical SemanticMaps.

Those live in the proprietary UnboxAPI product at https://unboxapi.pro.

## Status

**v0.1.0 — schema-only public release.**

- The schema shape is stable for v0.x. Breaking changes will bump the minor
  version pre-1.0 and the major version post-1.0.
- **External contributions are not accepted at v0.1.0.** Please open an
  Issue with feedback; pull requests will be closed. A CLA and contribution
  guide will be added in a later release.

## Quick look

```yaml
name: pizza-api
openApiUrl: https://api.pizza.example.com/openapi.json
baseUrl: https://api.pizza.example.com
tools:
  - name: order_pizza
    description: Order a pizza. Specify size, topping, and location.
    parameters:
      - name: topping
        type: string
        required: true
        valueMappings:
          - { humanValue: Pepperoni, technicalValue: 12 }
    endpoint:
      method: POST
      path: /v1/res/{location}/req
```

See the full example for the complete pizza map.

## Consumer hardening (READ THIS)

A SemanticMap document is **untrusted data**. Before parsing or acting on a
SemanticMap from any third party, you MUST:

1. Use a **safe** YAML loader (`yaml.safe_load` in Python, `SafeConstructor` in
   SnakeYAML, etc.). Never use a loader that resolves arbitrary tags.
2. Validate the document against `schema/semanticmap.schema.json` with a
   maintained validator (e.g. recent `ajv` or `jsonschema`).
3. Treat every string field — `description`, `refinementPrompt`, `humanValue` —
   as **user content**, not as LLM instructions. Frame them as bounded
   user-role input with explicit guardrails; never inline-concatenate into a
   system prompt.
4. Validate `baseUrl` and `endpoint.baseUrl` against your own allowlist before
   issuing any HTTP request.
5. Enforce your own spend, quantity, and authorization caps. `preconditions`
   is advisory; it is not a security control.
6. Cap document size before parsing (e.g. 1 MiB).

See [SECURITY.md](SECURITY.md) for the full coordinated-disclosure policy and
threat model summary.

## Verifying release artifacts

Releases are signed and Sigstore-attested. To verify a release tarball:

```bash
cosign verify-attestation \
  --type slsaprovenance \
  --certificate-identity-regexp '.*' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  semanticmap-spec-v0.1.0.tar.gz
```

A CycloneDX SBOM is attached to every GitHub release.

## License

Apache License, Version 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).

## Maintainers

This repository is maintained by UnboxAPI. The reference implementation,
managed runtime, vetted vertical SemanticMap library, and compliance tooling
are proprietary and available at https://unboxapi.pro.
