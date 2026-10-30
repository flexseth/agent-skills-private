# Plan: Improper Input Validation (Inputs and Sanitization)

Working draft, not part of the skill. Analysis of the Inputs rules in `references/security.md` (lines 112–237, as committed in `ea04663`) against their code samples, to separate each directive into its own Markdown file.

Attack class: **Improper Input Validation (CWE-20)**. Related classes the section already cites: CWE-915 (option overwrite), CWE-352 (CSRF), CWE-862 and CWE-285 (missing authorization).

## Status

Branch `try/migration-with-opus` (local only, not pushed). An Opus subagent drafted all six files, committed unchanged as `5f19130`. The full review is in `input/002_opus-migration-notes.md` (`0f47fd6`). `security.md` and `SKILL.md` are unchanged, so the original Inputs text is still in `security.md`, and the check script reports the input files as unlisted until they are linked.

| File | Status |
|---|---|
| `input/01-nonce-input.md` | Drafted. Needs fix 6 |
| `input/02-explicit-keys.md` | Drafted. Needs fix 5 |
| `input/03-validate-sanitize-type.md` | Drafted. Needs fix 4 |
| `input/04-cast-ids.md` | Drafted. **Needs fix 1 (factual error)**, then fixes 4 and 5 |
| `input/05-secrets.md` | Drafted. Needs fixes 2, 3, 4 and 5 |
| `input/06-allowlist-sensitive-positions.md` | Drafted. Needs fixes 4, 5 and 7 |

### Fixes to make

- [ ] **1. `04-cast-ids.md`, Required bullet 3.** Replace "(non-numeric input becomes `0`)". Proposed: "`absint()` returns a non-negative integer but does not reject bad input: `absint( '5abc' )` returns `5` and `absint( array( 'x' ) )` returns `1`. When malformed input must be rejected, check `is_scalar()` first, or validate with `filter_var( $v, FILTER_VALIDATE_INT, array( 'options' => array( 'min_range' => 1 ) ) )`." Also add this limit to issue #5.
- [ ] **2. `05-secrets.md`, Prohibited and Required repeat each other.** Keep the two prohibitions in Prohibited. Reduce Required to the positive steps only: unslash, use the secret only for its purpose, validate without changing it, fail closed when the stored secret is empty.
- [ ] **3. `05-secrets.md`, code unchecked.** Run the sample through PHPCS with WPCS (the `phpcs-harness` in this folder). Confirm the `phpcs:ignore` sniff code, whether `file_get_contents( 'php://input' )` is accepted, and drop the redundant `is_string()` after `wp_unslash()` if it is.
- [ ] **4. "None" lines.** Make "None specific to this directive." plain text, not a bullet, in all six input files, like the database files.
- [ ] **5. CWE and OWASP mapping.** Pick one OWASP category for CWE-915 and use it in `02` and `06`. Check CWE-470 (`06`), CWE-532 (`05`) and CWE-639 (`04`) against their CWE entries. These came from the agent's memory.
- [ ] **6. `01-nonce-input.md`.** Add a Required bullet: "Check the result of `wp_verify_nonce()` and stop the request when it fails. `check_admin_referer()` and `check_ajax_referer()` stop it for you."
- [ ] **7. `06-allowlist-sensitive-positions.md`.** Add a `current_user_can()` check to the safe callback-map sample, or change the attack bullet so it no longer says "plus a capability check".

### After the fixes

- [ ] Compare with the same files written by the main model (Sonnet) for `01`, `04` and `05`.
- [ ] Remove the Inputs text from `security.md` only after every rule, sample and write-up has a home, with your approval. Then add the IN-1 to IN-6 lines, the `SKILL.md` entries, and run the check script.
- [ ] Move the nonce attack write-ups to the authorization surface when that surface is split.
- [ ] Keep `002_opus-migration-notes.md` out of anything pushed.

## Rules, Verbatim

**Prohibited**

- P1. Never treat sanitization as authorization.

**Required**

- R1. Understand that `sanitize_key()` or `sanitize_text_field()` makes a value well-formed; it does not make it safe to *choose* which option, table, file, callback, or capability the code acts on. For those, use an allowlist.
- R2. Validate and sanitize input before storing it, displaying it, or passing it to a function whose behavior you don't control (hooks, third-party APIs).
- R3. Never process a whole `$_POST` / `$_GET` / `$_REQUEST` array; read explicit keys and apply `wp_unslash()` before sanitizing.
- R4. Sanitize a nonce before passing it to `wp_verify_nonce()`: `sanitize_text_field( wp_unslash( $_POST['_wpnonce'] ?? '' ) )`. `check_admin_referer()` and `check_ajax_referer()` read the request themselves.

**Unfinished (the `TODO - elaborate / organize` block, lines 230–237)**

- T1. Treat input as unsafe in sensitive positions even after sanitization. A fixed allowlist, strict comparison (`in_array( $v, $allowed, true )`), or hardcoded map must decide:
  - option, meta, transient, table, column, or capability names;
  - file paths;
  - hooks, routes, callbacks, classes, methods, or any dynamic call (`call_user_func()`, `$obj->$method()`, `new $class`, `do_action( $name )`);
  - a privileged target, scope, or execution path (which user, which site, which mode).
