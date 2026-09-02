# Lingjing Persona — JSON-LD Schema Context

Published at **https://schema.lingjing-persona.org**

This repository serves the JSON-LD `@context` documents for Verifiable Persona
Claims (VPC), so that standard-compliant verifiers — schools, games, workplaces —
can resolve them by URL.

| Version | URL | Status |
|---|---|---|
| v1 | https://schema.lingjing-persona.org/schema/persona/v1.jsonld | **Frozen.** Existing v1 credentials verify against it. |
| v2 | https://schema.lingjing-persona.org/schema/persona/v2.jsonld | Current. Newly issued credentials point here. |

## What changed in v2

The vocabulary is unchanged — v2 is v1 with the IRI base rebased from
`.../persona/v1#` to `.../persona/v2#`. What changed is the **value semantics**
of the credential subject:

| | v1 | v2 |
|---|---|---|
| `credentialSubject.id` | `did:key:{actorId}` | `urn:lingjing:persona:subject:{tenant}:{actor}` |
| `credentialSubject.role` | defaulted to `"student"` | omitted unless the issuer states it |

`did:key` is a DID method whose identifier **is** a multibase-encoded public key;
v1 put an opaque actor id there, which is syntactically valid and semantically
false — a resolver can extract no key material from it. v2 uses a URN, which
honestly denotes a name and claims no key ownership.

**v1 is not deprecated for existing credentials.** Versioning a breaking change
must not invalidate already-issued data, so both contexts stay resolvable and
conforming verifiers accept either.

## The eight persona sections

`competence`, `health`, `psycho`, `behavior`, `social`, `wellbeing`, `support`,
`affiliation` — in that order, which is product-visible. Each carries its own
nested vocabulary (e.g. `competence` → `logicalReasoning`,
`proceduralToolMastery`, `skillTreeNodes`).

## This repository is generated

These files are materialized from the `lingjing-persona` package, which owns the
single source of truth (`contracts/dimensions.json`) and a byte-comparison gate.
**Do not hand-edit anything here** — regenerate from the package instead.

Published bytes are pinned by the package's test suite:

```
v1  4530 B  sha256 6a7bc372b3e6d86490d6f0db197052f0fd47278e0c7194b1db8ab979f68c49d2
```

## Licence

MIT — see `LICENSE`.
