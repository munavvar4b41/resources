# Transfer a WordPress database

Run these statements after the database is imported on the new host. They rewrite the old site address to the new one in options, post content, post GUIDs, and post meta.

Replace `http://www.oldurl` and `http://www.newurl` in every statement. If the table prefix is not `wp_`, change that prefix in every statement too. Take a backup of the database first.

WordPress stores some option and meta values as serialized PHP. Each serialized string records its own length. When the new address has the same number of characters as the old one, `replace()` keeps those rows valid. When the lengths differ, use WP-CLI so the serialized lengths are rewritten with the text:

```bash
wp search-replace 'http://www.oldurl' 'http://www.newurl' --all-tables --skip-columns=guid
```

The `guid` column is a permanent identifier. Feeds treat a changed GUID as a new post. The SQL below rewrites GUIDs along with the other columns. Omit that statement when the GUIDs should stay as they are. The WP-CLI command above already skips them.

```sql
UPDATE wp_options SET option_value = replace(option_value, 'http://www.oldurl', 'http://www.newurl') WHERE option_name = 'home' OR option_name = 'siteurl';

UPDATE wp_posts SET guid = replace(guid, 'http://www.oldurl', 'http://www.newurl');

UPDATE wp_posts SET post_content = replace(post_content, 'http://www.oldurl', 'http://www.newurl');

UPDATE wp_postmeta SET meta_value = replace(meta_value, 'http://www.oldurl', 'http://www.newurl');
```

After the rewrite, set `WP_HOME` and `WP_SITEURL` in `wp-config.php` when those constants are in use, then open Settings → Permalinks and save so the rewrite rules match the new address.
