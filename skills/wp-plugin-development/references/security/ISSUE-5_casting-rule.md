## Summary

The security guidance has no rule for casting a single numeric ID. `absint()` appears only in the list handling of `db-03-dynamic-lists.md` and `db-06-combined-query.md`. Neither `SKILL.md` nor `security.md` mentions casting, so a model following the skill is never told to cast a single ID or to reject `0`.

## The rule (from the acceptance criteria)

Cast numeric IDs to positive integers with `absint()` before use, and reject `0`. A cast makes the value well-formed; it does not authorize access to that object, so check the matching capability (for example `current_user_can( 'read_post', $post_id )`) before acting on it, and use `%d` in `$wpdb->prepare()` for SQL.

Rule type: **Required**. Related **Prohibited**: treating a cast as authorization.

```php
/*
 * Unsafe: the ID is sanitized as text, not cast to an integer, so a value such
 * as "1 UNION SELECT user_pass FROM wp_users" stays intact and is interpolated
 * into the query. Nothing checks that the current user may read the post, so
 * changing the number also exposes private and draft posts.
 */
$post_id = sanitize_text_field( wp_unslash( $_POST['post_id'] ?? '' ) );
$title   = $wpdb->get_var( "SELECT post_title FROM {$wpdb->posts} WHERE ID = $post_id" );

// Safe: positive integer, 0 rejected, object-level capability checked, value prepared.
$post_id = absint( wp_unslash( $_POST['post_id'] ?? '' ) );
if ( ! $post_id || ! current_user_can( 'read_post', $post_id ) ) {
	wp_send_json_error( null, 403 );
}
$title = $wpdb->get_var(
	$wpdb->prepare( "SELECT post_title FROM {$wpdb->posts} WHERE ID = %d", $post_id )
);
```

**Attack methodology**

- **Uncast ID → SQL injection → credential disclosure.** CWE-89, OWASP A03:2021.
- **Cast ID, no capability check → insecure direct object reference.** CWE-639, OWASP A01:2021. Casting does not stop this.

## What the current files show

| File | Casting today |
|---|---|
| `SKILL.md` | Not mentioned. The baseline says "Use `$wpdb->prepare()` for SQL" with no `%d` |
| `references/security.md` | Not mentioned |
| `references/security/db-02-prepare-values.md` | Samples use `%d` for `$item_id`, but the comment says "sanitized", never cast, and `0` is not rejected |
| `references/security/db-03-dynamic-lists.md` | `array_map( 'absint', … )` plus `array_filter()` (drops `0` as a side effect, not stated as a rule) |
| `references/security/db-06-combined-query.md` | Same as `db-03` |

## Notes on accuracy

- `%d` in `prepare()` already casts to an integer, so an uncast value cannot inject through a `%d` placeholder. The injection in the unsafe sample comes from interpolating the value. `absint()` adds rejection of negatives and `0`, and gives the capability check a valid ID.
- `read_post` applies to posts. For a plugin's custom table, the object-level check is a plugin capability or an owner comparison, and the rule should say so.
- Not yet verified: no sample has been run through PHPCS, and Plugin Check has no sniff for casting.

## Plan

One rule, defined once, referenced from each surface. Keep the rule text in a single file and have the others point to it by ID, as the DB directives do.

### 1. Rule home: Inputs (`references/security.md`)

- Add a **Required** bullet under *Inputs → Validate and Sanitize Input*: cast numeric IDs with `absint()`, reject `0`.
- Add a **Prohibited** bullet: treating a cast as authorization.
- Add the unsafe/safe sample above. Per the file-split work, decide whether it stays inline or moves to a directive file (see "Open decisions").

### 2. Database surface

- `db-02-prepare-values.md`: Required bullet "Use `%d` for numeric values and cast IDs with `absint()` first; reject `0`". Make the `$item_id` sample cast it so the comment and code agree.
- `db-02` attack write-up: split the injection (interpolated value) from the IDOR (no capability check), so it matches how `%d` behaves.
- `db-03` and `db-06`: state the "reject `0`" rule that `array_filter()` already implements, and add a one-line note that casting a list does not authorize access to each object.
- Run `scripts/check_security_refs.mjs` after the edits.

### 3. Authorization surface (`references/security.md`, Request Authorization and REST Endpoints)

- Cross-reference at the object-level capability bullet (`current_user_can( 'edit_post', $post_id )`): cast the ID first, reject `0`, then check the capability.
- Note that custom-table objects need a plugin capability or an owner check, because `read_post` is post-specific.

### 4. REST and AJAX handlers

- REST: show `args` with `sanitize_callback` set to `absint` and a `validate_callback` that rejects `0`, ahead of the `permission_callback` capability check.
- AJAX and `admin-post`: the cast comes after `wp_unslash()`, before the capability check.

### 5. Other places an ID enters the code

Confirm the rule is stated or referenced for each:
- Shortcode attributes (`[myplugin id="3"]`) and dynamic block attributes.
- Option and post meta values that store IDs (read back as strings).
- Attachment IDs from an upload or media picker (`wp_get_attachment_url()`), including the capability to read that attachment.
- IDs used as URL path segments in external requests (cast, then `rawurlencode()` for non-numeric parts).
- IDs in `WP_Query` arguments (`p`, `post__in`, `author`), where the cast and the capability both still apply.

### 6. Output

- Cast IDs are safe for output as numbers, but IDs printed into attributes or URLs still follow the existing late-escaping rules (`esc_attr()`, `esc_url()`). Add a cross-reference, not a new rule.

### 7. Checklist, TOC, and `SKILL.md`

- `security.md` Checklist: add the cast-and-authorize rule next to "Input is read by explicit key, unslashed, validated, and sanitized for its type".
- Table of contents: add the new bullet.
- `SKILL.md` baseline: one short bullet, "Cast numeric IDs with `absint()` and reject `0`; a cast is not authorization, check the capability."
- `plugin-security_skill-acceptance-criteria.md`: tick "Type Casting Integer IDs" once done; update the surface checklist.

### 8. Checks and evals

- Run the new samples through PHPCS with WPCS (the `phpcs-harness` notes in the working folder are a starting point).
- Add an `eval/scenarios/*.json` scenario: an AJAX handler that reads a post title by ID, expecting `absint()`, rejection of `0`, a capability check, and `%d`. Format: name, skills, query, expected_behavior, success_criteria.
- Confirm the one-hop rule still holds: any new file must be named in `SKILL.md`, and `scripts/check_security_refs.mjs` must pass.

## Open decisions

- Should the rule's sample live in `security.md` (Inputs) or in a new directive file referenced from `SKILL.md`?
- Should the capability part stay in the authorization surface only, with the DB files carrying a one-line pointer?
- Include `intval()` and `(int)` casts as accepted alternatives, or require `absint()`?

## Acceptance

- [ ] Rule text exists in one place and is referenced from the other surfaces.
- [ ] `SKILL.md` baseline mentions it.
- [ ] The DB samples and comments agree on how IDs are cast.
- [ ] Samples follow WPCS and pass PHPCS.
- [ ] `scripts/check_security_refs.mjs` passes.
- [ ] An eval scenario covers it.
