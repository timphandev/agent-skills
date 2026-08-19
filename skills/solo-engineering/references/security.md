# Security

These are floors, not a security review. They are the ones that get skipped under time pressure and are expensive to add back once the shape is wrong.

## Input from a client

**Every field bound from client input carries an explicit length or size bound.** No unbounded strings, no unbounded arrays. Pick the ceiling from something real — the longest value your system actually stores, or the limit the downstream imposes — not from a round number, and not from an assumption about the world. Names, addresses and identifiers run longer in more places than most limits assume, so check before you choose.

An array parameter that drives a database write needs a bound most of all: it decides how much work one request can cost you.

**Validate at the boundary, not in the core.** Request shape and field constraints belong at the transport edge, expressed once, so the layers behind it can assume well-formed input. Business rules — "this state transition is not allowed", "this name already exists" — belong in the domain, where they can be tested without constructing a request.

**Treat a client-supplied path or URL as hostile.** A filename becomes a path traversal the moment it is joined to a directory: resolve it and confirm the result is still inside the intended root. A URL the server will fetch becomes a request forgery aimed at your internal network: allowlist the destination rather than blocklisting the obvious internal ranges, since redirects and DNS get you back inside.

## Uploads

An upload is client input that gets stored, and often served back. Every rule above applies, plus these.

**Never trust the declared type.** The `Content-Type` header and the file extension are both supplied by whoever is uploading. Determine the type from the content itself — the magic bytes — and reject anything whose real type is not on an allowlist. Allowlist, not blocklist: a blocklist is a list of the formats you thought of.

**Bound the size where the bytes arrive, not after.** A limit checked in the handler is a limit checked once the whole file is already in memory or on disk, which is the resource exhaustion it was meant to prevent. The cap belongs at the server or proxy that reads the body, and the count of files in one request needs a cap too.

**Never store an upload under a client-supplied name, and never inside the web root.** Generate the stored name yourself and keep the original only as metadata to display — that removes path traversal and extension-based execution in one move. Serving from a path the server will execute is how an uploaded file becomes code.

**Serve uploads from a separate origin, or with headers that neutralize them** — `Content-Disposition: attachment`, a fixed `Content-Type` rather than a guessed one, and `X-Content-Type-Options: nosniff`. An SVG or an HTML file served inline from your own origin runs script with your session's access.

## Authorization

**Every route that mutates or reads restricted data carries an explicit authorization check.** Not "the UI does not show the button" — the UI is a suggestion.

**A permission is not ownership.** A route-level check establishes that the caller may read invoices; it says nothing about whether they may read *this* invoice. Every handler taking a resource identifier verifies the caller's scope over that specific record — tenant, owner, team — in the same query that loads it, rather than in a second one somebody later forgets. This is the most common real authorization bug in ordinary CRUD code, and it passes every route-level check as written.

**Check a permission, never a role name.** Role names are usually admin-editable display text, so a handler branching on one breaks the moment somebody renames it, and it silently grants rather than denies. A permission key is a fixed identifier the code owns.

**A permission that no code checks is dead.** Wherever a catalog of permissions exists separately from the checks enforcing them, they drift, and the drift is invisible: the UI offers a permission that grants nothing. Adding a key and adding its check belong in the same change.

**Resolve permissions per request rather than caching them in the session.** A revoked permission that stays live until the user logs out is a revocation that did not happen.

**Authorize a batch in one query.** A handler checking many resources resolves their permissions with a single batched lookup — the per-item version is both an N+1 and, more often, the place someone forgets the check on one branch.

## Credentials and secrets

**Hash passwords through one function in one place, never inline.** Every implementation has a trap and the traps are not portable: bcrypt ignores input past 72 bytes, and implementations disagree on whether that is an error or a silent truncation — some disagree between their own hash and compare paths, so a hash stored by a lenient version is satisfied at login by any input sharing its first 72 bytes. Verify what your specific library does rather than assuming, bound the length at the boundary regardless, and never truncate quietly. Concentrating this in one function is what makes the trap fixable once.

**A stored credential that must be recoverable is encrypted at rest, masked on read, and revealed only through its own permissioned, audited endpoint.**

**Masking creates a trap on the way back in.** A client that reads an object and saves it whole sends the mask back, and a naive write stores `***` over the real secret — data loss that surfaces later as a failed connection nobody connects to a save. The fix lives on the write path, not the read path: treat the mask sentinel, or an omitted field, as "unchanged", or accept the secret only through a dedicated rotate endpoint that never returns it.

**Never let a credential reach a log, an error message, an audit record, or a response body.** Errors carrying request context are the common leak — check what the context holds before attaching it.

**A password change revokes that user's other sessions.** Otherwise an attacker already holding one keeps it for the full session lifetime after the password meant to lock them out changes. Re-issue the session the user is currently on rather than dropping it, or the change logs them out of the flow they are in. The same trigger applies to an email change, an MFA enrollment change, and a role downgrade.

## Transport

**Cookie-based sessions need CSRF protection beyond `SameSite`.** A same-site cookie is a mitigation, not a control. Mount the CSRF check behind authentication and have it read the session from what authentication resolved rather than from the cookie again — reading the cookie twice means the two can disagree.

**Session cookies are `HttpOnly`, `Secure`, `SameSite` and explicitly scoped** — all four, every time. `HttpOnly` keeps a script from reading the session, `Secure` keeps it off plaintext transport, and `SameSite` is still worth setting even though the previous rule says it is not sufficient on its own: insufficient is not the same as useless, and leaving it unset gets you the browser's default rather than a decision. A token the frontend is *required* to read — a CSRF token in the double-submit pattern — is the deliberate exception to `HttpOnly`, and it is a different cookie from the session.

**Compare secrets and MACs with a constant-time comparison, never `==`.** A timing side channel on a token comparison is hard to exploit across a network and trivial to avoid, and the cost of avoiding it is one function call. Compare fixed-width digests: a constant-time compare still reveals length.

**Rate-limit every unauthenticated endpoint that checks a credential**, per source address. A slow hash raises the cost per attempt but does not cap attempts per second.

**Set security response headers globally, not per route.** Per-route means the one route somebody adds later is missing them — and it will be the error page or the static fallback, which is exactly where it matters.

## Error responses

**Client-facing errors stay generic; the detail is logged server-side.** Stack traces, database errors and internal error wrappers must not cross the boundary. Map an internal error to a status and a stable client-facing code, and log the rest. This is not only disclosure — a client that starts parsing your internal error text has coupled to it.

**Generic is not the same as useless.** "Generic" constrains what the message may reveal, not whether it helps: an error still has to tell the person who hit it what to do next, and a wall of `An error occurred` sends them to you instead. Withhold the internals and keep the actionable part — which field was rejected and what would be accepted, that the operation was not performed, an identifier they can quote when asking. [`prose-in-the-repo.md`](prose-in-the-repo.md) covers how to write the message itself; this rule only bounds what may go in it.

**Authentication failures are the deliberate exception.** "No such account" and "wrong password" together are an account enumeration oracle, and the same applies to password reset and signup. One message for both, and the same response timing, even though that is worse for the honest user who mistyped their email. That trade is intentional; do not let a later "helpful error messages" pass undo it.
