# Serve the public directory

Use this when the host document root is the project root, as with a default shared-hosting folder. Put the file in the project root as `.htaccess`. Requests are rewritten into `public/`, so a URL such as `/` is served from `public/`.

`mod_rewrite` must be enabled, and `AllowOverride` must permit `FileInfo` or `All`.

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule ^(.*)$ public/$1 [L]
</IfModule>
```
