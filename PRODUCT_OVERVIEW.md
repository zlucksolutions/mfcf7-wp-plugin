# MultiLine Files for Contact Form 7: Product Overview

*Free edition, version 3.1.0, by Zluck Solutions*

---

## What is it?

MultiLine Files for Contact Form 7 is a free add-on for WordPress websites. It lets visitors attach **several files at once** when they fill in a contact form.

Many WordPress sites build their forms with **Contact Form 7**, one of the most widely used form plugins. On its own, Contact Form 7 lets a visitor attach only one file per upload field. If a site needs a CV *and* a cover letter *and* a portfolio, the site owner has to add three separate upload boxes and guess in advance how many files people will send.

This plugin replaces that with a single **"Upload" button that visitors can press as many times as they like**. Each time they pick one or more files, the file names appear in a list under the form. When the form is sent, the site owner receives all the files by email. If there are two or more, they arrive bundled into **one ZIP file**, so the email has a single tidy attachment instead of a pile of separate ones.

In short, it turns an ordinary contact form into a multi-file submission form without the site owner writing any code.

---

## Who is it for?

**Site owners and administrators** who already use Contact Form 7 and need to collect more than one file from their visitors. Typical examples:

| Use case | What visitors send |
|---|---|
| Job applications and recruitment | CV, cover letter, certificates, portfolio samples |
| Client onboarding and agencies | Brand assets, logos, briefs, reference images |
| Support and help desks | Several screenshots, log files, invoices |
| Quotes and estimates (print shops, builders, designers) | Drawings, photos, specifications |
| Schools, events and competitions | Entry forms, artwork, supporting documents |
| Content and media submissions | Photos, audio clips, short videos |

**Web designers and freelancers** who build sites for clients are a second audience. They can add a multi-file field to a client's form in a couple of minutes, with no custom development, and style it to match the client's theme.

**Visitors** are the people who fill in the forms. They get a familiar upload button, can add files a few at a time, see what they have attached, and remove something they added by mistake before sending.

---

## Key features (free edition)

### For the visitor filling in the form
- **Add files in several rounds.** Pressing the upload button again adds more files; it never replaces the earlier ones. Visitors can pick one file or several at once each time.
- **See what's attached.** The name of every chosen file is listed under the form.
- **Remove mistakes.** Each group of files added has an ❌ button to take it off the list before sending.
- **Clear error messages.** If a file is the wrong type or too large, or a required upload is missing, the visitor is told why when they try to send.
- **Works on phones, tablets and desktops,** and in all major browsers, including Safari. Earlier releases fixed several Safari-specific problems.

### For the site owner
- **Point-and-click setup.** A "multilinefile" button appears in the Contact Form 7 form editor. A small settings panel builds the upload field for the owner. It covers:
  - Field name, and whether an upload is **required**
  - The **button's label** (for example "Attach your CV" instead of "Upload")
  - **Maximum size per file**, which can be typed as bytes, KB or MB
  - **Allowed file types** (for example only PDFs and Word documents)
  - Optional styling hooks (ID and CSS class) for designers
- **Sensible defaults.** With no settings changed, the field accepts common images, documents, audio and video files (JPG, PNG, GIF, PDF, Word, PowerPoint, MP3, MP4, MOV and similar) up to about 10 MB each.
- **One ZIP attachment.** When a visitor sends two or more files, they reach the owner's inbox as a single compressed ZIP file. A single file is attached as it is, without zipping. This also tidies up Contact Form 7's own standard file fields on the same form.
- **Editable messages.** The plugin's messages (for example "Uploaded file is too big") appear in Contact Form 7's normal *Messages* tab and can be reworded or translated there.
- **Custom look.** The button and file list can be restyled with a few lines of CSS in the site's theme. The readme includes ready-made examples.
- **Built-in safety checks.**
  - Files are checked by type and size before the form is accepted.
  - The owner is warned if the server can't create ZIP files.
  - The owner is warned if the temporary upload folder isn't writable.
  - The plugin switches itself off with a clear message if Contact Form 7 isn't installed, so the site doesn't break.
- **Translations.** The plugin ships with English and Spanish language files and is ready for further translation.

---

## How it differs from similar products

