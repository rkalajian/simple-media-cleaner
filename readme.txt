=== Simple Media Cleaner ===
Contributors: robkalajian
Requires at least: 6.0
Tested up to: 6.9
Stable tag: 1.5.0
Requires PHP: 8.1
License: GPL-2.0+
License URI: https://www.gnu.org/licenses/gpl-2.0.html

Finds and permanently removes unused, unattached, and broken media — DB records, original files, and all WordPress-generated sizes in one pass.

== Description ==

Simple Media Cleaner finds and permanently removes unused, unattached, and broken media from your WordPress site — DB records, original files, and all WordPress-generated sizes in one pass.

= Features =

* Batched library scan with live progress bar — handles large libraries without timeouts
* Seven-point usage check verifies each attachment is genuinely in use before flagging it
* Broken reference detection — flags DB records whose file no longer exists on disk
* Orphaned file detection — scans the uploads directory for files with no DB record
* Unattached only filter — limit scans to media with no parent post
* Card grid UI with thumbnail previews, file type badges, and reason badges (Missing file / No DB record)
* Multi-select with floating action bar
* Confirmation modal listing files before deletion
* Live stats: total scanned, removable count, missing file count, estimated disk savings

= How it works =

**Scan (Phase 1 — Library)** — Fetches attachments in batches of 100 via AJAX. Each batch runs a file_exists() pass that flags broken references immediately, and a seven-point usage analysis on healthy attachments. Both sets of results are returned as removable items.

**Scan (Phase 2 — Filesystem)** — After the library scan completes, the uploads directory is scanned recursively for media files not registered in the DB. Known paths (original files + all generated sizes) are excluded. Matches appear with a "No DB record" badge.

**Delete** — Selected items are sent in chunks of 50. DB-backed items use wp_delete_attachment( $id, true ), which permanently removes the post row, all postmeta, the original file, and every resized variant — the item disappears from the Media Library immediately. Orphaned filesystem files are deleted via unlink() after path validation.

= Usage checks =

An attachment is considered in use if any of the following are true. All postmeta checks join against wp_posts so stale rows from deleted posts are ignored.

1. Has a post_parent pointing to a non-trashed, existing post — Batch INNER JOIN
2. Used as a featured image (_thumbnail_id) on an active post — Batch INNER JOIN
3. URL appears in any non-trashed post content — Per-item LIKE query
4. Included in a WooCommerce product gallery on an active product — Single query + PHP
5. Stored as a postmeta value (ACF image/file fields) on an active post — Batch INNER JOIN
6. Set as site icon or custom logo — get_option / get_theme_mod
7. URL or quoted ID appears in the options table — Per-item LIKE query

== Installation ==

1. Drop the `simple-media-cleaner/` folder into `wp-content/plugins/`
2. Activate via **Plugins → Installed Plugins**
3. The tool appears under **Media → Media Cleaner**

== Changelog ==

= 1.5.0 =
* Bug fix (critical): Checks 1, 2, 4, and 5 now INNER JOIN against wp_posts to verify the referencing post still exists and is not trashed or auto-draft. Stale postmeta rows left behind when a post was permanently deleted were keeping attachments flagged as "in use" indefinitely — this was the primary reason unused media wasn't surfaced by the scanner.
* New: Broken reference detection — attachments with a DB record but no file on disk are flagged with a "Missing file" badge. Deletion removes the dangling DB record, cleaning up the Media Library.
* New: Orphaned file scan (Phase 2) — the uploads directory is recursively scanned after the library scan for media files with no DB record. Orphan files appear with a "No DB record" badge.
* New: Unattached only checkbox — limits the scan to attachments with post_parent = 0 for faster targeted cleanup.
* New: Missing Files stat counter — dedicated count of broken DB references found during the scan.
* Fix: Progress bar total now sums all attachment statuses to prevent exceeding 100% on some sites.

= 1.4.0 =
* Renamed plugin from WP Media Cleaner to Simple Media Cleaner

= 1.3.0 =
* Fixed delete doing nothing for large selections: IDs are now sent in chunks of 50, staying well under PHP's max_input_vars limit
* Deletion progress shown in the floating action bar so it remains visible regardless of scroll position
* Progress bar fills per-chunk with file count and percentage; holds at 100% briefly before restoring the selection UI

= 1.2.0 =
* Replaced per-attachment tymc_is_unused() with tymc_filter_unused() — batch DB queries reduce scan query count from ~6N to ~6–8 per 100-item batch
* Fixed multi-file delete: URLSearchParams array serialization bug meant only the first selected ID was deleted
* URL computed during content check is cached and reused for the options check
* Added itemById Map for O(1) modal lookups; post-delete filtering uses a Set

= 1.1.0 =
* Added WooCommerce product gallery check
* Added options table check for ACF options fields
* Added site icon and custom logo exclusions
* Progress bar with shimmer animation

= 1.0.0 =
* Initial release
