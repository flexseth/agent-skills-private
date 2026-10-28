# Input: Read Explicit Keys

**Prohibited**

- None specific to this directive.

**Required**

- Never process a whole `$_POST` / `$_GET` / `$_REQUEST` array; read explicit keys and apply `wp_unslash()` before sanitizing.

**Exceptions:** None.

## Code

Shows: reading explicit keys with `wp_unslash()` and a sanitizer matched to the type, instead of passing a whole request array on. NEW.

```php
// NEW. Nonce and capability checks are omitted here; see 01-nonce-input.md.

// Unsafe: the request chooses every field. post_status, post_author, post_type and ID are all attacker-controlled.
$post_id = wp_insert_post( wp_unslash( $_POST ) );

// Safe: the code names each key, unslashes it, and sanitizes it for its type. Everything else is fixed.
$post_id = wp_insert_post(
	array(
		'post_type'    => 'myplugin_item',
		'post_status'  => 'draft',
		'post_title'   => sanitize_text_field( wp_unslash( $_POST['title'] ?? '' ) ),
		'post_content' => wp_kses_post( wp_unslash( $_POST['content'] ?? '' ) ),
	)
);
```

## Attack methodology

Mass assignment (NEW)

Security bulletins:
- CWE-915, Improperly Controlled Modification of Dynamically-Determined Object Attributes
- OWASP A01:2021, Broken Access Control

- **Whole request array → mass assignment → published or reassigned content (NEW).** The attacker adds fields the form never showed, such as `post_status=publish`, `post_author=1` or `ID=<existing post>`, and the function accepts them. Fix: read only the named keys and set every other field in code.
