---
name: wp-coding-standards
description: >
  Enforces WordPress Coding Standards (WPCS) across PHP code. Covers WordPress-Core,
  WordPress-Extra, and WordPress-VIP rulesets. Provides definitive reference for data
  sanitization, output escaping, nonce verification, PHPDoc, hook patterns, database
  query safety, and auto-enforcement tooling setup.
tools:
  - Bash
  - Read
  - Edit
  - Write
  - Grep
  - Glob
---

# WordPress Coding Standards Enforcement

This skill is the definitive reference for writing PHP code that passes WordPress
Coding Standards (WPCS) sniffs. Every pattern includes WRONG vs RIGHT examples.

---

## 1. WordPress-Core Standard

### Yoda Conditions

Place the constant or literal on the LEFT side of comparisons. This prevents
accidental assignment (`=` instead of `===`) from silently succeeding.

```php
// WRONG — variable on left risks accidental assignment
if ( $type === 'post' ) { ... }
if ( $count == 0 ) { ... }

// RIGHT — literal on left (Yoda style)
if ( 'post' === $type ) { ... }
if ( 0 === $count ) { ... }
```

### Array Syntax

Always use `array()`. Short array syntax `[]` is not permitted by WordPress-Core.

```php
// WRONG
$items = [];
$config = ['key' => 'value'];

// RIGHT
$items = array();
$config = array( 'key' => 'value' );
```

### Spaces Inside Parentheses

Add a single space after opening and before closing parentheses in control
structures, function declarations, and function calls.

```php
// WRONG
if ($foo) {
    my_function($bar, $baz);
}

// RIGHT
if ( $foo ) {
    my_function( $bar, $baz );
}
```

### Naming Conventions

Functions, variables, and action/filter names use `snake_case`. Classes use
`Upper_Snake_Case`. Constants use `UPPER_SNAKE_CASE`. File names use lowercase
hyphens (`class-my-widget.php`).

```php
// WRONG
function getData() { ... }
$firstName = 'Jen';
class myWidget { ... }

// RIGHT
function get_data() { ... }
$first_name = 'Jen';
class My_Widget { ... }
```

### Indentation and Braces

Use tabs for indentation, never spaces. Opening braces go on the same line for
functions, classes, and control structures. Closing braces go on their own line.

```php
// WRONG — spaces, brace on new line
function get_items()
{
    if ($condition)
    {
        return true;
    }
}

// RIGHT — tabs, braces on same line
function get_items() {
	if ( $condition ) {
		return true;
	}
}
```

### String Interpolation

Use single quotes when the string contains no variables. Use double quotes or
concatenation for variable strings.

```php
// WRONG — double quotes with no variables
$msg = "No items found";

// RIGHT
$msg = 'No items found';
$msg = "Found {$count} items";
$msg = 'Found ' . $count . ' items';
```

---

## 2. WordPress-Extra Standard

WordPress-Extra extends Core with additional sniffs for code quality.

### Loose Comparisons

Never use `==` or `!=` for comparisons. Always use strict `===` or `!==`.

```php
// WRONG — loose comparison, type coercion bugs
if ( $value == false ) { ... }
if ( $id != 0 ) { ... }

// RIGHT — strict comparison
if ( false === $value ) { ... }
if ( 0 !== $id ) { ... }
```

### Assignment in Conditions

Do not assign variables inside `if`, `while`, or `for` conditions.

```php
// WRONG — assignment in condition
if ( $result = get_post( $id ) ) { ... }

// RIGHT — assign first, then check
$result = get_post( $id );
if ( $result ) { ... }
```

### Complex Expressions in Function Calls

Do not nest complex operations inside function call arguments.

```php
// WRONG — complex expression inside call
update_option( 'count', absint( $_POST['count'] ) + get_option( 'offset' ) );

// RIGHT — compute first, then call
$count  = absint( $_POST['count'] ) + get_option( 'offset' );
update_option( 'count', $count );
```

### Default Switch Case

Every `switch` statement must include a `default` case.

```php
// WRONG — missing default
switch ( $status ) {
	case 'publish':
		break;
	case 'draft':
		break;
}

// RIGHT
switch ( $status ) {
	case 'publish':
		break;
	case 'draft':
		break;
	default:
		break;
}
```

---

## 3. WordPress-VIP Standard

VIP sniffs enforce performance and security for high-traffic WordPress sites.

### No Direct Database Queries

