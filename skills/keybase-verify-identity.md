---
name: Verify a Keybase identity
description: >-
  Resolve a Keybase user and cryptographically verify who they are — look them
  up by username or by a linked social account, fetch their PGP public key, and
  confirm their proofs against the Merkle transparency log.
api: keybase-api (https://keybase.io/docs/api/1.0)
operations:
  - user/lookup
  - key/fetch
  - merkle/root
---

# Verify a Keybase identity

Use the public Keybase API 1.0 (base URL `https://keybase.io/_/api/1.0`, JSON,
CORS-enabled, no auth for read endpoints). Every response carries a
`status: { code, name }` envelope — `code: 0` / `"OK"` means success.

## Steps

1. **Look up the user.** `GET /user/lookup.json?usernames=<username>` (or by
   proof: `?github=<handle>`, `?twitter=<handle>`, `?domain=<domain>`). Read the
   `them[]` array. A `null` entry means the user was not found — this is NOT an
   error (`status.code` is still `0`). Use `fields=basics,profile,public_keys,proofs_summary`
   to limit the payload.

2. **Read the proofs.** In `proofs_summary`, confirm the `state` of each proof
   (twitter, github, reddit, hackernews, dns, generic_web_site). Only treat a
   proof as valid when its state indicates it is active/verified.

3. **Fetch the key.** `GET /<username>/pgp_keys.asc` (or `key/fetch`) to retrieve
   the ASCII-armored PGP public key. Compare the fingerprint to any
   out-of-band-known fingerprint before trusting it.

4. **Anchor to the transparency log.** `GET /merkle/root.json` returns the signed
   Merkle root; use it (with `merkle/block`) to confirm the user's sigchain is
   published in Keybase's transparency log rather than trusting the lookup alone.

## Rules

- Read endpoints need no credentials; do not send secrets. Authenticated/mutating
  flows require the salt/login session-token handshake — see
  `authentication/keybase-authentication.yml`.
- Always branch on `status.code`, not HTTP status alone.
- Not-found (`null` in `them[]`) is a normal result — handle it, don't retry.
- The API is alpha and may change without version bumps; don't hard-code
  response shapes you can avoid.
