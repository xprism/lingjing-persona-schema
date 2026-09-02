# Verifying a Lingjing Persona credential

Read this before accepting any credential that names
`did:web:schema.lingjing-persona.org` as its issuer.

## Current status: credentials from this issuer are NOT yet verifiable

`did:web:schema.lingjing-persona.org` resolves, per the did:web method, to

    https://schema.lingjing-persona.org/.well-known/did.json

**That document does not exist yet** (HTTP 404 as of 2026-09-02). Without it
there is no published key material, so no conforming verifier can check the
signature on any credential this issuer has produced.

Until that document is published:

> **Do not accept a Lingjing Persona credential as evidence of anything.**
> Treat the payload as unauthenticated, self-reported data.

Publishing the JSON-LD context (which you are reading) makes the *shape* of a
credential interoperable. It does not make the credential *trustworthy* — those
are two separate things, and only the first is done.

## Two shapes you must reject

### 1. `proof.jws` equal to the literal `mock-signature-payload`

An earlier version of the issuing code emitted, when no signing key was supplied:

```json
"proof": {
  "type": "JsonWebSignature2020",
  "jws": "mock-signature-payload"
}
```

That is **not a signature**. A credential carrying it is unsigned, whatever the
`proof.type` says. Some credentials in this shape may already be in circulation.

Reject any credential whose `proof.jws` is that literal, regardless of the rest
of the envelope.

### 2. A `proof` block you cannot actually verify

`proof.type: "JsonWebSignature2020"` is a claim about the envelope, not evidence.
If you cannot resolve the issuer's key and check the signature, the presence of a
`proof` block tells you nothing. A verifier that accepts a credential because it
*has* a proof — rather than because the proof *verified* — provides no security.

The reference implementation refuses to verify without a public key, by design:
`verifyVerifiableCredential(credential, publicKey)` takes the key as a **required**
argument. If your verifier makes it optional, that is a bypass.

## An unsigned credential is a legitimate thing

Current issuing code omits `proof` entirely when no signer is configured, rather
than fabricating one. A credential with **no** `proof` is honest: it says "this
is data, not an attestation." Handle it as such.

An unsigned credential is safe to reject. A fake-signed one is not, because it
invites acceptance.

## Context versions

| Version | Subject identifier | Status |
|---|---|---|
| `.../persona/v1.jsonld` | `did:key:{actorId}` | **Frozen.** Still accepted for already-issued credentials. |
| `.../persona/v2.jsonld` | `urn:lingjing:persona:subject:{tenant}:{actor}` | Current. |

v1 misuses the `did:key` method: that method's identifier **is** a multibase-encoded
public key, and v1 put an opaque actor id there instead. A `did:key` value from a v1
credential yields no key material — **do not attempt to resolve it as one**.

v2 uses a URN, which honestly denotes a name and claims no key ownership.

Versioning a breaking change must not invalidate already-issued data, so both
contexts stay resolvable and conforming verifiers should accept either. But the
v1 subject identifier is a name, not a key — in both versions.

## What has to happen before this page can say something different

1. Publish `/.well-known/did.json` with the issuer's public key material.
2. Issue with a real signer (key custody on the issuing side).
3. Account for credentials already handed out with the sentinel `jws`, and
   re-issue them if they were relied upon.

Until all three are done, the honest answer to "can I trust this credential" is
**no**, and this page will keep saying so.
