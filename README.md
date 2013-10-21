Varnish 3 for Drupal 7 on Debian Wheezy 
=======================================

Install Varnish
---------------
Use the varnish-cache.org repository and install varnish:

```
curl http://repo.varnish-cache.org/debian/GPG-key.txt | apt-key add -
echo "deb http://repo.varnish-cache.org/debian/ wheezy varnish-3.0" >> /etc/apt/sources.list
apt-get update
apt-get install varnish
```
Edit Default Varnish
--------------------
Edit /etc/default/varnish

Listen on port 80, administration on localhost:6082, and forward to
one content server selected by the vcl file, based on the request.  Use a 1GB
fixed-size cache file.
```
DAEMON_OPTS="-a :80 \
-T localhost:6082 \
-f /etc/varnish/default.vcl \
-S /etc/varnish/secret \
-s file,/var/lib/varnish/$INSTANCE/varnish_storage.bin,1GB"
```
An example /etc/default/varnish is included.

VCL
---
default.vcl included should be moved to /etc/varnish/default.vcl

Ports
-----
Apache vhosts need to be bound to port 8080 instead of the default port 80 because now Varnish is listening to port 80 as described above. 
Edit ports.conf and change 80 to 8080:
```
NameVirtualHost *:8080
Listen 8080
```
An example ports.conf is included.

Virtualhost
-----------
vhost conf files in sites-available/sites-enabled will then require the same port 8080 directives eg. 
```
<Virtualhost *:8080>
```

Restart
-------
```
service apache2 restart
service varnish restart 
```

Install Drupal Varnish module
-----------------------------
https://drupal.org/project/varnish

Enable and configure module:
admin/config/development/varnish

Varnish version: 3.x

Varnish Control Terminal:
127.0.0.1:6082

Add Varnish Control Key from /etc/varnish/secret

Edit settings.php
-----------------
```
$conf['cache_backends'][] = '/sites/all/modules/contrib/varnish/varnish.cache.inc';
$conf['cache_class_cache_page'] = 'VarnishCache';
$conf['page_cache_invoke_hooks'] = FALSE;
$conf['reverse_proxy'] = TRUE;
$conf['cache'] = 1;
$conf['cache_lifetime'] = 0;
$conf['page_cache_maximum_age'] = 900;
$conf['reverse_proxy_header'] = 'HTTP_X_FORWARDED_FOR';
$conf['reverse_proxy_addresses'] = array('127.0.0.1');
$conf['omit_vary_cookie'] = TRUE;
```

Clear Drupal cache.

Test Varnish working
--------------------
http://www.isvarnishworking.com/



