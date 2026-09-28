# WordPress Plugin Conventions

How PHP and JS are written in this plugin. Visual and CSS rules are in `design-system.md`.

## Structure
- All code is procedural functions in two files: `multiline-files-upload-for-contact-form-7.php` (front end, validation, mail) and `multiline-admin.php` (admin side). Keep new code in whichever of these fits, unless a spec says to add a file.
- Declare each hook right next to its callback: `add_action(...)` / `add_filter(...)` on the line above `function ...`.
- Every PHP file must block direct access: `if (!defined('ABSPATH')) { exit; }`.
- Before calling a Contact Form 7 function or class, the Contact Form 7 plugin must be confirmed active. Existing code checks with `is_plugin_active('contact-form-7/wp-contact-form-7.php')`, `function_exists('wpcf7_add_form_tag')`, and by hooking into `wpcf7_init` / `wpcf7_admin_init`.

## Naming
- New functions, options, transients and nonces: `mfcf7_zl_` prefix, snake_case (for example `mfcf7_zl_multilinefile_create_zip`). Older names that break this (`mfcf7_plugin_meta_links`, `custom_plugin_deactivate`, `custom_plugin_ajax_object`) stay as they are.
- New CSS classes: `mfcf7-zl-` prefix (kebab-case) or `mfcf7_zl_` (to match the existing IDs). New JS lives in the existing `js/zl-multine-*.js` files.
- Text domain is always `'zl-mfcf7'`.

## Public contracts: never rename or remove
These are stored in customers' forms and databases or used by their own code:
- Form tags `multilinefile` / `multilinefile*` and their options `filetypes:`, `limit:`, `accept:`, `accept_wildcard:`, `minfile:`, `maxfile:`, `id:`, `class:`
- Filters `cf7_multilinefile_atts`, `cf7_multilinefile_input`, `cf7_multilinefile_max_size`
- Message keys added through `wpcf7_messages` (`upload_failed`, `zipping_failed`, `upload_file_type_invalid`, `upload_file_too_large`, `upload_failed_php_error`, `zl_min_file_count_validation_msg`, `zl_max_file_count_validation_msg`)
- Options `mfcf7-zl-admin-do-not-show-pro-tip`, `mfcf7-zl-admin-do-not-show-rating-tip`, `mfcf7_zl_plugin_deactivate_request`
- CSS hooks that the readme tells customers to style: `#mfcf7_zl_add_file`, `.mfcf7_zl_delete_file`, `.mfcf7-zl-multifile-name`, `#mfcf7_zl_multifilecontainer`

## Security (always)
- Escape all output: `esc_html()`, `esc_attr()`, `esc_url()`, `esc_html__()`. Some existing code outputs values unescaped: the button label in the field HTML and `$query_string` in the notices. Don't copy that pattern.
- Sanitise every `$_GET` / `$_POST` / `$_FILES` value before using it (`sanitize_text_field`, `intval`, `sanitize_file_name`).
- Every AJAX handler checks a nonce (`wp_verify_nonce`) **and** a capability (`current_user_can`).
- Validate uploads by extension and size, as the validation functions already do. Never trust the browser's `type` value.

## Translation
- Wrap every user-facing string in `__()` / `esc_html__()` / `_e()` with the `'zl-mfcf7'` domain. Several strings in the admin Tag Generator panel are not wrapped yet.
- When you add strings, update `languages/zl-mfcf7-*.po` and regenerate the `.mo` files.

## Assets
- Load assets only through `wp_enqueue_script` / `wp_enqueue_style` with a version argument. Don't put `?12` query strings in URLs and don't use `time()` as the version. Existing code does both.
- Admin assets currently load on every admin page. New admin assets should load only on the screens that need them.

## Compatibility and releases
- Supported: WordPress 5.6+, PHP 7.4+ (tested up to 8.3), and Contact Form 7 including the Tag Generator v2 panel format (`array('version' => '2')`). Don't use PHP syntax newer than 7.4.
- When you change behaviour that exists in both `..._validation_filter` and `..._validation_filtero`, change both.
- A version bump changes three places together: the `Version:` header in the main PHP file, `Stable tag:` in `readme.txt`, and a new `== Changelog ==` entry.
