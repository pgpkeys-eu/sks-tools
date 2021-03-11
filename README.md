# sks-tools
Toolkit for spinning up legacy sks deployments

## duplicator

A script that takes a standalone dpkg-installed sks instance and duplicates it (default 4x) to make a cluster.
The duplicates will sync with each other and the primary, but not with the primary's external peers.

Note that we leave sks listening on 127.0.0.1:11371 so that it can announce the correct port on the status page.

## etc/apache2

Apache reverse-proxy configuration.
This assumes that port 11371 is directed to the primary node, and 80,443 to the duplicates.
NB it does not (yet?) perform any active load-balancing across the three cluster members.

Before deploying, invoke the following by hand to install all your prerequisites (including a letsencrypt cert):

```
apt install certbot apache2
a2enmod ssl rewrite proxy_http lbmethod_byrequests proxy_balancer
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

Now unpack the contents of etc/apache2 into the corresponding places.
* Edit /etc/apache2/ports.conf and add all your non-localhost listening addresses with port 11371 (see file comments).
  This is because sks will listen on localhost port 11371 and we mustn't step on its toes.
  Keep the `Listen 127.0.113.71:11371` entry as this is required for tor.
* Edit /etc/apache2/sites-available/sks.pgpkeys.eu and alter the number of duplicates in both Proxy directives (if you didn't use the default 4).

Finally, incant:

```
a2ensite sks.pgpkeys.eu
apache2ctl graceful
```

## etc/letsencrypt

A renewal-hook script to ensure that renewed certs are automatically applied.

## etc/logrotate.d

Limit apache log retention to 48h

## etc/tor

Hidden service configuration parameters.