- T2. Do not run passwords, signatures, tokens, or secrets through sanitizers that change them (`sanitize_text_field()` strips characters). Unslash them, use them only for their purpose (hash, compare with `hash_equals()`, send in a header), and never echo or log them.

**Exceptions**

- None stated in `security.md`.

## Directives and Proposed Files

File naming standard: one folder per surface, files numbered in encounter order with no surface prefix. Input files are `references/security/input/NN-name.md`.

| # | Directive | Rules | Proposed file | Sample today (security.md lines) | Status |
|---|---|---|---|---|---|
Numbered in encounter order: the order an agent meets each rule while handling a request.

| # | Directive | Rules | File | Sample today (security.md lines) | Status |
|---|---|---|---|---|---|
| 1 | Nonce input handling | R4 | `input/01-nonce-input.md` | Nonce sample (174–196) | Covered. Its attack write-ups belong to other classes (see below) |
| 2 | Read explicit keys; unslash before sanitizing | R3 | `input/02-explicit-keys.md` | One line inside the option-overwrite sample | No dedicated unsafe/safe pair |
| 3 | Validate and sanitize for the type, before use | R2 | `input/03-validate-sanitize-type.md` | None | **Gap.** No guidance on choosing a sanitizer, and no validation example |
| 4 | Cast numeric IDs, reject `0` | issue #5 | `input/04-cast-ids.md` | None in security.md | New rule |
| 5 | Secrets are not sanitized | T2 | `input/05-secrets.md` | None | **Gap.** Unfinished rule |
| 6 | Sanitization is not authorization or allowlisting | P1, R1, T1 | `input/06-allowlist-sensitive-positions.md` | Option-overwrite pair (127–147) | Covered for option names. No sample for dynamic calls (`call_user_func()`, `new $class`) |

Attack methodology moves with its directive:

- Arbitrary option update (lines 149–168; CWE-915, CWE-862, OWASP A01:2021): goes with file 1.
- CSRF from an ignored nonce result (198–215; CWE-352, CWE-79): the class is CSRF, not input validation. It belongs with the authorization surface. File 4 would keep only a pointer.
- Missing authorization, valid nonce and no capability check (217–228; CWE-862, CWE-285): same, the authorization surface.

## Rule-to-Sample Matrix

| Rule | Sample shows it | Unsafe and safe halves | Notes |
|---|---|---|---|
| P1 | Yes (option overwrite) | Yes | |
| R1 | Yes (option overwrite) | Yes | The "safe" half uses a fixed map |
| R2 | No | No | No sanitizer-by-type table (`absint`, `sanitize_email`, `esc_url_raw`, `sanitize_textarea_field`, `wp_kses_post`, arrays) and no validation example |
| R3 | Partly | Partly | `wp_unslash( $_POST['x'] ?? '' )` is used, but no pair shows a whole-array read |
| R4 | Yes (174–196) | Yes | The unsafe half also leaves the result unchecked, which is a fail-closed point |
| T1 | Option names only | n/a | Paths, callbacks, classes and privileged targets have no sample |
| T2 | No | No | |

## Problems in the Section

- The `TODO - elaborate / organize` block ships two real rules (T1, T2).
- The Prohibited text is one sentence under the Inputs heading, and Required is prose, not a bullet list. The other sections use bullets.
- "Sanitizing Nonces" contains two attack write-ups for other classes.
- The Checklist line "Input is read by explicit key, unslashed, validated, and sanitized for its type" bundles three directives (R2, R3, and unslashing).
- The sample's multi-line comment block has trailing spaces on the opening line (`/* `). Check WPCS.
- Casting numeric IDs is not mentioned anywhere (issue #5).

## Carried Over

- [ ] Casting rule: issue #5 (`flexseth/agent-skills`).
- [ ] Verify that `esc_js()` and similar functions are described correctly in the Escaping section (separate plan).
- [ ] Run every sample through PHPCS with WPCS. The `phpcs-harness` in the working notes is a starting point.
- [ ] Update the surface checklist line the user marked `[???]`.

## Split Mechanics

- `security.md` keeps one line per directive (IN-1 to IN-6) with no links, and `SKILL.md` names each file directly so every file is one hop away.
- Same headings in every file: Prohibited, Required, Exceptions (`**Exceptions:** None.` where empty), Code, Attack methodology.
- `scripts/check_security_refs.mjs` scans only `references/security/` for `db-NN` files. It does not yet scan surface folders such as `references/security/input/`. Update it before relying on it (planned, not done).
- The Escaping section is a separate plan. Do not touch it here.

## Open Decisions

- File prefix: decided. No prefix. The `input/` folder names the surface, and files are numbered `01` to `06`.
- Keep nonce handling here (R4), or move it to the authorization surface with its two attack write-ups.
- Casting as its own file (decided: `input/04-cast-ids.md`).
- Whether `input/03-validate-sanitize-type.md` carries a sanitizer-by-type table, or points to the WordPress sanitizing documentation.