| | Contact Form 7 on its own | Typical "drag-and-drop uploader" add-ons | **MultiLine Files (free)** |
|---|---|---|---|
| Files per field | One | Many | **Many; the free edition sets no limit on the number of files** |
| How files are added | Single browse button | Drag-and-drop area, often with progress bars | **A plain upload button, pressed again to add more** |
| What arrives by email | Separate attachments | Usually separate attachments or download links | **One ZIP file when two or more files are sent** |
| Look and feel | Theme default | A new, distinct upload widget | **Blends in with the existing form; easy to restyle** |
| Setup | — | Often has its own settings pages | **Everything happens inside the Contact Form 7 editor** |
| Cost | Free | Free and paid tiers | **Free, with an optional paid Pro upgrade** |

What sets it apart:

1. **The "one ZIP in your inbox" approach.** Most alternatives attach every file separately or send links. Bundling the files keeps emails manageable, especially for teams that forward submissions or file them in a shared folder.
2. **Simplicity.** There is no extra dashboard, account or cloud service. The feature appears as one more field type in the form editor people already know.
3. **Lightweight and self-contained.** Files stay on the website's own server and go out by email. They don't pass through any third-party storage service.
4. **Unobtrusive design.** It uses a standard button instead of a large drag-and-drop box, so it fits forms that should stay compact and clean.

The trade-off is that the free edition deliberately offers less than feature-heavy competitors. It has no drag-and-drop, no upload progress bars and no image thumbnails, and it supports one multi-file button per form.

---

## Free vs. Pro

The free edition is fully usable on its own. A paid **Pro** version is sold through the Envato marketplace and adds:

- **More than one upload button** in the same form or on the same page (for example, separate "CV" and "Portfolio" upload areas)
- **Minimum and maximum file counts** (for example "attach at least 2 and no more than 5 files")
- **Removing files one by one**, even when several were picked in the same round. In the free edition, a group picked together is removed together.
- **Moving the list of chosen files** to a different spot in the form
- **Priority support** and extra customisation options

The free plugin promotes Pro in two ways:
- A "Get Pro version" notice in the WordPress dashboard, shown from about 3 days after activation
- An "Upgrade to Pro" link on the Plugins page

A separate "please rate us" notice appears from about 5 days after activation. Both notices can be postponed or dismissed.

---

## What it needs

- A WordPress site (version 5.6 or newer; tested up to 6.8.2)
- The **Contact Form 7** plugin installed and active
- PHP 7.4 or newer (confirmed working with PHP 8.3)
- The server's standard **ZIP** feature enabled, which is available on most hosting. The plugin warns the owner if it's missing.

---

## How a site owner gets started

1. Install and activate the plugin from the WordPress plugin directory.
2. Open a Contact Form 7 form and click the **multilinefile** button.
3. Choose a name, a button label and, if needed, file-type and size limits, then click **Insert Tag**.
4. In the form's **Mail** tab, add the field name to **File Attachments** so the files are included in the email.
5. Save. Visitors can now attach as many files as they need.

---

## Company and licensing

- **Developer:** Zluck Solutions, which also offers WordPress development and customisation services and promotes them in the plugin's rating notice.
- **Licence:** GPL v2 or later. It is free to use on any number of sites.
- **Distribution:** The free edition is on WordPress.org and the Pro edition is on Envato.
- **Maturity:** The product has been maintained since version 1.0. There have been about 30 releases, mostly focused on compatibility with new WordPress, Contact Form 7 and browser versions, plus security and usability fixes. Version 3.0 moved to Contact Form 7's newer form-editor interface.

---

## Notes for the team: where the public description and the product differ

Reviewing the product against its public description (the WordPress.org readme) turned up a few points that should be kept in mind when describing it:

- **Privacy statement.** The readme says the plugin "does not collect, store, or transmit any personal data". However, when an administrator deactivates the plugin, an optional feedback pop-up asks why. If they submit it, the reason is sent to a Zluck feedback form together with the **site address and the administrator's email address**. Choosing "Cancel & Deactivate" skips this. The privacy wording should be updated to reflect this.
- **"Preview" of files.** Visitors see the *names* of their chosen files, not previews or thumbnails.
- **File count limits.** The readme presents minimum and maximum file counts as Pro-only. The free edition's validation code actually honours these settings if they are typed into the form tag by hand, even though the free settings panel doesn't offer them.
- **Default size limit.** A code comment refers to a 1 MB default, but the actual default is about 10 MB per file.
