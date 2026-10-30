# Input: Cast Numeric IDs

**Prohibited**

- Treating a cast as authorization. `absint()` makes the value well-formed; it does not authorize access to that object.

**Required**

- Cast numeric IDs to positive integers with `absint()` before use, and reject `0`.
- Check the matching capability before acting on the object (for example `current_user_can( 'read_post', $post_id )`). `read_post` is post-specific; for a plugin's custom table, use a plugin capability or an owner comparison.
- Use `%d` in `$wpdb->prepare()` for SQL. `%d` already casts to an integer, so an uncast value cannot inject through a placeholder; the injection in the unsafe sample comes from interpolating the value.
- Casting is the integer case of file 03. Use the sanitizer and validator table there for other types.

**Exceptions:** None.

## Code

Shows: casting an ID with `absint()`, rejecting `0`, checking an object-level capability, and preparing the value with `%d`.

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

## Attack methodology

SQL injection

Security bulletins:
- CWE-89, Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)
- OWASP A03:2021, Injection

- **Uncast ID → SQL injection → credential disclosure.** The ID is interpolated into the query as text, so the attacker appends a `UNION SELECT` and reads other tables. Fix: cast with `absint()` and use `%d` in `prepare()`.

Insecure direct object reference

Security bulletins:
- CWE-639, Authorization Bypass Through User-Controlled Key
- OWASP A01:2021, Broken Access Control

- **Cast ID, no capability check → insecure direct object reference.** Casting does not stop this: the attacker changes the number and reads objects they do not own. Fix: check an object-level capability or owner after the cast.
