# Input: Cast Numeric IDs

**Prohibited**

- Treating a cast as authorization. A cast makes the value well-formed; it does not authorize access to that object.

**Required**

- Cast numeric IDs to positive integers with `absint()` before use, and reject `0`.
- Check the matching capability before acting on the object, for example `current_user_can( 'read_post', $post_id )`. `read_post` is post-specific; for a plugin's custom table, the object-level check is a plugin capability or an owner comparison.
- Use `%d` in `$wpdb->prepare()` for SQL. `%d` already casts, so an uncast value cannot inject through a placeholder; the injection in the unsafe sample comes from interpolating the value into the query.
- `absint()` converts, it does not validate: `absint( '5abc' )` returns `5`. Casting is the integer case of `03-validate-sanitize-type.md`; follow that file for the general rule.

**Exceptions:** None.

## Code

Shows: casting an ID with `absint()`, rejecting `0`, checking the object-level capability, and using `%d` in `prepare()`.

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

- **Uncast ID → SQL injection → credential disclosure.** The ID is interpolated into the query, so the attacker sends SQL in place of a number. Fix: cast with `absint()`, and pass the value through `%d`.

Insecure direct object reference

Security bulletins:
- CWE-639, Authorization Bypass Through User-Controlled Key
- OWASP A01:2021, Broken Access Control

- **Cast ID, no capability check → insecure direct object reference.** The attacker changes the number and reads or changes objects that belong to others. Casting does not stop this. Fix: check the object-level capability after the cast.