Never use `$wpdb->query()` with inline SQL. Always use `$wpdb->prepare()`.

```php
// WRONG — direct query, SQL injection risk
$wpdb->query( "DELETE FROM {$wpdb->posts} WHERE ID = {$id}" );

// RIGHT — prepared statement
$wpdb->query( $wpdb->prepare( "DELETE FROM {$wpdb->posts} WHERE ID = %d", $id ) );
```

### Restricted Functions

These functions are forbidden on VIP: `eval()`, `extract()`, `create_function()`,
`file_put_contents()`, `file_get_contents()` (use `wp_remote_get()` instead),
`fwrite()`, `fopen()` for writing, `error_log()` (use `trigger_error()` with
`E_USER_WARNING`), `ini_set()`, `exec()`, `shell_exec()`, `system()`,
`passthru()`, `proc_open()`.

```php
// WRONG — direct file operation
$data = file_get_contents( 'https://api.example.com/data' );

// RIGHT — WordPress HTTP API
$response = wp_remote_get( 'https://api.example.com/data' );
if ( ! is_wp_error( $response ) ) {
	$data = wp_remote_retrieve_body( $response );
}
```

### Caching Requirements

Any function that queries the database must use object caching.

```php
// WRONG — uncached query on every page load
function get_featured_ids() {
	return get_posts( array( 'fields' => 'ids', 'meta_key' => 'featured' ) );
}

// RIGHT — cached with expiration
function get_featured_ids() {
	$cache_key = 'featured_post_ids';
	$ids       = wp_cache_get( $cache_key, 'zentratec' );

	if ( false === $ids ) {
		$ids = get_posts( array( 'fields' => 'ids', 'meta_key' => 'featured' ) );
		wp_cache_set( $cache_key, $ids, 'zentratec', HOUR_IN_SECONDS );
	}

	return $ids;
}
```

---

## 4. Data Sanitization

Every piece of user input MUST be sanitized before use. Match the sanitizer to
the data type and context.

| Context              | Sanitizer                              |
|----------------------|----------------------------------------|
| Plain text field     | `sanitize_text_field()`                |
| Textarea             | `sanitize_textarea_field()`            |
| Email address        | `sanitize_email()`                     |
| URL                  | `esc_url_raw()` (for DB storage)       |
| Filename             | `sanitize_file_name()`                 |
| HTML class           | `sanitize_html_class()`               |
| Post slug / key      | `sanitize_key()` / `sanitize_title()` |
| Integer              | `absint()` or `intval()`              |
| Rich HTML (post)     | `wp_kses_post()`                      |
| Rich HTML (custom)   | `wp_kses()` with allowed tags array   |
| Array of values      | `array_map( 'sanitize_text_field', $arr )` |

```php
// WRONG — raw superglobal used directly
$name = $_POST['name'];
$page = $_GET['page'];

// RIGHT — sanitized immediately on access
$name = sanitize_text_field( wp_unslash( $_POST['name'] ) );
$page = absint( $_GET['page'] );
```

Always call `wp_unslash()` before sanitization on superglobals because WordPress
adds magic quotes via `wp_magic_quotes()`.

---

## 5. Data Escaping (Output)

Every value rendered in HTML MUST be escaped at the point of output. This is the
**late escaping principle** -- escape as late as possible, never at storage.

| Output Context         | Escaping Function            |
|------------------------|------------------------------|
| HTML element content   | `esc_html()`                 |
| HTML attribute value   | `esc_attr()`                 |
| URL (href, src)        | `esc_url()`                  |
| JavaScript string      | `esc_js()`                   |
| Rich HTML (post body)  | `wp_kses_post()`             |
| Translated + escaped   | `esc_html__()`, `esc_attr__()`, `esc_html_e()` |

```php
// WRONG — unescaped output
echo '<a href="' . $url . '">' . $title . '</a>';
echo '<input value="' . $value . '">';
_e( $user_string, 'textdomain' );

// RIGHT — escaped at point of output
echo '<a href="' . esc_url( $url ) . '">' . esc_html( $title ) . '</a>';
echo '<input value="' . esc_attr( $value ) . '">';
echo esc_html( $user_string );
```

For `printf`/`sprintf` patterns, escape within the arguments.

```php
// RIGHT — escape inside sprintf arguments
printf(
	'<h2 class="%1$s">%2$s</h2>',
	esc_attr( $class ),
	esc_html( $heading )
);
```

