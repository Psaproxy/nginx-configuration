# nginx-configuration
Example of high security configuration for Nginx with Certbot.

# Structure
```
nginx.conf
conf.d
  |-location
  |-security
  |-ssl
sites-available
  |-example.com.conf
```

# Symlink
For each configuration files in `sites-available` folder, we will create a symlink for it in `/etc/nginx/sites-enabled/`, then it will be visible for `nginx.conf`.

```bash
$ sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/
```

# Let's Encrypt - Certbot

## Create SSL certificates:

- `certonly`: If you want certbot only generate certs and do not proceed any further task.

- `-d example.com`: Your domain name.

```bash
$ sudo certbot certonly --nginx -d example.com -d www.example.com
```

Optional parameters:
- `--rsa-key-size 4096`: Optional parameter, in case you want to specify RSA key size is 4096.
- `--staging`: For testing purpose, add this param to avoid the rate limit of certs creation.
- `--force-renewal`: To force re-create / renew the certs that overwrite the existing one.

## Auto renew:

```bash
$ crontab -e
```

Copy and save this line:
```
0 12 * * * /usr/bin/certbot renew --quiet --post-hook "systemctl reload nginx"
```

Done 🎉 So after that:
1. Cronjob will run every 12 hours / day.
2. Certbot will check if can renew your SSL certs (either can or cannot, it will not prompt any message `--quiet`)
3. `--post-hook` after all certs are got renew, it will execute `systemctl reload nginx` to reload Nginx.

# DHParam

To improve the security, we add the Diffie-Hellman (DH) key exchange parameters. The key size which is following your SSL certs `2048` or `4096`.

```bash
$ openssl dhparam -out /etc/ssl/certs/dhparam-2048.pem 2048
```

# Test and reload Nginx

```bash
$ nginx -t && nginx -s reload
```

# Result in Qualys SSL Labs

