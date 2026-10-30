# Input: Validate and Sanitize for the Type

**Prohibited**

- None specific to this directive.

**Required**

- Validate and sanitize input before storing it, displaying it, or passing it to a function whose behavior you don't control (hooks, third-party APIs).

**Exceptions:** None.

## Code

Shows: choosing a sanitizer and a validator that fit the data type, and rejecting a value that fails validation. All of this file's content below is NEW.

Sanitizing makes a value well-formed. Validation decides whether it is acceptable. A sanitizer can return an empty or altered value without any error, so check the result.

| Data type | Sanitize | Validate |
|---|---|---|
| Single-line text | `sanitize_text_field()` | Length or format check for the field |
| Multi-line text | `sanitize_textarea_field()` | Length check |
| Key or slug | `sanitize_key()`, `sanitize_title()` | `in_array( $v, $allowed, true )` |
| Email | `sanitize_email()` | `is_email()` |
| URL | `esc_url_raw()` | `wp_http_validate_url()` for URLs the server will request |
| Integer | `absint()` (see `04-cast-ids.md`) | `filter_var( $v, FILTER_VALIDATE_INT )` |
| Boolean | `rest_sanitize_boolean()` | `filter_var( $v, FILTER_VALIDATE_BOOLEAN, FILTER_NULL_ON_FAILURE )` |
| Date | None | `wp_checkdate()` or `DateTime::createFromFormat()` |
| File name | `sanitize_file_name()` | `wp_check_filetype()` |
| Hex color | `sanitize_hex_color()` | Empty string means invalid |
| Limited HTML | `wp_kses_post()`, `wp_kses()` | None |
| Array of values | `array_map()` with one of the sanitizers above | Check each item |

```php
// Unsafe: sanitize_email() returns '' for a bad address and the empty value is stored without a check.
// The URL is stored as sent, so a javascript: URL reaches the page.
update_option( 'myplugin_email', sanitize_email( wp_unslash( $_POST['email'] ?? '' ) ) );
update_option( 'myplugin_site', wp_unslash( $_POST['site'] ?? '' ) );

// Safe: sanitize, then validate, and stop when the value is not acceptable.
$email = sanitize_email( wp_unslash( $_POST['email'] ?? '' ) );
$site  = esc_url_raw( wp_unslash( $_POST['site'] ?? '' ) );
if ( ! is_email( $email ) || '' === $site ) {
	wp_send_json_error( null, 400 );
}
update_option( 'myplugin_email', $email );
update_option( 'myplugin_site', $site );
```

## Attack methodology

Improper input validation (NEW)

Security bulletins:
- CWE-20, Improper Input Validation
- CWE-79, Cross-Site Scripting
- OWASP A03:2021, Injection

- **Unvalidated value → stored malicious value → script runs in a visitor's browser (NEW).** The attacker submits `javascript:...` as the site URL. It is stored unchanged and later printed as a link target. Fix: sanitize and validate for the type on input (`esc_url_raw()` drops disallowed protocols), *and* escape on output (`esc_url()`).