---

## 6. Nonce Verification

Every form submission, AJAX request, and state-changing action MUST include and
verify a nonce to prevent CSRF attacks.

### Form Pattern

```php
// In form output:
wp_nonce_field( 'zentratec_save_settings', 'zentratec_nonce' );

// In form handler:
if ( ! isset( $_POST['zentratec_nonce'] )
	|| ! wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST['zentratec_nonce'] ) ), 'zentratec_save_settings' )
) {
	wp_die( 'Security check failed.' );
}
```

### AJAX Pattern

```php
// Enqueue with nonce:
wp_localize_script( 'zentratec-app', 'zentratecData', array(
	'ajax_url' => admin_url( 'admin-ajax.php' ),
	'nonce'    => wp_create_nonce( 'zentratec_ajax' ),
) );

// In AJAX handler:
function zentratec_ajax_handler() {
	check_ajax_referer( 'zentratec_ajax', 'nonce' );
	// ... process request
	wp_send_json_success( $data );
}
add_action( 'wp_ajax_zentratec_action', 'zentratec_ajax_handler' );
```

### REST API Pattern

```php
register_rest_route( 'zentratec/v1', '/items', array(
	'methods'             => 'POST',
	'callback'            => 'zentratec_create_item',
	'permission_callback' => function () {
		return current_user_can( 'edit_posts' );
	},
) );
// REST API uses built-in nonce via X-WP-Nonce header (wp.apiFetch handles it).
```

---

## 7. PHPDoc Requirements

### File Header

Every PHP file must begin with a file-level docblock.

```php
/**
 * Zentratec custom post type registration.
 *
 * @package Zentratec
 * @since   1.0.0
 */
```

### Function Documentation

Every function must have a docblock with description, `@since`, `@param`, and
`@return` tags.

```php
/**
 * Retrieve featured post IDs with caching.
 *
 * @since 1.2.0
 *
 * @param int    $count  Number of posts to return.
 * @param string $status Post status to query. Default 'publish'.
 * @return int[] Array of post IDs, empty array if none found.
 */
function zentratec_get_featured_ids( $count = 10, $status = 'publish' ) { ... }
```

### Hook Documentation

Document `do_action()` and `apply_filters()` calls with `@since` and `@param`.

```php
/**
 * Fires after a zentratec item is saved.
 *
 * @since 1.3.0
 *
 * @param int   $item_id   The saved item ID.
 * @param array $item_data The item data array.
 */
do_action( 'zentratec_item_saved', $item_id, $item_data );
```

---

## 8. Hook Callback Patterns

### Named Functions Over Closures

Always use named functions for hook callbacks. Closures cannot be removed with
`remove_action()` / `remove_filter()`.

```php
// WRONG — anonymous closure, cannot be removed
add_action( 'init', function () {
	register_post_type( 'zt_item', array() );
} );

// RIGHT — named function
function zentratec_register_post_types() {
	register_post_type( 'zt_item', array() );
}
add_action( 'init', 'zentratec_register_post_types' );
```

### Priority Management

Document non-default priorities. Use constants or comments to explain why.

```php
// Late priority to run after other plugins register their types.
add_action( 'init', 'zentratec_modify_post_types', 99 );
```

### Removal Pattern

To remove a callback, match the function name and priority exactly.

```php
remove_action( 'wp_head', 'wp_generator' );
remove_filter( 'the_content', 'wpautop', 10 );
```

---

## 9. Database Queries

### Always Prepare

Every query containing user-supplied or variable data MUST use `$wpdb->prepare()`.

```php
// WRONG — direct interpolation, SQL injection
$results = $wpdb->get_results(
	"SELECT * FROM {$wpdb->postmeta} WHERE meta_key = '{$key}' AND meta_value = '{$value}'"
);

// RIGHT — prepared with typed placeholders
$results = $wpdb->get_results(
	$wpdb->prepare(
		"SELECT * FROM {$wpdb->postmeta} WHERE meta_key = %s AND meta_value = %s",
		$key,
		$value
	)
);
```

### Prefer WP Abstractions

Use `WP_Query`, `get_posts()`, `get_post_meta()`, `get_option()` instead of raw
SQL whenever possible.

```php
// WRONG — raw SQL for something WP_Query handles
$wpdb->get_results( "SELECT ID FROM wp_posts WHERE post_type = 'page' LIMIT 10" );

// RIGHT — WordPress API
$pages = get_posts( array(
	'post_type'      => 'page',
	'posts_per_page' => 10,
	'fields'         => 'ids',
) );
```

