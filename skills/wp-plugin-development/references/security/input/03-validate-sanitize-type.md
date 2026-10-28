# Input: Validate and Sanitize for the Type

**Prohibited**

- None specific to this directive.

**Required**

- Validate and sanitize input before storing it, displaying it, or passing it to a function whose behavior you don't control (hooks, third-party APIs).
- (NEW) Sanitizing makes a value well-formed. Validation decides whether it is acceptable. Do both, and reject the request when validation fails.

**Exceptions:** None.

## Code

Shows: choosing a sanitizer and validator for the data type, and rejecting a value that fails validation. (NEW table and sample.)

NEW. Which function fits which type:

| Data type | Sanitize | Validate |
|---|---|---|
| Positive integer ID | `absint()` (see file 04) | Reject `0` |
| Integer, negatives allowed | `intval()` | Range check |
| Float | `floatval()` | Range check |
| Boolean | `rest_sanitize_boolean()` | None needed |
| Single-line text | `sanitize_text_field()` | Length check |
| Multi-line text | `sanitize_textarea_field()` | Length check |
| Email | `sanitize_email()` | `is_email()` |
| URL to store or redirect to | `esc_url_raw()` | `wp_http_validate_url()`, `wp_validate_redirect()` for redirects |
| Hex color | `sanitize_hex_color()` | Empty result means invalid |
| File name | `sanitize_file_name()` | Extension check with `wp_check_filetype()` |
| CSS class | `sanitize_html_class()` | None needed |
| Limited HTML | `wp_kses_post()` | None needed |
| One of a fixed set | `sanitize_key()` | `in_array( $value, $allowed, true )` |
| Date | `sanitize_text_field()` | `DateTime::createFromFormat()` and check its errors |
| Array | Sanitize each element with `array_map()` or `map_deep()` | Validate each element |

```php
// NEW. The nonce and capability checks have already passed.

// Unsafe: sanitized, but never validated, so "not an email" is stored and mailed.
$email = sanitize_text_field( wp_unslash( $_POST['email'] ?? '' ) );
update_option( 'myplugin_contact', $email );

// Safe: sanitize for the type, then validate, and reject on failure.
$email = sanitize_email( wp_unslash( $_POST['email'] ?? '' ) );
if ( ! is_email( $email ) ) {
	wp_send_json_error( null, 400 );
}
update_option( 'myplugin_contact', $email );
```

## Attack methodology

Improper input validation

Security bulletins:
- CWE-20, Improper Input Validation
- OWASP A03:2021, Injection

- **Unvalidated value → stored in a trusted field → downstream misuse.** (NEW) A value that is well-formed but wrong (a malformed email, a URL with an unexpected scheme, an out-of-range number) is stored or passed on, and a later consumer such as `wp_mail()`, a redirect, or a third-party API acts on it. Fix: sanitize for the type, then validate, and reject the request on failure.
