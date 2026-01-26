put this on data/nextcloud/config/config.php if your smtp is giving you troubles :)

```php
'mail_smtpstreamoptions' => array(
    'ssl' => array(
        'allow_self_signed' => true,
        'verify_peer' => false,
        'verify_peer_name' => false,
    ),
  ),
```

you are getting some config.php cant be overwritten? its a permissions issue. just enter to your nextcloud container with `podman exec -it nextcloud bash` and execute `chown -R www-data:www-data .`
