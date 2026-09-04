# Verifying a Lingjing Persona credential

Read this before accepting any credential that names
`did:web:schema.lingjing-persona.org` as its issuer.

## Current status: the trust anchor is published

`did:web:schema.lingjing-persona.org` resolves, per the did:web method, to

    https://schema.lingjing-persona.org/.well-known/did.json

**That document is now published** (2026-09-04). It carries one verification
method, `#kms-persona-issuer-vc-signing-1`, whose `publicKeyJwk` is a P-256
public key. Verify `proof.jws` against it with ES256 — the signature is a raw
DER ECDSA signature over `JSON.stringify(credential)` with `proof` removed,
base64url-encoded without padding.

The private half lives in Google Cloud KMS at HSM protection level and cannot be
exported by anyone, including this issuer.

Before 2026-09-04 that document returned 404 and nothing from this issuer could
be verified by anyone. If you hold a credential from that period, it is unsigned
or carries the sentinel described below; treat it accordingly.

### This does not mean every credential from this issuer is trustworthy

Two separate things: publishing the JSON-LD context makes a credential's *shape*
interoperable, and publishing the DID document makes a *signature* checkable.
Neither makes an unsigned payload true.

> **A credential without a verifying `proof` is unauthenticated, self-reported
> data.** It may be perfectly honest data; it is not an attestation. Reject it
> as evidence.

The value of the published anchor is that you can now tell the two apart.

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

## Where this stands

1. **Publish `/.well-known/did.json` with the issuer's public key material.**
   Done, 2026-09-04. HSM-held P-256 key; the document is derived from the key
   rather than written by hand, so it cannot silently disagree with it.
2. **Issue with a real signer.** Key custody is on the issuing side, in KMS.
   Whether any given deployment has signing switched on is visible in the
   credential itself: a `proof` block that verifies, or no `proof` at all.
   This issuer does not fabricate one when it cannot sign.
3. **Account for credentials handed out earlier with the sentinel `jws`.**
   Not done. Anything issued before 2026-09-04 is unverifiable by construction —
   the anchor did not exist. If you relied on such a credential, ask for a
   re-issued one and check that it verifies.

So the honest answer to "can I trust this credential" is now: **check the
signature.** That is what changed. If there is no signature to check, the answer
is still no.
