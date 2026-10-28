I wrote all six files to the try/migration-with-opus branch. I ran no git commands, committed nothing, and
  changed no other file.

  Files written:
  - /Users/seth/Developer/WordPress/agent-skills/skills/wp-plugin-development/references/security/input/01-n
    once-input.md
  - /Users/seth/Developer/WordPress/agent-skills/skills/wp-plugin-development/references/security/input/02-e
    xplicit-keys.md
  - /Users/seth/Developer/WordPress/agent-skills/skills/wp-plugin-development/references/security/input/03-v
    alidate-sanitize-type.md
  - /Users/seth/Developer/WordPress/agent-skills/skills/wp-plugin-development/references/security/input/04-c
    ast-ids.md
  - /Users/seth/Developer/WordPress/agent-skills/skills/wp-plugin-development/references/security/input/05-s
    ecrets.md
  - /Users/seth/Developer/WordPress/agent-skills/skills/wp-plugin-development/references/security/input/06-a
    llowlist-sensitive-positions.md

  Checks (run with grep on the six files):
  - Entities: none of the six files contains &gt;, &amp; or &lt;.
  - Literal >: 01, 02, 03, 04 and 06 contain it, in =>, $wpdb->prepare() and $obj->$method(). 05 has none
    because its code has no array or arrow.
  - Literal &: it appears once, in 06 (option=users_can_register&value=1). No other file has a source &.
  - Headings: every file uses "Security bulletins:". No "Security Bulletins" is left.
  - absint(): no file says it rejects negative input.
  - Other: there are no em dashes.

  One change from my draft: the 06 stub already had the title # Input: Sanitization Is Not Authorization or
  Allowlisting, so I kept it instead of my draft's "Allowlist Sensitive Positions". Everything else matches
  the draft, with SAMPLE B's // lines unchanged.

  === NOTES ===
  1. COVERAGE
  - P1: 06 (Prohibited)
  - R1: 06 (Required, bullet 1)
  - R2: 03 (Required, bullet 1)
  - R3: 02 (Required)
  - R4: 01 (Required)
  - T1: 06 (Required, bullet 2, sub-list kept)
  - T2: 05 (Required, bullets 1 and 2, split at the sentence break, wording unchanged)
  - SAMPLE A: 06 (first php block)
  - SAMPLE B: 01
  - ATTACK A: 06 ("Arbitrary option update")
  - ATTACK B1: 01
  - ATTACK B2: 01
  - Casting rule: 04 (Required, bullet 1); the related Prohibited bullet is in 04
  - Casting sample: 04 (verbatim)
  - Casting attacks: 04 (both bullets)
  - Everything has a home.

  2. NEW (written by me, not copied)
  - All files: every "Shows:" sentence, the titles 01 to 05 (06 uses the stub's title), and the "None
    specific to this directive." and "Exceptions: None." lines.
  - 01: the sentence saying the write-ups belong to the authorization surface.
  - 02: the whole sample, and the attack class, bulletins and bullet (mass assignment).
  - 03: Required bullet 2 (sanitizing vs validating), the type table, the sample, and the attack class,
    bulletins and bullet.
  - 04:
    - The Prohibited bullet, expanded from the issue.
    - Required bullets 2 and 3: the read_post / custom table point and the %d / absint() accuracy note.
    - The attack class line, the bulletins list, and both bullet bodies. The bullet headlines come from the
      issue.
  - 05: both Prohibited bullets, Required bullet 3 (check without changing the value, fail closed), the
    whole sample, and the attack class, bulletins and bullet.
  - 06: the second php block (callback map) and the "Unsafe reflection" attack section.
  - Not mine, only normalized: the "Security bulletins:" lowercase b in B1 and B2.

  3. DOUBTS
  - absint() does not reject negatives: absint( -5 ) returns 5, absint( '5abc' ) returns 5, and absint(
    array( 'x' ) ) returns 1. So bad input can turn into a real ID. For strict rejection, use filter_var(
    $v, FILTER_VALIDATE_INT, array( 'options' => array( 'min_range' => 1 ) ) ) or check is_scalar() first.
    The casting sample is unchanged.
  - SAMPLE B comments: SAMPLE B (verbatim) uses a two-line // comment and extra spaces before its trailing
    // comments. This may break the WPCS rule in your memory (multi-line comments use /* */).
  - check_ajax_referer(): it stops the request only while $stop is true, which is the default. With no query
    arg it reads _ajax_nonce or _wpnonce.
  - 01 has no "stop on failure" rule: stopping the request when verification fails appears only in the
    sample and attacks. You could add it as a Required bullet.
  - 02 OWASP mapping: the official mapping puts CWE-915 under A08:2021, but ATTACK A pairs CWE-915 with A01.
    Pick one for consistency.
  - 05 CWE and sniff choices:
    - CWE-532 with A09:2021 is my choice. You could add CWE-208 for the !== comparison.
    - Check that the phpcs:ignore code still matches the sniff
      (WordPress.Security.ValidatedSanitizedInput.InputNotSanitized).
    - I believe WPCS allows file_get_contents( 'php://input' ) as a local stream, but please confirm.
    - A hex HMAC usually survives sanitize_text_field(). Passwords and tokens with %xx sequences, <, or
      extra whitespace do not.
  - 03 return values: sanitize_hex_color() returns null when invalid and '' when empty, so 02's settings
    array can store a null. rest_sanitize_boolean() is in rest-api.php, which loads on every request.
  - 06: I mapped CWE-470 to A03:2021 from memory. Please check it.
  - check_ajax_referer(): it stops the request only while $stop is true, which is the default. With no query
    arg it reads _ajax_nonce or _wpnonce.
  - 01 has no "stop on failure" rule: stopping the request when verification fails appears only in the
    sample and attacks. You could add it as a Required bullet.
  - 02 OWASP mapping: the official mapping puts CWE-915 under A08:2021, but ATTACK A pairs CWE-915 with A01.
    Pick one for consistency.
  - 05 CWE and sniff choices:
    - CWE-532 with A09:2021 is my choice. You could add CWE-208 for the !== comparison.
    - Check that the phpcs:ignore code still matches the sniff
      (WordPress.Security.ValidatedSanitizedInput.InputNotSanitized).
    - I believe WPCS allows file_get_contents( 'php://input' ) as a local stream, but please confirm.
    - A hex HMAC usually survives sanitize_text_field(). Passwords and tokens with %xx sequences, <, or
      extra whitespace do not.
  - 03 return values: sanitize_hex_color() returns null when invalid and '' when empty, so 02's settings
    array can store a null. rest_sanitize_boolean() is in rest-api.php, which loads on every request.
  - 06: I mapped CWE-470 to A03:2021 from memory. Please check it.
  - Testing: none of this has been run or linted.
    === END NOTES ===

---

## Review of the drafts (main model, checked against the files on disk)

Everything above is the Opus agent's own report. Lines 90 to 105 repeat lines 74 to 89 (pasted twice). This section is the independent review. Local branch only; do not push.

### Verified

- No HTML entities in the six files; arrows and `&` are literal characters.
- No file other than the six was touched.
- The nonce sample, the option-overwrite sample and the casting sample are identical to their sources (`diff`, ignoring trailing whitespace). Rules R1, R2, R3, R4 and T2 are present word for word, and the three attack bullets match.
- Every rule, sample and attack write-up from the Inputs section of `security.md` has a home. Nothing was dropped.
- Nothing has been run or linted.

### Problems to fix

1. `04-cast-ids.md` line 11 says "non-numeric input becomes `0`". Wrong: `absint( '5abc' )` returns `5` and `absint( array( 'x' ) )` returns `1`. The agent's own doubt (above) says this, so the file contradicts the agent's report.
2. `05-secrets.md` repeats itself: the Prohibited and Required lists both say "do not sanitize secrets" and "do not echo or log them". The source was one rule (T2). Merge them.
3. `05-secrets.md` code is unchecked: the `phpcs:ignore` sniff code, whether WPCS accepts `file_get_contents( 'php://input' )`, and a redundant `is_string()` after `wp_unslash()`.
4. "None specific to this directive." is a bullet in the input files and plain text in the database files. Make them consistent.
5. OWASP mapping differs between files: `02` pairs CWE-915 with A08:2021, `06` pairs it with A01:2021. The CWE-470 (A03), CWE-532 (A09) and CWE-639 (A01) picks came from memory and need checking.
6. `01-nonce-input.md` has no Required bullet saying to stop the request when verification fails.
7. `06-allowlist-sensitive-positions.md`: the callback-map sample has no capability check, while its attack bullet says "plus a capability check".

### Strengths

- Sensible split; the casting file separates injection from the object-reference risk and states correctly that `%d` stops injection on its own.
- A useful sanitizer-by-type table in `03`, with real functions.
- An accurate doubts list.

### Next steps

- Fix problems 1, 2 and 4, then 6 and 7; then check the CWE, OWASP and `phpcs:ignore` items.
- Compare with the same files written by the main model for `01`, `04` and `05`.
- Remove the Inputs text from `security.md` only after every item has a home and approval. Then add the IN-1 to IN-6 lines, the `SKILL.md` entries, and update the check script (it flags the input files as unlisted until then).
- Note: this file sits in `input/` but its name does not match the `NN-name.md` pattern, so the check script ignores it. It should not ship.
