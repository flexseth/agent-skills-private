# Input: Secrets

**Prohibited**

- Running a password, signature, token, or secret through a sanitizer that changes it (`sanitize_text_field()` strips characters). (NEW bullet, from the T2 rule.)

**Required**

- Do not run passwords, signatures, tokens, or secrets through sanitizers that change them (`sanitize_text_field()` strips characters). Unslash them, use them only for their purpose (hash, compare with `hash_equals()`, send in a header), and never echo or log them.

**Exceptions:** None.

## Code

Shows: unslashing a secret without sanitizing it, comparing it with `hash_equals()`, and never logging it. (NEW sample.)

```php
// NEW. $expected holds the stored token.
$expected = (string) get_option( 'myplugin_api_token', '' );

// Unsafe: the sanitizer can change a valid token so it no longer matches, == is not constant-time, and the secret is logged.
$token = sanitize_text_field( wp_unslash( $_POST['token'] ?? '' ) );
error_log( 'Token received: ' . $token );
if ( $token == $expected ) {
	myplugin_run_import();
}

// Safe: unslashed only, compared in constant time, never logged or echoed.
// phpcs:ignore WordPress.Security.ValidatedSanitizedInput.InputNotSanitized -- Secret: a sanitizer would change it. Only compared with hash_equals().
$token = wp_unslash( $_POST['token'] ?? '' );
if ( ! is_string( $token ) || '' === $expected || ! hash_equals( $expected, $token ) ) {
	wp_send_json_error( null, 403 );
}
myplugin_run_import();
```

## Attack methodology

Sensitive information exposure (NEW)

Security bulletins:
- CWE-532, Insertion of Sensitive Information into Log File (unsure of fit)
- CWE-208, Observable Timing Discrepancy (unsure of fit)
- OWASP A02:2021, Cryptographic Failures

- **Secret logged or echoed → disclosure → account or API takeover.** (NEW) A token written to a log or printed in a response is readable by anyone with log or page access, who can then replay it. Fix: never echo or log secrets.
- **Non-constant-time compare → timing side channel → guessed token.** (NEW) `==` can leak how many leading characters matched. Fix: `hash_equals()`.
