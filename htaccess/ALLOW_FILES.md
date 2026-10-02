# Allow specific files

Apache 2.4 rules for a directory whose parent config denies access. Put the block in `.htaccess` in the directory it should affect. `AllowOverride` must permit `AuthConfig` or `All` for that directory.

## One file

Grants access to `index.php` in this directory.

```apache
<Files "index.php">
    Require all granted
</Files>
```

## One extension

Grants access to every file whose name ends in `.php`.

```apache
<FilesMatch "\.php$">
    Require all granted
</FilesMatch>
```
