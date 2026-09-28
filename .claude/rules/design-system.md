# Design System

This plugin has no token file or style framework. Its design is deliberately minimal and inherits from the host: the front end inherits the site's theme and Contact Form 7's styles, and the admin side uses WordPress admin styles. The values below are everything the plugin itself defines.

## Where design values live
- `css/style.css`: front-end form field (loaded on every front-end page as `mfcf7_zl_button_style`)
- `css/admin-style.css`: admin notices, deactivation pop-up and Tag Generator tweak
- `css/mfcf7_zl_button_style.min.css`: an old minified copy of `style.css`. **It isn't loaded anywhere.** Don't edit it; its removal should be handled under a cleanup spec.
- Inline markup, including class names, lives in `mfcf7_zl_multilinefile_shortcode_handler()` (main PHP file) and `mfcf7_zl_deactivation_popup()` (`multiline-admin.php`).

## Front end (`css/style.css`)
- Colours: none. The upload button uses the theme's classes `button button-primary qbutton`, and the remove icon is the ❌ emoji (`&#x274C;`).
- Spacing: container `margin-top: 15px`; each file row `margin-bottom: 10px` and `padding: 6px 0`; icon `margin: 0 5px`
- Type: icon `font-size: 15px`; everything else inherits from the theme
- Customer-facing style hooks (listed in `readme.txt`): `#mfcf7_zl_add_file`, `.mfcf7_zl_delete_file`, `.mfcf7-zl-multifile-name`

## Admin (`css/admin-style.css`)
| Use | Value |
|---|---|
| Notice button background / text | `#009FCE` / `#fff` |
| Notice button shape | `padding: 7px 10px; border-radius: 3px` |
| Pop-up panel | `#fff` background, `padding: 20px`, `border-radius: 5px`, `box-shadow: 0 0 10px rgba(0,0,0,0.3)` |
| Dividers | `1px solid #eee` |
| Image border | `1px solid #bebebe` |
| Loading spinner | `#f3f3f3` track, `#3498db` accent, 4px border, 20px size |
| Mobile breakpoint | `max-width: 480px` (pop-up 70% wide, buttons full width) |

Admin buttons use WordPress core classes (`button`, `button-primary`, `button-secondary`).

## Hard rules
- **Never add a new colour, font or font size** that isn't listed above. On the front end, add none at all; the field must inherit from the customer's theme.
- **Reuse WordPress and Contact Form 7 classes** (`button`, `button-primary`, `notice notice-*`, `wpcf7-form-control-wrap`) before writing new CSS.
- **Reuse existing markup and classes** before creating new ones. New admin UI should look like the existing notices and pop-up.
- **Don't change or remove** the customer-facing style hooks listed above (see `wordpress-plugin.md`, "Public contracts").
- Front-end CSS must stay scoped under `#mfcf7_zl_multifilecontainer`, `.zl-form-control-wrap` or `#mfcf7_zl_add_file`, so it never affects the rest of the customer's site.
- Keep the `!important` rules in `style.css` that hide the template input and remove button. The upload flow depends on them.
- Every layout must work at 480px wide and below.
