# sks-tools
Toolkit for spinning up legacy sks deployments

## duplicator

A script that takes a standalone dpkg-installed sks instance and duplicates it (default twice) to make a cluster.
The duplicates will sync with each other and the primary, but not with the primary's external peers.

## etc/apache2

Apache reverse-proxy configuration.
This assumes that port 11371 is directed to the primary node, and 80,443 to the duplicates.
NB it does not (yet?) perform any active load-balancing across the three cluster members.

Before deploying, invoke the following by hand:

```
apt install certbot apache2
a2enmod ssl rewrite proxy_http
openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
cat <<EOF >> /etc/apache2/sites-enabled/sks.pgpkeys.eu.conf
<VirtualHost *:80>
	ServerName sks.pgpkeys.eu
	DocumentRoot /var/www/pgpkeyserver-lite
</VirtualHost>
EOF
apache2ctl graceful
certbot -d sks.pgpkeys.eu --webroot --webroot-path /var/www/pgpkeyserver-lite
rm /etc/apache2/sites-enabled/sks.pgpkeys.eu.conf
```

Now unpack the contents of etc/apache2 into the corresponding places, and incant:

```
a2ensite sks.pgpkeys.eu
apache2ctl graceful
```

## etc/letsencrypt

A renewal-hook script to ensure that renewed certs are automatically applied.

## etc/logrotate.d

Limit apache log retention to 48h
