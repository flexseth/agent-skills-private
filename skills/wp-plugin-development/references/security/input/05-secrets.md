# Input: Secrets Are Not Sanitized

**Prohibited**

- Running passwords, signatures, tokens, or secrets through sanitizers that change them (`sanitize_text_field()` strips characters).
- Echoing or logging them.

**Required**

- Unslash them.
- Use them only for their purpose: hash them, compare with `hash_equals()`, or send them in a header.

**Exceptions:** None.

## Code

Shows: leaving a secret unsanitized, comparing it with `hash_equals()`, and keeping it out of logs. All of this file's content below is NEW.

```php
// NEW.

// Unsafe: the sanitizer can alter the token, the loose comparison leaks timing, and the token is logged.
$token = sanitize_text_field( wp_unslash( $_SERVER['HTTP_X_MYPLUGIN_TOKEN'] ?? '' ) );
error_log( 'Token received: ' . $token );
if ( get_option( 'myplugin_token_hash' ) == hash( 'sha256', $token ) ) {
	myplugin_run_import();
}

// Safe: unslash only, compare in constant time, fail closed, never log the value.
$token  = wp_unslash( $_SERVER['HTTP_X_MYPLUGIN_TOKEN'] ?? '' );
$stored = (string) get_option( 'myplugin_token_hash', '' );
if ( ! is_string( $token ) || '' === $stored || ! hash_equals( $stored, hash( 'sha256', $token ) ) ) {
	wp_send_json_error( null, 403 );
}
myplugin_run_import();
```

## Attack methodology

Sensitive data exposure and timing attack (NEW)

Security bulletins:
- CWE-208, Observable Timing Discrepancy
- CWE-532, Insertion of Sensitive Information into Log File
- OWASP A07:2021, Identification and Authentication Failures

- **Secret in a log or compared with `==` → leaked or guessed credential → authenticated access (NEW).** Anyone who can read the log gets the token, and timing differences in a non-constant-time comparison let an attacker recover a value one position at a time. A sanitizer that strips characters can also make a valid token fail, or make different tokens compare equal after stripping. Fix: unslash only, compare with `hash_equals()`, and never echo or log the value.
