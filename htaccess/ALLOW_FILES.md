# Allow specific file

This will allow the index.php file to be executed on the root directory.
```htaccess
<Files "index.php">
    Require all granted
</Files>
```

# Allow specific file extension

This will allow the php file to be executed on the directory.
```htaccess
<FilesMatch ".php$">
    Require all granted
</FilesMatch>
```