### When Raw SQL Is Needed

For complex queries that WP_Query cannot express, use the appropriate `$wpdb`
method: `get_var()`, `get_row()`, `get_col()`, `get_results()`, `insert()`,
`update()`, `delete()`, `replace()`.

```php
$total = $wpdb->get_var(
	$wpdb->prepare(
		"SELECT COUNT(*) FROM {$wpdb->posts}
		 WHERE post_type = %s AND post_status = %s AND post_date > %s",
		'zt_item',
		'publish',
		$since_date
	)
);
```

---

## 10. Auto-Enforcement Setup

### Install phpcs and WordPress-Coding-Standards

```bash
# Project-level installation with Composer
composer require --dev squizlabs/php_codesniffer
composer require --dev wp-coding-standards/wpcs
composer require --dev phpcompatibility/phpcompatibility-wp
composer require --dev dealerdirect/phpcodesniffer-composer-installer

# Verify standards are registered
vendor/bin/phpcs -i
# Should list: WordPress, WordPress-Core, WordPress-Extra, WordPress-Docs, WordPress-VIP-Go
```

### Configuration File (phpcs.xml.dist)

Place at project root.

```xml
<?xml version="1.0"?>
<ruleset name="Zentratec">
    <description>WordPress Coding Standards for Zentratec</description>

    <file>./wp-content/themes/oshin_child/</file>
    <file>./wp-content/plugins/zentratec-*/</file>

    <exclude-pattern>*/vendor/*</exclude-pattern>
    <exclude-pattern>*/node_modules/*</exclude-pattern>

    <arg name="extensions" value="php"/>
    <arg name="colors"/>
    <arg value="sp"/>

    <config name="minimum_supported_wp_version" value="6.4"/>
    <config name="testVersion" value="8.0-"/>

    <rule ref="WordPress"/>
    <rule ref="WordPress-Docs"/>
    <rule ref="PHPCompatibilityWP"/>

    <rule ref="WordPress.NamingConventions.PrefixAllGlobals">
        <properties>
            <property name="prefixes" type="array">
                <element value="zentratec"/>
                <element value="zt_"/>
            </property>
        </properties>
    </rule>

    <rule ref="WordPress.WP.I18n">
        <properties>
            <property name="text_domain" type="array">
                <element value="zentratec"/>
            </property>
        </properties>
    </rule>
</ruleset>
```

### Run Commands

```bash
# Check for violations
vendor/bin/phpcs

# Auto-fix what can be fixed
vendor/bin/phpcbf

# Check a single file
vendor/bin/phpcs wp-content/themes/oshin_child/functions.php

# Check with specific standard
vendor/bin/phpcs --standard=WordPress-Extra path/to/file.php
```

### VS Code Integration

Install the `wongjn.php-sniffer` extension. Add to `.vscode/settings.json`:

```json
{
    "phpSniffer.executablesFolder": "./vendor/bin/",
    "phpSniffer.standard": "./phpcs.xml.dist",
    "phpSniffer.autoDetect": true,
    "phpSniffer.onTypeDelay": 500
}
```

### PhpStorm Integration

1. Settings > PHP > Quality Tools > PHP_CodeSniffer
2. Set path to `vendor/bin/phpcs`
3. Inspections > PHP > Quality Tools > PHP_CodeSniffer validation
4. Set coding standard to "Custom" and point to `phpcs.xml.dist`

### CI/CD Integration (GitHub Actions)

```yaml
name: WPCS
on: [pull_request]
jobs:
  phpcs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          tools: composer
      - run: composer install --no-interaction
      - run: vendor/bin/phpcs --report=checkstyle | cs2pr
```

---

## Quick Reference: Pre-Commit Checklist

Before committing any PHP file, verify:

1. All user input is sanitized with the correct function
2. All output is escaped at the point of rendering
3. All form and AJAX handlers verify a nonce
4. All database queries use `$wpdb->prepare()` or WP abstractions
5. All functions have PHPDoc with `@since`, `@param`, `@return`
6. All hook callbacks are named functions (not closures)
7. Yoda conditions are used for all comparisons
8. `array()` syntax is used (not `[]`)
9. Spaces inside parentheses are present
10. Tabs are used for indentation, not spaces
11. `phpcs` reports zero errors on changed files
