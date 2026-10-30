# Input: Explicit Keys

**Prohibited**

- None specific to this directive.

**Required**

- Never process a whole `$_POST` / `$_GET` / `$_REQUEST` array; read explicit keys and apply `wp_unslash()` before sanitizing.

**Exceptions:** None.

## Code

Shows: reading explicit keys with `wp_unslash()` and a sanitizer matched to each type, instead of processing the whole array. (NEW sample.)

```php
// NEW. The nonce and capability checks have already passed.

// Unsafe: every key the request carries is sanitized and stored, including keys the form never had.
update_option( 'myplugin_settings', array_map( 'sanitize_text_field', $_POST ) );

// Safe: only the expected keys are read, each unslashed and sanitized for its type.
$settings = array(
	'title' => sanitize_text_field( wp_unslash( $_POST['title'] ?? '' ) ),
	'color' => sanitize_hex_color( wp_unslash( $_POST['color'] ?? '' ) ),
	'limit' => absint( wp_unslash( $_POST['limit'] ?? 0 ) ),
);
update_option( 'myplugin_settings', $settings );
```

## Attack methodology

Mass assignment (NEW)

Security bulletins:
- CWE-915, Improperly Controlled Modification of Dynamically-Determined Object Attributes
- OWASP A01:2021, Broken Access Control

- **Whole-array write → extra attacker-supplied keys → unintended settings changed.** (NEW) The handler stores or loops over everything in `$_POST`, so the attacker adds fields such as `role` or `enabled` that the form never offered. Fix: read only the keys the handler expects.
