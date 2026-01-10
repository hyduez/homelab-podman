you need to generate roundcube tables manually: `podman exec -it mailserver-roundcube bin/initdb.sh --dir /var/www/html/SQL`. When you log in into postfix-admin, you will need to grant permissions inside MariaDB: `podman exec -it mailserver-database mysql -u root -p`.

```sql
GRANT ALL PRIVILEGES ON postfix.* TO 'postfix'@'%';
FLUSH PRIVILEGES;
EXIT;
```

also `podman exec -it mailserver-database mysql -u postfix -p${PF_DB_PASS} postfix -e "ALTER TABLE mailbox ADD COLUMN smtp_active tinyint(1) NOT NULL DEFAULT 1 AFTER active;"`

to fix bcrypt mismatch: `podman exec -it mailserver-database mysql -u root -p${MYSQL_ROOT_PASSWORD} postfix -e "UPDATE mailbox SET password = CONCAT('{BLF-CRYPT}', password) WHERE username = 'user@yourdomain.tld';"`. replace user with your actual user. Then to fix it for all future users `podman exec -it mailserver-postfixadmin bash -c "echo \"\$CONF['encrypt'] = 'dovecot:BLF-CRYPT';\" >> /var/www/html/config.local.php && apache2ctl graceful"`


bcrypt: `podman exec -u root mailserver-postfix sed -i "s|password_query =.*|password_query = SELECT username AS user, CONCAT('{BLF-CRYPT}', password) AS password FROM mailbox WHERE username = '%u' AND active = '1'|" /etc/dovecot/dovecot-sql.conf`
